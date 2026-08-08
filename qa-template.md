# QA: <title>

**Date:** YYYY-MM-DD
**Slug:** <slug>
**Ticket:** #42 / PROJ-123
**Environment:** local | <env name from config>
**Branch:** branch-name

## Scope

What was implemented. Which plan phases this QA pass covers.

## Test Scenarios

### <Scenario 1: Happy path>

**Steps:**
1. ...

**Expected result:** ...

### <Scenario 2: Edge case>

...

## QA Pass 1

### Services / Environment Validated

| Service | Mode/Env | Status |
|---------|----------|--------|

### Test Results

| # | Scenario | Method | Result | Details |
|---|----------|--------|--------|---------|
| 1 | ... | API / UI / DB / Logs | PASS/FAIL | ... |

### Findings

| # | Severity | Description | Repro command | Plan phase to fix | Status |
|---|----------|-------------|---------------|-------------------|--------|
| 1 | ... | ... | curl / UI path / SQL | ... | open |

Status values: `open`, `fixed`, `wontfix`. The repro command is mandatory - the fix subagent re-runs it green before the finding flips to `fixed`.

### Verdict

**PASS** / **NEEDS_CHANGES** / **FAIL**

<!-- Subsequent passes: copy this section as "## QA Pass 2", etc.
     Previous pass findings with status "open" carry forward.
     Only the LATEST pass's "open" findings feed /flow implement fix generation,
     which flips them to "fixed" as fixes commit. -->
