# Review Steps: review

Cross-model review with inverted roles: a **second model reviews** the implementation diff, **Claude adversarially validates** every finding before it can demand action, fixes loop back to the reviewer until clean. Simplicity has equal rank to correctness throughout.

## Reviewer resolution

Resolve the reviewer once per run, in this order (config `review.reviewer` can force a choice):

1. **Codex MCP** (`mcp__codex__codex` + `mcp__codex__codex-reply` available in the session) - the preferred mode: a persistent thread carries the whole review loop.
2. **A second-model CLI** named in config `review.reviewer` (e.g. `codex exec`, `gemini`) - write the brief to a file, invoke the command with it via Bash. No thread exists, so each re-review invocation restates the context: the original brief reference, what changed (fix commits), and the killed findings.
3. **A Claude subagent** carrying the same brief - the fallback when no second model is available. Its findings still go through the validation layer; the adversarial check matters MORE when reviewer and validator share a model family. Re-reviews are fresh subagents given the fix commits and killed findings.

If a preferred mode fails mid-run (call errors, MCP down), fall back down the list and note it in the review doc.

## Fail-fast preflight

Fail here, not inside the review:

- `state.yaml` and `spec.md` exist in the slug folder.
- Resolve working trees from state.yaml (`git worktree list` to confirm when worktrees are in use), determine each repo's default branch, then per repo `git rev-parse origin/<default>` and check `git diff origin/<default>...HEAD`. A repo with an empty diff is noted and excluded from the review (`repos` means involved, not necessarily changed); fail only if every repo's diff is empty. Capture the diff command and `git log --oneline origin/<default>..HEAD` once per included repo for the brief.

## The review (the reviewer's half)

**One reviewer thread per review, even cross-repo** - one reviewer seeing the whole change is the point, and cross-repo implementations are directly related (shared contracts, dependencies). Long calls are fine. Split per repo ONLY when the implementation is huge across multiple repositories - and the split is an attention tactic, never a fragmentation of the review: every brief carries the cross-repo contract points (API signatures, event/DTO shapes, published lib versions) and a one-paragraph summary of the sibling repos' changes, each thread is explicitly asked to flag mismatches against that contract context, and all threads' findings consolidate into the single review pass below. Never split within a repo.

**Build the brief** (under ~1500 words, pasted discipline text excluded). Data goes by reference, instructions by value: the reviewer reads data sources fully itself (no lossy middleman - a summary written by the implementing model family would filter the spec through the same lens that produced the code, hiding exactly the omissions spec-fidelity review exists to catch), while the review criteria are pasted in full so their uptake is guaranteed, not optional reading.

By reference (paths/commands, never inlined):

- The spec: `.flow/<slug>/spec.md` - read it in full; the Accepted Risks & Tradeoffs section is binding (an accepted risk is never a finding).
- The working tree path(s), diff command (`git diff origin/<default>...HEAD`), and commit list.
- The repo's `CLAUDE.md` (or `AGENTS.md`) path as the conventions source.

By value (pasted in full): `simplicity-discipline.md` (both parts - Part 2 is the tag vocabulary) and this smell baseline (each reads: what it is, then how to fix):

- **Mysterious Name** - a function, variable, or type whose name doesn't reveal what it does or holds. Rename it; if no honest name comes, the design's murky.
- **Speculative Generality** (`yagni:`) - abstraction, parameters, or hooks added for needs the spec doesn't have. Delete it; inline back until a real need shows.
- **Middle Man** (`delete:`) - a class or function that mostly just delegates onward. Cut it, call the real target direct.
- **Refused Bequest** - a subclass or implementer that ignores or overrides most of what it inherits. Drop the inheritance, use composition.
- **Divergent Change** - one file or module edited for several unrelated reasons. Split so each module changes for one reason.

Three hedges bind the conventions/smell portion: a documented repo standard always wins; every smell is a labelled heuristic, never a hard violation; skip anything tooling already enforces.

**The ask** - report findings across these categories, every finding with a `file:line` anchor:

