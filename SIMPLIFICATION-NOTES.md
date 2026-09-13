# Flow simplification, 2026-09-13

This pass builds on the staged simplification and unstaged portability corrections already present in the repository. It replaces the earlier proposed-diff plan with the implemented design and an evaluation plan. The user's index is preserved; this pass edits only the working tree.

## Assessment of the earlier local changes

Keep the removal of mandated test-first ordering, fixed phase counts, speculative abstraction rules, extensive-story quotas, mandatory refactoring deferral and model-specific delegation. Keep approved acceptance interfaces while permitting supporting tests that follow repository practice. Reviewer capability discovery improves portability. The later cross-model requirement below replaces automatic current-agent fallback with an explicit opt-out.

The prior changes also retained problems: automatic execution still stopped at phase transitions; context pressure and a second retry forced a new session; worker and coordinator repeated the same gate; recovery reset dirty trees; add-code and changed-line-only review filters could reject real defects; QA lacked revision identity and could substitute markup for UI behavior. Those are corrected in this pass.

## What the discussion supports

The shared theme is human judgment at the beginning and end, with agents choosing implementation details inside a clear outcome and testable constraints. Less prescription does not imply less specification or no verification. Frontloaded research, design discussion and prototypes reduce uncertainty enough for useful autonomy.

