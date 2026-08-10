---
name: flow
description: Full product delivery workflow from idea to shipped feature. USE WHEN starting a new feature, service, bugfix, or refactor that needs structured planning and execution, or when the user wants a plan, decision, or idea grilled - stress-testing their thinking, or any "grill" trigger phrase ("grill me"). Subcommands: setup, chart, idea, research, prototype, spec, plan, ticket, implement, review, replan, qa, push, complete, status, list. Use when user says /flow or needs to take an idea through grilling, spec, planning, autonomous implementation via subagents, cross-model code review, and QA testing.
---

# Workflow

Full product delivery workflow: idea to shipped, reviewed, QA'd feature.

```
[chart (wayfinder map) - efforts too big for one spec; spawns one run per deliverable]
     ↓
idea (grill) → [research] → [prototype] → spec (synthesize) → plan → [ticket]   human-in-the-loop
     → implement → review → [qa local]                                          AFK-capable
     → push → [qa env] → complete                                               human checkpoints
          ↑ (review/QA findings re-enter implement as targeted fixes)
```

**HITL/AFK banding.** Idea through ticket is human-in-the-loop: the grilling, quizzes, and previews are the point. Implement, review, and local QA run autonomously end to end - inside that band, prefer a deterministic default over asking; ask only when an action is destructive or the default would be a guess. Push is silent when its preflight is green; complete verifies rather than asks where it can.

**Decisions are made at idea, not at spec.** The idea step's relentless grilling session (every branch of the decision tree, one question at a time, a recommended answer per question, facts looked up rather than asked, nothing acted on until the user confirms shared understanding) resolves the decisions; research resolves the factual unknowns it surfaces; the spec synthesizes - it re-asks nothing.

## Subcommands

```
/flow setup                     - Create or update .flow/config.yml (runs automatically on first use)
/flow chart <desc|map> [ticket] - Wayfinder map for efforts too big for one spec (optional, sits above idea)
/flow idea <description>        - Capture the idea, then grill the decisions out of it
/flow research <slug>           - Codebase + external exploration of factual open questions (optional)
/flow prototype <slug>          - Throwaway code that answers one question (optional)
/flow spec <slug>               - Synthesize the spec; confirm test seams (the human checkpoint)
/flow plan <slug>               - Tracer-bullet phases with Blocked-by edges; quiz until approved
/flow ticket <slug>             - Tracker tickets from the plan (optional)
/flow implement <slug>          - Autonomous frontier-scheduled implementation via subagents
/flow review <slug>             - Cross-model review: a second model reviews the diff, Claude validates the findings
/flow replan <slug> [phase]     - Regenerate pending phases after spec changes
/flow qa <slug> [env]           - QA: local by default, or a named environment from config
/flow push <slug>               - Push branches and create PRs
/flow complete <slug>           - Verify merged, graduate lessons, delete the workflow folder
/flow status <slug>             - Progress from state.yaml
/flow list                      - All workflows
```

## Shared Context

### Project root and configuration

The **project root** is the directory the session runs in. Everything /flow persists lives under `<project-root>/.flow/`:

- `.flow/config.yml` - per-project configuration: layout, issue tracker, commit convention, worktrees, how to run the app, deployed environments, reviewer. Read it right after state.yaml on every subcommand that needs it; **if it doesn't exist, run the setup step first** (`setup-steps.md`, this skill's directory), then continue with the original subcommand.
- One self-contained folder per workflow, fixed filenames, no globbing.

### Paths

