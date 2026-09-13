---
name: flow
description: Deliver substantial features and refactors through collaborative discovery, specs and plans, then autonomous implementation, review and QA. USE WHEN the user invokes flow, wants an idea taken through delivery, or asks to grill an idea or plan.
---

# Flow

Agree on the outcome with the human, execute the approved work autonomously, then return a working result for human acceptance.

```text
[chart] → idea → [research / prototype] → spec → plan → [ticket]   human decisions
       → implement → review ↔ fixes → local QA ↔ fixes           autonomous
       → human acceptance → push → [environment QA] → complete
```

## Paths and state

Bundled instruction and template paths below are relative to this skill's directory. Workflow data lives under the **project root**, established when the workflow starts. Preserve its absolute path when delegating or resuming; a worker's checkout is not a new project root.

- `<project-root>/.flow/config.yml`: project layout, repo paths, tracker, worktrees, run instructions, reviewer. If absent, read `./setup-steps.md`, configure the project, then continue.
- `<project-root>/.flow/<slug>/`: `state.yaml`, `idea.md`, `research.md`, `spec.md`, `plan.md`, `review-code.md`, `qa-local.md` / `qa-<env>.md`, and `logs/`. Read `state.yaml` first except for setup, new idea, chart, and list.
- `state.yaml` holds scheduling and evidence references; `./state-schema.md` defines it. Read the schema when creating or extending state. Update `updated`, `current`, and relevant steps as work progresses. Narrative belongs in reports; `journal.md` is optional and is not read by default.
- Single-repo name `root` resolves to the project root. Multi-repo names resolve through config. Code paths come from `state.worktrees`, never from an invented repo-name path.
- Worktrees: `.flow/worktrees/<name>--<slug>`; prototypes use `<name>--proto-<slug>`. For `root`, `<name>` is the project-root basename. Pass absolute paths to git.

## Commands

Use `/flow` in Claude Code or `$flow` in Codex; the argument semantics are the same. Read only the supporting file needed now, loading subsequent steps as execution reaches them.

| Command | Purpose | Read |
|---------|---------|------|
| setup | Project configuration | `./setup-steps.md` |
| chart <description or map> [ticket] | Decision map for work too large for one spec | `./chart-steps.md` |
| idea <description> | Problem, outcomes and consequential decisions | `./ideation-steps.md` |
| research <slug> / prototype <slug> | Resolve factual or design uncertainties | `./ideation-steps.md` |
| spec <slug> | Synthesize decisions; agree acceptance-test interfaces | `./spec-steps.md` |
| plan <slug> / ticket <slug> | Verifiable slices and dependency edges; optional tracker tickets | `./plan-steps.md` |
| implement <slug> | Run or resume implementation, review, fixes and local QA through human acceptance readiness | `./execution-steps.md` |
| review <slug> | Review and repair existing work; partial when phases remain | `./execution-steps.md`, then `./review-steps.md` |
| qa <slug> [env] | Verify behavior locally or in a configured environment | `./qa-steps.md` |
| replan <slug> [phase] | Adjust remaining work | `./replan-instructions.md` |
| push <slug> / complete <slug> | Publish branches and PRs / verify merged and clean up | `./shipping-steps.md` |
| status <slug> / list | Summarize progress, active work, blockers and next action | `./state-schema.md` |

For status, show phase counts, execution status, latest review and QA evidence, and `next` / `blockers`. For list, read `.flow/*/state.yaml`, sort by `updated`, and list maps (`map.md` without state) separately. Missing workflow folders mean completed or never created; do not reconstruct them by guessing.

## Decision boundaries

- Humans settle outcomes, public contracts, consequential trade-offs, acceptance criteria and taste. Research facts yourself. Reversible implementation choices belong to the agent. Respect existing approvals; ask only about an unresolved material decision.
- `implement` authorizes the local implementation → review → repair → QA loop within the approved spec. Continue across phase boundaries and compaction. Publishing, deployment and merging follow the user's actual authorization; the default endpoint is a result ready for human acceptance.
- Accepted risks apply under their recorded assumptions. Do not quietly expand scope to address them. If evidence invalidates an assumption or reveals a different consequence, surface that evidence.
- Required acceptance tests use the agreed interfaces; supporting tests follow repo practice. Preserve relevant lint, compile and test gates. Verify UI interactions with a browser; protocol checks alone cannot prove them.
- Read `./simplicity-discipline.md` for implementation and review. Prefer the smallest solution satisfying the contract; prioritize findings by impact, not line count.
- Use the capabilities exposed by the host: Claude Code in Desktop or CLI, or Codex in its desktop app or CLI. Native subagents can execute substantial bounded work; otherwise execute directly with the same checkpoints. No particular delegation API or supervisor is required.
- Review defaults to the other model family: Codex-led implementation gets Claude review; Claude-led implementation gets Codex review. The implementing agent validates findings, repairs valid issues and follows up with the reviewer until resolved or blocked. Read `./review-steps.md` and `./reviewer-transport.md` when reaching review or checking reviewer access. A same-model subagent does not satisfy cross-model review.
- Git messages and PRs describe engineering changes using project conventions, without internal phase labels. Commit only task-owned changes.
- Workflow documents are temporary. At complete, preserve useful decisions and evidence in durable project homes before removing the workflow folder. Prototypes never merge.
