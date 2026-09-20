# Workflow state

`<project-root>/.flow/<slug>/state.yaml` is the index a new session reads to continue. The coordinator writes it, whole and valid, after each change; workers write reports under `logs/` instead. Details live in the reports. State holds what is needed to pick the work up.

```yaml
slug: token-refresh
title: Token refresh for user sessions
ticket: "#42"                    # optional
repos: [root]                    # root = the project root
map: platform-observability      # optional parent map
updated: 2026-09-20T14:32:00Z
status: running                  # running | ready | blocked, set by execution
next: Continue phase 2
blockers: []
steps:                           # pending | in-progress | done | skipped
  idea: done
  research: skipped
  prototype: skipped
  spec: done
  plan: done
  ticket: skipped
  implement: in-progress
  review: pending
  qa: pending
  push: pending
phases:
  - {n: 1, title: Refresh an expired session, repo: root, blocked_by: [], status: committed}
  - {n: 2, title: Revoke on password change, repo: root, blocked_by: [1], status: in-progress}
worktrees: {root: /project/.flow/worktrees/project--token-refresh}
bases: {root: 3f2a9c1}           # commit each branch started from
reviewer: {provider: codex, session: 019a...}   # recorded when review starts
review: {verdict: clean, scope: full, revisions: {root: 8b1e0d4}, report: review-code.md}
qa:
  local: {verdict: PASS, revisions: {root: 8b1e0d4}, report: qa-local.md}
prs: {root: https://github.com/example/project/pull/42}
tickets: {1: "#43"}              # phase -> tracker reference, when ticketed
prototype_branches: {}           # repo -> parked prototype branch
```

Phase status is `pending`, `in-progress`, `committed` or `failed`. Omit keys until they have content. Review and QA entries name the revisions they ran on; a later commit makes them stale, and `ready` needs both to match the current heads.

`flow status` shows `status`, `next` and `blockers` first, then steps, the phases with the ready frontier, and the latest review and QA. `flow list` reads every `.flow/*/state.yaml` sorted by `updated` and lists map folders (`map.md`, no state) separately. A missing folder means the workflow completed or never existed.
