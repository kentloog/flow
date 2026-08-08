# Review: <title>

**Date:** YYYY-MM-DD
**Slug:** <slug>
**Ticket:** #42 / PROJ-123 (if applicable)
**Target:** diff origin/<default-branch>...HEAD per repo

<!-- Repeated reviews append a new "## Review Pass N" section; only the LATEST pass's
     "open" findings feed /flow implement fix generation, which flips them to
     "fixed" as fixes commit. Number findings sequentially across categories within a
     pass so every finding has a stable ID. -->

## Review Pass 1

**Reviewer:** Codex MCP | <CLI command> | Claude subagent (note why the fallback)
**Re-review loops:** N

Findings stay in their category's section. Do NOT merge or rerank findings across categories - a change can pass one category and fail another, and reporting them separately stops one category from masking the other.

### <Category: spec-fidelity | simplicity | conventions | bug | test | cross-repo>

| # | Anchor | Tag | Finding | Status |
|---|--------|-----|---------|--------|
| 1 | file:line | yagni: | ... | open |

Or: clean.

Status values: `open`, `fixed`, `wontfix`

### Killed in validation

- Finding and the reason it died (bad anchor, pre-existing code, accepted risk, scope inflation, unreachable claim disproven). Kept for transparency, not action; reported back to the reviewer so it isn't re-raised.

### Verdict

Per category: findings count and the worst issue within that category. No single winner across categories. Simplicity category ends with `net: -N lines possible` or `lean already`.

### Actions Taken

- What was updated (fix commits) in response, per re-review loop.
