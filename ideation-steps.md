# Ideation Steps: idea, research, prototype

## /flow idea <description>

Input may be a ticket reference (fetch it via the tracker configured in `.flow/config.yml`), a file path, or plain text. If the effort is too big or foggy for one deliverable - destination not nameable, multiple independent deliverables visible, open decisions exceeding one grilling session - recommend `/flow chart <description>` instead; continue with idea only if the user declines or the effort fits one deliverable.

### 1. Capture

Confirm the repo(s) (from config `layout`/`repos`; single-repo records as `[root]`) and a slug (`42-token-refresh` with a ticket, `sso-support` without - the slug doubles as the branch name, so it embeds a branch-safe form of the ticket ref per config `tracker.ref_format`: `#42` -> `42-...`, `PROJ-123` -> `proj-123-...`), create `.flow/<slug>/` with state.yaml per the schema, and write `idea.md` (template: idea-template.md) - a seed, not a spec.

When spawned from a `/flow chart` handoff, seed from the map instead: destination excerpt as the problem, map link in Context, the map's relevant decisions imported into Settled Decisions, and `map: <map-slug>` in state.yaml. Grilling already happened on the map - skip step 2 unless new questions surface.

### 2. Grill

Interview the user relentlessly about every aspect of the idea until a shared understanding is reached, per `grill-discipline.md` (this directory): walk down each branch of the decision tree, resolving dependencies between decisions one by one; one question at a time with a recommended answer; look up facts, ask decisions; the five challenge triggers; concrete-scenario stress tests. Do not move on until the user confirms the shared understanding. This is where the thinking happens - the later spec step synthesizes, it does not interview.

Record as you go, never batched:

- Resolved decisions -> idea.md **Settled Decisions**.
- Declined scenarios -> idea.md **Accepted Risks (draft)**.
- Unresolved questions -> idea.md **Open Questions**, each marked factual (research can answer it) or design-feel (needs a prototype or a human call).

Skippable only when the idea is trivial or arrives fully settled; when skipping, say so.

### 3. Route

Per grill-discipline's ending rules: factual Open Questions -> recommend `/flow research <slug>`; design-feel ones -> `/flow prototype <slug>`; neither ->

> Idea grilled and captured. Next: `/flow spec <slug>`.

## /flow research <slug> (optional)

Cache exploration findings in `research.md` (template: research-template.md). The doc is **temporary** - the whole workflow folder is deleted at complete, and stale research misleads later agents.

Derive the questions from idea.md's Open Questions (let the user amend), split them into codebase questions and external questions, and fan out: one Explore subagent per repo for codebase questions (`model: sonnet`; return key files, patterns, extension points - `path:line` anchor per claim), and one web-research subagent per external topic (inherits the session model), investigating primary sources (official docs, source code, specs) with a URL per claim. Combine into `research.md`; update state.yaml.

If Remaining Unknowns is non-empty, recommend `/flow prototype <slug>` to resolve them through experimentation; otherwise suggest `/flow prototype <slug>` or `/flow spec <slug>`.

## /flow prototype <slug> (optional)

A prototype is throwaway code that answers a question. Human-in-the-loop: follow the user's lead, implement what they describe, show results, iterate.

1. **State the question first**, seeded from the idea's Open Questions and research's Remaining Unknowns, and confirm it with the user - it later heads the Prototype Findings section.

2. **Isolate the code.** With config `worktrees: true`: one worktree per involved repo - `git -C <repo> worktree add <project-root>/.flow/worktrees/<name>--proto-<slug> -b proto/<slug>` (absolute path - a relative one would resolve inside the repo; `<name>` is the repo name, or the project-root basename for `root`), copy untracked env files the app needs (`.env`, `.envrc`, `.env.local`), install dependencies. All prototype code lives and runs in the worktree, never the main checkout. The worktree is temporary - removed in the lifecycle step regardless of whether the branch is parked. With `worktrees: false`: require a clean tree, create `proto/<slug>` in the main checkout, and return the checkout to the original branch in the lifecycle step.

3. **Route by question type:**
   - **"Does this logic / state model feel right?"** - isolate the logic in a portable pure module (a pure reducer, an explicit state machine, or a small set of pure functions - whichever fits the question, not whichever is easiest to drive) behind a thin throwaway driver (interactive terminal app or script). The driver imports the module and calls into it; nothing flows the other direction. When the question is answered, the validated core lifts into the real code on its own.
   - **"What should this look like?"** - N structurally different UI variants (default 3, cap 5) behind a `?variant=` URL param with a small floating switcher. Mount the variants inside an existing page whenever one plausibly hosts them - a throwaway route on its own is a vacuum. Variants must differ in structure (layout, hierarchy, primary affordance), not colors - the interesting feedback is "the header from B with the sidebar from C".

4. **Throwaway rules:** clearly marked as a prototype in name and location, obeying the project's routing/task-runner conventions; one command to run; no persistence unless persistence IS the question (then a scratch store named "PROTOTYPE - wipe me"); no tests, no error handling beyond runnable, no abstractions, no generalising; surface the full relevant state after every action or variant switch.

5. **Capture findings** when the user signals done: append `## Prototype Findings` to `idea.md` - the question (verbatim), the answer, decisions made ("tried X, rejected because Y"), what carries into the spec, what gets thrown away. Include a snippet only where it encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), trimmed to the decision-rich parts.

6. **Lifecycle - prototype code never merges.** Delete the code now (`git -C <repo> branch -D proto/<slug>` after removing the worktree), or commit it on the `proto/<slug>` branch and record the branch per repo in Prototype Findings and state.yaml `prototype_branches`. Remove the worktree(s) either way (`git -C <repo> worktree remove <path>`, `--force` if dirty and the code isn't being parked) - only the parked branch survives. Implement starts from a clean default branch; parked branches are deleted at `/flow complete`. When a validated core later lifts into the real code, it gets rewritten properly there - prototype code was written under prototype constraints (no tests, minimal error handling), so it is a reference, never a paste.

Update state.yaml and suggest: `/flow spec <slug>`.
