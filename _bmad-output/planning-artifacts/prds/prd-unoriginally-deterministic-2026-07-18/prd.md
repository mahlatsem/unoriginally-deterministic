---
title: unoriginally-deterministic
created: 2026-07-18
updated: 2026-07-19
status: final
---

# PRD: unoriginally-deterministic
*Working title — confirm.*

## 0. Document Purpose

This PRD scopes v1 of **unoriginally-deterministic**, a VS Code + TypeScript toolkit for building deterministic, code-derived comprehension tools (dependency maps, call-flows, structure views). It is written for the solo maintainer (also the sole v1 user) and for whoever picks up architecture, epics, and implementation next. It builds directly on prior work — the PRFAQ ([prfaq-unoriginally-deterministic.md](../../prfaq-unoriginally-deterministic.md)) and its PRD-facing distillate ([prfaq-unoriginally-deterministic-distillate.md](../../prfaq-unoriginally-deterministic-distillate.md)) — and does not repeat their competitive analysis or full Q&A; it distills them into requirements.

**Updated 2026-07-19** to reconcile with the finalized UX design spines ([DESIGN.md](../../ux-designs/ux-unoriginally-deterministic-2026-07-19/DESIGN.md) · [EXPERIENCE.md](../../ux-designs/ux-unoriginally-deterministic-2026-07-19/EXPERIENCE.md)), which closed OQ-3, expanded FR-5, added copybook expansion and further honest states to FR-1, and introduced the build phasing in §5.2. **Further updated 2026-07-19** to reconcile with the architecture spine ([ARCHITECTURE-SPINE.md](../../architecture/architecture-unoriginally-deterministic-2026-07-19/ARCHITECTURE-SPINE.md)), which resolved the layout-determinism fork this PRD handed it, added a fourth honest state, and grew the gate slice to include data artifacts. **Division of authority:** this PRD owns *what must be true*; the UX spines own *how it looks and behaves*; the architecture spine owns *how it is built* and resolves forks this PRD delegates to it. Conflicts were reconciled on 2026-07-19.

**Conventions.** Features are grouped with FRs nested and globally numbered (FR-1…). Inferred or deliberately-deferred content is indexed in §8 (Assumptions Index); where it's asserted at a specific point in the body, it's additionally tagged inline `[ASSUMPTION]`. Technical-how and options-considered detail that doesn't belong here lives in the companion `addendum.md`.

## 1. Vision

