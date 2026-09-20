# Spec: <title>

**Date:** YYYY-MM-DD
**Ticket:** #42 / PROJ-123 (if applicable)
**Repos:** repo1, repo2

## Problem Statement

The problem from the user's perspective. What pain exists today and why it matters.

## Solution

The solution from the user's perspective. What changes and how it addresses the problem.

## User Stories

A numbered list grouped by actor or capability area. Each story: As an <actor>, I want <feature>, so that <benefit>. Include enough stories to cover required behavior, edge cases, error states, and accepted omissions; do not optimize for length.

### <Capability Area 1>

1. As a ..., I want ..., so that ...
2. As a ..., I want ..., so that ...

### <Capability Area 2>

3. As a ..., I want ..., so that ...

## Implementation Decisions

Binding decisions synthesized from the idea's Settled Decisions, research, prototype findings, and the residual interview. Include only what constrains the outcome or a consequential contract:

- Module responsibilities and interfaces when they are agreed contracts
- Architectural decisions
- Schema changes
- API contracts (new or modified)
- Cross-repo integration points
- Key technical constraints or trade-offs

Leave internal decomposition and algorithms to implementation; label a suggested approach as provisional. No speculative file paths or implementation code. Include durable references that encode a decision more precisely than prose: schema shapes, type definitions, API contracts, prototype snippets or mockups (trimmed to the decision-rich parts).

## Accepted Risks & Tradeoffs

What we consciously do NOT handle and why. One entry per declined scenario or deliberate gap ("no retry - runs hourly, a missed run self-heals"). Lifted from idea.md's Accepted Risks (draft) plus anything the residual interview added. Record the assumptions and conditions under which each entry applies. Leave it unhandled while those conditions hold; surface evidence that invalidates them.

## Testing Decisions

- **Acceptance-test interfaces** confirmed with the user: the interfaces used to verify required acceptance criteria. Prefer existing interfaces and the highest level that keeps failures legible.
- Supporting unit, integration, contract, and regression tests follow repository practice and may use other established interfaces.
- What makes a good test here (test external behavior, not implementation details)
- Prior art - similar tests in the codebase to follow as patterns

## Acceptance and Human Review

Observable success criteria, including relevant failure cases and nonfunctional constraints. For UI or experience work, link agreed references and identify the qualities the human will judge at the final demonstration. Separate mechanically verifiable behavior from subjective acceptance.

## Out of Scope

What this spec explicitly does NOT cover.

## Open Questions & Risks

Unresolved items and uncertainties, including any provisional terms.

## ADR Candidates

Decisions that are hard to reverse, surprising without context, and the result of a real trade-off. One line each. (Omit this section if empty.)
