# Setup Steps: setup

Create or update `<project-root>/.flow/config.yml` - the per-project configuration every other subcommand reads. Runs standalone via `/flow setup`, and **automatically** when any subcommand needs config that doesn't exist yet: run the interview, write the file, then continue with the original subcommand.

Setup is a short interview using the grill mechanics (`grill-discipline.md`, this directory): one question at a time, a recommended answer per question, and facts looked up rather than asked.

## 1. Detect before asking

Look these up first so every question ships with an informed recommendation:

- **Layout:** is the project root itself a git repo (single-repo), or a folder containing several repos as subdirectories (multi-repo)?
- **Default branch** per repo: `git symbolic-ref refs/remotes/origin/HEAD` (fall back to `git remote show origin`).
- **Remote host:** GitHub remote? Is `gh` installed and authenticated?
- **Tracker hints:** `.github/ISSUE_TEMPLATE/`, ticket-key patterns in recent commit messages (`#NN`, `ABC-123`), tracker MCP tools available in the session.
- **Run hints:** `package.json` scripts, `Makefile`, `docker-compose.yml`, README run instructions.
- **Reviewer hints:** is the Codex MCP (`mcp__codex__codex`) available? Any other second-model CLI on PATH (`codex`, `gemini`)?

## 2. Interview

Ask only what detection couldn't settle; confirm the rest in one summary. Cover:

1. **Layout** - single-repo or multi-repo; for multi-repo, which subdirectories are repos.
2. **Issue tracker** - `github | jira | linear | markdown | none`. For anything but none: how to interact with it (CLI, MCP tool, or a file path for a markdown backlog) and how a ticket is referenced in branches/commits (e.g. `#42`, `PROJ-123`).
3. **Commit convention** - one example message the project uses (e.g. `#42: Add rate limiting to the export endpoint`, or conventional commits `feat: ...`). This becomes the template every implement subagent follows.
4. **Worktrees** - recommend `true` (AFK implement can't wreck the main checkout, and parallel multi-repo phases need it); `false` is fine for a solo single-repo project where working on a branch in place is acceptable.
5. **Running locally** - the commands, ports, and any DB/service dependencies needed to run the app for QA. Free text; stored verbatim for the QA step to follow.
6. **Deployed environments** (optional) - name, base URL, and free-text notes per environment: how to deploy a branch there, how to read its logs.
7. **Reviewer** - default `auto` (Codex MCP when available, else Claude). If detection found a second-model CLI and the user wants it as the reviewer, write the concrete command as the value (e.g. `"codex exec"`) - `auto` never resolves to a CLI on its own. Only ask when detection found options or the user raises it.

## 3. Write the config

```yaml
# /flow project configuration - edit freely; /flow setup updates it
layout: single-repo          # single-repo | multi-repo
repos: []                    # multi-repo only: repo subdirectory names, e.g. [backend, frontend]

tracker:
  type: github               # github | jira | linear | markdown | none
  usage: "gh issue view/create in this repo"   # free text: the tool/CLI/MCP/file and any conventions
  ref_format: "#42"          # how a ticket is referenced in commits/branches; omit when type is none

commit_convention: "#42: Imperative summary of what changed"   # example message; drop the ref when no ticket exists

worktrees: true              # false = code-touching steps work on a branch in the main checkout

run:
  notes: |                   # how to run the app locally for QA: commands, ports, DB/services
    npm run dev              # serves on :3000
    docker compose up -d db  # postgres on :5432

envs: []                     # optional deployed environments for `/flow qa <name>`
# - name: staging
#   url: https://staging.example.com
#   notes: "deploy: git push staging main; logs: flyctl logs -a myapp"

review:
  reviewer: auto             # auto (Codex MCP if available, else Claude) | claude | "<shell command>" for a second-model CLI
```

Omit optional keys that have no content rather than writing empty placeholders (keep the commented examples for `envs`).

## 4. Version control

Offer to add this to `.gitignore` (recommended):

```gitignore
.flow/*
!/.flow/config.yml
```

Workflow folders are temporary and machine-local by design; the config is worth sharing with collaborators. If the user prefers everything untracked, ignore `.flow/` wholesale.

> Config written to `.flow/config.yml`. <Continue with the original subcommand, or:> Start a workflow with `/flow idea <description>`.
