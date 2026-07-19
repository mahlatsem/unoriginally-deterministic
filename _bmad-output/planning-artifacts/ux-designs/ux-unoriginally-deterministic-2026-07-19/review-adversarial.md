---
type: adversarial-review
target: DESIGN.md + EXPERIENCE.md (ux-unoriginally-deterministic-2026-07-19)
sources-checked:
  - ../../prds/prd-unoriginally-deterministic-2026-07-18/prd.md
  - ../../prfaq-unoriginally-deterministic-distillate.md
  - .memlog.md
reviewer: cynical adversarial pass
date: 2026-07-19
---

# Adversarial Review — UX Spine Pair

## Steelman (what genuinely holds)

Before the cuts: this is an unusually honest spine. Theme inheritance instead of a brand palette is the positioning made structural. The state-machine perspective was *rejected as fabrication* and demoted to a moldable candidate (EXPERIENCE.md:141, memlog decision) — exemplary. Unparsed-as-warning-not-error, "never a blank canvas," the voice table's rejected phrasings, dual provenance grain (popover vs rail), the GT Miller-columns rejection with cited evidence, and the memlog's explicit upstream scope flag (memlog: "(override) UPSTREAM FLAG") are all the right instincts. The findings below are where the spine fails to apply its own discipline, not where it lacks one.

---

## CRITICAL

### C1 — Flow 3 demonstrates a capability the spine itself declares out of v1 (Java reach)

**Where:** EXPERIENCE.md:219–227 (Flow 3), contradicting EXPERIENCE.md:147 (Boundary) and PRD §4 ("Not cross-language (COBOL↔Java) tracing").

Nomsa's molded question is: *"trace everything this COBOL program reads, writes, and calls, **and show me where the Java invokes it**"* — and step 4 says "The new view draws the reach she asked for." v1 extraction is COBOL + toolkit-TS only (PRD FR-1/FR-2); there is no Java grammar, so no molded tool composed over v1 primitives can draw "where the Java invokes it." The spine's own Boundary paragraph (line 147) says the view "shows the COBOL-side boundary and stops there — it does not guess at, stub, or imply the Java side." The flagship molding flow — the one written to showcase the trust checks — quietly delivers the exact overpromise the Boundary paragraph forbids. Compounding it: step 5 claims the tool is "~20 lines," while a cross-file read/write/call reach analyzer is substantial enough that the spine bundles it as a *starter perspective* (Module-reach tracer). Downstream impact: whoever builds from this flow will treat Java-side reach as a v1 molding outcome, and the contract docs will be judged against a question they cannot answer.

**Fix:** Rewrite Flow 3's question to a COBOL-only gap no starter covers (e.g. "list every paragraph that MOVEs into `TAX-RATE` fields, grouped by copybook"). If the Java half must stay for narrative truth (it's her real problem), have the flow *show the honest refusal*: the tool draws COBOL-side reach and renders the Java boundary as a marked edge-of-map, and the copy names that as the v1 line. Drop "~20 lines" or replace with "short enough to read end-to-end."

### C2 — Dynamic CALL targets: the call graph is presented as resolved when COBOL guarantees it often isn't

**Where:** EXPERIENCE.md:134 (Dependency map — "Edges: `CALL`, `COPY`…"), 135 (Sequence — "messages are `CALL`s in source order… Every message traces to its `CALL` line"), 136 (Module-reach — "everything it reads, writes, and calls"); DESIGN.md:193 (sequence marks reuse node/edge tokens).

COBOL has two CALL forms: `CALL 'PROG'` (static, resolvable) and `CALL WS-PROG-NAME` (dynamic — the target is a runtime value). The spine specifies CALL edges, sequence messages, and transitive reach as if every CALL has a statically known target. A dependency edge to a *resolved* target for a dynamic CALL is a guess — exactly the fabrication class this product exists to refuse — and an *omitted* dynamic CALL is a silent hole, which the spine also forbids. "In source order, as written" (the sequence disclaimer) covers *order*, not *target identity*; it does not save this. Transitive reach in the Module-reach tracer is unknowable past the first dynamic CALL, so "everything it… calls" is unearnable. Nothing in DESIGN.md provides a visual vocabulary for "call with unresolved target" — it is neither a normal edge nor an unparsed (tree-sitter ERROR) node; the parse *succeeded*, the target is simply not static.

