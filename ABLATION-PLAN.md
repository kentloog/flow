# Evaluation plan for flow

The September 13 implementation supersedes the earlier unexecuted sequence of proposed prompt deletions. Evaluate behavior and delivery quality; reduced instruction size alone is not success.

## Baselines

Compare the repository version before the user's staged changes, the user's combined staged/unstaged version before this pass, and the resulting version. Preserve model, reasoning settings, permissions, repository snapshot, acceptance criteria and available tools per comparison. Never reset the user's checkout to run an experiment; use isolated copies.

For this session, the user's pre-edit working files and staged diff were copied to `/tmp/flow-before-improvements`. That temporary snapshot is useful for this review but is not a durable benchmark archive.

## Completed validation

- Skill frontmatter validator passed. YAML examples, bundled file references, local documentation links and both SVG documents parsed/checked successfully. Git whitespace checks passed, and the staged patch remained byte-for-byte identical to the pre-edit snapshot.
- Independent fresh-agent delivery exercise: an isolated stdlib Python CLI project completed two implementation phases, current-agent review and 14 direct CLI QA cases, ending ready without human questions or external services. The final phase gate covered integration without a duplicate coordinator test run.
- Resume exercise: a later commit introduced an hour-calculation regression while state still said ready and old QA said PASS. The agent detected stale revision evidence, reproduced eight failing valid-input subtests, repaired once, then obtained clean review and 14 passing CLI QA cases. Committed phases and historical QA remained intact; the repair counter advanced to one.
- Two small clarifications followed observed ambiguities: `current` remains the last subcommand while `execution.status` expresses readiness; integration failures route into repair even when all phases are committed.
- The entrypoint decreased from 156 to 56 lines. Combined entrypoint/execution/implementation/review/QA/schema text is roughly 35% fewer words than the pre-edit working version. This is a size measurement, not a quality claim.

Exercise artifacts: `/tmp/flow-forward-test/.flow/duration-format/forward-test-observations.md` and `resume-test-observations.md`; initial successful evidence is preserved at `/tmp/flow-forward-test-baseline-evidence/.flow/`. These are temporary local artifacts. The delivery agent used direct execution and current-agent review; it did not test native worker orchestration or cross-model review.

## Follow-up checks after Claude review

Isolated checks in `/tmp/flow-review-fix-checks` exercised the agreed corrections. Standalone review left pending phases untouched, recorded a clean partial review and returned a non-running status with the next action. Cosmetic spec reconciliation retained revisions and counters, appended evidence records for the new digest, and needed no verification rerun. One native implementation worker committed its assigned change, passed two tests and returned evidence without modifying shared state; the coordinator then updated phase status.

Observations: `/tmp/flow-review-fix-checks/observations.md`. These are instruction-following checks in the current harness, not tests of both desktop apps. Concurrent multi-repo writers, dirty-start behavior and process-loss recovery remain unexercised here.

## Cross-model transport checks

An isolated fixture exercised the documented `claude` and `codex` CLI start and explicit-session resume commands. Claude 2.1.236 and Codex 0.153.4 each detected a boundary bug. The coordinator reproduced it, corrected the fixture and verified 10,001 inputs; both reviewers then confirmed resolution in their original sessions. The reviewer processes performed no fixture edits and used no model overrides or bypass flags.

The outer execution sandbox initially blocked host runtime or credential access. The same restricted reviewer commands succeeded after tool-level approval for host execution. This demonstrates both CLI transports from a Codex desktop task, not execution from every desktop/CLI coordinator or every permission configuration. Report and evidence: `/private/tmp/flow-review-transport-al91w0mo/validation-report.json` and adjacent `logs/`.

The README's Mermaid sequence diagram passed Mermaid's parser. Skill frontmatter, YAML examples, bundled references, documentation links, SVG XML and git whitespace checks passed. The in-app browser could not reach the localhost preview, so the updated README layout was not visually verified there. No comparison of model quality or speed follows from these checks.

A separate Claude bridge review initially failed with a monthly spend-limit error. A later retry completed; its result was recovered from the same session after the bridge's five-minute response timeout. Validation disproved two claims: Claude read the sibling spec/diff/report files with no permission denials, and recorded Codex start/resume contexts both showed read-only filesystem access with approval policy `never`. Setting `mcp_servers={}` did not disable inherited servers, so the instructions require verification of effective MCP/plugin restrictions instead.

The follow-up review confirmed three corrections: explicit MCP/plugin preflight, a pre-recorded Claude session UUID, and clearer standalone-review routing text. Claude withdrew four unsupported or disproven findings and reduced the untracked transport file to a pre-commit reminder. No actionable finding remained from that review. Additional evidence: `/private/tmp/flow-sibling-read-check/result.json` and `/private/tmp/flow-review-transport-al91w0mo/permission-mcp-verification.json`. Review session: `4c99ff13-2a8f-4a30-8869-d29bc4ea4416`.

## Comparative trials still to run

| Scenario | Observable success |
|----------|--------------------|
| Medium single-repo feature | One implement invocation reaches current clean review and local QA PASS without transition questions |
| Cross-repo dependency | Consumer waits for provider, integration is exercised, no unapproved publication |
| Review repairs | Real omission/regression is fixed; unrelated pre-existing issue and applicable accepted risk are rejected; related fixes share a work unit |
| UI without usable browser | Required interactions stay unverified; no false PASS from markup |
| Compaction with active worker | Ownership reconciled, no duplicate writer, counters preserved, next stage runs |
| Process loss with dirty task work | Changes preserved, missing evidence obtained, no blind reset |
| QA passed before a subsequent code change | Earlier PASS cannot certify the changed revisions |
| Repeated failure | Budget persists across resume; concrete blocker and independent progress reported |
| Planning adaptation | Reversible pending-work changes proceed; changed outcome returns to the human |
| Human acceptance | A demonstrable result and taste questions are returned before unapproved publication |

Run representative tasks in Claude Code Desktop/CLI and Codex desktop/CLI. Where model access permits, compare Opus 5, Fable 5.1, GPT-5.6 Sol and GPT-6 Astra separately. Exercise native delegation and sequential implementation, automatic opposite-family selection, bridge and CLI continuation, and switching coordinators with a recorded reviewer session. Explicitly requested CLI implementation workers need separate tests if that route is adopted.

Record completion rate, unintended human interruptions, hidden defects, false review findings, lost/duplicated work, repeated commands, elapsed time and token/cost usage where exposed. Keep full logs and artifacts outside the installed skill. Repeat sufficiently to distinguish model variability from an instruction effect; change one instruction group at a time in follow-up ablations. Restore only rules whose removal produces a demonstrated regression.

An isolated forward test supports workflow feasibility. It does not prove real desktop permissions, multi-repo integration, process-crash recovery or superiority of one model/prompt variant.
