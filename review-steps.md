# Review

One review pass by the other model family, validated and fixed by the implementing side. The reviewer advises; the coordinator fixes.

## Reviewer

`review.reviewer` in `.flow/config.yml`:

| Value | Reviewer |
|-------|----------|
| `auto` (default) | Claude for Codex-led work, Codex for Claude-led work |
| `claude` / `codex` | That provider |
| `current` | Self-review; the report says so |
| a command | That command; confirm which provider it runs |

Family follows who wrote the code, not which app is open. When authorship is unknown, treat the current coordinator's family as the author. A Claude subagent reviewing Claude's code is self-review.

Reach the other family through the MCP bridge the host exposes. Discover the callable tools and read their schemas rather than assuming names; the usual shapes are a Codex bridge in Claude Code (`codex`, then `codex-reply` with the returned `threadId`) and a Claude reviewer bridge in Codex (`claude`, then `claude_reply` with the returned `sessionId`). Send the brief as the prompt and nothing else: the bridge's model, approval policy, sandbox and working directory come from the user's own configuration, and overriding them from here misconfigures the review. Record the provider and the returned session id in `state.yaml` as soon as the first call returns, and pass that id to every follow-up so the reviewer keeps its context.

The bridge inherits the host's permissions, so the brief carries the role: the reviewer reads the source and the diff, reports findings, and makes no edits, commits or state changes. It does not call another reviewer. Fixes belong to the implementing side.

No bridge configured is a blocker to resolve in setup, never a silent fall back to self-review. A timeout is not a finished reviewer: read the session for a result before sending anything again, and continue that session instead of starting a second one. A session id is valid only for the provider and host that issued it; after a host change, start a fresh session with the brief and the latest report.

## Brief

The reviewer sees none of your conversation. Write the brief to a file with absolute paths to `spec.md`, the worktrees and the base commits, the diff command (`git diff <base>...HEAD`), the accepted risks, and the repo's documented standards if any. Ask for two reports, each under 400 words:

- Spec: requirements missing or partial, behaviour nobody asked for, requirements that look implemented but wrong. Quote the spec line per finding.
- Standards and simplicity: breaches of documented standards, and places where a concrete simpler design keeps the behaviour. Judgement calls labelled as such. Skip what tooling already enforces.

Ask for everything with evidence; you filter. Asking the reviewer to be conservative makes it report less.

## Triage and fix

Check each finding against the code and the spec. Reject what the evidence does not support, what the spec accepted as a risk and pre-existing issues outside the change, and write down why. Group the valid findings into one fix unit per worktree. Reply in the reviewer's session with the dispositions, the fix commits and the check results, and ask it to reassess. Stop when no supported finding remains. A first pass with no findings needs no reply. A dispute that repeats without new evidence becomes an open decision for the human.

Depth follows the plan's readiness section: a single pass for small changes, pass plus reassessment when fixes were made or the plan asked for it.

Write `review-code.md` per `./review-template.md` before fixing, and record the verdict, scope and reviewed revisions in `state.yaml`. Only a full-scope clean review counts toward readiness and push. Fixed findings do not make an old review current; changed code gets re-reviewed.
