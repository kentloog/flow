# Spec: <title>

**Date:** YYYY-MM-DD
**Ticket:** #42 / PROJ-123 (if applicable)
**Repos:** repo1, repo2

## Problem Statement

The problem from the user's perspective. What pain exists today and why it matters.

## Solution

The solution from the user's perspective. What changes and how it addresses the problem.

## User Stories

A LONG numbered list, grouped by actor or capability area. Each story: As an <actor>, I want <feature>, so that <benefit>. Extremely extensive - cover all aspects of the feature, including edge cases and error states.

### <Capability Area 1>

1. As a ..., I want ..., so that ...
2. As a ..., I want ..., so that ...

### <Capability Area 2>

3. As a ..., I want ..., so that ...

## Implementation Decisions

Decisions synthesized from the idea's Settled Decisions, research, prototype findings, and the residual interview:

- The modules to build or modify, and their interfaces (prefer deep modules: simple interface, hidden complexity)
- Architectural decisions
- Schema changes
- API contracts (new or modified)
- Cross-repo integration points
- Key technical constraints or trade-offs

No speculative file paths or implementation code - they go stale. DO include durable references that encode a decision more precisely than prose: schema shapes, type definitions, API contracts, prototype snippets or mockups (trimmed to the decision-rich parts).

## Accepted Risks & Tradeoffs

What we consciously do NOT handle and why. One entry per declined scenario or deliberate gap ("no retry - runs hourly, a missed run self-heals"). Lifted from idea.md's Accepted Risks (draft) plus anything the residual interview added. Implementers never "fix" these; reviewers never flag them.

## Testing Decisions

- **Test seams** confirmed with the user: the interfaces the feature is tested through. Prefer existing seams; the highest seam possible; the ideal number is one.
- What makes a good test here (test external behavior, not implementation details)
- Prior art - similar tests in the codebase to follow as patterns

## Out of Scope

What this spec explicitly does NOT cover.

## Open Questions & Risks

Unresolved items and uncertainties, including any provisional terms.

## ADR Candidates

Decisions that are hard to reverse, surprising without context, and the result of a real trade-off. One line each. (Omit this section if empty.)
