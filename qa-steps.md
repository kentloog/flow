# QA Steps: qa

## /flow qa <slug> [env]

QA the implementation against the spec's acceptance criteria. Default environment is **local**; `<env>` names an entry in config `envs` for a deployed environment.

Read `spec.md`, `plan.md`, and `state.yaml` from the slug folder, plus `run` and `envs` from `.flow/config.yml`. Extract: repos with code changes, acceptance criteria to test, services involved. Working-tree paths come from state.yaml.

**Findings must be reproducible:** every finding recorded in the QA doc carries a repro command (curl line, UI path, or SQL query) - the fix subagent re-runs it green before the finding flips to `fixed`.

Method by feature type: API - curl; UI - browser tools if a browser MCP is available (screenshot key states), otherwise curl the rendered routes and verify markup; full-stack - API first, then UI reflects the data; migration - schema + integrity queries via the project's DB client; background job - trigger + logs + DB check; CLI - run the binary against fixture input.

## Local (default)

1. **Start the app yourself where non-interactively possible**, from the working tree so the feature branch is what gets tested, following config `run.notes` - falling back to the repo's README, package scripts, or Makefile when notes are missing. Background long-running processes with output to a log file. Ask the user to run only the commands that genuinely need interactivity. Poll a health/root endpoint until up; on failure, read the process log, fix what's fixable (port in use, missing dependency, un-run migration) and retry once before asking the user.

2. **Validate the setup** before testing: the served code really is the feature branch (a version endpoint if one exists, else confirm the process was started from the working-tree path), and declared dependencies (DB, queues) are up.

3. **Execute test scenarios** per acceptance criterion, logging results: API via curl (status codes, response shape, error cases); UI via browser tools; DB via the project's client.

4. **Write the report** to `.flow/<slug>/qa-local.md` per `qa-template.md` (this skill's directory). First run creates the file; later runs append `## QA Pass N`. Open findings from the previous pass carry forward.

5. **Stop what you started** (background processes), then **update state.yaml** (`qa[]` entry: pass, env `local`, date, verdict, open findings count; `steps.qa`: `done` on PASS, `in-progress` otherwise) and report:
   - **PASS**: `Next: /flow push <slug>`
   - **NEEDS_CHANGES / FAIL**: list findings. Fix route: `/flow implement <slug>` re-enters with the latest pass's open findings as targeted fix subagents; then re-run QA.

## Deployed environments

Named in config `envs`; each entry carries the base URL and free-text notes (how to deploy a branch there, how to read logs). Less direct access than local: no processes to start, no direct DB unless the user provides access.

**Prerequisites:** the feature branch is deployed to the target environment - verify via a version endpoint when one exists, ask the user otherwise (the env's notes say how to deploy). Any auth the environment needs (tokens, login) comes from the user - never guess credentials.

Execute scenarios as locally, but: curl against the environment's URL, browser tools on the public URLs, and logs per the environment's notes (platform CLI, dashboard, whatever the notes name). DB only if the user has provided access for this task; ask before querying - otherwise verify data via API responses. On failures, capture whatever correlation the platform offers (request IDs, trace IDs from response headers or logs) and record it with the finding.

**Write the report** to `qa-<env>.md` per qa-template.md (`## QA Pass N` appends on re-runs; include request/trace IDs in findings). **Update state.yaml** (`qa[]` entry with env, date, verdict, open findings; `steps.qa`: `done` on PASS, `in-progress` otherwise) and report:
- **PASS**: `Next: /flow complete <slug>`
- **NEEDS_CHANGES / FAIL**: findings with their correlation IDs. Fix route: `/flow implement <slug>` re-entry, then redeploy and re-run.
