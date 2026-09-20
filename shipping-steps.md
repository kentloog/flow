# Shipping: push, complete

## /flow push <slug>

Publish the feature branches and open pull requests. An explicit `push` authorizes publication; inside an autonomous run, publish only when the user asked for it up front.

Preflight, silent when green. Check the readiness conditions in `./state-schema.md`, including current evidence, a full clean review and local QA `PASS`. If any fail, list them and get the user's go-ahead for publishing that incomplete result. Authorization to publish does not change the recorded verification verdicts.

Per repo, from its worktree: skip repos with nothing ahead of `origin/<default>`. Push the branch (`git push -u origin <slug>`), including re-pushes after fixes. On a GitHub remote with `gh`, reuse an open PR for the branch if one exists, otherwise create one. Title per the project's commit convention, body per the template below. Without PR tooling, push and record `branch pushed`.

Record PR URLs in `prs`, then report them with the next step: `/flow qa <slug> <env>` or `/flow complete <slug>`.

### PR body

Adapted from Matt Pocock's `pr` skill, which credits Dex Horthy's show-me. Use the project's domain language and no internal phase names.

```markdown
Closes #42

## Summary

<the smallest view that makes the change clear: pseudocode for logic, a call tree
for control flow, a component or file tree for structure, a diff-sketch when the
shape already exists, Mermaid for interaction between parts. Usually one of these.>

## Evidence

- Before: <screenshot, output or failing test>
  After: <screenshot, output or passing test>

## Merge danger

Door: <one-way or two-way, and why>
Blast radius: <one word, then what it could affect>
```

Screenshots are the best evidence for visual changes, test or command output next. Point at the QA report's evidence rather than re-running it.

## /flow complete <slug>

1. Verify shipped. `git fetch origin` per repo, then check each PR is merged (`gh pr view --json state,mergedAt`) or each branch is in `origin/<default>`. Anything open: report and stop unless the user overrides.

2. Retro. The workflow folder is about to go, so read the reports once for anything that would improve the next run's environment. Categories, from Matt Pocock's `retro` skill:

   - Navigation: a file or dependency that took long to find wants a pointer in the repo's `CLAUDE.md` or `AGENTS.md`.
   - Automated checks: a mistake a linter, type check, test or hook would have caught wants that check. An existing check that is unwired or broken is the finding.
   - Standards: a mechanical rule (banned API, import shape, file location) becomes a check, never prose. A judgement call goes to the repo's coding standards, which reviewers read, not implementers.
   - Tool economy and information access: a slow tool, a log the agent could not see.
   - No-ops: an instruction in a steering file that changed nothing.

   Build the check over writing the rule. Keep `CLAUDE.md` to navigation pointers. A decision that is hard to reverse, surprising without context and the result of a real trade-off is an ADR candidate; the user decides. Propose diffs to shared files for the user to approve. Most runs need nothing.

3. Clean up. Remove worktrees (`git -C <repo> worktree remove <path>`; inspect dirty files first and keep anything unexplained), then delete merged local branches and parked prototype branches. With `worktrees: false`, check out the default branch first.

4. Delete `.flow/<slug>/`. Merged code, PRs and tickets are the record; a kept spec goes stale.

5. Report: PRs verified merged, retro items and where they landed, what was deleted.
