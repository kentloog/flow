# Chart Steps: chart (wayfinder)

A loose idea has arrived - too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. Chart the way as a **map** of **decision tickets** - questions whose resolution is a decision, not slices of a build to execute - and resolve them one at a time until the route is clear.

The destination varies per effort, and naming it is the first act of charting - it shapes every ticket. For `/flow chart` the destination is normally **one or more spec-able deliverables**, each a future `/flow` run. The map is domain-agnostic.

## /flow chart <description | map-slug> [ticket]

**Mode inference:** if the argument names an existing folder under `.flow/`: with `map.md` - work through that map (the optional `[ticket]` names which ticket); with `state.yaml` - it's a workflow slug, not a map: say so and stop. Otherwise treat the argument as a loose idea and chart a new map: confirm a map slug with the user (suggest one) and require that no `.flow/` folder of that name already exists - maps and workflows share the namespace.

## Plan, don't do

Each ticket resolves a decision, and the map is done when the way is clear - nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in the map's **Notes** - carrying execution into the map itself - but absent that, produce decisions, not deliverables.

## Refer by name

In everything the human reads, refer to maps and tickets by their title (with the file link riding inside the name as a markdown link), never by a bare number or filename.

## The Map (local markdown tracker)

One folder per map: `.flow/<map-slug>/`.

| Item | Path |
|------|------|
| Map | `map.md` (template: `map-template.md`, this directory) |
| Tickets | `tickets/NN-<slug>.md`, numbered from `01` - one file per ticket, never a combined file (template: `ticket-template.md`, this directory) |
| Assets | `assets/` - raw research notes, temporary; the durable answer lives in the ticket |

The map is an **index**, not a store. It lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place - its ticket - so the map never restates it, only gists it and links. Open tickets are **not** listed in map.md - they are found by scanning `tickets/`.

### Operations

- **Closed** means `Status: resolved` or `Status: out-of-scope`.
- **Blocking:** a `Blocked by: NN, NN` line in the ticket header (`none` if unblocked). A ticket is unblocked when every ticket it lists is closed.
- **Frontier:** scan `tickets/` for `Status: open` tickets whose blockers are all closed. First by number wins when picking.
- **Claim:** set `Status: claimed` and save **before any work**, so concurrent sessions skip it. The status line IS the claim.
- **Resolve:** append the answer under `## Answer`, set `Status: resolved`, then append a context pointer (one-line gist + link) to the map's Decisions-so-far.
- **Rule out of scope:** set `Status: out-of-scope`, add one line to the map's Out of scope section (gist + why + link). It stays out of Decisions-so-far, which records the route actually walked.

Expect concurrent sessions: the user may run unblocked tickets in parallel. Re-read a ticket's status right before claiming, and number new tickets only after a fresh scan of `tickets/`. No lock files - claim-before-work keeps the race window to seconds. When running sessions in parallel, name an explicit ticket per session rather than launching multiple bare next-frontier sessions.

## Ticket Types

Every ticket is either **HITL** - human in the loop, worked *with* a human who speaks for themselves - or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling session that answers its own questions has broken this). Each ticket's body is one question, sized to one agent session.

- **Research** (AFK): surface a fact a decision waits on. Codebase questions - one Explore subagent per repo (`model: sonnet`), `path:line` anchor per claim. External questions - a web-research subagent per topic (inherits the session model), investigating primary sources (official docs, source code, specs), URL per claim. Distill the answer into the ticket's `## Answer`; park long raw notes in `assets/` and link them. The Answer must stand alone without the asset - `assets/` is deleted when the way is clear, so an answer that only makes sense with its raw notes is under-distilled.
- **Prototype** (HITL): raise the fidelity of the discussion with a cheap, rough, concrete artifact to react to. Follow the prototype rules in `ideation-steps.md` (this directory). Record any parked branch in the ticket's Answer on a structured line - `Prototype branches: <repo>=<branch>, ...` - so map close can find and delete them. Use when "how should it look/behave" is the key question.
- **Grilling** (HITL, the default): conversation per `grill-discipline.md` (this directory) - one question at a time with a recommended answer, the five challenge triggers, concrete-scenario stress tests. Decisions land in the ticket's Answer on resolution.
- **Task** (HITL or AFK): manual work that must happen before a *decision* can be made - signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. The one type that *does* rather than decides - it earns its place by unblocking a decision, not by delivering the destination. AFK where the agent can drive it alone; otherwise hand the human a precise checklist. The answer records what was done and any resulting facts (credentials location, URLs, row counts) later tickets depend on.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the fog - decisions and investigations you can tell are coming but can't pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, until the way is clear and no tickets remain.