1. **Spec fidelity** (the three-way brief): (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. This hunts silent omissions - the false negatives validation cannot catch on its own.
2. **Simplicity** - introduced-only, worked from the spec's invariants per the pasted discipline; tag format per Part 2; the category summary ends `net: -N lines possible` or `lean already`.
3. **Conventions** - does this look like the rest of this repo? Repo CLAUDE.md + the smell baseline, under the three hedges.
4. **Obvious bugs.**
5. **Test sufficiency** - against the spec's Testing Decisions seams.
6. **Cross-repo contract consistency** (only when split).

Findings must be structured: number, anchor, category, tag (where the discipline defines one), what, verbatim `specLine` where the category requires a spec quote, and an `addsCode` flag (true if acting on the finding would ADD code).

**Optional dead-code pass (never a hard dependency):** if a dead-code tool is available in the repo (`knip` or similar for Node/TS), run it before the reviewer call and feed introduced-only hits (unused exports, speculative helpers) into the brief as candidate `delete:` findings for the reviewer to confirm. Deterministic checks run first; model attention goes only to what tools can't judge.

## Claude validation (adversarial - false positives die first)

One Claude verifier subagent per finding, launched in parallel in a single message via the Agent tool. Each verifier prompt is self-contained (the finding, the diff command, the spec path) and carries simplicity discipline Part 1 (the carve-outs are needed to judge `unreachable:` claims correctly). The verifier tries to KILL the finding, confirming mechanically against the actual sources:

1. **Anchor post-validation, both sides.** The `anchor` exists: the file:line/symbol is really in the diff. Where `specLine` is quoted, that line really exists in `spec.md`. A hallucinated anchor on either side kills the finding.
2. **Introduced-only.** The flagged code is part of THIS change, not pre-existing.
3. **Accepted-risk check.** A finding that flags something listed in the spec's Accepted Risks & Tradeoffs dies.
4. **Scope-inflation filter.** A finding with `addsCode: true` must cite a spec requirement or a violated accepted risk; otherwise it dies. Without this the findings-address-re-review loop is a complexity ratchet - reviewers reward overbuilding unless explicitly instructed not to.
5. **Unreachable claims.** For an `unreachable:` finding, confirm no caller or input in the codebase produces the defended state; if one does, the finding dies.

Verdict per finding: valid or killed, with the reason. Killed findings are recorded for transparency and reported back to the reviewer in the re-review so they aren't re-raised.

## Address and re-review loop

1. **Address:** launch fix subagents in the working tree for the validated findings (simplicity discipline Part 1 injected; each fix subagent re-runs the affected scoped tests and commits its own fix; the orchestrator runs implement's silent verify - clean tree, new commit, typecheck+lint exit code).
2. **Re-review:** send back to the reviewer - `mcp__codex__codex-reply` on the same thread in Codex mode; a re-invocation with restated context in CLI mode; a fresh subagent in Claude mode. Content: what changed (the fix commits), plus which findings were killed in validation and why. New findings from the re-review go through the same Claude validation. When split, threads loop independently - but a fix that touches a cross-repo contract is announced in EVERY affected repo's re-review message, not just the thread that raised it.
3. Loop until the reviewer reports clean, capped at 3 loops - then surface the remaining findings to the user and stop.

## Output and state

- Write or append `review-code.md` in the slug folder per `review-template.md` (this skill's directory) - a new `## Review Pass N` per run, findings grouped by category, killed findings listed with their kill reasons for transparency. Kept findings go in the template's numbered table with a Status column (`open` | `fixed` | `wontfix`), numbered sequentially across categories within the pass - `/flow implement` re-enters on the latest pass's `open` findings and flips them to `fixed` as fixes commit. Findings already fixed during this review's loop are recorded as `fixed`. Do NOT merge or rerank findings across categories; end with per-category counts and the worst issue within each category, never a single winner across categories. Record the reviewer mode, the loop count and, when split, the per-repo threads.
- Update state.yaml (`reviews.code`, steps).
- Suggest next: clean - `/flow qa <slug>` or `/flow push <slug>`; findings open - list them and the fix route (`/flow implement <slug>` re-entry).
