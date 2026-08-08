# /flow replan <slug> [phase] - Detailed Instructions

Adjust the plan after the spec changed mid-implementation. **Committed phases are immutable** - replan never rewrites or reverts completed work; corrective phases are appended instead.

## Rules

- A phase with `status: committed` in state.yaml stays. Its code is fact. Committed phases are a SET, not a prefix - frontier scheduling commits out of order (1 and 3 can be committed while 2 is pending).
- Phases still `pending` can be regenerated.
- A phase with `status: failed` is regenerable like `pending`, with user confirmation - a failure caused by a bad plan is exactly what replan exists for. Its uncommitted working-tree changes are handled by the next implement run (reset to the checkpoint tag before the fresh attempt).
- A phase with `status: in-progress` means an implement session died mid-phase. Do not replan over it: run `/flow implement <slug>` first so its recovery logic settles the phase to `committed` or `pending`, then replan.
- To change committed work, append a corrective phase (e.g. "Refactor token storage to Redis") that explicitly references which earlier phase it corrects and why.

## Steps

1. **Read** the updated `spec.md`, `plan.md`, and `state.yaml` from the slug folder, plus the phase logs (`logs/phase-N.md`) of committed phases - their summaries stand in for the committed code; do NOT re-explore the codebase (the committed code IS the current state, and re-exploring burns context on already-understood code).

2. **Scope from `[phase]`.** Naming a `pending` or `failed` phase scopes regeneration to that phase (plus any pending phases the change invalidates - e.g. their `blocked_by` edges or content reference what changed); omitting it regenerates all replannable phases. If `[phase]` names a committed phase, reject:
   > Phase N is already committed - I'll add corrective phases at the end instead. Proceed? (y/n)

3. **Diff the spec against the plan's assumptions:** new requirements uncovered, changed requirements affecting committed phases, removed requirements making pending phases unnecessary.

4. **Regenerate the replannable phases** per the slicing rules in `plan-steps.md` step 4 (tracer-bullet vertical slices, repo per phase, context-window sizing, expand-contract for wide refactors), under these constraints:
   - Committed phase numbers and content are immutable; never touch them in plan.md or state.yaml.
   - Brand-new phases take fresh numbers above the highest existing one; blocking edges may reference committed phases.
   - Corrective phases for committed work are appended at the end with `blocked_by` edges on the phases they correct.
   - Each regenerated phase gets a full plan-template phase section (What to build, Acceptance criteria with seams, User stories).

   Merge into plan.md: committed sections unchanged, regenerated sections replaced, corrective sections appended.

5. **Update state.yaml `phases[]`**: committed entries byte-identical, new/regenerated entries `pending` with fresh `blocked_by` edges. Quiz the user on the new edges as in the plan step (granularity, edges, merge/split) and iterate until approved.

6. **Report:**
   > Replanned. Committed phases (<numbers>) unchanged. [Regenerated / appended corrective] phases: ... New total: P.
   > Continue: `/flow implement <slug>`
