# QA: <title>

**Slug:** <slug>
**Ticket:** <reference, if applicable>
**Environment:** local | <configured environment>

## QA Pass 1

**Date:** YYYY-MM-DD
**Tested revisions:** <repo: SHA; deployed version evidence when applicable>
**Approved spec digest:** <SHA256>
**Services:** <start commands, process IDs owned by this pass, readiness and code-version evidence>

### Results

| Criterion / scenario | Method and reproduction steps | Expected | Observed | Result | Evidence |
|----------------------|-------------------------------|----------|----------|--------|----------|
| ... | API / browser / DB / CLI / job | ... | ... | PASS / FAIL / UNVERIFIED | log/screenshot/report path |

### Findings

| ID | Severity | Expected vs actual / impact | Repro command or UI steps | Plan phase to fix | Status | Resolution evidence |
|----|----------|-----------------------------|--------------------------|-------------------|--------|---------------------|
| Q1 | ... | ... | ... | ... | open | ... |

Status: `open`, `fixed`, `wontfix`. Retain IDs across passes and qualify by environment when passing to repairs. Fixed requires reproduction evidence; wontfix links an accepted decision. Carry unresolved findings into every new pass.

### Verdict

**PASS** / **NEEDS_CHANGES** / **BLOCKED** / **FAIL**

Unverified required criteria, blockers, and the reason any unaffected evidence was reused. Include restart/demo instructions and cleanup of processes owned by this pass. Append subsequent passes; preserve historical verdicts.
