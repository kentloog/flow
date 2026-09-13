# Implementation work units

Loaded by `./execution-steps.md` for phases and repairs. This file returns results to the coordinator; it does not end the overall run after implementation.

## Working trees and gates (coordinator only)

Prepare the assignment before dispatch. Delegated workers start at "Execute a phase or repair" and report setup gaps to the coordinator. In direct execution, the main agent performs both roles.

Reuse each recorded working tree after confirming its repo and branch. For a new workflow, resolve the default branch from repository configuration or the remote's HEAD; without a remote, use the known local base. Record its exact commit in `bases`. If no base can be established, surface that ambiguity rather than inventing one.

With `worktrees: true`, create the feature branch named for the slug in `.flow/worktrees/<name>--<slug>` using absolute paths. Reuse an existing matching branch/worktree after checking ownership, including when a previous run ended before recording it. With `worktrees: false`, require a clean main checkout before the first run, excluding workflow artifacts; if user changes are present, stop and report without stashing or resetting them. On resume, use the reconciliation rules instead of discarding unfinished changes. Record the feature-branch checkout path in `worktrees`.

Read applicable repository instructions and manifests to discover runtime, dependency installation, typecheck/compile, lint, scoped tests and integration/affected-test commands. Preserve required repo checks; omit tools the project does not use. Run a build when the change or project requires it. Pass the discovered commands in the assignment; the first phase's worker records them in its report for later phases to reference.

Provision local configuration the app needs without committing secrets. Copy only needed environment files, keep them ignored or explicitly excluded from task staging, and preserve existing values. Start services for QA using config `run.notes` and repository instructions.

## Execute a phase or repair

The assignment contains:

- Goal and acceptance criteria by reference to spec/plan, or finding IDs with repro evidence.
- Absolute project root, working-tree paths, approved spec digest, and report path `logs/phase-N.md` or `logs/fix-N.md`.
- Required repo gates, relevant predecessor report paths and commit convention/ticket reference.

Read the relevant source and `./simplicity-discipline.md`. Choose implementation approach, test ordering and internal structure yourself within the approved contract. Verify required acceptance behavior through the approved interfaces; supporting tests follow repo practice. Test expectations come from requirements, fixtures or independent examples, not the implementation under test.

Run the relevant gate, fix failures within the coordinator's budget, and commit only task-owned changes in the assigned working tree. Multiple coherent commits are fine. Use project commit conventions and engineering descriptions. Do not modify shared workflow state or schedule other workers.

Write the report before returning:

- Outcome and commits; relevant implementation decisions and source pointers for successors.
- Criteria or finding IDs addressed, verification commands, exit codes and full-output log paths; exact tested repo revisions and spec digest. When tests ran immediately before committing, confirm the committed content matches the tested content and no hook changed it.
- Unresolved failures or gaps. Record deliberate omissions and their add-when conditions only when they add information beyond the spec. Explain a new dependency's need when one was added.

For a repair, re-run each finding's reproduction or equivalent regression check. Report per-finding results even when one fix addresses several. A coordinator marks a finding fixed only from this evidence; an already-fixed finding needs evidence but no empty commit.

## Integration (coordinator only)

After the current frontier completes, run checks needed to cover the combined change before review. Use the recorded base-to-head diff to select affected checks and include dependency contracts across repos. Reuse phase evidence where it already covers the integrated revisions; run broader checks when integration changes what was exercised.

Cross-repo `blocked_by` edges establish ordering. Prefer local linking or a local package artifact to verify dependent repos. If execution requires publishing a dependency, use existing publication authorization or record the exact blocked action; local implementation authorization alone does not publish packages or push branches.