unoriginally-deterministic is an open-source toolkit that lets an engineer build small, deterministic tools that draw comprehension views — dependency maps, call-flows, structure — directly from code, instead of asking an LLM to explain the code and hoping the answer is right. The toolkit bundles no AI: it ships a runtime, a handful of bundled perspectives, and a moldable framework designed to be extended equally well by an AI assistant (Claude Code, the maintainer's own choice) or by hand. Using AI is recommended, especially for speed, but the extension contract must never assume it's the only way in.

The core bet: LLMs are coherence engines, not truth engines, so the non-determinism of "does this analysis make sense" is spent once, at tool-build time, where a human can review it — and after that, the tool runs deterministically, for free, forever, on-machine, re-deriving from current code so it never goes stale. Trust is enforced structurally, not by convention: every drawn element in a view carries a required source span back to the exact line it came from. No source, no node — a lazy or rushed tool doesn't silently lie, it just produces nothing.

v1 is scoped to the maintainer's own most pressing comprehension problem: making sense of the **COBOL side** of a COBOL-and-Java estate they need to modernize, without a scarce subject-matter expert reading every line. Cross-language tracing into the Java side is explicitly out of v1 (see §4). Success in v1 is not about growing a user base — it's about whether this actually works for the one real problem it was built to solve.

Everything past the vertical slice is gated on one unresolved question: whether Claude Code can reliably produce correct, plug-in tools against the extension contract at all (§7, OQ-1). The v1 surface is therefore sequenced so that gate fires as early and as cheaply as possible — one perspective, not six. Only that slice is unconditional; the rest of v1 is committed but waits on the answer (§5.2).

## 2. Target User

### 2.1 Jobs To Be Done
- As the builder and sole v1 user, I need to comprehend COBOL code I didn't write and must modernize, without reading every line myself or trusting an LLM's unverifiable summary of it.
- I need a way to build a comprehension tool once (with AI acceleration or by hand) and re-run it for free, forever, as the underlying code changes — so the investment doesn't go stale or need repeating.
- I need to be able to trust what a view shows me enough to act on it — every element traceable back to source, not just plausible-sounding.
- I want to be able to show others what this produces (a real modernization outcome) without needing to support or measure their usage of it.

### 2.2 Non-Users (v1)
- Other engineers are **not** a v1 target for adoption or support — they may see the showcased outcome (e.g. a LinkedIn post) and self-select in, but v1 makes no commitment to onboarding, supporting, or measuring them.
- Enterprises and teams are explicitly out of scope for v1 — see §4.

### 2.3 Key User Journeys
*Solo/hobby-scoped per the template's lighter dial — single-sentence form, each restating a JTBD concretely.*

- **UJ-1.** The maintainer, staring at an unfamiliar COBOL module ahead of a modernization task, molds a small dependency-map tool with Claude Code against the toolkit's primitives, runs it, and traces the resulting graph's edges back to exact source lines to confirm it's not fabricated before relying on it.
- **UJ-2.** Months later, the same maintainer re-runs that same tool against the COBOL codebase after it's changed, and the view re-derives correctly with no re-molding needed.
- **UJ-3.** The maintainer shares the outcome of a real modernization session (e.g. on LinkedIn) as a showcase; the toolkit makes no attempt to track who sees it or acts on it.

## 3. Features

### 3.1 Tree-Sitter Extraction — COBOL (primary) + TypeScript (self-documentation)
**Description:** The toolkit extracts a structured, queryable representation of source code using tree-sitter, in-process (WASM), with no JVM or external service dependency. v1 targets two languages, for two different reasons: **COBOL**, the maintainer's own most pressing comprehension problem, and **TypeScript**, needed because the toolkit's own runtime, primitives, and extension contract are themselves written in TypeScript — an engineer (or Claude Code) can't mold new COBOL-facing tools against an extension contract they can't comprehend. TypeScript extraction here is scoped to self-documenting the toolkit's own source, not general-purpose TypeScript-codebase support (see §4). Realizes UJ-1, UJ-2.

Extraction must also report the limits of what it could establish, and those limits fall into four general categories — **parse-failure regions**, **dynamic dispatch**, **unresolvable inclusion**, and **out-of-snapshot references** (a target named statically but absent from the extracted set). They are properties of grammars, not of COBOL: COBOL supplies v1's instances, and every language added later supplies its own. The categories and their required treatment therefore belong to the model, so adding a language does not reopen them (the per-language mapping lives in `EXPERIENCE.md`'s language-profile table).

#### FR-1: COBOL source extraction
The toolkit can parse COBOL source files (a tree-sitter grammar covering the enterprise dialect, including `EXEC SQL`/`EXEC CICS` as typed constructs), resolve copybook inclusion, and expose a queryable syntax representation to the extension contract's primitives.

**Consequences (testable):**
- Extraction runs in-process; no JVM, no external network call.
- A construct the grammar can't parse surfaces as a visible tree-sitter ERROR node, never a fabricated or guessed structure.
- When a view (FR-5) encounters an ERROR node, it renders the surrounding structure with that node explicitly marked as unparsed — it does not silently omit it or abort the whole render.
- **Data artifacts are extracted, not just referenced.** Declared files and the database tables a program accesses are surfaced as identifiable artifacts with their own source anchors, so they can be drawn as nodes and traced against (FR-5). *The extraction mechanism — including how embedded SQL is read — is an architecture decision; see `addendum.md`.*
- **Copybook expansion.** `COPY` (including `COPY … REPLACING`) is resolved during extraction, so a paragraph referenced in one program but defined in a copybook resolves correctly and copybook-shared data items participate in reach. In real COBOL estates this is the primary reach mechanism, not an edge case — which is why it precedes the reliability gate (§5.2) rather than following it.
- **Inclusion carries dual provenance.** An element derived through inclusion carries *both* source anchors — the inclusion site and the definition line — and where `REPLACING` substituted an identifier, that substitution is stated explicitly. Click-through must never land on a line that doesn't contain what the drawn element claimed; that surprise would damage the trust gesture (FR-4) more than not expanding at all.
- **Unresolved targets are a distinct honest state.** An invocation whose target is not statically resolvable (`CALL` on an identifier rather than a literal — dynamic dispatch) is neither resolved by heuristic nor silently dropped. It renders as its own state, visually distinct from an unparsed region: the parse *succeeded*, only the target is unknown, and conflating them would collapse two different kinds of not-knowing into a single vague one. This does **not** weaken FR-4: an unresolved target still carries a real source span — the invocation site — so "no source, no node" holds. What is unknown is the *destination*, never the *provenance*. **Corollary for inbound queries:** because a dynamic target may resolve anywhere at runtime, the set of units reaching a given unit can never be closed by static extraction — see FR-5.
- **An unresolvable inclusion is also an honest state.** Where a `COPY` member cannot be located or resolved, extraction marks the inclusion site as unresolved and continues; it never silently omits the reference, substitutes a guess, or aborts the file.
- **An out-of-snapshot reference is an honest state.** Where a target is named statically but is absent from the extracted set — a called program outside the scope, or missing from disk — it is marked as such. It is distinct from a dynamic target: the destination *is* known, it simply was not extracted. Without this state, one view would draw the reference and another drop it, both defensibly.

