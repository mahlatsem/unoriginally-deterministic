# Validation Report — unoriginally-deterministic

- **DESIGN.md:** `_bmad-output/planning-artifacts/ux-designs/ux-unoriginally-deterministic-2026-07-19/DESIGN.md`
- **EXPERIENCE.md:** `_bmad-output/planning-artifacts/ux-designs/ux-unoriginally-deterministic-2026-07-19/EXPERIENCE.md`
- **Run at:** 2026-07-19
- **Lenses:** rubric walker · accessibility · adversarial

## Overall verdict

A downstream consumer can source-extract this pair cleanly: every `{path.to.token}` reference in both files resolves, all three frontmatter sources exist, PRD/PRFAQ citations are verbatim-accurate, and the three invented sections carry the product's actual load. Shape fit and flow coverage are **strong**. The spine's honesty instincts are real and structural — theme inheritance instead of a brand palette, the state-machine perspective rejected *as fabrication*, unparsed-as-warning-not-error, the GT Miller-columns rejection with cited evidence.

Two lenses converge on the same underlying failure, and it is the one worth acting on: **the spine's picture is more resolved than the extraction can prove, in exactly the three places COBOL gets hard.** Dynamic `CALL identifier` targets, unparsed regions *inside* a rendered structure, and unexpanded copybooks each produce a view that looks uniformly verified while parts of it are inferred. That is the fabrication class this product exists to refuse, arriving through the back door. Separately, the accessibility floor names the right outcomes but commits no mechanism — for a node-edge canvas the mechanism *is* the hard part, and as written the screen-reader commitments are not implementable.

Two scope-level issues sit above the craft findings: the spine supersedes PRD OQ-3 and materially expands FR-5 with the flag recorded only in `.memlog.md`, so a consumer reading PRD + spines sees an unflagged contradiction with no declared winner; and "committed fully to v1" contains roughly five renderer-scale builds for a solo pre-gate maintainer, with no phasing against the PRD's blocking reliability gate. Neither requires redesign — the perspective model survives all three reviews intact. The claims, the sequencing, and the accessibility mechanisms need work.

## Category verdicts

| Category | Verdict |
|---|---|
| Flow coverage | **strong** |
| Token completeness | adequate |
| Component coverage | adequate |
| State coverage | adequate |
| Visual reference coverage | pending (mocks rendered post-review) |
| Bloat & overspecification | adequate |
| Inheritance discipline | adequate |
| Shape fit | **strong** |

**Counts:** Critical 5 · High 9 · Medium 22 · Low 14

---

## Findings by severity

### Critical (5)

**[Inheritance discipline]** — Spine supersedes its source without saying so (EXPERIENCE.md:118–147 vs PRD FR-5/OQ-3)
The spine resolves PRD OQ-3 and expands FR-5's "at least one working starter view" into six perspectives across three grains, plus semantic zoom and the trail. The upstream flag exists only in `.memlog.md`. EXPERIENCE.md:13 declares the spines win over "any mock or import" — spine-vs-PRD precedence is undeclared. A consumer reading PRD (`status: final`) + spines sees a contradiction with no winner, and may scope to either.
*Fix:* Add an explicit supersession note in Views & Perspectives; run the PRD update before architecture consumes the pair.

**[Adversarial C1]** — Flow 3 demonstrates a capability the spine declares out of v1 (EXPERIENCE.md:219–227 vs :147, PRD §4)
Nomsa's molded question includes "show me where the Java invokes it," and step 4 says the view draws it. v1 extraction is COBOL + toolkit-TS only; no molded tool over v1 primitives can draw Java-side reach. The spine's own Boundary paragraph forbids exactly this. The flagship molding flow delivers the overpromise the Boundary paragraph exists to prevent.
*Fix:* Rewrite the question to a COBOL-only gap, or have the flow *show the honest refusal* — draw COBOL-side reach, render the Java boundary as a marked edge-of-map. Drop "~20 lines."