The map's **Not yet specified** section holds that dim view: the suspected question, the area to revisit. Everything there is in scope, just not sharp enough to ticket.

**Fog or ticket?** The test is whether you can state the question precisely now - _not_ whether you can answer it now. Ticket when the question is sharp, even if blocked. Fog when it isn't - and don't pre-slice fog into ticket-sized pieces; one patch may graduate into several tickets, or none, once the frontier reaches it.

## Out of scope

Fog only gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope** - not fog, and not "Not yet specified". It gets the map's **Out of scope** section: work consciously ruled out of _this_ effort. It never graduates - the frontier stops at the destination - and returns only if the destination is redrawn, as a fresh effort. When an existing ticket turns out to sit past the destination, rule it out of scope (operation above) rather than resolving it on the route.

## Chart mode (new map)

1. **Name the destination.** A grilling exchange (grill-discipline mechanics) to pin down what this map is finding its way to. The destination fixes the scope, so it's settled first.
2. **Map the frontier.** Grill again, **breadth-first**: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no fog** - the way is already clear, the journey small enough for one session - you don't need a map. Stop and suggest `/flow idea` instead.
3. **Create the map folder and `map.md`**: Destination and Notes filled in, Decisions-so-far empty, the fog sketched into Not yet specified.
4. **Create the tickets you can specify now**, then wire `Blocked by:` lines in a **second pass** (tickets need numbers before they can reference each other). Everything you can't yet specify stays in the fog.
5. **Fire the research subagents**: resolve each research ticket in parallel now (AFK), recording answers and Decisions-so-far pointers like any resolution.
6. **Stop** - charting is one session's work; it hand-resolves nothing beyond the research. End with the frontier report and
   > Map charted: N tickets (M on the frontier), fog: <count> patches. Work it one session at a time: `/flow chart <map-slug>`.

## Work mode (existing map)

**Never resolve more than one ticket per session** - exceptions: research tickets, and the user explicitly asking to continue while the session's context is still light.

1. Load `map.md` - the low-res view, not every ticket body.
2. Choose the ticket: the one the user named, else the first frontier ticket. **Claim it** before any work.
3. Resolve it - **zoom as needed**: read the full body of related closed tickets on demand; use the skills the map's Notes name.
4. Record the resolution: `## Answer`, `Status: resolved`, Decisions-so-far pointer.
5. Tend the map: graduate fog the answer made specifiable into fresh tickets (create-then-wire), clearing each graduated patch from Not yet specified; rule mis-scoped tickets out of scope; update or delete tickets the decision invalidated.
6. End with the resolved ticket's gist, the new frontier, remaining fog, and the same next-step suggestion - or the handoff when the way is clear.

## Handoff - when the way is clear

No open tickets, Not yet specified empty. The `## Spawned workflows` section in map.md is the handoff's durable plan AND its progress record - a crash resumes from it, never from guessing at folders.

- **Propose the deliverable split** to the user: one deliverable = one `/flow` run, derived from the map's decisions. On agreement, **write the full plan before spawning anything**: `## Spawned workflows` in map.md, one line per agreed deliverable - `<slug> - <one-line deliverable> - pending`. If the section already exists, this is a resume: keep the recorded split, skip `spawned` lines, continue with `pending` ones.
- **Per `pending` deliverable, one at a time:** run the `/flow idea` step (ideation-steps.md), seeded from the map - problem statement from the destination, `idea.md` Context links the map, the relevant Decisions-so-far entries imported into idea.md's **Settled Decisions** section (grilling already happened on the map - the child skips it unless new questions surface), and `map: <map-slug>` in the child's state.yaml. **Immediately after the child is created**, flip its line to `spawned`. If a `pending` slug's folder already exists: with `state.yaml` carrying `map: <map-slug>` and an `idea.md`, a previous session died between creating it and flipping the line - flip to `spawned` and move on; otherwise the folder is unrelated or half-created - stop and ask the user.
- **Close the map - only after every line is `spawned`:** set `Status: way-is-clear`, delete `assets/` (raw research notes are temporary; the durable answers live in the tickets), and delete the prototype branches listed on `Prototype branches:` lines in ticket answers - their questions are answered.
- Report the spawned slugs and suggest the next step per child (`/flow spec <slug>`, or `research`/`prototype` where a child still warrants them).

## Boundaries

- A map folder is not a workflow: it has `map.md` and no `state.yaml`, never appears in `phases`/implement, and `/flow list` shows it in a separate Maps section.
- Decisions made on the map arrive at child specs as settled context; if a child's interview overturns one, update the map's ticket (append to its Answer) so the map stays the truthful decision record for its other children.
