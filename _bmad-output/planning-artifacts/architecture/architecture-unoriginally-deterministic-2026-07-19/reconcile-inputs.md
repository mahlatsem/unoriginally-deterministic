---
title: Architecture spine ↔ inputs reconciliation
target: ARCHITECTURE-SPINE.md (draft, 2026-07-19)
inputs:
  - prds/prd-unoriginally-deterministic-2026-07-18/prd.md
  - prds/prd-unoriginally-deterministic-2026-07-18/addendum.md
  - ux-designs/ux-unoriginally-deterministic-2026-07-19/EXPERIENCE.md
  - ux-designs/ux-unoriginally-deterministic-2026-07-19/DESIGN.md
created: 2026-07-20
---

# Reconciliation — what the spine did not carry

## Verdict

The spine is strong on the **derivation half** of the product — extraction, identity, purity, provenance enforcement, determinism — and that half is largely faithful to its inputs. It is thin-to-absent on three whole dimensions its inputs treat as load-bearing:

1. **Sequence.** PRD §5.2's three-tier phasing (gate slice / Phase A remainder / Phase B) and OQ-2's frozen-minimal-contract constraint appear nowhere in the spine except a parenthetical `*(post-gate)*` on one capability-map row. The spine reads as a single undifferentiated v1 build.
2. **The rendering surface.** DESIGN.md's theme-inheritance mechanism, contrast floor, fallback chains, and provenance-tick minimum size are rendering *requirements*, not decoration. No AD owns any of them. The word "theme" does not appear in the spine.
3. **The contract-as-taught-artifact.** SM-2 — the project's go/no-go — is a test of whether types + examples + the Claude Code skill teach an assistant cold. The spine names the contract's *shape* (AD-4, AD-5) but never its *teachability surface*, and never forbids reaching around the primitives into raw tree-sitter, which is an explicit SM-2 pass criterion.

There are also five places where the spine **asserts something its inputs contradict** — most seriously the paradigm section's "same code, same picture," which reverses a PRD §4 non-goal and an EXPERIENCE.md copy rule without recording that it did so.

The addendum's LOCKED decisions are mostly *compatible* with the spine but almost none are *ratified* by it, and one (delivery shape) is quietly contradicted between two sections of the spine itself.

**Gap count: 38.** Critical 6 · High 14 · Medium 14 · Low 4.

---

## Input 1 — PRD (`prd.md`)

### P-1. §5.2 build phasing is absent from the spine entirely — **CRITICAL**

PRD §5.2 defines three tiers and states "**Only the gate slice is unconditional** — the Phase A remainder and all of Phase B are committed but conditional on OQ-1 passing, and are not started before it resolves." The spine has no phasing section, no gate-slice concept, and its `binds:` frontmatter claims FR-1…FR-10 flat. A builder reading only the spine would build all fourteen ADs' worth of surface before the gate fires.

**Fix:** add a `## Phasing` section that mirrors PRD §5.2's table and marks, per AD, whether it is *gate-slice load-bearing* (AD-1, AD-2, AD-3, AD-4, AD-5, AD-9, AD-10, AD-11, AD-13, AD-14) or *post-gate* (AD-6's registration implications, AD-7 beyond the dependency map, AD-8's molded-tool distribution path, AD-12). Mark the capability map's FR-7 row as Phase A remainder and FR-8/FR-9/FR-10 as Phase B.

### P-2. OQ-2's frozen-minimal-contract constraint is never ratified — **CRITICAL**

OQ-2 states a hard constraint: "the gate runs against a *frozen, minimal* contract — drawing primitives plus required provenance. Every capability added before the gate changes what the gate measures." The spine gestures at this once, in AD-6's *Prevents* line ("contract growth before the gate"), but never as a rule anyone is bound by. Nothing in the spine stops a builder adding tool-metadata declarations, registration hooks, or richer draw primitives before the gate.

**Fix:** promote it to its own invariant — **AD-15 (contract freeze)**: until OQ-1 resolves, the exported surface of `@ud/toolkit` is exactly the draw primitives with required spans, the query primitives, and the Snapshot/ViewDoc types. Additions require the gate to be re-run, not merely reviewed.

### P-3. FR-10 is treated as an in-scope binding, contradicting its own PRD row — **CRITICAL**

The spine's frontmatter binds FR-10, and the capability map assigns it to `toolkit/model` governed by AD-2 and AD-5 — i.e. it is architected as though it lands with everything else. PRD FR-10 says explicitly: "**This is an expansion of FR-3's contract surface and therefore lands only after OQ-1 passes**." EXPERIENCE.md agrees ("registration lands as a post-gate contract extension"). The `*(post-gate)*` parenthetical in the capability map is the only trace, and it carries no force.

**Fix:** state FR-10's declaration mechanism as a *deferred* design under AD-15, not a mapped capability. Record why: designing it now leaks into the frozen surface.

### P-4. The "language profile" boundary is invoked but never defined — **HIGH**

The capability map lists "language-profile boundary" as a *governing* item for FR-2, and AD-9 binds "every language profile whose grammar embeds another language." But no AD defines what a language profile is or what it may touch. EXPERIENCE.md's *Language Profiles* section makes this a first-order structural claim: "Supporting a new language means adding a column here and a tree-sitter grammar, not touching the model, the perspectives, the trust rules, or any component."