**Out of Scope:** Commercial/heavy COBOL-dialect parsing — available only as a user-supplied plug-in via an extension point, never bundled in core. Cross-language tracing into other languages COBOL calls out to or is called from (e.g. Java) — see §4.

#### FR-2: TypeScript extraction for toolkit self-documentation
The toolkit can parse its own source (runtime, primitives, extension contract — all TypeScript) using tree-sitter, and expose the same queryable representation FR-1 exposes for COBOL, so a molded tool can be built against the toolkit's own codebase.

**Consequences (testable):**
- A molded tool can generate a comprehension view (e.g. a dependency map) of the toolkit's own extension-contract code, enabling self-documentation (§7, OQ-5).
- This is not general-purpose TypeScript support for arbitrary user codebases — it is scoped to the toolkit's own source tree. Broader TypeScript support for user projects is future roadmap, not v1 (see §4).

### 3.2 Extension Contract — Moldable Primitives with Enforced Provenance
**Description:** The core reliability mechanism and the single most important piece of design work in the project. A tool-builder — Claude Code or a human, working directly in the engineer's own repo — composes a new analysis/view tool over vetted primitives instead of writing a parser or renderer from scratch. The contract must be equally easy to build against for AI and for a human reading the API cold; AI use is recommended for speed but never assumed as the only path. Realizes UJ-1.

#### FR-3: Compose a molded tool over toolkit primitives
A tool-builder can compose a new analysis tool by calling toolkit-provided tree-sitter query primitives and drawing primitives, without authoring a parser or renderer from first principles.

**Consequences (testable):**
- The primitive API surface is small enough that its correct usage is the path of least resistance for both an AI assistant and a human, not just documented as a convention.
- **Audit-the-lens is a surfaced action, not an assumed property.** When the active view was drawn by a molded tool, the toolkit names that tool and offers one-click access to its source — the first of the two trust checks (§9); the other, tracing to source, is what FR-4 guarantees. Because nothing on a rendered view otherwise distinguishes an unaudited molded lens from a vetted starter, this check must be ambient rather than buried.
- **The toolkit states a molded tool's actual size; it does not promise one.** Narrow primitives make short tools the natural outcome, but nothing prevents a 400-line molded tool. The audit surface therefore shows a tool's real length, and the guarantee is the *action* — read the analyzer end-to-end; it is the whole lens — not an unenforceable "it will be short."
- Extraction and running a molded tool happen entirely on the user's machine (FR-4 covers the no-bundled-AI guarantee). This does not cover the molding step itself: when a tool is authored with an AI assistant (e.g. Claude Code), code context is necessarily shared with whichever assistant the engineer chooses to use — that exchange happens in the engineer's own assistant, outside the toolkit's control, exactly as it would for any other AI-assisted coding task.

#### FR-4: Structurally enforced provenance
Every drawing primitive (node/edge emission) requires a source span argument; a call omitting it does not emit anything.

