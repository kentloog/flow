# Design notes

Why flow is shaped the way it is, what it borrows, and what has not been tested yet.

## Principles

**Describe the outcome, not the path.** Give the agent the problem, the constraints and a clear definition of done, then let it choose how. Our working hypothesis is that this produces better solutions than prescribing the implementation. Programming expertise still helps define contracts, assess evidence and judge design; an early implementation sketch should remain provisional unless it encodes a real constraint.

**Instruction volume is not quality.** There was a period of micro-optimizing AGENTS.md and CLAUDE.md into long rulebooks. Current vendor guidance supports removing obsolete recipes and repeated instructions. Whether a specific instruction helps depends on the model and task. Every rule in this skill has to change what the agent does, or it comes out.

**The middle of the lifecycle belongs to agents.** Implementation, review, repair and local verification run end to end without a human in the loop. Resolve reversible choices from the spec and evidence. Stop affected work for an unapproved material decision, unresolved access or the repair limit; continue independent work.

**The beginning and the end belong to the human.** Judgement and taste decide what is worth building, what the contracts are, what risks are accepted, and whether the finished thing is any good. Grilling the idea, agreeing the spec and approving the plan are the point, not overhead. At the end the human tries the feature and judges it.

**Guardrails, not a harness.** Reasonable rules, linting, a sensible testing strategy and verifiable slices work. Drowning the agent in rules is the same failure as micromanaging it. When a run exposes a repeatable mistake, prefer an automated check over another written rule.

**Bounded autonomy.** The goal is not maximum autonomy. It is the least uncertainty for the autonomy being granted, which is why the outcome, the acceptance interfaces and the stop conditions are settled before the AFK band starts.

**Two hosts, one contract.** The same instructions run in Claude Code and in Codex, desktop app or CLI. Flow uses the capabilities a host exposes and assumes no particular tool, supervisor or orchestration API.

## Borrowed from Matt Pocock

Flow's shape comes largely from [mattpocock/skills](https://github.com/mattpocock/skills).