**[Adversarial C2]** — Dynamic `CALL` targets: the call graph is presented as resolved when COBOL guarantees it often isn't (EXPERIENCE.md:134–136)
COBOL has `CALL 'PROG'` (static) and `CALL WS-PROG-NAME` (dynamic, target is a runtime value). The spine specifies CALL edges, sequence messages, and transitive reach as if every target is statically known. Drawing a resolved edge for a dynamic CALL is a guess; omitting it is a silent hole. Both are forbidden. "In source order, as written" covers *order*, not *target identity*. No visual vocabulary exists for this — the parse *succeeded*, the target simply isn't static.
*Fix:* Introduce a third first-class honesty state, **unresolved target**: dashed edge to a distinct `‹dynamic: WS-PROG-NAME›` terminal node, own legend entry, popover shows the CALL line and identifier. Reach past a dynamic CALL states truncation. Add "resolving a dynamic CALL target by heuristic" to the banned list.

**[Accessibility C1]** — No accessible-structure model or linearized alternative for the graph canvas (EXPERIENCE.md:110–116, 95)
The spine commits to "every node/edge reachable" and meaningful announcement but never specifies what the accessibility tree contains. A node-edge canvas exposes nothing by default; edges have no native announceable representation. The one linear surface offered — the provenance rail — lists *source anchors*, not topology, so a screen-reader user can audit provenance but never comprehend the view. That inverts the product's purpose.
*Fix:* Commit to an explicit ARIA model (canvas as one tab stop, roving tabindex, nodes/edges as focusable elements with name + kind + relationships) **and** a per-perspective linearized equivalent — a "Structure view" sibling to the rail (adjacency list, ordered message list, depth-ordered reach list, nested block outline). This also answers WCAG 1.4.10 reflow.

**[Accessibility C2]** — Meaning-bearing diagram semantics have no non-visual representation (EXPERIENCE.md:135–138, 64)
Guards, cycles, `GO TO` distinction, and read/write/call kinds are all specified as *visual* marks. The spine's own "never color alone" rule names shape and legend — still visual-only cues. Nothing states these semantics enter the accessible name. A screen-reader Nomsa cannot learn the TAXCALC call is guarded by `IF RUN-MODE = 'YR-END'` — the literal climax of Flow 2.
*Fix:* Add a rule to the Accessibility Floor: every meaning-bearing mark has a text equivalent in the accessible name/description ("calls TAXCALC, conditional: IF RUN-MODE = 'YR-END'"; "GO TO (unstructured jump)"; "writes EMPFILE"; "decision block, lines 214–220").

### High (9)

**[Token completeness + Accessibility H3]** — `charts-*` bindings have no fallback chain and theme inheritance carries no contrast contract (DESIGN.md:23, 28–33, 80–92)
An undefined CSS custom property resolves to nothing — the provenance tick, whose "absence is impossible by construction," silently vanishes in themes lacking chart tokens. Beyond that, delegating to the theme inherits its *choices*, not a contrast *contract*: `charts-*` carries no guarantee against `editorWidget-background`; `editorWarning-foreground` as label text is tuned for squiggles, frequently <4.5:1, exactly on the honest hole whose legibility is a brand commitment; the tick is a meaning-bearing graphic with no minimum size.
*Fix:* Declare CSS fallback chains for every `charts-*` binding; add a contrast-floor clause (≥3:1 marks, ≥4.5:1 text) with in-system fallbacks; give the tick a minimum rendered size; never set multi-word label text in `editorWarning-foreground`.

**[Adversarial H1]** — Edges and order across unparsed holes imply continuity the parser never verified (EXPERIENCE.md:138, 135, 137)
The spine specifies unparsed *nodes* thoroughly but never how *edges and order* behave across a hole. A flowchart drawing A → `‹unparsed›` → B asserts fall-through the parser never confirmed (the hole could contain a `GO TO` or an exit). So "mechanically derivable for *any* unit" is false precisely when a hole is present. Same for sequence: an ERROR region may contain CALLs, so "in source order" is silently incomplete. The view renders uniformly, and the uniformity is the dishonesty.
*Fix:* Specify hole-adjacent semantics per perspective — edges entering/leaving an unparsed block render in the unparsed treatment; never draw an A→B edge *around* a hole; sequence/call-flow render a marked gap in the order. Ban "drawing a verified-style edge whose derivation crosses an ERROR region."

