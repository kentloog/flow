# Spec Steps: spec

## /flow spec <slug>

Produce the spec: the destination document with user stories, implementation decisions, and testing decisions. (The document is a PRD by another name.)

**Synthesis, not interview.** The thinking happened at the idea step's grilling session (and on the wayfinder map, for chart-spawned runs). Everything idea.md and research.md settle is synthesized, never re-asked. A residual interview covers only decisions that are still genuinely open - there should be few.

### 1. Gather context

Read `idea.md` fully - Settled Decisions, Accepted Risks (draft), Open Questions, Prototype Findings - and `research.md` if present. Do NOT proceed without a clear problem statement; if none exists, elicit it first (that's a sign the idea step was skipped - consider running its grilling now).

### 2. Light exploration

If the codebase hasn't been explored in this conversation: targeted exploration to verify assertions, understand the domain models, APIs, and modules involved, and identify existing patterns. Keep it focused - enough to write an informed spec, not everything. Read the repo's CONTEXT.md or glossary for vocabulary if one exists (read-only - this workflow never creates or edits such files), and respect any ADRs in the area being touched.

### 3. Residual interview

For decisions still open after synthesis, apply `grill-discipline.md` (this directory). If the residue reveals scope too large for one deliverable, recommend `/flow chart` - the current slug can become the map's first spawned deliverable.

### 4. Acceptance-test interface sketch (the human checkpoint)

Before writing, sketch the interfaces through which the feature's acceptance criteria will be tested. Prefer existing interfaces and the highest level that keeps failures legible. A stable interface lets the code underneath change without moving the acceptance tests.

Confirm the interfaces with the user and record them in Testing Decisions. They bind required acceptance tests. Supporting unit, integration, contract, and regression tests may use established repository interfaces.

### 5. Write the spec

Write to `.flow/<slug>/spec.md` per `spec-template.md` (this directory). Rules:

- **Accepted Risks & Tradeoffs is mandatory**: lift idea.md's draft entries plus anything the residual interview added. Record the conditions under which each risk is accepted. Leave it unhandled within those conditions; surface new evidence that invalidates them.
- **Durable references, not speculative ones.** No speculative file paths or implementation code - they go stale. DO include references that encode a decision more precisely than prose can: schema shapes, type definitions, API contracts, a prototype snippet or mockup (trimmed to the decision-rich parts, noted as coming from the prototype), and test rubrics for the confirmed interfaces.
- **User stories are the coverage mechanism**: include enough stories to cover required behavior, edge cases, and error states. Coverage is the target, not length.

### 6. ADR candidates and review

Append the ADR Candidates section per grill-discipline.md (only decisions that are hard to reverse AND surprising without context AND the result of a real trade-off; omit if empty). Surface the list; the user decides whether to record any in the project's ADR convention - do not write ADRs yourself.

Ask the user to review and iterate on any sections. Update state.yaml.

> Spec written. Next: `/flow plan <slug>`.
