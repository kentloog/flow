# state.yaml Contract

Single source of truth for a workflow's state. Path: `<project-root>/.flow/<slug>/state.yaml`. Created by `idea`, read first and updated by every subsequent subcommand. On every write: bump `updated` and set `current` to the subcommand that is running. (Project-level configuration lives in `.flow/config.yml`, not here - state.yaml is per-workflow.)

**Schema keys only.** state.yaml holds the keys below and nothing else - narrative prose, deploy records, and session notes go to `journal.md` in the slug folder (which no subcommand reads by default). Two sanctioned homes for operational state the next session must see: `blockers:` (short strings - interlocks, cutoffs, do-not-merge-before conditions) and `next:` (one line). Every generated state.yaml starts with the header comment:

```yaml
# schema keys only (see state-schema.md); narrative goes to journal.md
```

## Schema

```yaml
slug: 42-token-refresh
title: Token refresh for user sessions
ticket: "#42"                  # primary ticket (or epic) in the tracker's ref format; null until known
map: platform-observability    # wayfinder map slug this workflow was spawned from; omit otherwise
repos: [backend, frontend]     # multi-repo: config repo names. Single-repo: [root] (root = the project root, path ".")
created: 2026-08-08
updated: 2026-08-08T14:32:00
current: implement             # the step that ran last or is running now
next: run /flow qa after the lib bump lands        # optional, one line
blockers:                      # optional, short strings only
  - "Do not push frontend before backend's API change is merged"

steps:                         # pending | in-progress | done | skipped
  idea: done
  research: skipped            # optional steps: research, prototype, ticket, qa
  prototype: skipped
  spec: done
  plan: done
  ticket: done
  implement: in-progress
  review-code: pending
  qa: skipped                  # optional steps start skipped; a non-PASS qa pass flips it to in-progress, done only on PASS
  push: pending
  complete: pending

worktrees:                     # written by implement; complete removes the worktrees, then deletes this file
  backend: /home/user/projects/myapp/.flow/worktrees/backend--42-token-refresh

phases:                        # structure written by plan; status owned by implement
  - n: 1
    title: Schema and session repository
    repo: backend
    blocked_by: []             # phase numbers that must be committed first
    status: committed          # pending | in-progress | committed | failed
  - n: 2
    title: Token refresh endpoint and UI wiring
    repo: backend
    blocked_by: [1]
    status: pending

reviews:
  code: { passes: 2, verdict: clean }        # clean | findings-open

qa:                            # appended per QA pass
  - pass: 1
    env: local                 # local | <env name from config>
    date: 2026-08-09
    verdict: NEEDS_CHANGES     # PASS | NEEDS_CHANGES | FAIL
    open_findings: 2

prs:                           # written by push: PR/MR URL, or "branch pushed: <branch>" when no PR flow exists
  backend: https://github.com/user/myapp/pull/57

tickets:                       # only for one-ticket-per-phase work; phase n -> ticket ref
  1: "#43"
  2: "#44"

prototype_branches:            # only if prototype code was parked; complete deletes the branches
  backend: proto/42-token-refresh
```

Omit empty optional keys (`map`, `next`, `blockers`, `worktrees`, `phases`, `reviews`, `qa`, `prs`, `tickets`, `prototype_branches`) until they have content.

## Update duties per subcommand

| Subcommand    | Writes to state.yaml |
| ------------- | -------------------- |
| idea          | Creates folder + file: header comment, slug, title, repos, ticket if known, `map` when spawned from a wayfinder map, all steps pending (optionals skipped), `steps.idea: done` |
| research      | `steps.research: done` |
| prototype     | `steps.prototype: done`; `prototype_branches` if code was parked |
| spec          | `steps.spec: done` |
| plan          | `steps.plan: done`; `phases[]` with titles, repos, `blocked_by` edges, all statuses `pending` |
| ticket        | `ticket:` (primary); `tickets:` phase-to-ref map for one-ticket-per-phase work; `steps.ticket: done` |
| implement     | `steps.implement: in-progress` at start, `done` when every phase is `committed`; `worktrees`; per-phase status transitions. Findings-fix runs also flip each fixed finding's Status to `fixed` in the source review/QA doc after its commit - the doc is the durable fix state |
| replan        | Rewrites `phases[]`; entries with `status: committed` stay byte-identical |
| review        | `steps.review-code: done`; `reviews.code` |
| qa            | Append a `qa[]` entry per pass (pass, env, date, verdict, open_findings); `steps.qa: done` when a pass ends PASS, `in-progress` after a non-PASS pass |
| push          | `steps.push: done`; `prs` |
| complete      | Deletes the entire workflow folder, state.yaml included - a completed workflow leaves no local state (`steps.complete` never persists as `done`) |

## Derived reads

- **QA gate**: `qa[]` non-empty with the latest entry's verdict not `PASS` means QA has open findings - status shows it and "next" is the fix route (`/flow implement <slug>`), even though `steps.qa` is not `done`. `steps.qa: done` alone never proves the latest environment passed; always read the latest `qa[]` entry.
- **status**: render the checklist from `steps`, phase progress from `phases`, latest QA verdict from `qa`.
- **list**: read every `.flow/*/state.yaml`, sort by `updated`.
