# Scope and simplicity

Reduce the complexity a maintainer must understand while satisfying the approved outcome. Reuse what exists. Choose internal structure, ordering and test method yourself. A material change to an approved architectural or operational contract needs the human; an internal design choice does not.

Use these principles as design judgement, not a required sequence or a scorecard:

- Deep modules and information hiding: give callers a small, clear interface that hides substantial behaviour and decisions. Fewer lines or smaller functions alone do not make a better design.
- Orthogonality and decoupling: keep unrelated concerns independent; changes and knowledge should stay local.
- DRY: give each business rule or piece of knowledge one authoritative home. Similar-looking code does not justify coupling different concepts.
- Strategic programming and software entropy: improve the design where this feature needs change; avoid expedient patches that spread complexity. Keep unrelated cleanup outside this delivery.
- Tracer bullets and don't outrun your headlights: get working feedback early, and let it reshape pending work within the approved outcome.
- Avoid programming by coincidence: understand why the change works. Use observed behaviour and failure evidence to test assumptions and guide repairs.

If, while working or testing, you find a pre-existing bug, a performance concern or behaviour the task doesn't mention, leave it alone unless the requested behaviour cannot work without it, and report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption, and build nothing for the other readings. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or the repository already keeps tests for this kind of change, sized like the neighbouring test files.

Accepted risks and out-of-scope entries are decisions under their stated conditions. Keep trust-boundary validation, security controls and data integrity. Evidence that an accepted assumption is wrong goes to the human; the product is not quietly redesigned around it.

In review, introduced complexity is a finding when a concrete simpler alternative keeps the behaviour and is easier to maintain. Check real callers before calling defensive code unreachable. Treat design smells as hypotheses supported by code and impact, not automatic violations.

Record a new dependency's reason and a deliberate omission's add-when condition when the spec does not already say so. Match the repository's conventions and comment density.
