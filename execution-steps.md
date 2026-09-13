# Autonomous execution

`flow implement <slug>` owns the full local delivery loop below. A standalone `flow review` uses the shared recovery and repair rules, then enters at review of the existing implementation. It never starts pending phases or continues into QA. With unfinished phases, report a partial review and the remaining planned work; a clean partial review does not establish delivery readiness. A delegated worker executes only its assigned work unit; it never starts another flow loop.

When standalone review ends, clear its finished active entries. Set `execution.status: ready` only if full delivery readiness is established; otherwise set `blocked` with `next` naming the remaining work outside this command's scope. Do not leave it marked running after the invocation ends.

## Start or resume

Read state, config, the approved spec and plan. Resolve the absolute project root and working trees. Require an approved spec and plan; an invocation to implement an already prepared plan authorizes its execution. For older workflows without `approved_spec`, establish it from the current spec only when that approval is known. If the spec digest differs from `approved_spec`, reconcile the user's approved changes through `./replan-instructions.md` before continuing.

For implementation, confirm the acceptance criteria have a feasible local verification method and the needed app, service and browser access. Standalone review needs only the prerequisites for its review and scoped repairs. Use existing authorization and fix routine setup problems autonomously. Missing credentials, unavailable required capabilities, or an unresolved product decision are concrete blockers, not a reason to pretend QA passed. Record the blocker and continue independent safe work when useful.

Resolve reviewer access through `./reviewer-transport.md` before leaving the run AFK. Record the implementation lead's family in `execution.implementation_family`; workers inherit it unless explicitly configured otherwise. Preserve it across a host change, updating it when another family takes over implementation or repairs. Record mixed authorship in the review brief so independence is described accurately.

Initialize `execution` using `./state-schema.md` without resetting existing counters; set its status running while work is underway. Persist state before dispatch and after consuming each result. The coordinator alone writes state and consolidates finding statuses; workers write their own reports. Record checkpoint commits and any worker handles in `execution.active`. One writer owns a working tree at a time, including during fixes. Review and QA run on stable revisions after writers finish.

### Reconcile interrupted work

Before launching work, inspect recorded handles where the harness supports it, working-tree status, commits since the recorded checkpoint, and the task's report. A recorded `in-progress` status is not proof that a worker died. Reattach to live work; do not launch a duplicate. If its liveness cannot be established, resolve ownership before another writer starts there.

Preserve uncommitted work. Identify task-owned changes and continue from them; unrelated or unexplained edits require reconciliation. Never infer finding resolution from filename overlap, or reset a dirty tree merely because the session ended. A commit without complete evidence needs the missing checks, not automatic acceptance or rollback. Honor legacy `workflow-checkpoint-<slug>-*` tags as recovery references when newer checkpoint records are absent.

## Run loop

For `implement`, resume at the earliest unfinished or stale stage. For standalone `review`, use step 2's review/repair cycle on the existing work even when phases remain unfinished, then return its result. Reuse complete current evidence when its inputs are unchanged; an implementation workflow already ready on the same revisions returns the acceptance handoff without repeating review or QA.

1. **Implement the ready frontier.** Read `./implement-instructions.md`. A pending phase is ready when all `blocked_by` phases are committed. Run independent repos concurrently when useful; serialize phases sharing a tree. Keep committed phase identities. Adjust pending execution batches or edges within approved scope as new facts arrive, recording why and preserving requirement coverage and ticket references. Use `./replan-instructions.md` for changes to plan content.
   If integration fails, persist the failing command and reproduction in `logs/integration.md`, then repair within the integration attempt budget before proceeding to review. Keep already-committed phases intact; a failing combined result does not require rerunning their implementation.
2. **Review the integrated result.** When all phases are committed and integration checks pass, read `./review-steps.md` and perform one review pass. Validate and persist findings before starting repairs. Group related valid findings into one fix task per working tree; pass all affected finding IDs. Send dispositions, fix commits and verification evidence back to the reviewer in its recorded session. Continue the exchange on fixes and their consequences while retaining coverage of the full spec.
3. **Run local QA.** After a clean review of the current revisions, read `./qa-steps.md`. Fix reproducible failures within scope, then return to review for changed code and QA for affected behavior. Carry unaffected evidence forward only with an explicit reason; completion requires evidence for the current revisions and approved spec. A standalone QA call reports its result; in this loop it returns control here without asking the user to invoke the next step.
4. **Return for human acceptance.** Ready means all phases committed, no unresolved required findings or blockers, a clean full review meeting the configured reviewer requirement, and local QA PASS for the current repo revisions and spec. Set `execution.status: ready`, clear active work, and provide a short demonstration guide, evidence links, accepted limitations, and any areas needing human taste. Keep the working trees. `push` is a separate publication step unless already authorized.

When resuming with open review/QA findings, incorporate them into repair work instead of redoing completed phases. Keep unresolved findings from the latest pass of every applicable report, including deployed QA. A deployed failure may be fixed locally, but remains awaiting environment verification until that revision is deployed with authorization and retested there. It does not silently become local PASS.

## Verification and repair budget

The worker runs the required repo gate and records exact commands, exit codes, tested revisions and log paths. The coordinator checks that evidence is complete and matches the work; it does not repeat the same typecheck/lint/test commands by default. Repeat only for changed inputs, integration requirements, missing evidence or a concrete concern. Integration checks cover combined changes, including cross-repo contracts; do not substitute isolated phase passes for them.

A phase or failing integration/setup check gets an initial attempt and up to two repair attempts, counted by work-unit ID. Review/QA get up to three repair rounds across the run, grouping the current findings rather than spending a round per finding. Persist counters in `execution.attempts`; resuming or compacting does not reset them. Stop a recurring failure when its budget is exhausted, with what failed, approaches tried, evidence, and the exact resume command. Independent work may continue. A changed diagnosis or explicit user instruction can justify another budget; record that reason rather than silently resetting it.

Do not stop merely because a phase needed a second repair, a fixed number of phases ran, or the context is filling. Stop dependent work for a material contract change, a needed permission, an unavailable prerequisite, or exhausted recovery. Set `execution.status: blocked` once no useful authorized work remains and leave an actionable `next` entry.

## Keep the main context small

Give substantial work to a fresh native subagent where available. Send the work-unit goal, absolute working-tree and workflow paths, relevant spec/plan sections, predecessor report pointers, required gates and report path. Include the absolute path to the relevant step instruction file; its bundled references resolve against the skill directory. Workers read necessary source directly. Return only status, commits, evidence/report paths, material decisions and blockers. Keep build output and research in files; load details only to resolve a specific failure.

On compaction, retain the workflow path, approved spec digest, current stage, active handles and ownership, outstanding blockers and next action. Re-read state after compaction and reconcile before dispatch. Files carry the full requirements and evidence; a summary is an index, not a substitute for source.

Use capabilities actually exposed in the desktop or CLI task. When native delegation is absent, implement sequentially with the same checkpoints and the harness's compaction; reviewer access is a separate requirement. An available review bridge or the other provider's CLI supplies it as described in `./reviewer-transport.md`. A skill cannot keep a closed app running or restart a crashed process. After actual process loss, resume with the same `flow implement <slug>` invocation. Across hosts, use the original project root and reconcile active ownership before continuing; do not run two coordinators on one workflow.