**Fix:** add **AD-16 (language profile is the only language-aware surface)** — a profile supplies grammar, the concept→construct mapping (EXPERIENCE.md's table), and nothing else; no perspective, primitive, honest-state definition, or renderer component may branch on language.

### P-5. FR-2's scope limit is not carried — **MEDIUM**

PRD FR-2 and §4 both bound TypeScript extraction to the toolkit's own source tree, explicitly *not* general-purpose TS support. The spine's capability map says only "`toolkit/extract` (profile)," which reads as full TS support and would justify building one.

**Fix:** state the scope limit in the capability map row and in AD-16 — the TS profile is provisioned for self-documentation (OQ-5 showcase) and carries no user-codebase commitment.

### P-6. The v1 accessibility floor has no architectural owner — **HIGH**

PRD §4 does not merely drop accessibility; it makes a positive, testable commitment: "v1 commits to **keyboard operability, theme fidelity with a stated contrast floor, and never relying on color alone**." EXPERIENCE.md's *Accessibility Floor* expands this into six concrete requirements. The spine mentions none of them. Because these are quiet — stated as a non-goal's carve-out rather than as an FR — they are exactly the kind of requirement the AD structure drops.

**Fix:** add **AD-17 (rendering floor)** owning: single-tab-stop canvas with arrow-key traversal, visible focus, keyboard-reachable provenance popover, non-colour cue mandatory per kind, reduced-motion compliance, and the DESIGN.md contrast floor (see D-1…D-4 below).

### P-7. The commercial-dialect plug-in extension point is missing — **MEDIUM**

PRD §4 and FR-1 *Out of Scope* both say heavy/commercial COBOL dialect parsing is "available only as a user-supplied plug-in via an extension point, never bundled in core." An extension point is an architectural commitment. The spine has no such seam; AD-16 (proposed) is adjacent but the dialect case is a *replacement* parser, not a new profile.

**Fix:** state the seam explicitly — a profile's grammar binding is user-overridable, and an overridden grammar is recorded in the snapshot hash (AD-3) so determinism claims stay scoped to the grammar actually used.

### P-8. "No telemetry or network call" is not an invariant — **MEDIUM**

FR-6's consequences include "No telemetry or network call is made when a molded tool runs," and FR-4's include "The toolkit itself calls no LLM/model at analysis-run time." AD-1 forbids I/O in the *core*, and AD-13 forbids a *required* registry call — but the extension (the shell) is unconstrained, and AD-1's "all I/O lives in the extension" reads as permission.

**Fix:** extend AD-1 or AD-13 with a negative rule on the shell: the extension's permitted I/O is exactly filesystem reads of the workspace, writes to the output folder and cache, and editor commands. No network egress in any code path, including diagnostics.

### P-9. Gallery tool metadata (OQ-2 requirement 3) is unowned — **HIGH**

OQ-2 lists as a contract requirement: "Tool metadata sufficient for the gallery: the question a tool answers, and its origin (FR-7)." FR-7's consequences require each entry to state *where it lives and how it gets updated* — and PRD calls this asymmetry "the FR-6 ownership guarantee, so it is shown rather than left to inference." The spine maps FR-7 to `extension` governed by AD-5/AD-11 and stops. Nothing says ViewDoc or a tool manifest carries the question-answered string or the bundled/molded origin.

**Fix:** name the carrier. Recommend: ViewDoc header block carries `producer: { kind: 'bundled'|'molded', name, source, questionAnswered }`, validated at the render boundary alongside AD-4. Note the phasing tension — this is contract surface, so it lands with FR-7 in the Phase A remainder, *after* the gate (P-2).

### P-10. Load-failure diagnostics (OQ-2 requirement 4) only half-covered — **MEDIUM**

The spine's Errors convention covers *run* failure ("Tool failures surface with `file:line`, never as a partial view"). OQ-2 and FR-7 additionally require *load* failure to be diagnosable: "a molded tool that fails to load is listed with its error and source location — never silently absent from the list." A tool that throws at import time is a different code path from one that throws mid-run.

**Fix:** extend the Errors convention to load time — tool discovery is fault-isolated per tool; a load fault yields a gallery entry, never a dropped entry.

### P-11. SM-2's "no reaching around the primitives" has no enforcement surface — **CRITICAL**

The addendum's pass criteria (§5) include: "calls only approved primitives (**no reaching around them into raw APIs**)." That is a packaging invariant, not a review habit: if `@ud/toolkit` re-exports `web-tree-sitter` or exposes raw AST nodes, a generated tool can bypass AD-4 entirely — and would still pass AD-4's render-boundary check if it happened to attach a span. The spine says nothing about what the toolkit's public surface excludes.

**Fix:** add to AD-4 or AD-15 — `@ud/toolkit`'s public entry point exports only the query primitives, draw primitives, minter, and types. Raw parser handles, raw tree-sitter nodes, and grammar objects are not exported. Enforce with an export-surface test in the AD-14 gate.

### P-12. The contract's teachability surface is not an architectural artifact — **CRITICAL**

SM-2 is the project's go/no-go, and its protocol (addendum §3) tests whether "the contract's own docs/types/examples teach it cold every time." The addendum's primitive-exposure decision makes the artifact explicit: "the extension contract effectively *being* its types + a handful of examples (taught to Claude Code via the accompanying skill)." The spine treats the contract purely as a type surface. There is no folder for examples, no skill artifact, no rule that the examples are gate-tested. The single thing SM-2 measures has no home in the structural seed.

**Fix:** add to the structural seed a first-class `contract/` (or `toolkit/contract/`) containing the primitive types, the worked examples, and the Claude Code skill; add a rule that examples compile and run in CI against the same fixture corpus as AD-14, so the taught surface cannot drift from the shipped one.

### P-13. The failed-gate branch implies a seam the spine does not name — **MEDIUM**

PRD §5.2: "A failed gate indicts the **extension contract**, not the thesis... The response is to redesign the contract using what the failed attempts exposed, then re-run." That requires the contract to be replaceable without invalidating extraction, the snapshot, or the renderer. AD-5 gives ViewDoc that property for the *renderer* side; nothing gives it for the *authoring* side (draw/query primitives over the Snapshot).

**Fix:** state that the primitive layer is a facade over Snapshot + ViewDoc and carries no state of its own, so a contract redesign is a rewrite of one layer, not a migration.

### P-14. AD-11's output folder collides with FR-5's zero-setup claim — **MEDIUM**

FR-5: "A bundled perspective renders on an unmolded repo with no setup." AD-11 requires ViewDocs to be written to "one conventional workspace-relative output folder." First run therefore creates a directory in the user's repo. The spine gitignores the *cache* but says nothing about the output folder's git status, nor what happens on a read-only or non-workspace-rooted folder.

**Fix:** state that ViewDocs are derived artifacts, git-ignored by default like the cache, and that the extension degrades to an in-memory path when the workspace is not writable (see A-1, which this interacts with).

### P-15. §5.2's retry bound is an open assumption the spine does not surface — **LOW**

Tagged `[ASSUMPTION: retry bound not yet set]` in the PRD. Not architectural, but the spine's *Deferred* section is where unresolved inputs are supposed to be visible to the next reader, and this one gates the whole build sequence.

**Fix:** note it in Deferred as an inherited open item.

---

## Input 2 — Addendum (`addendum.md`)

### A-1. The LOCKED delivery shape is contradicted *within* the spine — **CRITICAL**

The addendum locks: thin extension, two jobs only — host a webview and **watch a conventional output folder for rendered-view JSON**; bundled starters run as "an in-process call from the extension's command handler straight into the bundled script... Output goes to the same conventional location a molded tool's output would."

The spine ratifies half of this in AD-11 (one output folder, extension watches it). But the **Structural Seed** flow diagram shows a fully in-memory pipeline — `perspective() → ViewDoc → validation → layout → webview` — with no output folder and no watch. And the dependency graph shows `host --> persp` directly. So the spine describes two incompatible delivery paths and never says which governs for bundled perspectives, or whether the folder is the transport or merely a byproduct.

This matters beyond tidiness: if bundled perspectives bypass the folder, the addendum's "the renderer is blind to which kind produced its input" stops being true by construction and becomes true only by discipline — the exact failure mode AD-1's paradigm section says it exists to prevent.

**Fix:** ratify the addendum's shape explicitly. The output folder is *the* transport for both producers; the in-process call for bundled perspectives is an execution convenience that still lands in the folder. Redraw the structural seed to route through it. If the architecture genuinely wants an in-memory fast path, that is a *reversal* of a locked decision and must be recorded as one, with the blindness invariant re-established some other way.

### A-2. The rejected primary architectures are not ratified — **MEDIUM**

The addendum rejected (a) Mermaid/DOT via built-in previewers, killed by the day-one pixel-perfect-clickable requirement, and (b) LSP-as-primary, "parked as a possible v2 complementary layer." The spine records neither. It does not re-open them, but it also does not foreclose them: nothing in the spine's renderer story says "a diagram-markup previewer cannot serve AD-4's click-to-source," so a later reader optimizing for less rendering code can rediscover the rejected option and re-litigate it.

**Fix:** add a short *Ratified constraints inherited from the addendum* block. It costs four lines and stops the two most attractive wrong turns. Include the LSP parking note as a forward constraint — a v2 LSP layer sits *underneath* the locked design, so the extension's editor integration should not assume it is the only path to the editor.

### A-3. "Library, not MCP for v1" is neither ratified nor given its revisit trigger — **MEDIUM**

AD-13 ("one library, two consumers") is *consistent* with the decision but is framed as a distribution rule, not as the primitive-exposure decision. The addendum's actual call — small well-typed TS library, MCP **deferred not rejected**, "revisit **after** OQ-1 proves the primitive surface is learnable" — is nowhere. A spine that doesn't carry the deferral will be read as having rejected MCP outright, and a spine that doesn't carry the trigger loses the revisit.

**Fix:** state it in AD-13 or the ratified-constraints block, with the OQ-1 trigger attached. Note the corollary the addendum implies: because the library is the contract, the *persisted readable script* is the default artifact — which is what FR-3's audit-the-source check depends on.

### A-4. The EXEC SQL question is closed without naming the second parser or deferring the choice — **HIGH**

The addendum flags this as the one known architectural risk and leaves it open. AD-9 closes it with a two-stage parse and preserves the honest fallback correctly — good. But AD-9 names no stage-2 parser, and unlike the COBOL grammar fork (which the spine *does* defer, with candidates), the SQL parser choice appears in neither the ADs nor *Deferred*. The result reads as settled when a load-bearing component is unselected and its feasibility unmeasured against the maintainer's real dialect.

**Fix:** add an entry to *Deferred* mirroring the grammar-fork entry — stage-2 parser candidates, the measurement that decides (table-name recovery rate on the real corpus), and the statement that AD-9's honest fallback holds whichever wins. Also note the addendum's warning that "BYO plug-in is not the intended answer here," so the dialect extension point (P-7) must not become the SQL escape hatch.

### A-5. The commons-graduation mechanism is structurally blocked by the spine's own asymmetry — **HIGH**

The addendum defines graduation: a useful molded tool is PR'd upstream and "graduates from *living in one user's repo* to *bundled as a starter tool in the next extension release*." Post-v1, but it constrains v1's shape.

The spine makes graduation a rewrite. Bundled perspectives are pure functions inside `toolkit/perspectives` with the signature `perspective(snapshot, scope) → ViewDoc`. Molded tools are standalone Node modules that write ViewDocs to a folder (AD-8, AD-11, AD-13). Those are different artifacts with different entry shapes. And AD-12 adds a *behavioural* asymmetry on top — molded views are document-scoped and cannot serve drills the tool did not emit — so a graduated tool would also change behaviour on graduation.

**Fix:** make the two shapes one. A molded tool should be a module exporting the same `(snapshot, scope) → ViewDoc` function, with the folder-writing being a thin runner the toolkit supplies. Then graduation is a move, not a port — and AD-12's asymmetry mostly dissolves (see E-3).

### A-6. Bundled perspectives live in the wrong package relative to the locked shape — **LOW**

The addendum: starter tools are "pre-written by the maintainer, **bundled inside the extension package** (compiled into the `.vsix`)." The spine's structural seed puts `perspectives/` inside `toolkit/`. Both can be true (toolkit-authored, extension-bundled via AD-13's "the extension bundles the same package") but the spine never says so, and the seed reads as a contradiction.