- [Maximilian's first post](https://x.com/maxedapps/status/2098397503724880058) describes research, collaborative planning, self-verification and focused human review, with manageable concurrency.
- [His second post](https://x.com/maxedapps/status/2098445366479786151) favors sensible deterministic checks and manageable tasks over sprawling agent fleets or piles of instructions.
- [Uncle Bob's post](https://x.com/unclebobmartin/status/2098432570887217520) argues that elaborate constraints can outlive their usefulness as models improve, while still valuing architectural discussion and tests.
- [Spark Tsai's reply](https://x.com/SparkTsaiX/status/2098415553195192398) frames autonomy relative to uncertainty. Flow applies that through explicit outcomes, current evidence and concrete stop conditions.
- The user-supplied screenshot shows Maximilian describing substantial upfront planning and variable review effort depending on task complexity. It supports proportional human involvement, not eliminating it.
- [Matt Pocock's reply](https://x.com/mattpocockuk/status/2098452455839457588) defends deterministic guardrails, well-designed skills and prototypes, and describes his spec/ticket/implementation combination. This supports retaining workflow structure without prescribing the coding method.
- [Captain Nym0's reply](https://x.com/nym0_speaks/status/2098487188497236426) proposes fixed interfaces and module isolation. Flow retains stable acceptance interfaces and working-tree ownership; it does not hide integration context from agents.

The signed-out X preview exposed only a subset of replies. The named replies above were expanded and read; the supplied screenshot provided additional evidence. Social posts are practitioner experience, not controlled evidence of model performance.

## Matt Pocock principles retained and adapted

| Source | Applied in flow | Deliberately not made mandatory |
|--------|-----------------|---------------------------------|
| [wayfinder](https://github.com/mattpocock/skills/tree/main/skills/engineering/wayfinder) | Map as an index; decision tickets; ready frontier and claims; research before dependent human decisions; defer unresolved work instead of inventing detail | A specific external tracker, a fixed context-token target, or execution inside the discovery map |
| [grilling](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) | Current version asks independent frontier questions in rounds, with recommendations; facts are researched; humans answer consequential decisions | The previous one-question-at-a-time rule for every situation |
| [to-spec](https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/SKILL.md) | Synthesize settled decisions; confirm stable acceptance interfaces; use domain vocabulary and durable contracts | An extensive-story quota, exactly one test interface, automatic external publication |
| [implement-spec](https://github.com/mattpocock/skills/blob/main/skills/in-progress/implement-spec/SKILL.md) | Dependency-driven work; sparse communication through artifact pointers; integrated review; grouped review repairs | A worker fleet, one worktree per task plus a merger agent, or automatic draft-PR publication for every project |

`implement-spec` is in the upstream repository's in-progress directory. These adaptations are design choices to evaluate, not claims that every upstream detail is established best practice.

## Current model guidance

[Anthropic's Opus 5 guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) supports complete task descriptions and reducing redundant self-check and verifier scaffolding. Flow removes repeated coordinator gates and generic review bureaucracy while preserving project checks and feature evidence.

[The Fable 5.1 guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) also supports long autonomous tasks, but explicit continuation, scope and durable progress still matter. Its advice should not be summarized as removing all verification instructions.

[OpenAI's GPT-5.6 guidance](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6) favors lean, consistent instructions with clear scope and approval boundaries. [The GPT-6 Astra skills article](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) supports short entrypoints, selective loading and removing obsolete recipes. Flow applies these principles without pinning a model.

[Claude Code subagents](https://code.claude.com/docs/en/subagents) provide separate contexts and support compact handoffs. Flow uses native delegation when available and direct execution otherwise; it does not assume every harness supports nested subagents. Compaction is a harness capability, while state files make recovery understandable across harnesses. API compaction controls are not assumed to be desktop controls.

The claimed 80% prompt reduction is not a target for flow. The linked interview transcript was not available for verification here. Instruction length alone cannot establish quality, reliability or lower cost.

## Implemented decisions

1. `implement` owns implementation, review, repair and local QA until ready for human acceptance. Standalone review and QA remain available.
2. The coordinator tracks work and evidence through files; sizeable workers receive outcomes and pointers. Automatic compaction continues the run. Actual process loss resumes through the same command.
3. Active worker ownership, checkpoints and retry counts survive compaction. Recovery preserves unfinished work; no blind reset or filename-based finding closure.
4. Workers own verification. The coordinator checks evidence and revision identity; integration runs checks that phase results did not cover. Review and QA evidence refers to the approved spec and tested repo revisions.
5. Review findings are prioritized by impact and validated against real contracts. Omissions and regressions do not require a changed-line anchor or an explicit spec sentence permitting additional code.
6. Local QA is required for readiness. Unavailable required capabilities produce BLOCKED, and a deployed failure remains unresolved until the fixed deployment is tested.
7. Planning covers run/QA feasibility and final acceptance. Pending execution choices can adapt without another human quiz when the outcome is unchanged.
8. Both desktop apps and CLIs use the same contract. Cross-model review needs access to the other provider through a bridge or CLI; no permission bypass or external supervisor is introduced.
9. Lessons become deterministic checks or focused documentation when useful; completed runs do not automatically add more global instructions.

## Follow-up review corrections

Claude reviewed the staged design against these source summaries; it did not independently fetch the sources. After validation, two medium issues remained: conflicting state-write duties in worker-facing instructions and an ambiguous standalone-review entry point. The corrections make coordinator ownership explicit and keep standalone review on existing work. Partial reviews are recorded as partial and cannot establish delivery readiness.

Three smaller clarifications cover dirty first-run checkouts, comparison with an approved-spec snapshot for cosmetic edits, and stable per-repo integration/setup attempt IDs. Cosmetic reconciliation retains tested revisions, review scope and unresolved findings; it does not make stale evidence current. Retry limits and evidence checks remain unchanged.

## Harness portability and iterative review

The user explicitly requested bidirectional cross-model review after the initial simplification. `auto` now selects Claude for Codex-led work and Codex for Claude-led work. The implementing agent validates findings, repairs supported issues and sends dispositions and evidence back to the same reviewer session. Session replacement retains the report history and repair budget. Missing required reviewer access cannot silently become self-review or acceptance readiness; `current` remains an explicit opt-out.

Transport details live in [reviewer-transport.md](reviewer-transport.md), loaded for setup or review. Native delegation remains optional for implementation. Review uses the tools exposed in the host, with CLI fallback, explicit session IDs and read-only scope. The README distinguishes Claude Code's desktop experience from ordinary chat and gives both harnesses' invocation and installation paths.

This is a user-selected workflow policy, not a claim that every project needs two models. The invocation and continuation guidance is grounded in [Claude's programmatic sessions](https://code.claude.com/docs/en/headless), [Claude Desktop's shared configuration](https://code.claude.com/docs/en/desktop), [Codex non-interactive sessions](https://learn.chatgpt.com/docs/non-interactive-mode), and installed CLI help. These sources establish available mechanisms, not a guarantee that every host exposes them or that cross-model review improves every task.

Claude's portability review led to three narrow corrections: explicit verification of MCP/plugin restrictions on both CLI calls, recording Claude's session UUID before launch, and clearer wording for partial standalone reviews. Live access checks and recorded effective Codex permissions disproved two proposed CLI defects. Extra namespaces and review counters were not added without evidence of failure. Claude confirmed these dispositions in the same review session.

## Validation limits

See [ABLATION-PLAN.md](ABLATION-PLAN.md) for checks and remaining experiments. This pass does not claim measured improvement across Opus 5, Fable 5.1, GPT-5.6 Sol and GPT-6 Astra. Installed skill copies outside this repository are not synchronized here.
