# Scope and simplicity

Build the smallest solution that satisfies the approved outcome and fits the repository. Reuse existing capabilities where they serve the need. Choose internal structure and test ordering without a prescribed recipe.

Accepted risks and out-of-scope entries are decisions under their stated conditions. Preserve trust-boundary validation, security controls, data integrity and required behavior. Surface evidence that invalidates an accepted assumption; do not quietly redesign the product around it.

A material architectural or operational change needs the human's judgment unless already approved. A familiar implementation technique does not need permission merely because it is called a queue, state machine or abstraction.

Review introduced complexity when a concrete simpler alternative preserves behavior and improves maintainability. Check real callers and external inputs before declaring defensive code unreachable. Shorter code alone is not evidence of a better solution; prioritize findings by their impact.

Record new dependencies with their rationale and deliberate omissions with add-when conditions when they are not already clear in the spec. Match repository conventions and comment density. Reports carry decisions and evidence; avoid repeating a boilerplate checklist in every handoff.