**Consequences (testable):**
- A tool that skips provenance produces no output for the affected element — it cannot silently draw an unsourced node or edge.
- Every element a molded tool draws is traceable back to its exact source location (click-to-source or equivalent).
- The toolkit itself calls no LLM/model at analysis-run time.
- **Provenance claims traceability, not correctness.** The guarantee is that a drawn element carries a source span you can jump to — not that the element is right, and not that the span was read and understood. Product surfaces must therefore say "traces to ⟨location⟩" and never "verified": a molded tool can draw a wrong element backed by a real but irrelevant span, and the structural check cannot catch that. Trust check #1 (FR-3) is what covers it.
- **The provenance mark must remain perceivable.** Provenance carried in the UI but not perceivable is a provenance failure with a visual cause — so the mark has a stated minimum size and a contrast floor, and degrades to a legible fallback rather than disappearing under any user theme (visual specification: `DESIGN.md`).

### 3.3 Perspectives Across Three Grains
**Description:** A fixed v1 set of bundled perspectives plus the rendering primitives they're built from — not a general-purpose visualization framework. The viewing model is **one scope, many perspectives**: a *scope* (what code you're looking at) projected through a *perspective* (how it's drawn), with drilling moving the scope down a grain. The model is language-agnostic; concrete constructs live in the UX spine's language profiles. Realizes UJ-1, UJ-2. Full behavioral specification: `EXPERIENCE.md`, § Views & Perspectives.

#### FR-5: Working perspectives over real COBOL
The maintainer can generate and run comprehension views over a real COBOL codebase using v1 extraction + primitives, switching perspective on a fixed scope and drilling between grains.

| Grain | A node is… | Perspectives |
|---|---|---|
| System | a compilation unit **or a data artifact** (file, table) | Dependency map · Sequence diagram · Reach tracer *(bidirectional)* |
| Component | a named callable | Call-flow |
| Unit | control flow within one callable | Flowchart · Code (native editor split) |

**Consequences (testable):**
- Each view renders using only toolkit-provided rendering primitives (graph/tree/list + click-to-source).
- Every element in every rendered view satisfies FR-4's provenance requirement.
- A bundled perspective renders on an unmolded repo with no setup, no account, and no AI/tokens spent — it works immediately on install.
- Switching perspective re-projects the existing extraction snapshot: no re-parse, selection preserved.
- A perspective is offered only where it can honestly derive something for the current scope and grain; where it applies but resolves poorly, it renders and states its own thinness rather than being silently withheld.
- **Data artifacts are first-class nodes.** The files and database tables a program declares and accesses are selectable nodes at system grain, not attributes of a program-to-program edge. Data access is typed (`read` / `write`) and points at the artifact it names. Data nodes are leaves — no code sits inside them, so they carry no component grain.
- **Reach is traceable in both directions, from any system-grain node.** *Outbound* — what a node statically reads, writes, and invokes ("once this is invoked, what else does it reach"). *Inbound* — what statically reaches it ("what will I break by changing this"), which must work from a file or table as readily as from a program, since "what writes to this table" is the question a modernization estate asks most. Both directions are available at once with direction distinguished. Blast-radius analysis needs the inbound half, and no outbound trace can supply it.
- **An inbound result is never presented as complete.** Because any dynamic-dispatch site may resolve to the target at runtime, inbound counts always report the unresolvable-dispatch population alongside the resolvable one, and an empty inbound result reads as "none found statically" — never "unused." No dead-code, disposal, or safe-to-remove affordance may be derived from inbound emptiness: that false negative is the one failure mode with the power to break the running business.
- Delivery is phased against the reliability gate — see §5.2. **This resolves OQ-3**, which previously left view count and shape open.

**Out of Scope:** "Tool builds the tool" (AI composing wholly novel view types over primitives) — direction, not a v1 requirement. A bundled state-machine perspective — only truthful where code explicitly encodes one, so it ships as a moldable-tool candidate rather than a bundled perspective.

### 3.4 Tool Ownership & Packaging
**Description:** A molded tool is a plain-TypeScript artifact that lives and is versioned in the *user's own* repository — never the toolkit's — so it keeps working even if the toolkit project stalls. Realizes UJ-2.

#### FR-6: Molded tools run independent of the toolkit project
A molded tool, once generated, is committed as plain TypeScript inside the engineer's own repository and continues to run without any dependency on the toolkit project's continued existence.

**Consequences (testable):**
- No toolkit-hosted registry, service, or marketplace call is required for a molded tool to execute.
- Re-running a molded tool against changed code re-derives the view with no re-molding step required (UJ-2).
- No telemetry or network call is made when a molded tool runs.

### 3.5 View Navigation, Discovery & Audit Surfaces
**Description:** The surfaces that let a user find a tool, move through a view, and audit a whole view at once. Behavioral specification lives in `EXPERIENCE.md`; this section states only what must be true. Realizes UJ-1, UJ-2.

#### FR-7: View gallery *(Phase A)*
A user can discover and launch every available view — bundled perspectives and in-repo molded tools alike — from one list, without knowing where a tool lives on disk.

**Consequences (testable):**
- Bundled perspectives and molded tools appear in the same list, each with a one-line statement of the question it answers.
- Each entry states where it lives and how it gets updated: bundled entries update with the toolkit release; molded entries live in the user's repo and stay frozen until re-molded. This asymmetry *is* the FR-6 ownership guarantee, so it is shown rather than left to inference.
- A molded tool that fails to load is listed with its error and source location — never silently absent from the list.

#### FR-8: View-trail history *(Phase B)*
A user can retrace their own navigation — returning to a previous scope, perspective, and selection as one restored step.

**Consequences (testable):**
- Back/forward restores scope, perspective, *and* selection together, not just position.
- A restored step whose source anchors no longer exist reports that plainly rather than rendering as if current.
- Retracing never re-runs extraction; it replays over the existing snapshot (FR-5).

#### FR-9: Provenance audit rail *(Phase B)*
A user can review the source anchors for many drawn elements in one pass, rather than checking them one at a time.

**Consequences (testable):**
- Lists anchors across the elements currently drawn, each actionable to its source location.
- Opt-in: it never displaces the view by default. Single-element checking (FR-4) remains available without it — spot-check and full audit are different jobs, and making the cheap check expensive would discourage the check that matters most.

#### FR-10: Molded-tool perspective registration *(Phase B — post-gate contract expansion)*
A molded tool can declare the grain and node kinds it applies to, and then appear as a perspective wherever that context matches — not merely as a gallery entry.

**Consequences (testable):**
- A registered molded perspective is offered under the same availability rule as a bundled one, and is bound by the same provenance requirement (FR-4).
- The drawing primitives an assistant must target for a *registered* perspective are a strict superset of the minimal set. **This is an expansion of FR-3's contract surface and therefore lands only after OQ-1 passes** — see §5.2 and OQ-2. Growing the contract before the gate would change what the gate measures.

**Cross-cutting consequence (FR-7 – FR-10, and every view interaction):** every navigation and trust action is invocable as a named, remappable command, not only by a default key. Views render in a webview, which does not inherit editor keybindings, and platform defaults differ (the editor's own back-navigation binding is not the same on macOS as elsewhere) — so a default key is a convenience and never the only route to an action.

## 4. Non-Goals (Explicit)

- **Not measuring or tracking other users.** v1 makes no commitment to installs, stars, active users, or any usage analytics — success is judged entirely on the maintainer's own outcome (§6).
- **Not proving cross-AI-assistant portability.** The reliability gate (§7, OQ-1) validates Claude Code specifically. The toolkit is assistant-agnostic by design, but v1 does not test or claim reliability with other assistants — that's left to whoever chooses one.
- **Not broad language coverage.** COBOL is the v1 target for user-facing comprehension work. TypeScript extraction also ships in v1, but only to self-document the toolkit's own source (§3.1, FR-2) — not as general-purpose support for arbitrary user TypeScript codebases. Other user-facing languages (e.g. Java) are future roadmap, not a v1 commitment.
- **Not cross-language (COBOL↔Java) tracing.** The maintainer's real modernization estate entangles COBOL with Java, but v1 extraction is COBOL-only for user-facing comprehension (plus TypeScript, scoped to self-documentation, per above). Tracing where Java calls into COBOL (or vice versa) is explicitly out of v1 — a deliberate choice: v1 solves the COBOL structural-comprehension portion of the real problem, not the full cross-language picture.
- **Not a general-purpose rendering framework.** A fixed set of bundled perspectives + primitives only (§3.3).
- **Not enterprise features, not hosted/SaaS anything.** No multi-tenant, no SSO, no server component.
- **Not a polished external contribution pipeline at v1.** The repo is MIT-licensed already, but community contributions are opened later, not at v1 launch.
- **No bundled AI model, API key, or model dependency inside the toolkit itself** (§3.2, FR-4).
- **No bundled commercial/heavy COBOL-dialect parsing.** Available only as a user-supplied plug-in via an extension point (§3.1, FR-1).
- **No assistive-technology / screen-reader support at v1.** v1 commits to keyboard operability, theme fidelity with a stated contrast floor, and never relying on color alone. It does **not** commit to an accessible-structure model for rendered views, a linearized alternative to the graph, or an announcement contract — a scoping call that follows from §2.2 (sole v1 user), stated here rather than left implicit because an unmet accessibility claim would be the overclaim this product refuses everywhere else. **Re-open trigger:** the first rollout phase targeting users beyond the maintainer (§5.3) — and because retrofitting a node-edge canvas is expensive, reconsider *before* invited beta, not after.
- ~~**Not pixel-stable view layout at v1.**~~ **RESOLVED 2026-07-19 — architecture took the other branch.** This PRD handed the layout-reproducibility fork to the architecture phase; that phase adopted deterministic layout (spine AD-7, AD-14). Determinism is therefore claimed over the **picture**, not merely the derived structure: the same code, under the same layout selection, yields the same arrangement on any machine. This is what makes a rendered view usable as auditable evidence rather than an illustration. Reproducibility is enforced by canonical ordering plus a layout engine configured as a pure function of its input, and protected by a golden-file release gate.
- **Does not catch semantic/local correctness bugs** (e.g. rounding errors, date edge cases). The toolkit directs human attention to the right place; correctness judgment stays human. This is an accepted product ceiling, not a gap to close.

## 5. MVP Scope

### 5.1 In Scope
- Tree-sitter-based COBOL extraction with copybook expansion, in-process, no JVM (§3.1, FR-1).
- Tree-sitter-based TypeScript extraction scoped to the toolkit's own source, for self-documentation (§3.1, FR-2).
- Extension contract with structurally enforced provenance, usable equally by AI or by hand (§3.2).
- Six perspectives across three grains over real COBOL, using toolkit rendering primitives (§3.3), delivered in the phases below.
- Molded tools packaged as versioned TypeScript in the user's own repo (§3.4).
- View discovery, navigation, and audit surfaces (§3.5, FR-7 – FR-10), phased below.
- The reliability gate experiment itself (§7, OQ-1) — this is a pre-broad-build validation activity, not a shipped feature, but it is in scope for v1's timeline since later scope depends on its outcome.

### 5.2 Build Phasing — the gate comes early and cheap

The viewing *model* (§3.3) is committed in full, so nothing needs redesign later. The *surface* is sequenced so the reliability gate (OQ-1) fires as early as possible, because a failed gate should not have been preceded by six renderers. This applies the same principle that governs the MCP deferral in `addendum.md` — don't pay a cost before proving the core bet.

Everything in §5.1 is v1 *scope*; the gate determines v1 *sequence*. **Only the gate slice is unconditional** — the Phase A remainder and all of Phase B are committed but conditional on OQ-1 passing, and are not started before it resolves.

| Tier | Contains | Rationale |
|---|---|---|
| **Gate slice** *(subset of Phase A — what OQ-1/SM-2 actually runs against)* | COBOL extraction **including copybook expansion and data artifacts (files and tables)** · provenance-enforcing primitives · **dependency map only** · click-to-source · unparsed + unresolved rendering | Matches the SM-2 protocol's thin vertical slice (`addendum.md`), narrowed to the dependency map. Copybook expansion is *inside* the slice because real COBOL is copybook-saturated — a gate passed against non-expanded extraction might not transfer to the actual dogfooding target. |
| **Phase A remainder** *(post-gate)* | Unit flowchart · view gallery | The flowchart is the second perspective, and the one that proves the grain model and the switcher actually work. |
| **Phase B** *(post-gate)* | Sequence diagram · reach tracer · call-flow · view-trail history · provenance rail · Code-perspective selection sync · molded-tool perspective registration | Each is a substantial build (bespoke layout engines, data-flow extraction, bidirectional editor sync). None is needed to answer OQ-1. |

Phase A must stand alone as a complete, honest product — not a demo with gaps.

**If the gate fails.** A failed gate indicts the **extension contract**, not the thesis: it means the contract did not narrow the job enough for an assistant to hit it cold. The response is to redesign the contract using what the failed attempts exposed, then re-run the experiment. The Phase A remainder and Phase B stay blocked throughout. Retries are **bounded** — the point of a go/no-go is that it can say no, and an unbounded retry loop quietly converts the gate into a formality. On exhausting the bound, the decision escalates to a deliberate choice between shipping the gate slice as a hand-authored-tools product and stopping the project on its own stated terms. `[ASSUMPTION: retry bound not yet set — maintainer to confirm the number before the gate runs]`

### 5.3 Rollout Plan
Phased: **(1) Dogfood** — maintainer uses v1 on real COBOL modernization work (§6, SM-1); **(2) Invited beta** — a small, informal group tries it, no formal support commitment; **(3) Public listing** — published to the VS Code Marketplace. Community contributions stay closed until after public listing (§7, OQ-4) — no defined trigger yet for when they open. Graduation criteria between phases, and what happens if a regression or trust-model break is found during beta, are not yet defined (§7, OQ-6).

## 6. Success Metrics

*Personal/dogfood-scale per the maintainer's explicit framing — depth here is closer to a hobby PRD's single-sentence bar than a launch PRD's quantitative breakdown, even though the toolkit ships publicly.*

**Primary**
- **SM-1**: The maintainer uses at least one molded tool on real COBOL modernization work, and it measurably speeds or clarifies comprehension versus reading the code directly (self-assessed, no formal instrument). Validates FR-5, FR-6.
- **SM-2** *(reliability gate)*: Across at least 5 independent molding tasks (distinct comprehension questions, no shared context between sessions), Claude Code generates molded tools against the extension contract that plug in and pass both trust checks (§9) unedited in at least 4 of 5 attempts. This is the pre-broad-build go/no-go — see OQ-1. Validates FR-3, FR-4. Confirmed threshold — see `addendum.md` for the test protocol (task selection, "independent"/"unedited" definitions, pass criteria).

**Secondary**
- **SM-3**: At least one real dogfooding outcome is published as a public showcase (e.g. LinkedIn), with no tracking of who engages with it. Validates FR-5. Candidate showcase artifact: self-documentation, now buildable within v1's own scope via FR-2 (see OQ-5).

**Counter-metrics (do not optimize)**
- **SM-C1**: Installs, stars, and any other adoption/growth signal are explicitly not tracked or optimized for. Counterbalances the temptation to build for breadth over the maintainer's own real need.
- **SM-C2**: View-type or language count is never traded against provenance correctness (FR-4). Counterbalances scope creep on SM-1/FR-5.

## 7. Open Questions

1. **OQ-1 (reliability gate, blocking):** Does Claude Code, given the extension contract, reliably produce correct, plug-in molded tools that pass both trust checks unedited, across several independent tasks? Must resolve before any broad build proceeds (cross-ref SM-2). The slice it runs against, and the branch taken if it fails, are both defined in §5.2; the one remaining unknown is the retry bound.
2. **OQ-2:** Detailed extension-contract design (exact API shape, conventions, templates) — the mechanism (constrained primitives + required provenance) is known; the design itself is not yet done. The UX run added four concrete requirements the design must satisfy, plus one hard constraint:
   - Provenance passed as structured data (a locatable span), not a formatted string, since the same anchor drives click-to-source, the audit rail (FR-9), and dual-anchor inclusion reporting (FR-1).
   - A declaration mechanism for perspective registration — grain plus applicable node kinds (FR-10).
   - Tool metadata sufficient for the gallery: the question a tool answers, and its origin (FR-7).
   - Load-failure diagnostics good enough to report a malformed tool with its source location rather than dropping it (FR-7).
   - **Constraint:** the gate runs against a *frozen, minimal* contract — drawing primitives plus required provenance. Every capability added before the gate changes what the gate measures, so a pass would not transfer to the larger surface. Registration and any other expansion land only after OQ-1 resolves (§5.2, FR-10).
3. ~~**OQ-3:** Exact starter view count and shape beyond the first working view.~~ **RESOLVED 2026-07-19** by the UX design run: six perspectives across three grains, phased against the reliability gate (§3.3, FR-5 and §5.2). The resolution came from UX rather than build, superseding this question's original "resolve during build" disposition.
4. **OQ-4:** Trigger/timing for opening community contributions post-v1 — no defined condition yet.
5. **OQ-5:** What counts as a "striking enough" dogfood outcome to showcase (SM-3) — no explicit qualitative bar defined yet. **Leading candidate:** self-documentation — running a molded tool against the toolkit's own TypeScript source (FR-2) to produce a comprehension view of the extension-contract code, showcased directly in VS Code. Mirrors Glamorous Toolkit's own book (book.gtoolkit.com) — itself built live as proof-by-use of moldable development — but without GT's cost of entry, since it stays inside VS Code. Not committed as THE answer, but the strongest candidate so far, and buildable within v1's own scope.
6. **OQ-6:** No defined graduation criteria between rollout phases (§5.3) — what has to be true to move from invited beta to public listing — nor a rollback/pause/communication plan if a regression or trust-model break (e.g. a molded tool silently producing wrong output) is discovered during invited beta.

## 8. Assumptions Index

- ~~§3.3, FR-5 — exact starter view count/shape left open.~~ Resolved 2026-07-19 (see OQ-3).
- Cross-cutting NFRs (performance budgets, supported VS Code versions/OS) are explicitly deferred to the architecture phase per maintainer direction — not resolved in this PRD.
- Toolkit-level (not molded-tool-level) versioning/deprecation policy is undefined at v1; assumed non-blocking while the toolkit has a single (maintainer) user. This also covers the currently-undefined behavior when a previously-molded tool's assumptions are broken by a later change to the toolkit's own extension contract.

## 9. Glossary

- **Molded tool** — a small, deterministic analysis/view tool built (by AI or by hand) against the toolkit's extension contract; plain TypeScript, versioned in the *user's own* repository, not the toolkit's.
- **Moldable framework / toolkit** — the runtime + bundled perspectives + primitives this PRD scopes. Bundles no AI.
- **Extension contract** — the API, conventions, and primitives an AI assistant or a human uses to author a molded tool. THE make-or-break design surface (§7, OQ-2).
- **Provenance** — the required source span (exact source location) that a drawing primitive must receive before it will emit a node or edge. Structural, not conventional: omit it, nothing is drawn. Claims traceability only, never correctness (FR-4).
- **Trust checks** — the two independent ways any molded tool can be verified: (a) reading its source end-to-end — it is the whole lens, and its actual length is stated rather than promised (FR-3); or (b) tracing every drawn element back to its source span (FR-4).
- **Bundled perspective** (formerly "starter view") — a perspective shipped with the toolkit rather than molded by the user. Bundled perspectives update with the toolkit release; molded tools stay frozen until re-molded (FR-7).
- **Scope** — the code a view is currently showing. Changes by drilling, never by switching perspective.
- **Perspective** — a way of projecting the current scope (dependency map, sequence, flowchart, …). Switching perspective re-projects the same extraction snapshot; it does not re-parse or change scope (FR-5).
- **Grain** — the level a scope sits at: **system** (a compilation unit or a data artifact), **component** (a named callable), **unit** (control flow within one callable). Drilling moves between grains; data artifacts are leaves and have no component grain.
- **Gate slice** — the minimal build the reliability gate runs against: extraction with copybook expansion and data artifacts, provenance-enforcing primitives, the dependency map, and click-to-source (§5.2). Deliberately smaller than Phase A.
- **Honest state** — a rendered element that reports the limit of what extraction could establish rather than guessing past it (§3.1, FR-1).
- **Data artifact** — a file or database table the code declares and accesses, modelled as a first-class system-grain node rather than an edge attribute. A leaf: it has no component grain (§3.3, FR-5).
- **Outbound / inbound reach** — the two directions of the reach tracer. *Outbound* = what a node statically reads, writes, and invokes ("once this is invoked, what else does it reach"). *Inbound* = what statically reaches it ("what will I break by changing this"). Outbound can be bounded honestly; inbound never closes, because dynamic dispatch may reach anything (§3.3, FR-5).
- **Beachhead** — the v1 wedge: existing-code structure comprehension, COBOL-first, for the maintainer's own legacy-modernization work (the COBOL side of it — see §4 for the Java boundary).
- **Reliability gate** — the pre-broad-build go/no-go experiment validating that Claude Code can reliably produce correct, plug-in molded tools against the extension contract, unedited (§7, OQ-1).