**Fix:** one sentence in the seed — perspectives are authored in the toolkit and ship in the `.vsix` by virtue of the extension bundling the package.

---

## Input 3 — EXPERIENCE.md

### E-1. "No re-parse on perspective switch" has no architectural expression — **HIGH**

This is stated four separate times in EXPERIENCE.md (Component Patterns, Interaction Primitives, the viewing model, the glossary) and once in PRD FR-5, always as behaviour: "Switching re-projects the existing snapshot: no re-parse, selection preserved." The architectural requirements it implies are: (a) the Snapshot outlives a single view render and is retained, keyed by scope + content hash; (b) node identity is stable *across perspectives* so a selection survives re-projection; (c) re-projection is synchronous enough to feel instant. The spine's AD-3 establishes one snapshot owner and content-addressing, but never says the snapshot is retained across renders — the structural seed shows a one-shot pipeline, and "the cached snapshot" appears only once, in an FR-8 capability-map cell.

**Fix:** extend AD-3 — the Snapshot is a retained, hash-keyed value with a lifetime spanning many projections; perspective switch and reach-direction toggle are pure re-projections that never touch extraction. State that AD-2 ids are the selection-preservation mechanism.

### E-2. The stated snapshot's draw timestamp collides with AD-1 and AD-3 — **HIGH**

