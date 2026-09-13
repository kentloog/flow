# Workflow state

`<project-root>/.flow/<slug>/state.yaml` is a compact index of durable work. The coordinator alone writes it. Write a complete valid file atomically, preserving unrelated fields; bump `updated` and set `current` on updates. Reports hold details; active work, blockers and the next action must be recoverable without conversation history.

## Schema

```yaml
slug: token-refresh
title: Token refresh for user sessions
ticket: null                  # tracker reference; optional map/tickets below
repos: [root]                 # root = project root; otherwise config repo names
created: 2026-09-13
updated: 2026-09-13T14:32:00Z
current: implement
next: Continue ready phases
blockers: []                  # short actionable strings; omit when empty
approved_spec: "<sha256 of approved spec.md bytes>"
steps:                        # pending | in-progress | done | skipped
  idea: done
  research: skipped           # research, prototype and ticket are optional
  prototype: skipped
  spec: done
  plan: done
  ticket: skipped
  implement: in-progress
  review-code: pending
  qa: pending                 # local QA is required for autonomous readiness
  push: pending
  complete: pending
worktrees:
  root: /project/.flow/worktrees/project--token-refresh
bases:
  root: "<base commit SHA>"  # frozen when feature branch starts
phases:
  - n: 1
    title: Refresh an expired session
    repo: root
    blocked_by: []            # phase IDs; all must be committed before dispatch
    status: in-progress      # pending | in-progress | committed | failed
execution:
  status: running            # running | blocked | ready
  implementation_family: codex  # codex | claude | other; lead for current implementation/repairs
  reviewer:                 # optional; persists after active call entries are cleared
    provider: claude
    transport: "<bridge tool or CLI>"
    cwd: /project/.flow/worktrees/project--token-refresh
    session_id: "<provider session ID>"  # null until returned; record host if remote
  attempts:                  # durable attempt counts, not reset on compaction
    phase-1: 1               # initial attempt + at most two repairs; also applies to integration/setup IDs
    repair: 0                # review/QA repair rounds; at most three by default
  active:
    - id: phase-1
      kind: phase            # phase | fix | review | qa | integration
      phases: [1]            # optional; IDs covered by this work unit
      finding_ids: []         # optional; qualified IDs, e.g. review-code:R1
      repo: root             # writer's repo; omit for read-only multi-repo work
      handle: null               # native handle when delegated; null for direct work
      checkpoint: "<HEAD before work>"
      report: logs/phase-1.md
reviews:
  code:
    passes: 1
    scope: full              # full | partial; only full can satisfy readiness
    implementation_family: codex
    reviewer_family: claude  # actual provenance; current/unknown is not cross-model proof
    verdict: clean           # clean | findings-open
    revisions: {root: "<reviewed SHA>"}
    spec_digest: "<approved spec SHA256>"
    report: review-code.md
qa:
  - pass: 1
    env: local
    date: 2026-09-13
    verdict: PASS            # PASS | NEEDS_CHANGES | BLOCKED | FAIL
    open_findings: 0
    revisions: {root: "<tested SHA>"}
    spec_digest: "<approved spec SHA256>"
    report: qa-local.md
prs:
  root: https://github.com/example/project/pull/42
map: platform-observability   # optional parent map slug
tickets: {}                  # phase ID -> tracker reference
prototype_branches: {}       # repo -> parked prototype branch
```

`tickets` is a top-level phase-ID → tracker-reference mapping (for example `tickets: {1: "#43"}`). Omit optional keys until used: `next`, `blockers`, `approved_spec`, `worktrees`, `bases`, `phases`, `execution`, `reviews`, `qa`, `prs`, `map`, `tickets`, `prototype_branches`. Initialize execution counters once; clear `active` only after reconciling results or confirmed termination. Phase reports retain commit and verification details after active entries are removed.

## Ownership and transitions

These writes belong to the coordinator, including when it performs a step directly. Delegated workers return reports and never update shared state.

- Idea creates identity and step defaults; optional steps start skipped. Research/prototype/spec/ticket update their step and related references.
- Plan writes approved phase content to `plan.md`, phase IDs/edges/status to state, and `approved_spec`, with the matching snapshot in `logs/approved-spec.md`. Later spec edits use the cosmetic/material reconciliation in `./replan-instructions.md`.
- Execution owns worktrees, bases, active work, attempts, per-phase status and `steps.implement`. Mark a phase committed only with committed work and complete evidence, or evidence the intended behavior already exists. Mark implement done once all phases are committed and required integration checks pass.
- Review preserves its session in `execution.reviewer`, records pass provenance and sets `steps.review-code` done only on a clean full review meeting the configured reviewer requirement. A clean partial review cannot satisfy readiness or shipping gates. QA appends pass metadata and sets its step according to verdict.
- Repairs close findings in their source reports based on evidence; they do not rewrite old QA verdicts or automatically upgrade review verdicts.
- Push writes `prs` and its step; complete removes the workflow folder after shipment verification and cleanup.

`current` names the last/running subcommand, not a lifecycle status. At acceptance readiness it may remain `qa`; `execution.status: ready` and `next` express the handoff. A later code or material spec change, or a failed check, invalidates readiness even if older step flags still say done.

## Stable IDs

New attempt counters use `phase-<n>`, `setup-<repo>`, `integration-<repo>` (or `integration` for a cross-repo check), and `repair` for the shared review/QA repair-round budget. Use the same work-unit ID in `execution.active` and its attempt counter where applicable. Retry and resume reuse the existing ID and count; a new attempt never gets a new ID. Preserve legacy keys for their existing work instead of renaming them or creating parallel counters.

Qualified finding IDs use `<report-basename>:<finding-id>`, such as `review-code:R1` or `qa-local:Q1`. Keep previously recorded forms as aliases for the same findings; do not create duplicates when resuming an older workflow.

## Evidence and compatibility

Review and QA evidence applies to the recorded spec digest and repo revisions, with no unexplained task changes in the trees. After a change, perform the affected verification and explicitly account for any reused unaffected evidence in a new report/pass covering the current revisions. Historical PASS is not current PASS.

Reviewer sessions are optional recovery pointers, not proof of completed review. Keep them when clearing finished `active` entries. Record transport/host changes and replacement sessions in the report; preserve findings and attempt counters. Changing the reviewer requirement invalidates evidence that does not meet it. Recover missing provider metadata only from known provenance, never infer cross-model review from a clean verdict or a tool's name.

Older states without the new fields remain readable. Recover bases from known branch creation/checkpoint information or a verified merge base; preserve existing phase IDs, worktrees, findings and tags. Infer a missing review scope from its report only when full-spec coverage is established. Missing revision/digest evidence means unverified, not failed code: obtain the missing review/QA evidence before marking ready. Treat legacy `steps.qa: skipped` as pending for autonomous readiness. Existing numbered finding IDs remain valid; qualify them by source report to avoid collisions.

## Status and list

Render steps, phase counts and the ready frontier (pending phases whose blockers are committed), execution status and active work, latest review, and latest QA per environment. For readiness/push gates, require current local evidence and account for any open findings in other environments; never use the last array entry alone as proof that local QA passed. Prefer explicit `next` and `blockers` to a derived next command.

List `.flow/*/state.yaml` by `updated`. Completed workflows disappear because complete deletes the folder. List map folders with `map.md` and no state separately, showing title, status and open/total decision tickets. Do not treat map tickets as implementation phases.
