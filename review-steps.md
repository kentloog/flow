# Review pass

Review the integrated change against the full approved spec and relevant repo conventions. This procedure produces one durable pass; `./execution-steps.md` owns repair and re-review. A standalone `flow review` reviews and repairs existing work, then returns. When phases remain unfinished, label the scope partial and list those phases as remaining work, not defects to implement during review.

## Reviewer and inputs (coordinator only)

Resolve config `review.reviewer` using `./reviewer-transport.md`. With `auto`, select the opposite of `execution.implementation_family`: Codex-led work gets Claude, Claude-led work gets Codex. For existing work without that field, use known implementation provenance; for standalone review where authorship is unknown, select the opposite of the current coordinator and disclose that limit. An explicit user cross-model requirement takes precedence over a same-family or `current` config. Missing required reviewer access blocks clean completion; never silently downgrade to self-review.

Use one independent reviewer session for the whole change where practical. Split a large review into bounded areas only when useful; every area gets the full spec path and relevant integration contracts, and one pass consolidates the results. The implementing agent validates the findings; a reviewer fleet or separate validator is not required. Reviewers inspect and advise, while the implementation side owns all fixes.

Pass absolute spec and working-tree paths, recorded base commits, current heads and report path. Include repo conventions and a self-contained statement of scope, accepted risks and review criteria; the reviewer cannot see the coordinator's conversation. Review each base-to-head diff and surrounding source needed to assess it. An empty diff is not automatically a failure: verify whether the approved behavior already exists and record that conclusion with evidence. Read full requirements to catch omissions; implementation summaries are navigation aids, not the requirements source.

## Review criteria

- Spec fidelity: missing, partial, incorrect or unrequested behavior.
- Correctness and regressions introduced or exposed by the change, including security and data integrity.
- Simplicity and conventions: use `./simplicity-discipline.md`; a simpler design needs a concrete maintenance or correctness benefit, not a line-count target.
- Test sufficiency at approved acceptance interfaces and relevant supporting interfaces.
- Integration contracts across changed components and repos.

Findings need a concrete trigger, impact, evidence and a repair direction. Anchor to the actual source/symbol or to an omitted requirement; an omission need not have a changed line. Spec-fidelity findings cite the requirement. Regressions can cite an existing behavioral contract or reproduction even when the spec did not restate it.

## Triage and persist

Validate findings against source and evidence in one pass. Reject unsupported claims, unrelated pre-existing issues and requests outside scope. Accepted risks are not defects under their stated conditions; new evidence that invalidates those conditions is a decision to surface. Code additions are justified when needed for approved behavior or an introduced regression. Check callers and trust boundaries before accepting a claim that a guard is unnecessary.

Prioritize by impact and confidence; categories organize findings but do not make cosmetic changes equal to data loss. Record rejected findings with reasons so they do not recur. A required issue cannot be marked `wontfix` merely to get a clean verdict; link the existing accepted decision or obtain one for a material change.

A delegated reviewer returns findings. The coordinator saves the output and writes or appends `review-code.md` using `./review-template.md` **before** fixes start. Carry unresolved findings into the latest pass, retaining their IDs and decision history.

The coordinator sets `reviews.code` to the pass, scope (`full` or `partial`), verdict, implementation and reviewer families, report path, current repo revisions and approved spec digest; `steps.review-code` is `done` only for a clean full pass meeting the reviewer requirement. A partial pass leaves it `in-progress` and cannot satisfy readiness or shipping gates. Fixed findings alone do not make an old review current: re-review the changed result.

## Continue the review conversation

After triage and any repairs, reply in the same recorded reviewer session with finding IDs, accepted/rejected dispositions and reasons, fix commits, reproduction/test results, current heads and the report path. Ask the reviewer to check the fixes and consequences, and reconsider disputed findings against that evidence. Validate new or persistent findings yourself; agreement alone is not proof. Persist the resulting pass before another repair round.

Close the exchange when the reviewer has assessed the current result and no supported required finding remains. Explain evidence-based disagreements in the report; neither rubber-stamp findings nor require agreement on preferences. If every finding was rejected without code changes, still send the reasons back once. An initial clean review needs no ceremonial extra round. QA repairs return to this same loop. The shared repair budget applies across sessions; if a dispute repeats without new evidence, record the unresolved decision and stop dependent work.

Reuse the session for the same target and focused fixes. Start fresh if it is unavailable, context is no longer useful, or the target or reviewer family changes; pass the requirements, latest report, decisions and current revisions. Follow the recovery rules in `./reviewer-transport.md` before replacing an uncertain running call. A new session never resets findings or budgets. In the full run a clean review leads directly to local QA.
