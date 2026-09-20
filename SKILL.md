---
name: flow
description: Deliver substantial features and refactors through collaborative discovery, specs and plans, then autonomous implementation, review and QA. USE WHEN the user invokes flow, wants an idea taken through delivery, or asks to grill an idea or plan.
---

# Flow

Agree the outcome with the human, execute the approved work autonomously, return a working result for human acceptance.

```text
[chart] → idea → [research / prototype] → spec → plan → [ticket]   human decisions
       → implement (review ↔ fixes, local QA ↔ fixes)              autonomous
       → human acceptance → push → [environment QA] → complete
```

## Paths and state

Bundled paths are relative to this skill's directory. Workflow data lives under the project root, fixed when the workflow starts; a worker's checkout is never a new project root.

- `<project-root>/.flow/config.yml`: layout, repos, tracker, worktrees, run notes, reviewer. If absent, read `./setup-steps.md`, configure, then continue.
- `<project-root>/.flow/<slug>/`: `state.yaml` (`./state-schema.md`), `idea.md`, `research.md`, `spec.md`, `plan.md`, `review-code.md`, `qa-local.md` / `qa-<env>.md`, `logs/`. Read `state.yaml` first except for setup, a new idea, chart and list.
- Worktrees: `.flow/worktrees/<name>--<slug>`, where `<name>` is the repo name or the project-root basename for `root`; prototypes use `<name>--proto-<slug>`. Code paths come from `state.worktrees`.

## Commands

`/flow` in Claude Code, `$flow` in Codex. Read only the file for the command at hand.

| Command | Read |
|---------|------|
| setup | `./setup-steps.md` |
| chart <description or map> | `./chart-steps.md` |
| idea <description>, research <slug>, prototype <slug> | `./ideation-steps.md` |
| spec <slug> | `./spec-steps.md` |
| plan <slug>, ticket <slug> | `./plan-steps.md` |
| implement <slug>, review <slug>, qa <slug> [env], replan <slug> [phase] | `./execution.md` |
| push <slug>, complete <slug> | `./shipping-steps.md` |
| status <slug>, list | `./state-schema.md` |

## Decision boundaries

- Humans settle outcomes, public contracts, consequential trade-offs, acceptance criteria and taste. Facts are researched, never asked. Reversible implementation choices belong to the agent. Ask only about an unresolved material decision.
- `implement` authorizes the local implement, review, fix and verify loop within the approved spec, across phases and compactions. Publishing, deploying and merging follow the user's actual authorization; the default endpoint is a result ready for human acceptance.
- Accepted risks hold under their recorded conditions. Evidence that breaks a condition is surfaced, not designed around.
- Required acceptance tests use the interfaces agreed in the spec; supporting tests follow repo practice. UI behaviour is verified in a browser.
- Review is by the other model family unless config says `current`. Read `./simplicity-discipline.md` when implementing or reviewing.
- Commits and PRs use the project's conventions and describe engineering changes, never internal phase names.
- Workflow documents are temporary; complete moves durable lessons into the repo and deletes the folder. Prototypes never merge.
