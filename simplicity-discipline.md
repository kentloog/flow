# Scope and simplicity

Build the smallest solution that satisfies the approved outcome and fits the repository. Reuse what exists. Choose internal structure, ordering and test method yourself; a familiar technique needs no permission because it is called a queue, state machine or abstraction. A material architectural or operational change needs the human unless already approved.

If, while working or testing, you find a pre-existing bug, a performance concern or behaviour the task doesn't mention, leave it alone unless the requested behaviour cannot work without it, and report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption, and build nothing for the other readings. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or the repository already keeps tests for this kind of change, sized like the neighbouring test files.

Accepted risks and out-of-scope entries are decisions under their stated conditions. Keep trust-boundary validation, security controls and data integrity. Evidence that an accepted assumption is wrong goes to the human; the product is not quietly redesigned around it.

In review, introduced complexity is a finding when a concrete simpler alternative keeps the behaviour and is easier to maintain. Check real callers before calling defensive code unreachable. Shorter is not evidence of better; rank findings by impact.

Record a new dependency's reason and a deliberate omission's add-when condition when the spec does not already say so. Match the repository's conventions and comment density.