**Fix:** Introduce a third first-class honesty state alongside provenance and unparsed: **unresolved target** (e.g. an edge to a visibly-distinct `‹dynamic: WS-PROG-NAME›` terminal node, dashed edge, own legend entry, popover shows the CALL line and the identifier). Sequence: a message to an unresolved lifeline, marked. Module-reach: reach past a dynamic CALL is stated as truncated ("reach ends at 2 dynamic CALLs — targets not static"), same discipline as depth truncation (line 136 already has the pattern). Add it to DESIGN.md components and to the banned list ("resolving a dynamic CALL target by heuristic").

---

## HIGH

### H1 — Unparsed regions *inside* a rendered structure: hole-adjacent edges are unspecified and currently imply knowledge

**Where:** EXPERIENCE.md:138 (Flowchart — "control flow is mechanically derivable from the AST for *any* unit"), 135 (Sequence — "CALLs in source order"), 137 (Call-flow), 63 (Unparsed node), 79 (Honest holes state); DESIGN.md:185.

The spine specifies unparsed *nodes* thoroughly but never how *edges and order* behave across a hole. If a paragraph contains a tree-sitter ERROR region: a flowchart that draws block A → `‹unparsed›` → block B asserts fall-through continuity the parser never verified (the hole could contain a `GO TO`, a `PERFORM`, an exit). So "mechanically derivable for *any* unit" is false precisely when a hole is present — the flowchart around a hole is partially inferred. Same for Sequence: an ERROR region may contain CALLs, so "CALLs in source order" is silently incomplete and the order claim spans a gap; a guard condition inside a hole is invisible. The header count ("3 regions unparsed — shown, unread") tells the user holes exist but not that *the connective tissue around holes is weaker than the rest of the view*. This is the availability-rule question in miniature: the view renders, looks uniform, and the uniformity is the dishonesty.

**Fix:** Specify hole-adjacent semantics per perspective: (a) Flowchart — edges entering/leaving an unparsed block render in the unparsed treatment (dashed, warning hue), meaning "control may flow here; the hole was not read"; never draw an A→B edge *around* a hole. (b) Sequence/Call-flow — a hole in the source range renders as a marked gap in the order ("‹unparsed region here — calls within it unknown›"), so "in source order" is visibly interrupted, not silently bridged. (c) Add to the banned list: "drawing a verified-style edge whose derivation crosses an ERROR region."

### H2 — Full v1 commitment vs the reliability gate and a solo nights-and-weekends builder: no phasing, five quietly-enormous items

**Where:** EXPERIENCE.md:120 ("committed fully to v1"), 122–145; PRD §1 ("Nothing else in this document proceeds until that's answered"), FR-5 ("at least one working starter view"), SM-2/OQ-1; distillate ("No-hurry, long-horizon, solo (nights/weekends)").

The memlog *flags* the expansion upstream (correct), but the spine text commits without phasing: 3 grains, 6 perspectives, semantic zoom, temporal trail, molded-tools-as-perspectives, selection-synced editor split, dual provenance surfaces, gallery, full keyboard/AT parity. The PRD gates *everything* on a vertical slice + a molding-reliability experiment. The quietly enormous items, each a project on its own:

1. **Deterministic graph layout** for a 214-file dependency graph with collapse-by-subtree and a layout toggle — stable, readable, *reproducible* layout (see M6) is a hard problem; "layout toggle" implies ≥2 algorithms.
2. **Sequence-diagram rendering** — lifelines, guard fragments, loop fragments: a bespoke renderer with its own layout engine.
3. **Bidirectional selection sync** with the native editor (webview↔editor, decorations, event echo suppression, spans going stale while the user types).
4. **Module-reach reads/writes** — requires data-item→file mapping (SELECT/ASSIGN, FD) and cross-program data usage through copybooks; by far the heaviest extraction in the bundle, sitting in one table row.
5. **Trail restore of exact scope+perspective+selection** — including across re-derives (see M1).

None of this is needed to run the reliability gate, which needs: extraction, provenance-enforcing primitives, *one* view, click-to-source.

**Fix:** Split v1 into named phases inside the spine. **Phase A (gate-sufficient):** dependency map + flowchart, anchor popover, F12/click-to-source, unparsed rendering, gallery-as-command-list. **Phase B (post-gate):** sequence, module-reach, call-flow, trail, rail, Code-perspective sync, molded-perspective registration. Keep the perspective *model* (grains, switcher, availability rule) as the committed architecture so Phase B slots in without redesign — commit the *shape* fully, the *surface* incrementally.