EXPERIENCE.md requires the header to state *what was drawn and when* — "drawn from `payroll/` at 14:02" — and the stale state to read "drawn 14:02 · source changed since — re-run to re-derive." DESIGN.md puts the draw timestamp in `{components.view-header}`.

The spine's AD-1 forbids the core from reading a clock, and AD-3 says "staleness is detected by hash, never by timestamp." Neither is wrong, but together they leave a required UI field with no producer, and a reader could take AD-3 as banning the displayed timestamp altogether. Additionally, the *jump-to-source-into-a-changed-file* warning is a per-file freshness check at navigation time — a third mechanism the spine never mentions.

**Fix:** state the seam. The shell stamps wall-clock draw time onto the ViewDoc envelope at write time (shell-side, so AD-1 holds); the hash is what *decides* staleness; per-file hashes are retained in the Snapshot so a jump into a changed file can be flagged without a full re-derive. Amend AD-3's wording from "never by timestamp" to "detected by hash; timestamps are display-only."

### E-3. The anchor popover's drill preview has no data path — **HIGH**

EXPERIENCE.md, Component Patterns: the popover "Holds the source anchor(s) plus a one-line drill preview (`↵ Flowchart · 9 blocks`) — GT's peek-before-spawn affordance." Called out in *Inspiration* as one of three patterns deliberately lifted from Glamorous Toolkit, so it is not incidental.

This requires, for a node in the current ViewDoc, knowledge of *what drilling would produce and how many elements* — i.e. cross-grain information the current ViewDoc by definition does not carry. AD-6 says producers emit topology only; AD-12 says drilling into an unemitted grain requires re-running the tool. So the spine as written makes the preview impossible for molded views and unspecified for bundled ones.

**Fix:** decide and state it. Cleanest: the preview is computed by the shell against the retained Snapshot (E-1), not carried in the ViewDoc — which keeps AD-6 intact and makes the preview a bundled-perspective affordance that molded views may lack, consistent with AD-12. Whichever way, the spine must own it, because the popover is the primary provenance surface.

### E-4. Every primitive being a remappable VS Code command is entirely absent — **HIGH**

Both PRD and EXPERIENCE.md state this, and both give the same reason. PRD §3.5 cross-cutting consequence: "every navigation and trust action is invocable as a named, remappable command, not only by a default key. Views render in a webview, which does not inherit editor keybindings, and platform defaults differ." EXPERIENCE.md, Accessibility Floor: "Every primitive is also a VS Code command — contributable, remappable, and visible in the Command Palette."

