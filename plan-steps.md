# Plan Steps: plan, ticket

## /flow plan <slug>

Break the spec into a phased implementation plan of **tracer-bullet vertical slices**, each declaring the phases that **block** it. Each phase is a thin end-to-end slice through all integration layers - not a horizontal slice of one layer.

### 1. Read the spec

Read `.flow/<slug>/spec.md` fully.

### 2. Explore the codebase

If the relevant code hasn't been explored in this conversation: the modules and services the spec touches, prior art for similar features, integration boundaries (schema, API, UI), and testing patterns. Use `Explore` subagents (`model: sonnet`) for heavy exploration - one per repo if cross-repo.

Look for opportunities to **prefactor** the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Identify durable architectural decisions

Before slicing, find the decisions unlikely to change as phases are built: route structures and URL patterns, DB schema shape, key data models, auth approach, third-party service boundaries. These go in the plan header so every phase can reference them - including **Test seams**, copied from the spec's Testing Decisions, which every phase's tests must go through.

### 4. Draft vertical slices

Break the work into tracer-bullet phases.

Vertical slice rules:

- Each phase cuts a narrow but COMPLETE path through every layer it needs (schema, API, UI, tests) - vertical, NOT a horizontal slice of one layer.
- A completed phase is demoable or verifiable on its own. If a phase's only verification is "compilation succeeds", merge it with the next - every phase needs a meaningful gate. Tests live in the same phase as the implementation.
- Each phase is sized to fit in a single fresh context window - it will run as a fresh agent session, so sizing is a plan-time property, not a runtime hope.
- Every phase declares its **Repo** (state.yaml tracks phases per repo; single-repo projects use `root`). Granularity: small-to-medium features get 2-3 phases (typically one per repo), not 5-8. Split by repo boundary or independently verifiable behavior, never by code layer.
- Prefactoring first: where prefactoring would make the implementation easier, make it its own early phase.
- Self-containment: each phase section plus the plan header must be sufficient to implement the phase without reading other phases.
- Do NOT include specific file names, function names, or implementation details likely to change as later phases are built. DO include durable decisions: route paths, schema shapes, data model names. Exception: a prototype snippet that encodes a decision more precisely than prose can, trimmed to the decision-rich parts.

Give each phase its **blocking edges** - the phases that must complete before it can start. Declare only genuine gates; a phase with no blockers can start immediately, and independent phases can run in parallel off these edges (phases in the same repo share a working tree, so implement serializes them - keep that in mind when weighing granularity against parallelism).

**Wide refactors are the exception to vertical slicing.** A wide refactor is one mechanical change - rename a column, retype a shared symbol - whose blast radius fans across the whole codebase, so no vertical slice can land green. Don't force it into a tracer bullet; sequence it as expand-contract. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites in blast-radius-sized batches (per package, per directory), each batch its own phase blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a phase blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify phase - green is promised only there.

### 5. Quiz the user

Present the proposed breakdown as a numbered list. For each phase, show:

- **Title**: short descriptive name
- **Repo**: where it lands
- **Blocked by**: which phases (if any) must complete first
- **What it delivers**: the end-to-end behavior this phase makes work

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct - does each phase only depend on phases that genuinely gate it?
- Should any phases be merged or split?

Iterate until the user approves the breakdown.

### 6. Write the plan

Write `.flow/<slug>/plan.md` per `plan-template.md` (this directory), then write `phases[]` to state.yaml: n, title, repo, blocked_by (each phase's Blocked-by line as an integer array; "None - can start immediately" -> `[]`), status `pending`. The plan file holds the content; state.yaml holds the status.

> Plan ready with N phases. Next: `/flow ticket <slug>` for tracker tickets, or `/flow implement <slug>` to start.

## /flow ticket <slug>

Tickets come FROM plans, not the other way around. Read `plan.md` and `spec.md`, and the tracker configuration from `.flow/config.yml` (if `tracker.type` is `none`, say the project has no tracker configured and stop - `/flow setup` changes that).

Draft one ticket per deliverable - usually one per plan phase for multi-ticket work, or a single ticket for the whole feature (propose, let the user pick). Per ticket: the end-to-end behaviour from the user's perspective (not a layer-by-layer list), acceptance criteria from the plan phase, links to the spec and plan docs where the tracker can reach them (skip for local-only paths), no file paths or code snippets.

Show ONE combined preview of all drafted tickets and get one confirmation, then create them via the configured tracker (the config's `usage` note says how: `gh issue create` for GitHub, the relevant CLI/MCP for Jira or Linear, an entry appended to the backlog file for markdown). Map each phase's Blocked-by edges to whatever the tracker supports: native blocking links (Jira, Linear), or a "Blocked by #NN" line in the ticket body (GitHub, markdown).

Record in state.yaml: the primary ticket (or epic) in `ticket:`; a `tickets:` phase-to-ref map for one-ticket-per-phase work. Add the primary ticket to the idea/spec/plan doc headers (filenames never change). Report ticket refs with links where the tracker has URLs.

> Tickets created and linked. Next: `/flow implement <slug>`.
