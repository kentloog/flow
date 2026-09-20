# Execution

`flow implement <slug>` runs the whole local delivery loop: implement the plan, get it reviewed, fix, verify the behaviour, hand the result to the human. `flow review`, `flow qa` and `flow replan` enter the same loop at one stage and return after it.

## Operating mode

You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task. Complete your assigned scope without transition questions: the coordinator owns the delivery loop, a worker its assigned unit. Choose reversible implementation details yourself. Stop affected work only for an unauthorized destructive action, a material decision outside the approved spec, missing access you cannot resolve with available tools and permissions, or the repair limit below. Continue independent authorized work before handing back a blocker.

Put this section in every worker brief.

## Start

Read `state.yaml`, `.flow/config.yml`, `spec.md` and `plan.md`. If the spec changed, reconcile the plan using the replan rule below; existing approval stands. Resolve worktrees from state. Create missing ones as `.flow/worktrees/<name>--<slug>` on branch `<slug>` from the default branch and record the base commit. With `worktrees: false`, a new run starts on a branch in a checkout clean apart from `.flow/`; resumed work follows the recovery rule below.

Confirm the acceptance criteria can be verified with what this session has: app startup, services, a browser for UI. Resolve reviewer access per `./review-steps.md`. A missing capability is a blocker to record, never a reason to skip verification.

On resume, look before you launch: worktree status, commits since the last recorded one, the unit's report. An `in-progress` phase may still have a live worker in this session; check before starting another. Continue from unfinished work. Keep a dirty tree, and keep one writer per tree.

## Loop

1. Implement the frontier. A phase is ready when every phase in `blocked_by` is committed. Use background workers (subagents) when independent work or a fresh context warrants them; otherwise continue in the coordinator. Parallelize across separate trees when useful, with one writer per tree. Brief by pointer: absolute paths to the spec, the plan section, predecessor reports, the worktree, this file and `./simplicity-discipline.md`, plus the commit convention. Duplicate nothing the pointers already carry.
2. Review. When every phase is committed and the combined change passes the repo's checks, run one pass per `./review-steps.md`. Fix valid findings in one fix unit per tree, then reply to the reviewer in the same session with what changed and what was rejected and why.
3. Verify the behaviour. Exercise each acceptance criterion the way it is used, starting the app from the worktree per config `run.notes` when needed: requests for APIs, a browser with screenshots for UI, queries for migrations, a run for jobs, fixtures for CLIs. Markup or HTTP inspection alone does not verify a UI. Reuse current evidence; run missing or affected checks rather than repeating checks because the stage changed. Write `qa-local.md` per `./qa-template.md`. Fix defects, then re-review changed code and re-verify affected criteria.
4. Hand off. Ready means every phase committed, a full review clean and local verification PASS, with current evidence per `./state-schema.md`. Set `status: ready`, leave the worktrees and running services in place, and give the human a short demonstration guide, the report paths, the accepted limitations and the questions that need their taste.

## Worker brief

A worker owns one unit in one worktree: a phase, or a fix covering several findings. It reads the source and the pointers, chooses its own approach and internal structure, verifies required behaviour through the approved acceptance interfaces, runs the repo's checks and commits only its own changes with the project's commit convention. It writes `logs/<unit>.md`: what it did, the commits, the commands it ran and their results, decisions a successor needs, anything left undone. It leaves `state.yaml` to the coordinator.

## Done and stopped

Stop a unit after its second failed repair and record what failed, what was tried and where the evidence is in its report. A worker returns the blocker to the coordinator, which records it and the exact resume command in `state.yaml`. Other ready work continues. A new diagnosis or the user's instruction earns another attempt; a new session on its own does not. Set `status: blocked` with an actionable `next` when no authorized work remains.

Progress lives in files. After compaction or in a new session, re-read `state.yaml` and the latest reports before dispatching anything. One coordinator per workflow.

## Standalone entry points

Each returns with `status` set by the rules above: `ready` only when the hand-off conditions hold, `blocked` with a `next`, otherwise `running` with the next stage named. A commit or plan change after review or QA makes that evidence stale.

- `flow review <slug>`: review and fix what exists. With uncommitted phases, record the review as partial; it starts no phases and does not continue into verification.
- `flow qa <slug> [env]`: one verification pass and its report. For an environment from config `envs`, first confirm which revision is deployed. A local fix stays unverified there until redeployed and rechecked.
- `flow replan <slug> [phase]`: keep committed phases as they are, regroup or add pending ones, keep every acceptance criterion covered and the graph acyclic, note the reason in `plan.md`. Material changes to outcomes, contracts or acceptance criteria need the human unless already approved. Internal design, order and batching belong to the agent.