**[Adversarial H2]** — Full v1 commitment pre-gate hides five renderer-scale builds (EXPERIENCE.md:120; PRD §1, FR-5, SM-2/OQ-1)
The memlog flags the expansion; the spine text commits without phasing. Quietly enormous, each a project: deterministic graph layout for 214 files with collapse and a layout toggle; a bespoke sequence renderer with guard/loop fragments; bidirectional editor selection sync (decorations, echo suppression, stale spans); module-reach reads/writes (needs SELECT/ASSIGN/FD data-item mapping through copybooks — the heaviest extraction in the bundle, sitting in one table row); trail restore across re-derives. None is needed to run the reliability gate, which needs extraction, provenance-enforcing primitives, one view, click-to-source.
*Fix:* Split v1 into named phases inside the spine. Phase A (gate-sufficient): dependency map + flowchart, popover, jump-to-source, unparsed rendering, gallery-as-command-list. Phase B (post-gate): sequence, module-reach, call-flow, trail, rail, Code sync, molded-perspective registration. Keep the perspective *model* fully committed so Phase B slots in without redesign — commit the shape fully, the surface incrementally.

**[Adversarial H3]** — Copybook expansion is load-bearing for every perspective and completely unspecified (EXPERIENCE.md:126–139)
COBOL `COPY` is textual inclusion, often with `REPLACING`; tree-sitter parses files as written. So: a paragraph PERFORMed in `PAYROLL.cbl` but defined in a copybook resolves nowhere; with `REPLACING`, identifiers in the copybook don't textually match the identifiers in use, so the provenance anchor for a replaced identifier points at a line that *doesn't contain the name shown on the node* — the click-through lands somewhere that doesn't say what the node says, breaking the core trust gesture. Module-reach through copybook-shared data items is the *main* reach mechanism in real COBOL and is unmentioned.
*Fix:* State the model: v1 renders as-written; cross-COPY references render as edges to the copybook node with both anchors; `REPLACING` cases note the inclusion plainly in the popover so click-through never surprises. If expansion is deferred, add it to the Boundary paragraph.

**[Adversarial H4]** — The provenance tick over-signals, and its "absence impossible" claim collides with unparsed nodes (DESIGN.md:187; EXPERIENCE.md:67, 113)
(a) DESIGN says the tick rides every element and its absence is impossible; the unparsed-node spec is silent on it. An ERROR region *has* a span, so it can carry one — but a green "verified" mark on a region labeled *unread* conflates "we located it" with "we read it." (b) The tick guarantees a span *exists*, not that it *supports the drawn claim*: a molded tool can emit a wrong edge whose provenance is a real but irrelevant line, passing the structural check and rendering with starter-grade visual trust.
*Fix:* State that the tick means "carries a source span" — nothing more; unparsed nodes carry it but unread status dominates visually. Purge "verified to source" in favor of "traces to `PAYROLL.cbl:214`". When the active lens is a molded tool, the header names it with one-click "view analyzer source," making trust-check #2 ambient rather than buried.

**[Accessibility H1]** — `Tab` consumed by the canvas is a keyboard trap and orphans the chrome (EXPERIENCE.md:95 vs 116)
If `Tab` walks nodes and edges, a 214-file graph puts hundreds of stops between the user and the perspective tabs, breadcrumb, and rail toggle — contradicting the "ordinary tab-stops" claim, and in the worst reading trapping the canvas.
*Fix:* Standard composite-widget pattern — canvas is **one** tab stop; arrows (plus type-ahead) move within it; `Tab`/`Shift+Tab` exit to the next control. State the full focus ring explicitly, including the Code-perspective split.

**[Accessibility H2]** — The trace outcome, the core trust action, is visual-only (EXPERIENCE.md:64, 94, 203)
The #1 trust check is expressed as paired *lighting* of element and editor line. Nothing specifies an announcement or focus outcome. The spine's own line — "a verifiability guarantee that only works with a pointer isn't a guarantee" — applies equally to vision.
*Fix:* Jump-to-source moves focus into the editor at the line; an in-canvas trace fires a live-region status ("Traced: CALL TAXCALC at PAYROLL.cbl:214"); the linked state is exposed as an `aria-*` state.