### H3 — Copybook expansion is load-bearing for every perspective and completely unspecified

**Where:** EXPERIENCE.md:126–139 (grain table: copybook is a system-grain node; COPY is a dependency edge); DESIGN.md:192 (breadcrumb example).

COBOL `COPY` is textual inclusion, frequently with `REPLACING`. Tree-sitter parses files as written, unexpanded. Consequences the spine never addresses: (a) a paragraph PERFORMed in `PAYROLL.cbl` but *defined* in a copybook resolves nowhere in call-flow — is that a hole, an edge to the copybook node, or silently dropped? (b) With `COPY … REPLACING`, identifiers in the copybook file don't textually match the identifiers in use — so the provenance anchor for a replaced identifier points at a line that *doesn't contain the name shown on the node*. That breaks the core trust gesture: Nomsa clicks through and the line doesn't say what the node says. (c) Module-reach through data items defined in copybooks shared by 40 programs is the *main* reach mechanism in real COBOL, and it's unmentioned.

**Fix:** Decide and state the model: v1 renders *as-written* (no expansion) — cross-COPY references render as edges to the copybook node with both anchors (the COPY line and the copybook definition line); `REPLACING` cases render the as-written copybook text in the anchor popover with a plain note ("included via COPY … REPLACING at PAYROLL.cbl:88") so the click-through never surprises. If expansion is deferred entirely, add it to the Boundary paragraph as an honest edge of the map.

### H4 — The provenance tick over-signals: it certifies traceability, but reads as certifying truth; and its "absence impossible" claim collides with unparsed nodes

**Where:** DESIGN.md:89–92, 187 ("Its *absence is impossible by construction*"), 184–185 (node-unparsed spec — no tick mentioned); EXPERIENCE.md:67 (tick meaning), 62, 113 ("verified to source"), 72 (gallery marks molded origin).

Two related breaks. **(a) Unparsed nodes:** DESIGN says the tick rides "every drawn element" and its absence is impossible; the node-unparsed component spec is silent on the tick. A tree-sitter ERROR region *does* have a span, so structurally it can carry one — but then a green "verified" mark sits on a region explicitly labeled *unread*, conflating "we located it" with "we read it." Either answer is currently defensible from the text; that ambiguity is a spec bug in the product's signature motif. **(b) Molded tools:** the tick guarantees *a span exists*, not that the span *supports the drawn claim*. A molded tool can emit a wrong edge whose provenance is a real, irrelevant line — it passes the structural check, renders with the same tick as a starter view, and on the canvas nothing distinguishes a vetted starter from an unaudited molded lens. "Audit the lens" exists (EXPERIENCE.md:156) but only as a command; the view itself launders molded output into starter-grade visual trust.

**Fix:** (a) State explicitly: the tick means "this element carries a source span" — nothing more; unparsed nodes carry the tick (they are located) but their unread status dominates visually; write that sentence into DESIGN.md:187 and EXPERIENCE.md:67. Never use the word "verified" in tick-adjacent copy/screen-reader text — say "traces to source" ("verified to source" at EXPERIENCE.md:113 should become "traces to PAYROLL.cbl:214"). (b) When the active lens is a molded tool, the view header names it with a one-click "view analyzer source" ("Drawn by tools/reach.ts — read it"), making trust-check #2 ambient rather than buried in a command.

---

## MEDIUM

### M1 — "No re-parse" vs "faithful to the code as it is now": staleness semantics are undefined

**Where:** EXPERIENCE.md:70, 96, 122 ("no re-parse"), 83 (re-derive on re-run), 47 ("Drawn from PAYROLL.cbl as it is now"), 139 (editor "always current"), 97 (trail restores "the exact prior step").

The spine holds two invariants that collide the moment code changes under an open view: perspective switches "never re-parse" (so they project a *snapshot*), while the product's voice claims the picture is drawn from the code "as it is now," and the Code perspective splits a *live* editor beside a snapshot canvas — after one edit, the flowchart and the code beside it disagree, with no specified treatment. Trail restore of "the exact prior step" after a re-derive may reference anchors that no longer exist. Line 69 ("states… what the view was drawn from and when") hints at snapshot honesty but is never connected to these cases.

