# Design notes

Why flow is shaped the way it is, what it borrows, and what has not been tested yet.

## Principles

**Describe the outcome, not the path.** Give the agent the problem, the constraints and a clear definition of done, then let it choose how. Current models reach better solutions this way than when a human prescribes the implementation. Knowing a lot about programming worked against this for a while: it led to instructing agents to build things a particular way, and they complied precisely, which capped the result at what the human had already imagined.

**Instruction volume is not quality.** There was a period of micro-optimizing AGENTS.md and CLAUDE.md into long rulebooks. Vendors have since cut their own system prompts substantially, because models need less instruction and are actively harmed by over-prescription. Every rule in this skill has to change what the agent does, or it comes out.

**The middle of the lifecycle belongs to agents.** Implementation, review, repair and local verification run end to end without a human in the loop. Inside that band, prefer a deterministic default over a question. Ask only when an action is destructive, a product decision is open, or a capability is missing.

**The beginning and the end belong to the human.** Judgement and taste decide what is worth building, what the contracts are, what risks are accepted, and whether the finished thing is any good. Grilling the idea, agreeing the spec and approving the plan are the point, not overhead. At the end the human tries the feature and judges it.

**Guardrails, not a harness.** Reasonable rules, linting, a sensible testing strategy and verifiable slices work. Drowning the agent in rules is the same failure as micromanaging it. When a run exposes a repeatable mistake, prefer an automated check over another written rule.

**Bounded autonomy.** The goal is not maximum autonomy. It is the least uncertainty for the autonomy being granted, which is why the outcome, the acceptance interfaces and the stop conditions are settled before the AFK band starts.

**Two hosts, one contract.** The same instructions run in Claude Code and in Codex, desktop app or CLI. Flow uses the capabilities a host exposes and assumes no particular tool, supervisor or orchestration API.

## Borrowed from Matt Pocock

Flow's shape comes largely from [mattpocock/skills](https://github.com/mattpocock/skills).

| Source | Applied in flow | Deliberately not adopted |
|--------|-----------------|--------------------------|
| wayfinder | Map as an index, decision tickets, ready frontier and claims, fog kept unspecified rather than invented | A specific external tracker, or execution inside the discovery map |
| grilling | Design tree worked in rounds, frontier questions with recommended answers, facts researched rather than asked | One question at a time in every situation |
| to-spec | Synthesis without a second interview, agreed acceptance interfaces, durable contracts over file paths | An extensive-story quota, or automatic publication to a tracker |
| to-tickets | Tracer-bullet vertical slices with blocking edges, the granularity quiz, expand-contract for wide refactors | Publishing tickets by default |
| implement | A short brief: verify at the agreed interfaces, run checks as you go, review, commit | Mandated test-first ordering |
| implement-spec | Dependency-driven work, sparse communication through pointers, grouped review repairs | A worker fleet with a separate merger agent |
| code-review | Two axes reviewed separately, spec fidelity and standards, capped reports, no reranking across axes | Same-model review; flow requires the other model family |
| retro | Check over rule for mechanical mistakes, standards belong to the reviewer, CLAUDE.md holds navigation pointers | Automatic edits to shared files |
| pr | Summary sketch, before-and-after evidence, merge danger as door and blast radius | |
| writing-for-agents | One meaning in one place, positive phrasing over prohibition, no-op pruning, progressive disclosure | |

His portfolio moves. Skills get renamed, graduated out of `in-progress`, reworked or removed, and new ones appear that would fit here. Check it periodically, read the CHANGELOG for what changed and why, and propose adoptions. A human approves each one before it lands in flow.

## Current model guidance

[Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) supports complete task descriptions and says explicit self-check and verifier scaffolding causes over-verification. [Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) supports long autonomous tasks and supplies the autonomy and scope wording used in `execution.md`, while still expecting explicit continuation and durable progress. [GPT-5.6](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6) favours lean instructions stated once with clear approval boundaries. [GPT-6 Astra](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) supports short entrypoints, selective loading and removing obsolete recipes.

None of this means removing verification. It means removing instructions that duplicate what the model already does. Shorter instructions are not evidence of a better skill.

## What still needs testing

No comparative measurement has been run across Opus 5, Fable 5.1, GPT-5.6 Sol and GPT-6 Astra.

| Scenario | Observable success |
|----------|--------------------|
| Medium single-repo feature | One implement invocation reaches a clean review and local QA PASS with no transition questions |
| Cross-repo dependency | The consumer waits for the provider, integration is exercised, nothing is published without authorization |
| Review repairs | A real omission is fixed; an unrelated pre-existing issue and an accepted risk are rejected |
| UI without a usable browser | Required interactions stay unverified; markup never produces a false PASS |
| Compaction with a live worker | Ownership is reconciled, no duplicate writer, the next stage runs |
| Process loss with a dirty tree | Changes are preserved, missing evidence is obtained, nothing is reset blindly |
| Code change after a QA pass | The earlier PASS cannot certify the changed revisions |
| Repeated failure | The stop rule holds across resume, with a concrete blocker and any independent progress reported |
| Human acceptance | A demonstrable result and taste questions arrive before anything is published |

Run these in both hosts. Change one group of instructions at a time, and restore a rule only when its removal causes a demonstrated regression.
