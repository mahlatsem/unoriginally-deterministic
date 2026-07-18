---
title: unoriginally-deterministic
created: 2026-07-18
updated: 2026-07-18
status: final
---

# PRD: unoriginally-deterministic
*Working title — confirm.*

## 0. Document Purpose

This PRD scopes v1 of **unoriginally-deterministic**, a VS Code + TypeScript toolkit for building deterministic, code-derived comprehension tools (dependency maps, call-flows, structure views). It is written for the solo maintainer (also the sole v1 user) and for whoever picks up architecture, epics, and implementation next. It builds directly on prior work — the PRFAQ ([prfaq-unoriginally-deterministic.md](../../prfaq-unoriginally-deterministic.md)) and its PRD-facing distillate ([prfaq-unoriginally-deterministic-distillate.md](../../prfaq-unoriginally-deterministic-distillate.md)) — and does not repeat their competitive analysis or full Q&A; it distills them into requirements. Features are grouped with FRs nested and globally numbered (FR-1…). Inferred or deliberately-deferred content is indexed in §8 (Assumptions Index); where it's asserted at a specific point in the body, it's additionally tagged inline `[ASSUMPTION]`. Technical-how and options-considered detail that doesn't belong here lives in the companion `addendum.md`.

## 1. Vision

unoriginally-deterministic is an open-source toolkit that lets an engineer build small, deterministic tools that draw comprehension views — dependency maps, call-flows, structure — directly from code, instead of asking an LLM to explain the code and hoping the answer is right. The toolkit bundles no AI: it ships a runtime, a handful of starter tools, and a moldable framework designed to be extended equally well by an AI assistant (Claude Code, the maintainer's own choice) or by hand. Using AI is recommended, especially for speed, but the extension contract must never assume it's the only way in.

The core bet: LLMs are coherence engines, not truth engines, so the non-determinism of "does this analysis make sense" is spent once, at tool-build time, where a human can review it — and after that, the tool runs deterministically, for free, forever, on-machine, re-deriving from current code so it never goes stale. Trust is enforced structurally, not by convention: every drawn element in a view carries a required source span back to the exact line it came from. No source, no node — a lazy or rushed tool doesn't silently lie, it just produces nothing.

v1 is scoped to the maintainer's own most pressing comprehension problem: making sense of the **COBOL side** of a COBOL-and-Java estate they need to modernize, without a scarce subject-matter expert reading every line. Cross-language tracing into the Java side is explicitly out of v1 (see §4). Success in v1 is not about growing a user base — it's about whether this actually works for the one real problem it was built to solve.

Everything past the vertical slice is currently gated on one unresolved question: whether Claude Code can reliably produce correct, plug-in tools against the extension contract at all (§7, OQ-1). Nothing else in this document proceeds until that's answered.

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

#### FR-1: COBOL source extraction
The toolkit can parse COBOL source files (base tree-sitter grammar + community extensions) and expose a queryable syntax representation to the extension contract's primitives.

**Consequences (testable):**
- Extraction runs in-process; no JVM, no external network call.
- A construct the grammar can't parse surfaces as a visible tree-sitter ERROR node, never a fabricated or guessed structure.
- When a starter view (FR-5) encounters an ERROR node, it renders the surrounding structure with that node explicitly marked as unparsed — it does not silently omit it or abort the whole render.

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
- A molded tool's own source is short enough to be read end-to-end as one of the two trust checks (§9) — the other, tracing to source, is what FR-4 guarantees.
- Extraction and running a molded tool happen entirely on the user's machine (FR-4 covers the no-bundled-AI guarantee). This does not cover the molding step itself: when a tool is authored with an AI assistant (e.g. Claude Code), code context is necessarily shared with whichever assistant the engineer chooses to use — that exchange happens in the engineer's own assistant, outside the toolkit's control, exactly as it would for any other AI-assisted coding task.

#### FR-4: Structurally enforced provenance
Every drawing primitive (node/edge emission) requires a source span argument; a call omitting it does not emit anything.

**Consequences (testable):**
- A tool that skips provenance produces no output for the affected element — it cannot silently draw an unsourced node or edge.
- Every element a molded tool draws is traceable back to its exact source location (click-to-source or equivalent).
- The toolkit itself calls no LLM/model at analysis-run time.

### 3.3 Starter View Set
**Description:** A small, v1-only set of hardcoded view types plus the rendering primitives they're built from — not a general-purpose visualization framework. Realizes UJ-1, UJ-2.

#### FR-5: At least one working starter view over real COBOL
The maintainer can generate and run at least one starter view (e.g. dependency map or call-flow) over a real COBOL codebase using v1 extraction + primitives.

**Consequences (testable):**
- The view renders using only toolkit-provided rendering primitives (e.g. graph/tree/list + click-to-source).
- Every element in the rendered view satisfies FR-4's provenance requirement.
- A starter view renders on an unmolded repo with no setup, no account, and no AI/tokens spent — it works immediately on install.
- Exact additional view count/shape beyond this first one is deliberately left open. `[ASSUMPTION]`

**Out of Scope:** "Tool builds the tool" (AI composing wholly novel view types over primitives) — direction, not a v1 requirement.

### 3.4 Tool Ownership & Packaging
**Description:** A molded tool is a plain-TypeScript artifact that lives and is versioned in the *user's own* repository — never the toolkit's — so it keeps working even if the toolkit project stalls. Realizes UJ-2.

#### FR-6: Molded tools run independent of the toolkit project
A molded tool, once generated, is committed as plain TypeScript inside the engineer's own repository and continues to run without any dependency on the toolkit project's continued existence.

**Consequences (testable):**
- No toolkit-hosted registry, service, or marketplace call is required for a molded tool to execute.
- Re-running a molded tool against changed code re-derives the view with no re-molding step required (UJ-2).
- No telemetry or network call is made when a molded tool runs.

## 4. Non-Goals (Explicit)

- **Not measuring or tracking other users.** v1 makes no commitment to installs, stars, active users, or any usage analytics — success is judged entirely on the maintainer's own outcome (§6).
- **Not proving cross-AI-assistant portability.** The reliability gate (§7, OQ-1) validates Claude Code specifically. The toolkit is assistant-agnostic by design, but v1 does not test or claim reliability with other assistants — that's left to whoever chooses one.
- **Not broad language coverage.** COBOL is the v1 target for user-facing comprehension work. TypeScript extraction also ships in v1, but only to self-document the toolkit's own source (§3.1, FR-2) — not as general-purpose support for arbitrary user TypeScript codebases. Other user-facing languages (e.g. Java) are future roadmap, not a v1 commitment.
- **Not cross-language (COBOL↔Java) tracing.** The maintainer's real modernization estate entangles COBOL with Java, but v1 extraction is COBOL-only for user-facing comprehension (plus TypeScript, scoped to self-documentation, per above). Tracing where Java calls into COBOL (or vice versa) is explicitly out of v1 — a deliberate choice: v1 solves the COBOL structural-comprehension portion of the real problem, not the full cross-language picture.
- **Not a general-purpose rendering framework.** A few hardcoded views + primitives only (§3.3).
- **Not enterprise features, not hosted/SaaS anything.** No multi-tenant, no SSO, no server component.
- **Not a polished external contribution pipeline at v1.** The repo is MIT-licensed already, but community contributions are opened later, not at v1 launch.
- **No bundled AI model, API key, or model dependency inside the toolkit itself** (§3.2, FR-4).
- **No bundled commercial/heavy COBOL-dialect parsing.** Available only as a user-supplied plug-in via an extension point (§3.1, FR-1).
- **Does not catch semantic/local correctness bugs** (e.g. rounding errors, date edge cases). The toolkit directs human attention to the right place; correctness judgment stays human. This is an accepted product ceiling, not a gap to close.

## 5. MVP Scope

### 5.1 In Scope
- Tree-sitter-based COBOL extraction, in-process, no JVM (§3.1, FR-1).
- Tree-sitter-based TypeScript extraction scoped to the toolkit's own source, for self-documentation (§3.1, FR-2).
- Extension contract with structurally enforced provenance, usable equally by AI or by hand (§3.2).
- At least one working starter view over real COBOL, using toolkit rendering primitives (§3.3).
- Molded tools packaged as versioned TypeScript in the user's own repo (§3.4).
- The reliability gate experiment itself (§7, OQ-1) — this is a pre-broad-build validation activity, not a shipped feature, but it is in scope for v1's timeline since later scope depends on its outcome.

### 5.2 Rollout Plan
Phased: **(1) Dogfood** — maintainer uses v1 on real COBOL modernization work (§6, SM-1); **(2) Invited beta** — a small, informal group tries it, no formal support commitment; **(3) Public listing** — published to the VS Code Marketplace. Community contributions stay closed until after public listing (§7, OQ-4) — no defined trigger yet for when they open. Graduation criteria between phases, and what happens if a regression or trust-model break is found during beta, are not yet defined (§7, OQ-6).

### 5.3 Out of Scope for MVP
Every v1 exclusion is captured in §4 (Non-Goals) — nothing additional here.

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

1. **OQ-1 (reliability gate, blocking):** Does Claude Code, given the extension contract, reliably produce correct, plug-in molded tools that pass both trust checks unedited, across several independent tasks? Must resolve before any broad build proceeds (cross-ref SM-2).
2. **OQ-2:** Detailed extension-contract design (exact API shape, conventions, templates) — the mechanism (constrained primitives + required provenance) is known; the design itself is not yet done.
3. **OQ-3:** Exact starter view count and shape beyond the first working view — deliberately left open, to be resolved during build rather than pre-specified.
4. **OQ-4:** Trigger/timing for opening community contributions post-v1 — no defined condition yet.
5. **OQ-5:** What counts as a "striking enough" dogfood outcome to showcase (SM-3) — no explicit qualitative bar defined yet. **Leading candidate:** self-documentation — running a molded tool against the toolkit's own TypeScript source (FR-2) to produce a comprehension view of the extension-contract code, showcased directly in VS Code. Mirrors Glamorous Toolkit's own book (book.gtoolkit.com) — itself built live as proof-by-use of moldable development — but without GT's cost of entry, since it stays inside VS Code. Not committed as THE answer, but the strongest candidate so far, and buildable within v1's own scope.
6. **OQ-6:** No defined graduation criteria between rollout phases (§5.2) — what has to be true to move from invited beta to public listing — nor a rollback/pause/communication plan if a regression or trust-model break (e.g. a molded tool silently producing wrong output) is discovered during invited beta.

## 8. Assumptions Index

- §3.3, FR-5 — exact starter view count/shape beyond the first working view left open.
- Cross-cutting NFRs (performance budgets, supported VS Code versions/OS) are explicitly deferred to the architecture phase per maintainer direction — not resolved in this PRD.
- Toolkit-level (not molded-tool-level) versioning/deprecation policy is undefined at v1; assumed non-blocking while the toolkit has a single (maintainer) user. This also covers the currently-undefined behavior when a previously-molded tool's assumptions are broken by a later change to the toolkit's own extension contract.

## 9. Glossary

- **Molded tool** — a small, deterministic analysis/view tool built (by AI or by hand) against the toolkit's extension contract; plain TypeScript, versioned in the *user's own* repository, not the toolkit's.
- **Moldable framework / toolkit** — the runtime + starter tools + primitives this PRD scopes. Bundles no AI.
- **Extension contract** — the API, conventions, and primitives an AI assistant or a human uses to author a molded tool. THE make-or-break design surface (§7, OQ-2).
- **Provenance** — the required source span (exact source location) that a drawing primitive must receive before it will emit a node or edge. Structural, not conventional: omit it, nothing is drawn.
- **Trust checks** — the two independent ways any molded tool can be verified: (a) reading its source (short, bounded), or (b) tracing every drawn element back to its source span.
- **Starter view** — one of a small, v1 hardcoded set of view types (e.g. dependency map, call-flow) shipped with the toolkit as a starting point, not a general rendering framework.
- **Beachhead** — the v1 wedge: existing-code structure comprehension, COBOL-first, for the maintainer's own legacy-modernization work (the COBOL side of it — see §4 for the Java boundary).
- **Reliability gate** — the pre-broad-build go/no-go experiment validating that Claude Code can reliably produce correct, plug-in molded tools against the extension contract, unedited (§7, OQ-1).