The spine mentions commands nowhere. This is a concrete architectural obligation on the shell: a command registration per interaction primitive, plus bidirectional webview↔host message routing so a palette-invoked command reaches canvas state and a canvas key event resolves through the same handler. Building the canvas with DOM key handlers first and retrofitting commands later is the expensive order.

**Fix:** add to AD-17 (or a shell-specific AD) — every interaction primitive in EXPERIENCE.md's list is implemented as a registered VS Code command; webview key events dispatch *through* those commands, never around them. One handler, two entry points.

### E-5. AD-10 closes the honest-state vocabulary too tightly — **HIGH** *(also a reverse assertion — see R-2)*

AD-10: "**exactly three** honest states exist... perspectives render but never invent or suppress them." EXPERIENCE.md requires perspectives to state several further limits that are not any of those three:

- **Thin derivation** — "2 of 14 calls have static targets — the rest shown unresolved" (State Patterns; availability rule).
- **Depth truncation** — "reach depth 2 — 3 further hops collapsed"; "reach ends at 2 dynamic targets."
- **Collapse / filter** — collapse-by-subtree and category filter must "say what it hid," never silently truncate.
- **Legibility auto-collapse** — below the label-size threshold the view collapses and says what it collapsed.
- **Stale snapshot** (E-2) and **cross-language boundary** (E-9).

Under AD-10 as worded, a perspective reporting its own thinness is inventing a state it is forbidden to invent. The intent of AD-10 is right — a closed *doubt* vocabulary owned by extraction — but it over-reaches into a second category the spine never names.

**Fix:** split the concept. AD-10 governs **extraction-assigned honest states** (the three, closed). Add **AD-18 — perspective-assigned disclosure**: a closed, model-owned set of statements a perspective makes about its own projection (thinness, truncation, collapse, staleness, boundary), each carried in the ViewDoc header so the renderer displays it uniformly and no perspective invents private phrasing. Same discipline, correct layer.

### E-6. Unparsed-edge propagation is not carried — **MEDIUM**

EXPERIENCE.md is emphatic that this is the subtler lie: "edges entering or leaving an unparsed block render in the unparsed treatment... and an edge is **never** drawn *around* a hole. In Sequence and Call-flow a hole renders as a marked gap in the order." *Banned everywhere* also lists "drawing a verified-style edge whose derivation crosses an ERROR region." AD-10 defines the three states but not their propagation across edges and orderings.

**Fix:** add the propagation rule to AD-10 — adjacency to an unparsed region is inherited by touching edges and by any ordering the region interrupts; synthesizing an edge across a hole is a contract violation, enforceable at ViewDoc validation since both endpoints and the hole are addressable.

### E-7. The availability/derivability predicate is unowned — **MEDIUM**

EXPERIENCE.md's availability rule: "the switcher shows a perspective when it can honestly derive *something* for this scope at this grain — never a disabled tab... **The predicate is derivability, not merely grain and node kind**," and the overflow must name absent perspectives *with a one-line reason*. This is a real function — `canDerive(perspective, snapshot, scope) → yes | no-with-reason` — and it must be cheap (it runs to build the header). The spine mentions availability only in FR-5's capability-map row.

**Fix:** state that each perspective exports its derivability predicate alongside its projection function, that the predicate is pure over the Snapshot, and that its negative case carries a reason string. Note that this is contract surface, so the *molded* half of it lands with FR-10 post-gate (P-3).

### E-8. Edges have no identity, which the provenance rail requires — **MEDIUM**

AD-2 mints ids for nodes and calls edges "edge endpoints." But FR-9 / EXPERIENCE.md's provenance rail lists anchors "across all currently-shown elements," and "Selecting a row highlights its element on canvas **and vice versa**." Edges are elements with anchors (an edge's span is its invocation site) and must be addressable in both directions. Bidirectional highlight with no edge identity is unimplementable.

**Fix:** extend AD-2 to mint element ids for edges as well (endpoint pair + kind + span, canonicalized), and state that ViewDoc element ids are the rail's addressing scheme.

### E-9. The Code perspective's hard prohibition is not carried — **MEDIUM**

EXPERIENCE.md, twice, and DESIGN.md once: "The toolkit **never re-renders source in a webview** — the editor owns code, always current, with every editor feature intact." Plus selection-synced-both-ways between canvas and native editor split (Phase B). The spine's structural seed ends at "Editor — jump to source"; a bidirectional sync channel and the never-render-source prohibition are absent.

**Fix:** add the prohibition to AD-8's family (no primitive reaches into the editor; equally, no renderer reproduces the editor) and name the selection-sync channel as Phase B shell surface.

### E-10. Cross-language boundary reporting is not carried — **MEDIUM**

EXPERIENCE.md *Boundary*: each profile is extracted independently and where a unit is invoked from another language "the view shows the boundary of the language it knows and stops there. It does not guess at, stub, or imply the other side, and it *says so* when asked." Flow 3 makes this a climactic trust moment ("an honest 'no' she can plan around"). The spine treats cross-language as simply out of scope — which silently omits rather than states.

**Fix:** fold the boundary into AD-18's disclosure set — an out-of-profile reference is a stated boundary, not an absence.

### E-11. Filter and collapse honesty is not carried — **MEDIUM**

