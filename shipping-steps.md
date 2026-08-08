# Shipping Steps: push, complete

## /flow push <slug>

Push feature branches and create pull requests. Separated from QA so PRs can go up independently. Read `state.yaml` (worktrees, ticket, repos), `spec.md`, `plan.md`, and the tracker/commit settings from `.flow/config.yml`.

**Preflight - warn, don't hard-block.** Silent when green; if any of these hold, list them and get the user's explicit go-ahead before pushing:

- Phases not `committed` (`pending`, `in-progress`, or `failed`).
- `review-code.md` missing, or its latest pass has `open` findings.
- The latest `qa[]` entry's verdict is not `PASS` (or QA never ran).
- state.yaml `blockers` entries whose conditions aren't verifiably met - they exist precisely to gate this step.

**Per repo, from the working-tree path.** Skip repos with no commits ahead of `origin/<default>` and note them; a repo with no `origin` remote has nothing to push - report it and treat its branch as local-only (complete verifies it against the local default branch). For each remaining repo:

- **Push the branch whenever it has commits the remote lacks** (`git push -u origin <slug>`) - including re-pushes after QA/review fix commits; an existing PR never suppresses the push.
- **Create the PR** when the remote supports it and the tooling exists (GitHub remote + `gh`: check for an existing open PR for the branch first - if one exists, record its URL and skip only the creation, since a previous push run may have died before writing state; otherwise `gh pr create`). Title per the project's commit convention + engineering language (principle 10 - never internal workflow naming); description summarizes what was built (from the spec) in engineering language, links the primary ticket per the tracker's convention (`Closes #42` on GitHub), and lists per-phase tickets when state.yaml has a `tickets:` map.
- **No PR tooling** (non-GitHub remote without a CLI, or no remote PR flow at all): push the branch, record `branch pushed: <slug>` in state.yaml `prs`, and tell the user - they can open an MR manually or merge locally if that's the project's style.

**Record PR URLs** in state.yaml (`prs`) and report:
> PR(s) created: <repo>: <url>
> Next: `/flow qa <slug> <env>` for environment QA, or `/flow complete <slug>` if QA is done.

## /flow complete <slug>

1. **Verify shipped:** `git fetch origin` first per repo (stale local refs report freshly-merged PRs as open), then for each `prs` entry with a PR URL check the merge state (`gh pr view --json state,mergedAt` on GitHub). For branch-only entries, verify the branch is merged into the default branch (`git branch -r --merged origin/<default>` contains it - or the local default branch for remote-less repos). All merged - proceed. Otherwise report what's still open and stop unless the user explicitly overrides.

2. **Graduate lessons.** The workflow folder is about to be deleted, so anything with lasting value must move to a durable home now. Mine the run's artifacts (phase logs, review-code.md, QA docs, journal.md if present) for what went wrong, took longer than expected, or would trip the next agent, then route each candidate by audience:

   | Audience | Home |
   |----------|------|
   | Everyone working in a repo (conventions, gotchas, toolchain facts) | That repo's `CLAUDE.md` / `AGENTS.md` - propose the exact diff; it's a shared file, so the user approves before it lands |
   | A durable architectural decision (three-part ADR test from grill-discipline.md) | Recommend an ADR in the project's convention; the user decides |
   | Only this user / this machine (personal tooling, env quirks) | The user's own memory/notes - offer, don't push |

   Most candidates won't clear the repo-CLAUDE.md bar ("useful to everybody") - that is the filter working, not a failure. Discard the rest with the folder.

3. **Clean up:**
   - Delete leftover checkpoint tags in each repo: `git tag -l "workflow-checkpoint-<slug>-*" | xargs -r git tag -d`
   - Delete the prototype branch(es) if `state.yaml` records `prototype_branches`.
   - Remove worktrees per entry in `state.yaml` `worktrees`: `git -C <repo> worktree remove <path>` (`--force` if the tree is dirty - QA and dev-server artifacts make that common), then delete the merged local feature branch: `git -C <repo> branch -d <slug>`. When `worktrees: false` recorded the main checkout instead: check out the default branch there, then delete the feature branch the same way.

4. **Delete the workflow folder** - the whole of `.flow/<slug>/`, state.yaml included. Once shipped, the durable truth is the merged code, the PRs, and the ticket; a kept spec describes the feature as designed at one moment and goes confidently stale. A completed workflow leaves no local state and disappears from `/flow list` - by design.

5. **Report:** PRs verified merged, lessons graduated (and where each landed), what was deleted.