| Item | Path |
|------|------|
| Config | `.flow/config.yml` - created by setup, survives across workflows |
| Workflow folder | `.flow/<slug>/` - deleted in full at complete |
| Map folder (chart) | `.flow/<map-slug>/` - `map.md` + `tickets/NN-<slug>.md`, no state.yaml |
| State | `state.yaml` (contract: `state-schema.md`, this skill's directory) |
| Journal | `journal.md` - free-form narrative and session notes; no subcommand reads it by default |
| Docs | `idea.md`, `research.md`, `spec.md`, `plan.md`, `review-code.md`, `qa-local.md` / `qa-<env>.md` |
| Phase logs | `logs/phase-N.md` |
| Repos | single-repo: the project root IS the repo (default) - state.yaml names it `root`, which resolves to the project root (`.`). Multi-repo: config lists the repo subdirectories. A repo name is a state.yaml field, never invented as a path segment |
| Worktrees | `.flow/worktrees/<name>--<slug>` (implement) and `.flow/worktrees/<name>--proto-<slug>` (prototype), where `<name>` is the repo name or the project-root basename for `root` |

Templates and disciplines (`idea-template.md`, `spec-template.md`, `plan-template.md`, `map-template.md`, `ticket-template.md`, `research-template.md`, `review-template.md`, `qa-template.md`, `grill-discipline.md`, `simplicity-discipline.md`) live in this skill's directory.

Read `state.yaml` first on every subcommand - except `setup`, `idea` (which creates it), `chart` (maps have no state.yaml; map.md is their state) and `list`. On every update: bump `updated`, set `current` to the running subcommand, flip `steps.<subcommand>` (`done` on success; the review subcommand's key is `review-code`; qa flips to `done` only when a pass ends PASS, `in-progress` after a non-PASS pass), plus the step file's stated writes. Open `state-schema.md` (the full contract) only when creating the file at idea, when writing a structure the step file doesn't spell out, or for status/list's derived-read rules.

### Tool Dependencies

| Tool | When | Notes |
|------|------|-------|
| Issue tracker | ticket step, fetching tickets | Whatever config `tracker` names: GitHub Issues via `gh`, Jira, Linear, a markdown backlog file, or none. The config's `usage` note says how to interact with it |
| `git` | everywhere | Plain git. Phase/fix subagents commit with the message rules their prompt carries; checkpoint tags and reset/recovery are deliberate raw-git territory |
| `gh` CLI | push, complete | PRs and merge checks when the remote is GitHub; otherwise push branches and verify merges with plain git |
| `git worktree` | code-touching steps, when config `worktrees: true` | Worktrees live under `.flow/worktrees/` (see the Paths table) - always created with absolute paths |
| Second-model reviewer | review | Codex MCP (`mcp__codex__codex` + `-reply`) when available, else a CLI named in config, else a Claude subagent - the adversarial validation layer runs regardless. No Workflow orchestration tool anywhere: its resume is same-session only, and implement needs state.yaml + checkpoint tags to survive session death |
| Browser tools | qa (UI features) | Whatever the session has - Claude Code's built-in browser (no setup) or a browser MCP (e.g. chrome-devtools, which adds perf traces and Lighthouse). The agent picks; curl-level QA only when neither exists |

### Worktrees

When config `worktrees: true` (recommended): code-touching steps (prototype, implement, review, local QA, push) run in git worktrees under `.flow/worktrees/` so the user's main checkout stays untouched; docs-only steps need none. implement creates them (recorded in state.yaml), later steps reuse them from state.yaml, complete removes them; prototype manages its own temporary ones.

When `worktrees: false`: those steps run on a feature branch in the main checkout. implement requires a clean tree to start, and the checkout should be left alone during AFK runs.

### Context Management

Delegate heavy work to subagents with self-contained prompts; keep the main context for orchestration.

### Subagent Models

Subagents inherit the session model - do not pass a `model` override except where a step says so. Two standing exceptions run on a fast tier (`model: sonnet`): Explore subagents doing codebase sweeps, and implement's repo test sweep - both are mechanical enough that the fast tier loses nothing.

## Subcommand Routing

Read only the instruction file for the invoked subcommand (all in this skill's directory):

| Subcommand | Read |
|------------|------|
| setup | `setup-steps.md` |
| chart | `chart-steps.md` |
| idea, research, prototype | `ideation-steps.md` |
| spec | `spec-steps.md` |
| plan, ticket | `plan-steps.md` |
| review | `review-steps.md` |
| implement | `implement-instructions.md` |
| replan | `replan-instructions.md` |
| qa (any env) | `qa-steps.md` |
| push, complete | `shipping-steps.md` |

### status

Read `.flow/<slug>/state.yaml` and render:

```
Workflow: <slug>  (#42)
Repos: <repos>    Worktrees: <from state, or "none">
Map: <map-slug, only when state.yaml has one>

  [x] Idea          [x] Spec        [x] Plan (N phases)
  [ ] Research (skipped)            [ ] Prototype (skipped)
  [~] Implement - 3/5 phases committed (frontier: 4)
  [ ] Review        [ ] QA          [ ] Push   [ ] Complete

Latest QA: pass 1, local, NEEDS_CHANGES (2 open)
Blockers: <state.yaml blockers, when present>
Next: /flow implement <slug>
```

Derive "next" from the first non-done pipeline step (skipped optionals don't block); when state.yaml carries `next:` or `blockers:`, render them - they exist precisely so the next session sees them. A missing slug folder means the workflow either completed (complete deletes folders) or never existed - say so and stop.

### list

Read every `.flow/*/state.yaml`, sort by `updated`. Completed workflows don't appear - complete deletes their folders.

```
| Slug              | Repos  | Ticket | Current         | Updated    |
|-------------------|--------|--------|-----------------|------------|
| 42-token-refresh  | (root) | #42    | implement (3/5) | 2026-08-08 |
```

If none: `No active workflows. Start one with /flow idea <description>.`

Also list maps (folders with `map.md`, no state.yaml) in a separate Maps section: title, status (charting / way-is-clear), open/total tickets, started date. Omit the section if there are none.

## Design Principles

1. **Simple over impressive** - the simplicity discipline is injected into every implement/fix/review subagent; "a simpler version exists" carries the same severity as "this is broken".
2. **Accepted risks are decisions** - recorded in the spec, binding on implementers and reviewers.
3. **Plans are first-class artifacts** - content in versioned markdown, status in state.yaml.
4. **Autonomous inner loop** - implement, review, and QA run AFK; the human decides at idea (grilling), spec (seams), plan (breakdown quiz), and push.
5. **Tickets from plans, not plans from tickets.**
6. **Workflow docs are temporary** - complete deletes the whole folder; durable truth lives in merged code, PRs, the tracker, ADRs, and repo CLAUDE.md. Prototypes never merge.
7. **Too big for one spec means `/flow chart`.**
8. **Fail loud, fail early** - exact error + exact resume/rollback command.
9. **Test at pre-agreed seams** - the spec confirms them with the user; implementers never invent new ones, and a criterion untestable through a listed seam is a spec problem, reported not patched around.
10. **Git artifacts are public.** Commit messages and PR titles/descriptions follow the project's commit convention and use engineering language, never internal workflow naming ("Phase 3", "B1", "Track 2") - third parties reading git log or PRs have no context for it.