"Filter / collapse → narrow the view by category or subtree; never fabricates, only hides (and says what it hid)"; "never silently truncates." This is a renderer invariant with a data requirement: the renderer must know the full element population to report what it hid, so filtering happens *after* the ViewDoc, never by producing a smaller ViewDoc.

**Fix:** state it — filtering and collapse are view-state over a complete ViewDoc; a producer must never pre-filter for density.

### E-12. The two-histories seam is undocumented — **LOW**

"The trail applies **with canvas focus**; once focus is in the editor, the same gesture is VS Code's own navigation history. Two histories, one gesture — designed and documented rather than discovered." Phase B, but it is a focus-ownership rule the shell must implement deliberately.

**Fix:** note it against FR-8 in the capability map.

### E-13. The legibility floor is unowned — **MEDIUM** *(pairs with D-3)*

"Fit-to-canvas must not render labels below a readable size — below the threshold the view auto-collapses and *says* what it collapsed." A layout/renderer requirement with a numeric threshold, sitting in the same family as the provenance-tick minimum size.

**Fix:** own it in AD-17 with a stated threshold.

### E-14. The banned-list is not mirrored anywhere in the spine — **LOW**

EXPERIENCE.md's *Banned everywhere* list contains items that are architecture-enforceable, not merely design guidance: "resolving a dynamic `CALL` target by heuristic," "drawing a node/edge without a source span," "silently dropping an unparsed region," "a 'refresh' that implies the picture could differ for the same code." The spine covers the second and third structurally (AD-4, AD-10). The heuristic-resolution ban is implied by AD-10 but never stated as a prohibition on extraction.

**Fix:** state the heuristic-resolution ban in AD-10. It is the one that a well-meaning extraction improvement would violate.

---

## Input 4 — DESIGN.md

The spine has **no rendering-layer invariant of any kind**. All four gaps below are unowned in full; they are grouped rather than repeated.

### D-1. Theme inheritance and the fallback-chain rule are unowned — **HIGH**

DESIGN.md's entire colour model is a mechanism, not a palette: "All base surface/text/chrome tokens resolve to the user's active VS Code color theme at render time... **Never hardcode a hex where a `--vscode-*` token exists**." And the harder rule: "Every binding declares a **FALLBACK CHAIN**: not all themes define `charts-*`, and an undefined custom property resolves to nothing — which would **silently erase the very marks that carry the trust model**."

That last clause is a provenance failure with a CSS cause — the same failure mode PRD FR-4 flags ("Provenance carried in the UI but not perceivable is a provenance failure with a visual cause"). It belongs in the spine as an invariant, not in a stylesheet as a convention. The spine's AD-4 enforces provenance at the data layer and the validation boundary and then stops one layer short of where it can still be lost.

**Fix:** add **AD-19 — provenance survives rendering**: every meaning-bearing mark binds to a theme token via a fallback chain terminating in `currentColor`; no hardcoded colour values in the renderer; a mark that cannot resolve degrades, never disappears. This completes AD-4's "the guarantee attaches to the product, not to the happy path" into the layer where the user actually sees it.

### D-2. The contrast floor is a runtime component with no owner — **HIGH**

DESIGN.md: "**At render time, verify each composed pair (mark colour × its actual surface) against 3:1 for marks and 4.5:1 for text. On failure, fall back *in-system* — swap to `{colors.foreground}`, add a `contrastBorder` outline, or thicken the mark. Never hand-tune a hex.**"

This is not a review checklist — it describes computation the renderer performs on every draw, against colours that are only knowable at render time because they come from the user's theme. It needs a contrast-evaluation function, a resolved-colour probe, and a defined in-system fallback ladder per mark. None of this exists in the spine; nothing in the ADs, the conventions table, the structural seed, or the capability map touches rendering at all. PRD FR-4 backs it: "the mark has a stated minimum size and a contrast floor, and degrades to a legible fallback rather than disappearing under any user theme."

**Fix:** name it in AD-19 as a required renderer component, with the 3:1 / 4.5:1 ratios and the fallback ladder stated numerically in the spine rather than referenced. Add it to the AD-14 release gate: render the fixture corpus against light, dark, and high-contrast themes and assert the floor.

### D-3. The provenance tick's minimum size is unowned — **HIGH**

`{components.provenance-tick}` sets `min-size: '6px'` with the annotation "meaning-bearing mark: never smaller, **at any zoom**," and DESIGN.md's contrast-floor section restates it: "The mark carrying the entire trust thesis must not be the most fragile pixel on screen."

"At any zoom" is the load-bearing part and it is a genuine implementation constraint: on a zoomable canvas the tick must be counter-scaled against the viewport transform rather than scaling with the graph. That decision has to be made when the canvas transform model is designed — it is very expensive to retrofit onto an SVG/canvas renderer that scales everything uniformly. It appears in the spine nowhere.

**Fix:** state it in AD-19 with the 6px floor and the counter-scaling requirement, and pair it with E-13's label legibility floor — same family, same mechanism.

### D-4. High-contrast survival and the mandatory non-colour cue are unowned — **HIGH**

DESIGN.md: "**High-contrast themes work by *removing* fills and leaning on borders. So every state distinction must survive on border, line style, or text alone**"; "Category colour is meaning-bearing and must **always** be paired with a non-colour cue (line style, shape, legend entry)"; legend entries carry "colour swatch **plus** a line-style sample plus the label. The non-colour cue is not optional."