**Fix:** Name the model: a view is a *stated snapshot* ("drawn from working tree at 14:02"); perspective switches project the snapshot (that's what "no re-parse" means — say so); when the underlying files change, the header states it plainly ("source changed since draw — re-run to re-derive") without auto-re-deriving mid-reasoning; trail steps whose anchors vanished restore with the same stale-banner rather than silently rendering. This is one paragraph in *Views & Perspectives* and one State-pattern row.

### M2 — The availability rule checks grain/kind, not derivability — the banned-list promise has no mechanism

**Where:** EXPERIENCE.md:145 (availability rule), 104 (banned: "offering a perspective the extraction can't honestly derive at the current grain"), 70; DESIGN.md:191.

The ban is about *derivability*; the rule as specified filters on *grain and node kind* only. Sequence at system grain is "valid" by rule even when 90% of the scope's CALLs are dynamic (see C2) and the diagram would be mostly unresolved; the rule prevents dead tabs, not thin truths. Conversely, silent absence has its own honesty cost: a user who expects Sequence on a copybook node gets no explanation of *why* it's absent — "absent, not disabled" trades legibility of limits for cleanliness, in a product whose whole ethos is legible limits.

**Fix:** Keep absence (no disabled tabs), but (a) redefine the rule's predicate as "the perspective can honestly derive *something* for this scope at this grain," and (b) let the perspective render its thinness honestly when chosen ("2 of 14 calls have static targets — the rest shown unresolved") rather than pre-filtering on quality. Optionally: a single "other perspectives" overflow listing what doesn't apply here and the one-line reason — honesty about the map's edges, consistent with line 147.

### M3 — Call-flow's framing is a runtime claim; Sequence got a disclaimer, Call-flow didn't

**Where:** EXPERIENCE.md:137 ("the 'what actually happens when this runs' perspective… directed and execution-ordered, so reading top-to-bottom mirrors control flow") vs 135 (Sequence: "as written in source, not a runtime trace — the header says so plainly").

"What actually happens when this runs" is precisely the register the voice section bans ("never say… 'understands your code'"). Static paragraph order in COBOL with `PERFORM THRU`, fall-through, and `GO TO` does not mirror execution in general — that's *why* the same paragraph lists `GO TO` as the mental-model breaker. The spine applied the honesty disclaimer to one system-grain perspective and skipped it on the component-grain one making the stronger claim.

**Fix:** Reword to "the static control-flow perspective inside one program: sections and paragraphs in *declared and PERFORM order, as written* — not a runtime trace" and give Call-flow the same plain header disclaimer Sequence has. Purge "what actually happens when this runs."

### M4 — F12 and Alt+←/→: two behaviors on one key, and a forked navigation history

**Where:** EXPERIENCE.md:94 (F12 = jump to source), 213 (Flow 2: F12 *opens the selection-synced editor split*), 97 (Alt+←/→ = view trail), 35, 139.

(a) F12 is specified as "jump to source" but Flow 2 shows it producing the Code-perspective split with two-way sync — two different behaviors, undisambiguated. (b) Alt+←/→ are VS Code's *global* navigation-history keys. The spine reassigns them (inside the webview) to the view trail. The moment a user F12s into the editor and presses Alt+← *there*, they get VS Code's history, not the trail — two interleaved histories behind one muscle-memory gesture, presented as pure familiarity win. (c) Practical: webviews swallow raw keys; F12/Alt+arrows won't forward without explicit keybinding contributions, and users who rebound go-to-definition get no benefit. None of this is fatal, but the spine sells these bindings as frictionless inheritance when they are contextual overloads.

**Fix:** Disambiguate: F12 = jump (editor gets focus, no sync contract); entering the *Code perspective* (tab or a distinct key) = the synced split. State the two-history reality in Interaction Primitives ("trail Back applies with canvas focus; editor focus uses VS Code's own history") so the seam is designed, not discovered. Note keybinding-contribution requirement for the architecture phase.

### M5 — "Everything it reads, writes, and calls" is a completeness claim the product forbids itself

**Where:** EXPERIENCE.md:136; cf. 181 (Rejected: "accurate / complete map" framing), distillate Rejected framings.

"Everything" is unearnable under dynamic CALLs (C2), unparsed regions (H1), copybooks (H3), and dialect gaps. The tracer's other copy already has the right pattern ("reach depth 2 — 3 further hops collapsed").

**Fix:** "Radiates from one program through what it *statically* reads, writes, and calls" + the unresolved/hole counts surfaced in the same header breath.

### M6 — "Same code, same picture" smuggles in an unscoped hard requirement: deterministic layout

**Where:** EXPERIENCE.md:51 ("Same code, same picture — re-run anytime, free"), 101 ("layout is cosmetic, the derived structure is fixed"); DESIGN.md (no layout determinism mention).

The voice line promises pictorial reproducibility; force-directed layouts (the default for graphs this size) are seed- and iteration-order-dependent. If two runs of identical code produce differently-arranged pictures, the flagship determinism copy is falsified by the user's own eyes, even though the *structure* is identical. Either layout must be deterministic (a real engineering constraint no artifact currently owns) or the copy must claim structure, not picture.

**Fix:** Add "layout is deterministic: same code + same layout choice ⇒ pixel-stable arrangement" as an explicit requirement handed to architecture, or soften the copy to "same code, same structure — re-run anytime, free." Pick one; don't ship the ambiguity.

### M7 — Molded-tools-as-perspectives grows the contract surface *before* the gate that tests the contract

**Where:** EXPERIENCE.md:143; PRD OQ-1/OQ-2, SM-2; memlog ("re-checked against the larger primitive surface").

Perspective registration (grain + node-kind declaration, switcher integration) is presented as settled contract behavior ("Carried into our contract: a molded tool can register itself **as a perspective**"). Every contract feature added pre-gate raises what Claude Code must pattern-match unedited in 4/5 attempts — it makes SM-2 harder, and a failed gate would then indict a contract fatter than it needed to be. The memlog flags sequencing; the spine text doesn't.

**Fix:** Mark registration as **contract-design input, contingent on the gate** (one clause in line 143): the gate runs against the minimal contract (draw primitives + provenance); registration is a post-gate contract extension. The UX (switcher shows molded perspectives) stands as the target; the commitment timing moves.

---

## LOW

### L1 — "Sequence — same 12 modules, entry PAYROLL" can be literally false

**Where:** EXPERIENCE.md:70, 116.

A sequence from one entry shows the *reachable* subset; if the dependency map showed 12 modules and only 7 are reachable from PAYROLL, "same 12 modules" is a false continuity claim in the one line designed to be plainly true. **Fix:** "Sequence — 7 of 12 modules reached from PAYROLL" (states continuity *and* the compression, which is more on-brand anyway).

### L2 — "Bounded reading (tens of lines)" promises a property the toolkit doesn't enforce

**Where:** EXPERIENCE.md:156 ("the tool's own short TypeScript — bounded reading (tens of lines)"), 224 ("~20 lines"); PRD FR-3 ("short enough to be read end-to-end").

Nothing stops a molded tool from being 400 lines; the trust check degrades silently and the UI copy that promised "short" becomes an overclaim. **Fix:** In spine copy, promise the *action* not the *size* ("read the analyzer end-to-end — it is the whole lens"), and optionally have the audit command state the real size ("tools/reach.ts — 214 lines") so length is a visible fact, not an assumed virtue.

---

## Summary table

| # | Severity | Finding | Betrays |
|---|---|---|---|
| C1 | Critical | Flow 3 molds Java-side reach — out of v1 by the spine's own Boundary | Scope honesty, internal consistency |
| C2 | Critical | Dynamic CALL targets have no honest treatment; call graph implied fully resolved | No-fabrication core |
| H1 | High | Edges/order across unparsed holes imply unverified continuity | Honest-holes contract |
| H2 | High | Full-v1 commitment pre-gate; 5 quietly-enormous builds; no phasing | Buildability, PRD gate |
| H3 | High | Copybook expansion/REPLACING/provenance unspecified | Click-through trust gesture |
| H4 | High | Tick reads as "verified truth"; unparsed-tick ambiguity; molded views visually starter-grade | Trust-signal semantics |
| M1 | Medium | Snapshot vs "as it is now": staleness undefined | Determinism/fidelity claims |
| M2 | Medium | Availability rule filters grain/kind, not derivability; ban has no mechanism | Availability honesty |
| M3 | Medium | Call-flow "what actually happens when this runs" — runtime overclaim, no disclaimer | Voice discipline |
| M4 | Medium | F12 dual behavior; Alt+←/→ forked histories; webview key forwarding | Muscle-memory promise |
| M5 | Medium | Module-reach "everything" — banned completeness framing | Rejected framings |
| M6 | Medium | "Same picture" requires deterministic layout — unowned requirement | Determinism claim |
| M7 | Medium | Perspective registration committed pre-gate | Gate sequencing |
| L1 | Low | "Same 12 modules" continuity copy can be false | Plain-truth copy |
| L2 | Low | "Tens of lines" unenforced promise | Overclaim in trust check |
