# Simplicity Discipline

Premise: shipping overengineered code is a bigger risk than shipping broken code. Broken code fails loudly and gets fixed; overengineering sails through review and quietly taxes every future change. The main failure mode is not long functions (linters catch those) but semantic over-defensiveness: guarding against states that cannot occur. Only the spec's invariants reveal which states those are.

## Part 1: Constraint block (implementers, fix agents, reviewers)

### Decision ladder

Before writing any code, walk down this ladder and stop at the first rung that holds:

1. Does this need to exist at all? The spec's Accepted Risks and Out of Scope sections list things deliberately not handled. Never "fix" an accepted risk.
2. Does the codebase already have it?
3. Does the stdlib or platform do it?
4. Does an already-installed dependency do it?
5. Can it be one line?
6. Only then: write the minimum code that satisfies the spec.

### Abstraction and configuration rules

- No new abstraction without a second concrete consumer that exists today. One caller means inline code, not an interface.
- No config option for a value that has exactly one value. Hardcode it; the spec's add-when trigger says when to lift it.
- If the urge is a queue, state machine, plugin system, or event bus: stop and report back instead of building it. That decision belongs to the human.

### Carve-outs: never simplify these away

- Input validation at trust boundaries. Trust-boundary input is never "unreachable".
- Error handling that prevents data loss or corruption.
- Security controls.
- Anything the spec explicitly requires.

A guard is over-defensive only if the defended state is unreachable given actual callers. Check the callers before deleting.

### Skipped / add-when contract

End every phase report with a `Skipped / add-when` list: each thing deliberately not handled, plus the concrete trigger that would justify adding it. Example: "No retry on the export job - runs hourly, a missed run self-heals. Add when a consumer needs sub-hour freshness." Reviewers check this list against the spec's Accepted Risks instead of discovering silent omissions.

### Dependency gate

Adding a new package requires a line in the phase report: why not native, stdlib, or an existing dependency. No micro-utility packages. New dependencies always surface to the user as decisions to confirm.

### Comment discipline

Match the file's existing comment density and idiom. No marker prefixes (`WORKFLOW:`, `NOTE(agent):`, ticket tags). Decisions live in the spec's Accepted Risks and the phase report's Skipped / add-when list, not in code markers.

## Part 2: Review tag format (reviewers)

Findings use one line each:

```
file:line: tag - what. replacement.
```

Tags:

| Tag           | Meaning | Example (each smell reads: what it is, then how to fix) |
| ------------- | ------- | -------------------------------------------------------- |
| `delete:`     | Code that serves no requirement | Middle Man: a function that mostly delegates onward. Cut it, call the real target direct. |
| `stdlib:`     | Hand-rolled logic the stdlib provides | Custom deep-clone. Replace with `structuredClone`. |
| `native:`     | Hand-rolled logic the platform or an installed dependency provides | Manual query-string builder. Replace with `URLSearchParams`. |
| `yagni:`      | Built for a need the spec does not have | Speculative Generality: abstraction, parameters, or hooks added for needs the spec doesn't have. Delete it; inline back until a real need shows. |
| `shrink:`     | Same behavior achievable in less code | Three near-identical branches. Collapse to one parameterized path. |
| `unreachable:`| Guard, lock, retry, or catch for a state that cannot occur | Null-check on a value the schema makes non-null. Name the caller or input that produces the state; none exists, so delete or downgrade to a documented accepted risk. |

Rules:

- **Introduced-only.** Flag only what THIS change introduced. Pre-existing complexity is context, never a finding.
- **Work from the spec's invariants**, not code shape. For every defensive construct ask: can this state actually occur given what the spec guarantees? For every `unreachable:` finding, name the concrete caller or input that would produce the defended state; if none exists in the codebase, the guard goes.
- **Respect the carve-outs** in Part 1 and the spec's Accepted Risks: an accepted risk is never a finding, and neither is its absence of handling.
- **Obvious or narrative comments are `delete:` findings** (introduced-only).
- End the report with `net: -N lines possible` or `lean already`.