This constrains the *data model*, not just the stylesheet. ViewDoc carries "kinds" (AD-6) with no statement that the kind vocabulary is closed or registered — so a molded tool can emit an arbitrary kind, and the renderer has no non-colour cue to assign it. Combined with the six-category cap ("more than the six-category ramp in one view — the view is too dense"), the kind vocabulary needs to be a registry with a cue slot per entry, not a free string.

**Fix:** state that ViewDoc element kinds resolve against a registry that assigns each kind a colour *and* a mandatory non-colour cue (line style / shape / legend entry); a view exceeding six simultaneous categories is a producer error, not a rendering compromise. Note the phasing: how a *molded* tool declares a kind is contract surface and must wait for the post-gate expansion (P-2).

### D-5. The code/UI typography split is unowned — **LOW**

"If it's code, it's monospace" — identifiers, `file:line` spans, primitive names, unparsed spans, breadcrumb segments. The spine's conventions table specifies the *structured* span shape (`{file, startLine, …}` — good, and correctly matched to OQ-2) but says nothing about the rendering rule that makes a span visibly literal. Minor, but it is the cue that carries "this is a string from your code."

**Fix:** one row in the conventions table.

---

## Reverse — what the spine asserts that its inputs contradict

### R-1. "Same code, same picture" reverses a PRD non-goal and an EXPERIENCE copy rule — **CRITICAL**

The spine's Design Paradigm: "This makes **'same code, same picture'** true by construction rather than by discipline."

PRD §4, non-goal: "**Not pixel-stable view layout at v1.** Determinism is claimed over derived *structure*... **so product copy claims structure, never 'the same picture.'** Architecture inherits this as an explicit fork: adopt deterministic layout, or keep the narrower claim."

EXPERIENCE.md, Voice and Tone: "**Determinism copy names structure, not pixels.**... Either the architecture owns pixel-stable layout as an explicit requirement, or the copy claims structure — **the spine picks structure**." Its Do/Don't table gives the sanctioned line: "Same code, same *structure* — re-run anytime, free."

The architecture *does* take the deterministic-layout fork (AD-7 excludes force-directed engines; AD-14 byte-compares laid-out output), which is a defensible and arguably correct choice. But the spine takes it **silently**: it never states that it is exercising the fork the PRD offered, never notes that doing so unlocks a copy change the UX spine deliberately declined, and meanwhile its own Consistency Conventions table still carries the narrow-claim copy rule ("say 'traces to ⟨location⟩', never 'verified'") without touching the structure-vs-picture rule. So the spine simultaneously asserts the stronger claim in prose and leaves the weaker claim governing the copy.

Compounding it: AD-7's guarantee is real only within a pinned engine version. A layout-engine upgrade changes the picture for unchanged code. And EXPERIENCE.md's *Interaction Primitives* has a user-facing **layout toggle** — "re-arrange the same deterministic graph; layout is cosmetic" — which by design produces different pictures from the same code. Under the spine's phrasing that toggle is a contradiction; under "same structure" it is fine.

**Fix:** replace the phrase with "same code, same structure" and add an explicit fork record: architecture adopts deterministic layout (AD-7 + AD-14) as an *engineering* property, while product copy continues to claim structure per PRD §4 and EXPERIENCE.md. If the maintainer wants the stronger claim surfaced to users, that is a PRD/UX change to request, not one for the spine to assume.

### R-2. AD-10's "exactly three honest states... closed" over-constrains its inputs — **HIGH**

Detailed at E-5. As worded, the invariant forbids behaviour EXPERIENCE.md requires (thinness statements, depth truncation, collapse reporting, staleness, boundary). The fix is a layer split, not a loosening.

### R-3. AD-12's molded/bundled behavioural asymmetry is not in any input — **MEDIUM**

AD-12: "a molded view supports exactly the interactions its ViewDoc can serve. Drilling into a grain the tool did not emit requires re-running the tool, and the UI states that rather than offering an affordance it cannot honour."

