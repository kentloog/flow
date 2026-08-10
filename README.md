# flow

A product delivery workflow skill for [Claude Code](https://claude.com/claude-code): take an idea through a relentless grilling interview, a synthesized spec with agreed test seams, a tracer-bullet phase plan, autonomous implementation via subagents, cross-model code review, and QA - all the way to merged PRs.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/pipeline-dark.svg">
  <img alt="The flow pipeline. An optional chart (wayfinder map) step feeds a human-in-the-loop band: idea (grill) → research (optional) → prototype (optional) → spec (confirm seams) → plan (tracer bullets) → ticket (optional). Then an autonomous AFK band: implement (frontier subagents) → review (cross-model) → local QA (optional), where review and QA findings re-enter implement as targeted fixes. Then a ship band with human checkpoints: push (branches and PRs) → env QA (optional) → complete (verify and clean up), with env QA findings also looping back to implement." src="assets/pipeline-light.svg" width="940">
</picture>

## Why

Most agent-assisted feature work fails in the same places: decisions that were never actually made surface as guesses mid-implementation, tests get invented at whatever seam was convenient, reviews reward overbuilding, and "done" is declared without anyone reproducing the feature working. flow attacks each of those directly:

- **Decisions are made at idea, not at spec.** The idea step grills you - one question at a time, a recommended answer per question, facts looked up rather than asked - until shared understanding is confirmed. Scenarios you decline to handle become **Accepted Risks**: documented decisions that bind implementers (never "fix" one) and reviewers (never flag one).
- **Tests go through pre-agreed seams.** The spec step's one human checkpoint is confirming the interfaces the feature will be tested through. Implementation subagents test only there; a criterion untestable through a listed seam is a spec problem, reported rather than patched around.
- **Implementation runs AFK and survives crashes.** One fresh subagent per plan phase, frontier-scheduled off explicit blocking edges, each committing its own work behind a typecheck/lint/test gate. State lives in `state.yaml` plus git checkpoint tags, so a dead session resumes with the same command.
- **Review is adversarial in both directions.** A second model (Codex MCP when available, any CLI you configure, or a Claude subagent) reviews the diff against the spec; then one Claude verifier per finding tries to *kill* it - hallucinated anchors, pre-existing code, accepted risks, and code-adding findings without a spec citation all die before they can demand action. Simplicity findings carry the same weight as bugs.
- **Simplicity is enforced, not hoped for.** Every implementer, fix agent, and reviewer gets the same constraint block: a decision ladder ending at "the minimum code that satisfies the spec", no abstraction without a second consumer, and a mandatory Skipped/add-when list so omissions are visible decisions instead of silent gaps.
- **Workflow docs are temporary.** `complete` verifies the PRs merged, graduates lessons to durable homes (repo CLAUDE.md, ADRs, your notes), and deletes the whole workflow folder. Durable truth is the merged code, the PRs, and the tracker.

## Requirements

- [Claude Code](https://claude.com/claude-code) and `git`. That's it for the core loop.
- Optional, used when present:
  - `gh` CLI - PR creation and merge verification on GitHub remotes
  - An issue tracker - GitHub Issues, Jira, Linear, or a plain markdown backlog file (configured per project)
  - A second-model reviewer - the [Codex MCP server](https://developers.openai.com/codex/mcp/) or any CLI (`codex exec`, `gemini`, ...); flow falls back to a Claude-only review with the same adversarial validation
  - Browser tools - UI QA with screenshots; Claude Code's built-in browser needs no setup, and a browser MCP (e.g. chrome-devtools) adds performance traces and Lighthouse on top

## Install

Clone into your Claude Code skills directory (all projects):

```bash
git clone https://github.com/kentloog/flow.git ~/.claude/skills/flow
```

Or scope it to a single project by cloning into `<project>/.claude/skills/flow` instead.

Skills are discovered when a session starts, so open a **new** Claude Code session afterwards and type `/flow` - it should appear in the command suggestions. If you already have a skill named `flow`, clone to a different directory name (the directory name is the skill name; adjust the `name:` field in `SKILL.md` to match).

To update later:

```bash
git -C ~/.claude/skills/flow pull
```

## Quick start

```
cd your-project
claude
> /flow idea add rate limiting to the public API
```

The first run creates `.flow/config.yml` through a short setup interview (issue tracker, commit convention, worktrees, how to run the app locally). After that:

| Command | What it does |
|---------|--------------|
| `/flow idea <description>` | Capture the idea, then grill the decisions out of it |
| `/flow research <slug>` | Codebase + external exploration of factual open questions (optional) |
| `/flow prototype <slug>` | Throwaway code that answers one question (optional) |
| `/flow spec <slug>` | Synthesize the spec; confirm test seams |
| `/flow plan <slug>` | Tracer-bullet phases with blocking edges; quiz until approved |
| `/flow ticket <slug>` | Tracker tickets from the plan (optional) |
| `/flow implement <slug>` | Autonomous frontier-scheduled implementation via subagents |
| `/flow review <slug>` | Cross-model review with adversarial validation |
| `/flow replan <slug>` | Regenerate pending phases after spec changes |
| `/flow qa <slug> [env]` | QA locally, or against a configured deployed environment |
| `/flow push <slug>` | Push branches and create PRs |
| `/flow complete <slug>` | Verify merged, graduate lessons, delete the workflow folder |
| `/flow chart <description>` | Wayfinder map for efforts too big for one spec |
| `/flow status <slug>` / `/flow list` | Progress |

For a fully AFK implementation run: `claude --dangerously-skip-permissions`, then `/flow implement <slug>`, and walk away - if the session dies, the same command resumes from state.

You can also just say **"grill me"** about any plan or decision to get the interview without the pipeline.

## Configuration

`/flow setup` writes `.flow/config.yml` in your project root (and re-runs any time to change it):

```yaml
layout: single-repo          # single-repo | multi-repo (repos as subdirectories)
repos: []                    # multi-repo only: repo subdirectory names

tracker:
  type: github               # github | jira | linear | markdown | none
  usage: "gh issue view/create in this repo"   # free text: how to interact with it
  ref_format: "#42"          # how a ticket is referenced in commits/branches

commit_convention: "#42: Imperative summary of what changed"

worktrees: true              # false = work on a branch in the main checkout

run:
  notes: |                   # how to run the app locally for QA
    npm run dev              # serves on :3000

envs:                        # optional deployed environments for `/flow qa <name>`
  - name: staging
    url: https://staging.example.com
    notes: "deploy: git push staging main; logs: flyctl logs -a myapp"

review:
  reviewer: auto             # auto (Codex MCP if available, else Claude) | claude | "<shell command>"
```

Everything else flow creates lives under `.flow/<slug>/` - one self-contained folder per workflow, deleted in full at `/flow complete`.

## How it's built

`SKILL.md` is a thin router; each subcommand loads only its own instruction file. Disciplines (`grill-discipline.md`, `simplicity-discipline.md`) are shared prompts injected into subagents; templates define every document the workflow writes; `state-schema.md` is the contract for `state.yaml`. Read `SKILL.md` first if you want to modify anything.

## Credits

The grilling interview discipline is sourced from Matt Pocock's grilling skill.

## License

[MIT](LICENSE)