Compared with [source revision c55ee46](https://github.com/mattpocock/skills/tree/c55ee46073ed923f86ce59a5eb3b6d895095d1b7) and its [CHANGELOG](https://github.com/mattpocock/skills/blob/c55ee46073ed923f86ce59a5eb3b6d895095d1b7/CHANGELOG.md) on 2026-09-20. Adoption means preserving useful behaviour within flow's contract, not copying every upstream instruction.

| Source | Applied in flow | Deliberately not adopted |
|--------|-----------------|--------------------------|
| wayfinder | Map as an index, decision tickets, ready frontier and claims, fog kept unspecified rather than invented | A specific external tracker, or execution inside the discovery map |
| grilling | Design tree worked in rounds, frontier questions with recommended answers, facts researched rather than asked | Asking humans to choose reversible implementation mechanics |
| to-spec | Synthesis without a second interview, agreed acceptance interfaces, durable contracts over file paths | An extensive-story quota, or automatic publication to a tracker |
| to-tickets | Tracer-bullet vertical slices with blocking edges, the granularity quiz, expand-contract for wide refactors | Publishing tickets by default |
| implement | A short brief: verify at the agreed interfaces, run checks as you go, review, commit | Mandated test-first ordering |
| implement-spec | Dependency-driven work, sparse communication through pointers, grouped review repairs | A worker fleet with a separate merger agent |
| code-review | Separate spec and standards verdicts, concise evidence-backed findings, design smells as judgement calls | Mandatory parallel reviewers, a fixed smell checklist, same-family review by default |
| codebase-design | Deep modules, information hiding, locality of change, testing through interfaces | Enforced vocabulary, universal adapter counts, automatic parallel interface designs |
| retro | Check over rule for mechanical mistakes, standards belong to the reviewer, CLAUDE.md holds navigation pointers | Automatic edits to shared files |
| pr | Summary sketch, before-and-after evidence, merge danger as door and blast radius | |
| writing-for-agents | One meaning in one place, positive phrasing, no-op pruning, progressive disclosure, leading words | Treating a named principle as a mandatory recipe |
| prototype | A concrete question, runnable exploration, decisions carried into the spec | Permanent prototype retention; flow still removes temporary branches at complete |

His portfolio moves. Skills get renamed, graduated out of `in-progress`, reworked or removed, and new ones appear that would fit here. Check it periodically, read the CHANGELOG for what changed and why, and propose adoptions. A human approves each one before it lands in flow.

## Engineering fundamentals

The [Pragmatic Programmer tips](https://books.pragprog.com/tips/) and [Ousterhout's design work](https://web.stanford.edu/~ouster/cgi-bin/book.php) inform the judgement in `simplicity-discipline.md`. That file is reached during design, implementation and review. The short definitions are leading words with enough context to prevent common misreadings; they prescribe no file layout, pattern or test order.

| Principle | Where flow puts it to work |
|-----------|----------------------------|
| Software entropy; strategic programming | Improve design in the area the feature changes; report unrelated debt rather than expanding scope |
| Don't outrun your headlights | Small verifiable slices, experimental feedback, autonomous adjustment of pending work |
| Programming by coincidence | Evidence for assumptions, diagnoses and acceptance; a green command alone is not proof of the feature |
| Tracer bullets | `plan-steps.md` requires verifiable vertical slices with real dependency edges |
| Orthogonality and decoupling | Keep unrelated concerns independent and changes local |
| DRY | Give knowledge one authoritative home; avoid abstractions based only on similar syntax |
| Complexity reduction; deep modules | Judge the burden on callers and maintainers, not line counts; hide decisions behind useful interfaces |

[DRY concerns knowledge](https://media.pragprog.com/titles/tpp20/dry.pdf), which is why plans point to the spec's test contracts instead of copying them. [Feedback limits safe step size](https://media.pragprog.com/titles/tpp20/dont-outrun-your-headlights.pdf), which is why a plan can change during execution without reopening settled product decisions.

## Current model guidance

[Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) supports complete task descriptions and says explicit self-check and verifier scaffolding causes over-verification. [Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) supports long autonomous tasks and supplies the autonomy and scope wording used in `execution.md`, while still expecting explicit continuation and durable progress. [GPT-5.6](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6) favours lean instructions stated once with clear approval boundaries. [GPT-6 Astra](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) supports short entrypoints, selective loading and removing obsolete recipes.

None of this means removing verification. It means removing instructions that duplicate what the model already does. Shorter instructions are not evidence of a better skill.

The distinction matters for the September 2026 review. [Max's first post](https://x.com/maxedapps/status/2098397503724880058) describes collaborative planning and task-specific verification; [his second](https://x.com/maxedapps/status/2098445366479786151) favours bounded tasks over both micromanagement and unrestricted fleets. [Robert Martin's post](https://x.com/unclebobmartin/status/2098432570887217520) questions elaborate coordination while retaining testing. These are practitioner reports, not controlled comparisons of flow. The posts were read through public syndication because X's page retrieval failed.

The [Boris Cherny interview](https://www.youtube.com/watch?v=qyPCVqFUyDo) discusses an 80% prompt reduction. That percentage is not a target for this skill. Flow keeps independent review and demonstrated acceptance because they are part of the requested delivery outcome; it avoids extra verifier agents and repeated checks without changed inputs or a specific concern.

This review retained the existing lifecycle and added focused corrections: explicit design principles, provisional internal design, selective delegation, separate review verdicts, continuing independent work when blocked, and evidence tied to source and acceptance documents. The entrypoint stays a router. No implementation pattern, test-first sequence, model override or new stage was added.

## What still needs testing

No comparative measurement has been run across Opus 5, Fable 5.1, GPT-5.6 Sol and GPT-6 Astra.

The 2026-09-20 review passed skill validation, bundled-reference checks, YAML example parsing and a diff whitespace check. An independent Codex instruction walkthrough covered dependencies, access blockers, dirty source, changed criteria, QA repairs, internal design decisions and repair-limit resume. Its identified ambiguities were corrected and rechecked. This validates the written contract, not live execution or comparative model performance.

| Scenario | Observable success |
|----------|--------------------|
| Medium single-repo feature | One implement invocation reaches a clean review and local QA PASS with no transition questions |
| Cross-repo dependency | The consumer waits for the provider, integration is exercised, nothing is published without authorization |
| Review repairs | A real omission is fixed; an unrelated pre-existing issue and an accepted risk are rejected |
| UI without a usable browser | Required interactions stay unverified; markup never produces a false PASS |
| Compaction with a live worker | Ownership is reconciled, no duplicate writer, the next stage runs |
| Process loss with a dirty tree | Changes are preserved, missing evidence is obtained, nothing is reset blindly |
| Code change after a QA pass | The earlier PASS cannot certify the changed revisions |
| Dirty code or a changed spec after a pass | Matching HEAD is insufficient; affected review and acceptance evidence is refreshed |
| Reversible internal design change | The agent improves the design without another human checkpoint |
| One blocked phase with independent work ready | The worker reports its blocker; the coordinator continues authorized work |
| Repeated failure | The stop rule holds across resume, with a concrete blocker and any independent progress reported |
| Human acceptance | A demonstrable result and taste questions arrive before anything is published |

Run these in both hosts. Change one group of instructions at a time, and restore a rule only when its removal causes a demonstrated regression.
