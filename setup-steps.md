# Setup

Create or update `<project-root>/.flow/config.yml`, the per-project configuration every other subcommand reads. Runs as `/flow setup`, and automatically when a subcommand finds no config: interview, write the file, continue with the original subcommand.

Setup is a short interview using the grill mechanics in `./grill-discipline.md`: a round of independent questions with recommended answers, facts looked up rather than asked.

## 1. Detect before asking

- Layout: is the project root itself a git repo (single-repo), or a folder of repos (multi-repo)?
- Default branch per repo: `git symbolic-ref refs/remotes/origin/HEAD`, else `git remote show origin`.
- Remote host: GitHub? Is `gh` installed and authenticated?
- Tracker hints: `.github/ISSUE_TEMPLATE/`, ticket patterns in recent commits (`#42`, `ABC-123`), tracker tools in the session.
- Run hints: `package.json` scripts, `Makefile`, `docker-compose.yml`, README run instructions.
- Reviewer access: which MCP bridge reaches the other model family, a Codex bridge in Claude Code or a Claude reviewer bridge in Codex (see `./review-steps.md`).

## 2. Interview

Ask only what detection could not settle and confirm the rest in one summary:

1. Layout, and for multi-repo the repo subdirectories.
2. Issue tracker: `github | jira | linear | markdown | none`; how to reach it (CLI, tool, or backlog file) and how a ticket is referenced in branches and commits.
3. Commit convention: one example message.
4. Worktrees: recommend `true` so an AFK run never touches the main checkout; `false` suits a solo single-repo project happy to work on a branch in place.
5. Running locally: commands, ports, services needed for QA. Stored verbatim.
6. Deployed environments (optional): name, URL, how to deploy a branch and read logs.
7. Reviewer: `auto` reviews with the other model family through its MCP bridge. `claude`, `codex`, `current` or a command are explicit choices; `current` opts out of independent review. Configure the bridge before the first AFK run.

## 3. Write the config

```yaml
# /flow project configuration; /flow setup updates it
layout: single-repo          # single-repo | multi-repo
repos: []                    # multi-repo only, e.g. [backend, frontend]

tracker:
  type: github               # github | jira | linear | markdown | none
  usage: "gh issue view/create in this repo"
  ref_format: "#42"          # omit when type is none

commit_convention: "#42: Imperative summary of what changed"

worktrees: true

run:
  notes: |
    npm run dev              # serves on :3000
    docker compose up -d db  # postgres on :5432

envs: []                     # optional, for `/flow qa <name>`
# - name: staging
#   url: https://staging.example.com
#   notes: "deploy: git push staging main; logs: flyctl logs -a myapp"

review:
  reviewer: auto             # auto | claude | codex | current | "<review command>"
```

Omit optional keys with no content; keep the `envs` example comment.

## 4. Version control

Offer to add to `.gitignore`:

```gitignore
.flow/*
!/.flow/config.yml
```

Workflow folders are temporary and machine-local; the config is worth sharing. If the user prefers, ignore `.flow/` wholesale.

> Config written to `.flow/config.yml`. <Continue with the original subcommand, or:> Start a workflow with `/flow idea <description>`.