Its *Prevents* rationale (don't execute arbitrary user code during navigation) is sound and worth keeping. But every input describes molded and bundled tools as behaviourally indistinguishable at the surface: the addendum — "the webview/renderer is **blind to which kind produced its input**"; EXPERIENCE.md — molded tools "**run identically to a starter**," and a registered molded perspective "appears in the switcher **exactly like a bundled one**"; PRD FR-10 — "offered under the **same availability rule** as a bundled one." The only asymmetry any input sanctions is *update behaviour* (frozen vs updates-on-release), which the addendum calls out as needing to read as ownership, not staleness.

AD-12 introduces a second, capability-based asymmetry that no input asked for, and it interacts badly with A-5 (a graduated tool would gain drill capability on graduation) and E-3 (the drill preview).

**Fix:** keep the no-arbitrary-execution prevention, but re-derive the asymmetry from *shape* rather than *origin*: any view — bundled or molded — supports drilling where the retained Snapshot can serve it (E-1, E-3), and neither kind executes producer code during navigation. That satisfies AD-12's actual concern without contradicting four separate input statements.

### R-4. AD-3's "never by timestamp" reads as banning a required UI field — **MEDIUM**

Detailed at E-2. The rule is right about *detection* and silent about *display*, and the absolute phrasing makes the required "drawn at 14:02" header field look prohibited.

### R-5. The frontmatter's flat `binds: [FR-1…FR-10]` overstates v1 commitment — **HIGH**

Detailed at P-1 and P-3. The binding list asserts that the spine governs all ten FRs as one body of work, which is precisely the framing PRD §5.2 exists to prevent. Given that OQ-1 may fail and block Phase B entirely, a spine that binds FR-8/FR-9/FR-10 unconditionally has committed architecture to work that may never be authorized.

**Fix:** annotate the binds list by tier, or add the phasing section (P-1) and reference it from the frontmatter.

---

## Summary table

| # | Input | Gap | Severity |
|---|---|---|---|
| P-1 | PRD | §5.2 build phasing absent entirely | Critical |
| P-2 | PRD | OQ-2 frozen-minimal-contract constraint not an invariant | Critical |
| P-3 | PRD | FR-10 architected as in-scope despite post-gate lock | Critical |
| P-11 | PRD/addendum | No export-surface rule; "no reaching around primitives" unenforceable | Critical |
| P-12 | PRD/addendum | Contract teachability artifact (types + examples + skill) has no home | Critical |
| A-1 | Addendum | Locked delivery shape contradicted between AD-11 and the structural seed | Critical |
| R-1 | PRD/EXPERIENCE | "Same code, same picture" reverses a non-goal silently | Critical |
| P-4 | PRD/EXPERIENCE | Language-profile boundary invoked but undefined | High |
| P-6 | PRD/EXPERIENCE | v1 accessibility floor unowned | High |
| P-9 | PRD | Gallery tool metadata (question + origin) unowned | High |
| A-4 | Addendum | EXEC SQL stage-2 parser unnamed and undeferred | High |
| A-5 | Addendum | Commons graduation blocked by bundled/molded shape asymmetry | High |
| E-1 | EXPERIENCE | Snapshot retention / no-reparse / selection preservation | High |
| E-2 | EXPERIENCE | Stated-snapshot timestamp collides with AD-1 / AD-3 | High |
| E-3 | EXPERIENCE | Anchor popover drill preview has no data path | High |
| E-4 | EXPERIENCE/PRD | Remappable VS Code commands absent | High |
| E-5 / R-2 | EXPERIENCE | AD-10 closes the honest-state vocabulary too tightly | High |
| D-1 | DESIGN | Theme inheritance + fallback chain unowned | High |
| D-2 | DESIGN | Contrast floor (3:1 / 4.5:1, render-time) unowned | High |
| D-3 | DESIGN | Provenance tick min-size at any zoom unowned | High |
| D-4 | DESIGN | High-contrast survival + mandatory non-colour cue unowned | High |
| R-5 | PRD | Flat `binds:` overstates v1 commitment | High |
| P-5 | PRD | FR-2 self-documentation scope limit dropped | Medium |
| P-7 | PRD | Commercial-dialect plug-in extension point missing | Medium |
| P-8 | PRD | No-telemetry / no-network not an invariant | Medium |
| P-10 | PRD | Load-failure diagnostics only half-covered | Medium |
| P-13 | PRD | Failed-gate contract-redesign seam unnamed | Medium |
| P-14 | PRD | AD-11 output folder vs FR-5 zero-setup | Medium |
| A-2 | Addendum | Rejected architectures (Mermaid/DOT, LSP) not ratified | Medium |
| A-3 | Addendum | "Library not MCP" + OQ-1 revisit trigger not ratified | Medium |
| E-6 | EXPERIENCE | Unparsed-edge propagation rule missing | Medium |
| E-7 | EXPERIENCE | Derivability predicate unowned | Medium |
| E-8 | EXPERIENCE | Edges have no identity; rail needs it | Medium |
| E-9 | EXPERIENCE | Never-render-source prohibition + selection sync | Medium |
| E-10 | EXPERIENCE | Cross-language boundary must be stated, not omitted | Medium |
| E-11 | EXPERIENCE | Filter/collapse honesty unowned | Medium |
| E-13 | EXPERIENCE | Label legibility floor unowned | Medium |
| R-3 | Addendum/EXPERIENCE | AD-12 asymmetry not sanctioned by any input | Medium |
| R-4 | EXPERIENCE | AD-3 "never by timestamp" reads as banning a required field | Medium |
| P-15 | PRD | Retry bound not surfaced in Deferred | Low |
| A-6 | Addendum | Perspectives' package location vs `.vsix` bundling | Low |
| E-12 | EXPERIENCE | Two-histories focus seam undocumented | Low |
| E-14 | EXPERIENCE | Heuristic dynamic-target resolution not banned explicitly | Low |
| D-5 | DESIGN | Code/UI typography split unowned | Low |

*(Row count exceeds 38 because E-5/R-2 and R-4/E-2 are cross-listed; unique gaps = 38.)*

## Recommended minimum set of new invariants

- **AD-15** — contract freeze until OQ-1 resolves; export surface enumerated and gate-tested (P-2, P-11).
- **AD-16** — language profile is the only language-aware surface; dialect grammar override seam (P-4, P-5, P-7).
- **AD-17** — interaction floor: commands, keyboard, focus, single tab stop, reduced motion (P-6, E-4).
- **AD-18** — perspective-assigned disclosure, closed set, carried in the ViewDoc header (E-5, E-10, E-11, E-13, R-2).
- **AD-19** — provenance survives rendering: token fallback chains, contrast floor, tick min-size, non-colour cue registry (D-1…D-4).
- **Amend AD-3** (retention + display timestamp), **AD-10** (propagation, heuristic ban, layer split), **AD-12** (shape not origin), and the paradigm's "same picture" phrase.
- **Add** a `## Phasing` section and a `## Ratified from the addendum` block.
