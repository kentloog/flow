# /flow implement <slug> - Detailed Instructions

Autonomous implementation of all plan phases. The orchestrator launches one fresh subagent per phase; subagents implement, verify, and commit; the orchestrator owns scheduling, cheap deterministic verification, retries, and state.

Key properties:

- **Frontier scheduling** - a phase launches the moment every phase in its `blocked_by` list is committed; independent phases in different repos run in parallel. Phases in the same repo share a working tree, so they run sequentially even when both are unblocked.
- **Subagents commit** - each phase subagent runs the full gate (typecheck, lint, scoped tests) and commits its own work. The orchestrator never pulls build output into its context; it verifies with exit codes and `git` plumbing only.
- **TDD at pre-agreed seams** - tests are written test-first at the seams the spec confirmed; the phase prompt carries them.
- **Bounded retries** - 1 implementation attempt + max 2 fix attempts per phase.
- **No Workflow orchestration tool here** - its resume is same-session only; state.yaml plus git checkpoint tags survive session death, which multi-hour AFK runs need.

AFK invocation: start Claude Code with `--dangerously-skip-permissions`, run `/flow implement <slug>`, walk away. If the session dies, re-run the same command in a new session - it resumes from state.yaml.

## Steps

1. **Read** `plan.md`, `spec.md`, and `state.yaml` from `.flow/<slug>/`, the `worktrees` and `commit_convention` settings from `.flow/config.yml`, and `simplicity-discipline.md` (this skill's directory) once - Part 1 is pasted into every subagent prompt below, implementers and fix agents alike. From the plan header, extract the **Test seams** for the phase prompts.

2. **Check for findings re-entry.** If `qa-*.md` or `review-code.md` exists in the slug folder, read each file's latest `## QA Pass N` / `## Review Pass N` section and collect findings with Status = `open` (both docs use numbered findings tables with a Status column). If any exist, this run generates targeted fix subagents instead of re-running phases: one subagent per finding, scoped by its "Plan phase to fix" column (QA) or its anchor (review). Continue through steps 4-5 for worktree and toolchain setup, then apply the fix-mode rules:
   - **Recovery first.** Per affected worktree: a dirty tree means a previous fix session died mid-fix (fix subagents commit their own work, so uncommitted changes are unfinished work) - `git reset --hard` to the newest fix checkpoint tag (HEAD if none) and let the finding re-run. Commits past the newest fix checkpoint tag with a clean tree: run the silent verify (step 7d); green - flip the matching finding(s) to `fixed` (review findings match by anchor-file overlap; QA findings by re-running their repro command green); red - reset to the tag.
   - **Checkpoint per fix:** before each fix subagent launches, `git tag -f workflow-checkpoint-<slug>-fix-<doc>-<n> HEAD` in its worktree (e.g. `-fix-review-3`, `-fix-qa-local-2`). Fixes touching the same worktree run sequentially; fixes in different worktrees may run in parallel.
   - **Fix subagent duties:** a QA finding's fix subagent gets the finding's repro command from the QA doc and must re-run it green; every fix subagent re-runs the relevant scoped tests, then commits its own work (same commit rules as phase subagents, step 7b).
   - After each fix, the orchestrator runs the silent verify (step 7d); green - flip that finding's Status to `fixed` in the source doc. The doc is the durable fix state; there is no `phases[]` entry for fixes. A crash mid-run resumes by re-collecting `open` findings. A fix subagent that discovers its finding is already addressed reports that without committing; the orchestrator flips the Status.
   - **After the fix run:** recount `open` findings in the latest `review-code.md` pass; zero means update state.yaml `reviews.code.verdict: clean`. QA verdicts are never recomputed - only a new QA pass changes them.

3. **Determine resume point.** From `state.yaml` `phases[]`: `committed` - skip; `in-progress` or `failed` - needs recovery. Recovery runs AFTER steps 4-5 (it needs the worktree paths and discovered toolchain commands). Because subagents commit their own work, the tree state is the signal:
   - **Dirty tree** - the session died mid-phase before the subagent committed: `git reset --hard workflow-checkpoint-<slug>-phase-N` and set the phase `pending` (the frontier re-runs it). For a `failed` phase, announce it - a phase that already exhausted retries may need plan or spec attention rather than a third identical run.
   - **Clean tree with commits past the checkpoint tag** - the phase's work landed but state.yaml wasn't updated: run the silent verify (step 7d); green - mark it `committed`; red - reset to the tag and set `pending`.
   - **Clean tree, no commits past the tag** - nothing happened: set `pending`.
   - Report: "Resuming: phases X committed, starting from the frontier."

4. **Set up the working tree(s).** In `state.yaml` `repos`, the name `root` resolves to the project root (`git -C .`); its worktree naming below uses the project-root basename. Determine each repo's default branch once: `git symbolic-ref refs/remotes/origin/HEAD` (fall back to `git remote show origin`); every diff and branch below uses `origin/<default>`. A repo with no `origin` remote uses its local default branch (`main` or `master`, whichever exists) and skips the fetches - push later reports it has nothing to push there. The feature branch is the slug.

   With config `worktrees: true`, per repo in `state.yaml` `repos` - **reuse first**: if state.yaml `worktrees` already records a path for the repo and `git -C <repo> worktree list` confirms it, use it as-is (crash recovery and findings-fix re-entry land here). Otherwise create it, passing an **absolute path** - a relative path would resolve against the repo and nest the worktree inside the checkout:

   ```bash
   git -C <repo> fetch origin
   git -C <repo> worktree add <project-root>/.flow/worktrees/<name>--<slug> -b <slug> origin/<default>
   # branch already exists (crash before the worktree was recorded):
   git -C <repo> worktree add <project-root>/.flow/worktrees/<name>--<slug> <slug>
   ```

   `<name>` is the repo name (`backend`), or the project-root basename when the repo is `root`. Copy untracked env files the app needs (`.env`, `.envrc`, `.env.local` and friends) from the main checkout into the worktree - these are normally git-ignored; where they aren't, the step 7d clean-tree check excludes them. Record the worktree paths in `state.yaml` `worktrees`. **Every subsequent command uses the worktree path, never the main repo path.** Install dependencies in each worktree per that repo's package manager.

   With `worktrees: false`: require a clean tree in the main checkout (ignoring `.flow/`; dirty - stop and report; never stash the user's work silently), `git fetch origin && git checkout -b <slug> origin/<default>` (or check out the existing branch on resume), and record the checkout path in `state.yaml` `worktrees` so later steps have a single source for "where the branch lives". Warn once that the checkout should be left alone during the run.

5. **Discover each repo's toolchain** from the working tree: `CLAUDE.md` (or `AGENTS.md`) at the repo root, plus the manifest (`package.json` scripts, `Makefile`, `pyproject.toml`, Gradle build files, `Cargo.toml`, ...). Extract the exact commands for: typecheck (or compile check), lint (and its autofix variant), scoped tests, single-test-file invocation, affected tests (`jest --findRelatedTests`, `nx affected --target=test`, or similar; fall back to scoped tests per changed module if none exists), plus any required runtime version and workspace structure. Prefer commands CLAUDE.md documents. A repo with no lint or typecheck step just has a shorter gate - note it, don't invent one.

   Subagents run the full gate (typecheck, lint, scoped tests) and commit; the orchestrator's verify is the same typecheck+lint run with output discarded - exit codes only - so a subagent that misreported green is caught without build output ever entering the orchestration context. Production builds are CI's job.

6. **Report the schedule** from the `blocked_by` edges ("Launching phases 1 and 3 now. Phase 2 launches when 1 commits.") and create the log directory: `mkdir -p .flow/<slug>/logs`

7. **Run the frontier loop.** Repeat until every phase is `committed`, `failed`, or transitively blocked by a failure:

   Launch every phase whose status is `pending` and whose `blocked_by` phases are all `committed` - in parallel, in a single message (serialize phases that share a repo). Per phase:

   **a. Checkpoint and mark.** In the phase's working tree: `git tag -f workflow-checkpoint-<slug>-phase-N HEAD`. Set the phase `in-progress` in state.yaml.

   **b. Construct the phase prompt** (structure below): phase content from the plan, relevant spec stories, the toolchain block, the plan header's test seams, simplicity discipline Part 1, and - only for phases with blockers - the implementation summaries from their blockers' phase logs. The prompt carries the commit rules: the config's commit convention with the phase's ticket ref (`tickets[n]` when state.yaml has a per-phase map, else the primary `ticket:`; a `<slug>:` prefix without one, so a workflow's commits stay greppable) and principle 10 (engineering language, never plan phase titles or workflow naming).

   **c. Launch the subagent** via the Agent tool; pick the subagent type per the work.

   **d. Verify (silent).** Deterministic checks only, no build output into this context:

   ```bash
   cd <working-tree-path> && git status --porcelain            # must be empty (env files copied in step 4 excepted)
   git log --oneline workflow-checkpoint-<slug>-phase-N..HEAD   # must be non-empty
   (<typecheck> && <lint>) >/tmp/gate-<slug>-N.log 2>&1; echo $?  # must be 0
   ```

   **e. On failure** (dirty tree, no commit, or non-zero gate): launch a fix subagent with the last 200 lines of the gate log, the original phase context, the test seams, and simplicity discipline Part 1; it re-runs the phase's scoped tests and commits its fix on top. A second fix attempt additionally gets a 3-5 line summary of the first attempt's approach and why it failed, so it explores a different path instead of repeating it. Max 2 fix attempts. Still failing:

   > Phase N failed after 2 fix attempts. Error: <summary>
   > Rollback: `git reset --hard workflow-checkpoint-<slug>-phase-N`
   > Resume after manual fix: `/flow implement <slug>`

   Set the phase `failed` in state.yaml, then **keep the frontier running**: continue launching phases not transitively blocked by the failure, and report all failures together at the end.

   **f. Update state and log.** On green verify: phase `committed` in state.yaml; write `.flow/<slug>/logs/phase-N.md` - an implementation summary at the top (10-20 lines: key files, signatures, data shapes, design decisions; consumed by dependent phases) plus the subagent's `Skipped / add-when` list.

   **g. Repo test sweep.** When a repo's LAST phase commits, launch a sweep subagent (`model: sonnet` - mechanical work): run the repo's affected-tests command over the branch's changed files (`git diff --name-only origin/<default>...HEAD`), fix any failures, re-run to green, commit. Same retry bounds as (e); the orchestrator runs the silent verify after it.

   **h. Clean-stop check.** Stop cleanly after ~6 committed phases in this session, or when any phase enters its second fix retry: finish the in-flight phases, verify them, update state.yaml, then:

   > Stopping at a clean boundary to preserve orchestration quality. Phases 1-4 committed. Resume with `/flow implement <slug>` in a fresh session.

8. **After all phases commit:** delete the checkpoint tags in each working tree (`git tag -l "workflow-checkpoint-<slug>-*" | xargs -r git tag -d`), set `steps.implement: done`, and report:

   > All N phases committed. Logs: `.flow/<slug>/logs/`. Next: `/flow review <slug>`.

   If any phase is `failed`, skip the tag cleanup and instead report every failure with its rollback and resume commands. Keep the worktrees either way - review, QA, and push need them; `complete` removes them.

## Cross-repo dependencies

When one repo's phases depend on another's (e.g. a shared library before the app that consumes it), the plan's `blocked_by` edges encode it - the frontier handles the ordering. Push the dependency repo's branch as soon as its phases commit if the dependent repo consumes it via a published artifact.

## Phase Prompt Structure

````markdown
# Task: <what this phase builds, in engineering language>

Focus on the repo's established patterns and idioms throughout.

## Context

- Phase N of M for <slug>. Ticket: <the phase's ticket ref, or none>.
- Repo: <repo> at <working-tree-path>

## Repo Toolchain

Use ONLY these commands - no substitutes:

- Typecheck: `<exact command>`
- Lint / autofix: `<exact commands>`
- Test (scoped): `<exact command>` - run only tests for modules you changed; never the full suite
- Test (single file): `<exact command>`
- Runtime version / workspace notes: <...>

Read the repo's CLAUDE.md for conventions.

## Spec Context

<Relevant user stories and decisions, including the Accepted Risks & Tradeoffs entries touching this phase>

## What to Build

<"What to build" from the plan phase>

## Acceptance Criteria

<From the plan phase - the definition of done>

## Test Seams

Tests go through these interfaces and no others:

<The plan header's Test seams entries relevant to this phase>

If an acceptance criterion cannot be verified through a listed seam, STOP and report it instead of inventing a new seam - that is a spec problem, not an implementation choice.

## Testing Discipline

- Work in tracer-bullet slices: write a failing test at a listed seam first, then the minimum implementation that makes it green, then the next slice. Never write all tests up front, never all implementation first.
- Tests verify behavior through the seam's public interface, not implementation details. The tell: a test that breaks when you refactor without changing behavior is testing the wrong thing.
- Expected values come from an independent source of truth (the spec, a hand-computed example, a fixture). Never recompute the expected value the same way the code under test computes it - a test that mirrors the implementation proves nothing.
- Refactoring is not part of the loop - it belongs to the review step. Note refactoring candidates in your report instead of doing them.

## Simplicity Discipline

<Paste Part 1 of simplicity-discipline.md>

## Previous Phase Context

<Only for phases with blockers: their logs' implementation summaries. These summaries are an index, not the truth - when a signature or shape matters, read the actual committed code in this working tree. Omit section otherwise.>

## Workflow

1. Implement per the testing discipline, staying inside this phase's scope.
2. Run typecheck, lint (autofix first), and scoped tests; iterate until all green.
3. Verify every acceptance criterion, and map each new source file to its test file and each criterion to the specific test(s) verifying it - justify any source file without tests or go back and write them.
4. Commit ALL changes yourself: a single commit in this working tree, message per the project's commit convention: `<convention example from config>`. Describe what the code does, never the plan phase title or any workflow naming.
   - Good: `#42: Add per-user notification preferences to SettingsService and expose them in the profile API`
   - Bad: `#42: Phase 3 cleanup - remove CORS env var per audit`
5. Report: implementation summary (10-20 lines: files, signatures, data shapes, decisions - dependent phases consume this), files changed, criteria confirmation, the test mapping, refactoring candidates noted for review, and the Skipped / add-when list plus any new-dependency justification per the simplicity discipline.
````
