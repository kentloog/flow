# QA

Verify the spec's acceptance criteria against working software. `flow qa <slug> [env]` runs one pass and reports it; the autonomous execution loop calls the same procedure and handles repairs itself.

Read state, spec, relevant plan criteria and config `run` / `envs`. Resolve working trees from state. Record the approved spec digest and exact repo revisions. Run on stable code after writers finish.

## Environment and methods

For local QA, start required services from the recorded working trees using config notes or repo instructions. Keep process IDs and logs in the report; verify service readiness and that the running app uses the intended code, including container mounts/build inputs. Diagnose ordinary setup failures and retry within the execution budget. A missing login, inaccessible service or unknown deployed revision is an explicit verification gap.

Choose the method that proves the behavior: API requests for APIs, browser interactions and screenshots for UI states, schema/integrity queries for migrations, trigger and observable effects for jobs, fixture input/output for CLIs. Use the browser capability exposed by the harness. HTTP/markup inspection can supplement UI QA but cannot prove interaction, layout or browser behavior. If the required method is unavailable, mark those criteria unverified and the pass BLOCKED.

For a named environment, verify the deployed revision through a version endpoint, deployment record or equivalent evidence. Use supplied credentials and authorized access. Environment config describes how to deploy; it does not by itself authorize deployment. Capture request/trace IDs when available. A local fix does not resolve deployed QA until the fixed revision is deployed and retested there.

## Report and return

Use `./qa-template.md` to create `qa-local.md` or `qa-<env>.md`; append a pass on later runs. Record each criterion's observed result, method and evidence path, including unverified criteria. Findings need reproducible commands or UI steps with expected/actual results. Carry unresolved findings from the previous pass with their IDs; close them only with reproduction evidence or a linked accepted decision.

Verdicts:

- **PASS**: all required criteria verified for the stated revisions, with no unresolved required findings.
- **NEEDS_CHANGES**: verified product defects needing repair.
- **BLOCKED**: required criteria cannot be verified with available environment, access or tools. Record known defects too.
- **FAIL**: execution failed before a reliable assessment; include diagnostic evidence.

Persist the report. A delegated QA worker returns the report and leaves shared state to the coordinator; in a standalone pass, the main agent performs both roles. Stop only services this pass started; leave reproduction instructions so human acceptance can restart them.

The coordinator appends `qa[]` with pass, env, date, verdict, open count, revisions, spec digest and report path. It sets `steps.qa: done` only on PASS, otherwise `in-progress`. A non-PASS result invalidates any prior execution readiness; record running while the coordinator can repair, or blocked when no authorized progress remains. Preserve previous pass verdicts as historical evidence.

Return the verdict and evidence paths to the coordinator when called within execution; it proceeds without a phase-transition question. A standalone pass reports the result and appropriate next action: local PASS is ready for human acceptance if all phases are committed and a clean full review meeting the configured reviewer requirement is also current; code defects resume via `flow implement`; deployed fixes await authorized redeployment and another environment pass.