**[Accessibility H4]** — Keybinding portability and conflicts (EXPERIENCE.md:94, 97, 99)
Webviews receive raw DOM key events — VS Code keybindings don't apply, so `F12`/`Alt+arrows` must be re-implemented and cannot honor a user's *remapped* bindings without explicit work. `Alt+←` is not Go Back on macOS, silently breaking the muscle-memory rationale on one of three named platforms. `Esc` collides with NVDA/JAWS focus-mode conventions, where it may drill out a grain as a destructive surprise.
*Fix:* Every primitive is also a contributable, remappable, palette-visible VS Code command; document macOS defaults; announce each `Esc` stage so layered behavior is perceivable.

**[Adversarial M4, elevated]** — `F12` carries two behaviors and the trail forks VS Code's history (EXPERIENCE.md:94 vs 213, 97)
`F12` is specified as "jump to source" but Flow 2 shows it producing the synced Code split — two behaviors, undisambiguated. `Alt+←/→` are reassigned inside the webview, so the same gesture yields the view trail on canvas and VS Code's history in the editor: two interleaved histories sold as pure familiarity.
*Fix:* `F12` = jump (editor focus, no sync contract); entering the Code perspective = the synced split. State the two-history seam in Interaction Primitives so it's designed, not discovered.

### Medium (22)

**Rubric (7):** view header has a behavioral row but no DESIGN component spec (named "Legend/controls" there) · category legend is load-bearing for accessibility yet has no visual spec or behavioral row · `GO TO`/cycle/guard/loop-fragment marks are required distinctions with no visual spec · view gallery surface type undecided (native QuickPick vs webview) · no state for a scope with *nothing parseable* · no state for a stale open view after an edit (anchor can land on the wrong line — the trust destination lying) · GT rationale duplicated across Views & Perspectives and Inspiration.

**Adversarial (7):** M1 snapshot-vs-"as it is now" staleness undefined ("no re-parse" projects a snapshot while the Code split shows a live editor) · M2 availability rule filters grain/kind, not derivability, so the banned-list promise has no mechanism · M3 Call-flow's "what actually happens when this runs" is a runtime overclaim with no disclaimer, unlike Sequence · M5 "everything it reads, writes, and calls" is banned completeness framing · M6 "same code, same picture" smuggles in deterministic *layout* as an unowned requirement · M7 molded-perspective registration grows the contract surface before the gate that tests the contract · (M4 elevated to High above).

**Accessibility (8):** M1 DESIGN says the trace highlight "fades," EXPERIENCE says it persists — EXPERIENCE is correct · M2 popover "dismisses on moving away" fails WCAG 1.4.13 hoverable/persistent · M3 `Esc` stages and trail restores are unannounced, making the temporal-vs-structural model imperceptible non-visually · M4 "high-contrast honored automatically" is an overclaim (HC removes fills; ticks/category hues may flatten) · M5 no live-region strategy for the header's honesty statements — SR users get the picture without the caveats · M6 Code-split focus/sync behavior can steal focus or move the virtual cursor · M7 no zoom/reflow/legibility-floor/`prefers-reduced-motion` commitments · M8 molded tools inherit no accessibility contract.

### Low (14)

Flow 4 has no failure path · UJ-3/SM-3 showcase never acknowledged (no export affordance stated) · `anchor-popover` bypasses the `colors` map · `{spacing.1}`–`{spacing.8}` implies undefined `5`/`7` · component naming variance across spines (perspective tabs/switcher, unparsed node, view header) · sequence entry-point ask unspecified · molded-tool registration failure uncovered · slightly off `PRD FR-4` citation · EXPERIENCE section order differs from the example (cosmetic) · "same 12 modules" continuity copy can be false when only a subset is reachable · "tens of lines" is an unenforced promise · SR-hostile glyphs (`↵`, `›`, `‹ ›`) in announced text · sub-24px target sizes on canvas marks · editorial flourish in spec prose.

## Reviewer files

- `review-rubric.md` — coverage and contract fitness (1 critical, 1 high, 7 medium, 11 low)
- `review-a11y.md` — WCAG 2.2 AA floor for a webview graph product (2 critical, 4 high, 8 medium, 2 low)
- `review-adversarial.md` — trust-model and scope-honesty audit (2 critical, 4 high, 7 medium, 2 low)
