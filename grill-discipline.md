# Grill Discipline

An interview to sharpen a plan, decision, or idea - put every decision that changes what gets built to the user until a shared understanding is reached. Applied by `idea` (the main grilling session), `spec` (residual open decisions only), and wayfinder grilling tickets; also applies whenever the user wants to stress-test their thinking or uses a "grill" trigger phrase ("grill me"). Sourced from Matt Pocock's grilling skill.

## The principle

Most miscommunication between human, AI, and codebase comes from missing shared language and unstated decisions. Surface both while the work is still cheap to change. When the user uses a domain term, do not assume - verify the meaning against the repo's existing language (read a `CONTEXT.md` or glossary at the repo root if one exists - read-only; this workflow never creates or edits such files).

## Interview mechanics

- Map consequential decisions and their dependencies. Ask the ready frontier: a short numbered round of independent questions, each with a recommendation. Ask dependent questions in the next round after their prerequisites are answered. Use one question when dependency or complexity calls for it.
- Wait for the human's answers; never simulate them. Research factual unknowns yourself while continuing questions that do not depend on those facts.
- For each question, provide your recommended answer - the user can accept with a word or override.
- If a fact can be found by exploring the environment (filesystem, code, tools), look it up rather than asking. The decisions, though, are the user's - put each one to them and wait for the answer. Never answer your own questions.
- Summarize decisions periodically to confirm alignment.
- Record each resolved decision in the run's decision record the moment it lands, never batched to the end. During ideation that is idea.md's **Settled Decisions**; during the spec's residual interview it is the spec itself.

## Decision boundary

Ask the user about choices that change the product outcome, a public contract, data semantics, security posture, operational behavior, architectural shape, or a costly-to-reverse trade-off. Also ask when the user wants control of a choice.

Leave reversible implementation mechanics to the implementation owner: file placement, helper shape, refactoring order, test-double style, and command sequencing. Escalate when a local-looking choice exposes a material trade-off or changes an approved contract.

## Discuss concrete scenarios (proactive)

The triggers below are reactive; this move is proactive. When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts. Scenarios the user declines to handle flow into the run's accepted-risks record - idea.md's **Accepted Risks (draft)** during ideation, the spec's Accepted Risks & Tradeoffs once one exists. A documented decision, not silence.

## When to challenge (5 triggers)

Do NOT challenge every domain term. Apply discipline only when one of these fires:

1. **Conflict with repo vocabulary** - the user's usage contradicts the repo's existing language (a CONTEXT.md/glossary read as reference, or consistent naming in code). Surface the conflict and resolve it in conversation; the resolution lands in the decision record, not in the glossary file.
2. **Ambiguous domain term** - the term could plausibly mean two or more distinct things in the domain (e.g., "account" = customer org or login user; "order" = the cart submission or the fulfillment record). Propose a canonical name and ask the user to pick.
3. **Fuzzy quantifier** - "many", "sometimes", "usually", "soon", "fast", "most" without a threshold. Ask for a concrete number or example.
4. **Cross-reference with code and grep contradiction** - the user asserts a behavior and a quick read of the relevant code contradicts it. Surface the contradiction.
5. **Fuzzy robustness language** - "should be resilient", "handle errors gracefully", "make it robust". Force it into one of two shapes: a concrete requirement (which failure, detected how, handled how) or an explicit accepted-risk entry. Fuzzy robustness left in a spec becomes invented defensive code at implement time.

If no trigger fires, let the interview flow.

## Provisional terms

If the user refuses to define a fuzzy term ("just call it X for now"), note it as provisional in idea.md's Open Questions (or the spec's Open Questions & Risks). Do not block the interview.

## Ending the session

The grilling ends when a shared understanding is reached - nothing left that would change what gets built. Do not act on the plan until the user confirms that shared understanding has been reached; the confirmation ends the grilling, nothing else does. Close by sorting what remains in Open Questions:

- **Factual unknowns** (answerable from docs, code, or external sources): recommend `/flow research <slug>`.
- **Design-feel unknowns** ("does this state model hold up?", "what should it look like?"): recommend `/flow prototype <slug>`.
- Neither: the idea is spec-ready.

## ADR Candidates (spec step only)

After the spec is written, append an "ADR Candidates" section listing decisions from the run that satisfy ALL THREE (omit any that miss one):

1. **Hard to reverse** - the cost of changing your mind later is meaningful.
2. **Surprising without context** - a future reader will wonder "why did they do it this way?"
3. **Result of a real trade-off** - genuine alternatives existed and one was picked for specific reasons.

What tends to qualify: architectural shape ("the write model is event-sourced"); integration patterns between contexts ("these services communicate via domain events, not synchronous HTTP"); technology choices that carry lock-in (database, message bus, auth provider - not every library); deliberate deviations from the obvious path ("manual SQL instead of the ORM because X" - these stop the next engineer from "fixing" something deliberate); constraints not visible in the code ("response times under 200ms because of the partner API contract").

One-line summary per candidate. Do NOT write ADRs yourself - surface the list; the user decides whether to record any (in the project's ADR convention, e.g. `docs/adr/`, if it keeps one).
