# Accessibility Review — unoriginally-deterministic UX Spine Pair

**Reviewed:** `DESIGN.md` + `EXPERIENCE.md` (2026-07-19 draft), accessibility lens only.
**Target floor:** WCAG 2.2 AA, applied as it reasonably applies to a VS Code webview extension (SC 1.1.1, 1.3.1, 1.4.3, 1.4.4, 1.4.10, 1.4.11, 1.4.13, 2.1.1, 2.1.2, 2.2.1, 2.4.3, 2.4.7, 2.5.8, 3.2.x, 4.1.2/4.1.3), plus VS Code-specific realities: webviews receive raw key events (editor keybindings do not apply inside them), and theme tokens carry **no contrast contract**.

**Overall:** The spine's accessibility *intent* is unusually strong for a graph product — keyboard parity, announced transitions, popover-not-hover-only, color-never-sole-cue are all stated as invariants. But the commitments are stated at the level of outcomes, not mechanisms, and for a node-edge canvas the mechanism *is* the hard part. As written, the two critical gaps (no accessible-structure model for the canvas; no non-visual representation for meaning-bearing diagram semantics) mean the SR commitments in `EXPERIENCE.md:110-116` are not yet implementable.

---

## CRITICAL

### C1. No accessible-structure model or linearized alternative for the graph canvas
**Location:** `EXPERIENCE.md:110-116` (Accessibility Floor), `EXPERIENCE.md:95` (arrow/Tab selection), `EXPERIENCE.md:118-147` (Views & Perspectives); `DESIGN.md:165` (canvas as primary region).
**Floor:** WCAG 1.3.1 Info and Relationships, 1.1.1 Non-text Content, 4.1.2 Name/Role/Value.

The spine commits to "every node/edge reachable" and meaningful SR announcement, but never specifies *what the accessibility tree contains*. A rendered node-edge canvas (SVG or `<canvas>`) exposes nothing by default; edges in particular have no native focusable/announcable representation. The one linear surface the spine offers — "the opt-in rail gives a fully linear, screen-reader-friendly pass over every anchor" (`EXPERIENCE.md:115`) — lists **source anchors**, not structure: it cannot convey topology (who calls whom, edge direction, edge kind, fan-in/fan-out), which is the entire content of the view. An SR user can audit provenance but cannot *comprehend the view*, which inverts the product's purpose.

Also note the tension: the rail is "never auto-opens... an audit surface" (`EXPERIENCE.md:34,167` in DESIGN) yet is simultaneously positioned as the SR-friendly pass. If the rail is the accessible alternative, it is doing double duty it was not designed for.

**Fix (spine-level commitments to add):**
1. Commit to an explicit ARIA model: canvas as a single tab stop (`role="application"` or grid/treegrid) with **roving tabindex**; every node and edge is a real focusable element with an accessible name (identifier + kind + grain), state (selected, unparsed, traced), and relationship description ("calls TAXCALC, called by 2").
2. Commit to a **per-perspective linearized equivalent** (a structure list/table, not the anchor rail): dependency map → adjacency list ("PAYROLL: calls TAXCALC, COPY WSCOMMON…"); sequence → ordered message list with guards and loop fragments; module-reach → depth-ordered read/write/call list; call-flow → execution-ordered paragraph list with cycle markers; flowchart → nested block outline (branch/loop structure as list nesting). This can be one component ("Structure view" toggle, sibling to the rail) and doubles as the 1.4.10 reflow answer (C-ref M7).
3. Specify keyboard graph traversal beyond "arrow keys": arrows for spatial/sibling movement is ambiguous in a 2D graph — commit to semantic traversal (e.g., arrows follow edges from the focused node; Home/End for entry/exit nodes) and document it.

### C2. Meaning-bearing diagram semantics have no non-visual representation
**Location:** `EXPERIENCE.md:135` (guards "carry their guard visibly", loop "fragments"), `EXPERIENCE.md:137` (cycles "marked explicitly", `GO TO` edges "visually distinguished"), `EXPERIENCE.md:138` (block kinds, `GO TO` distinction), `EXPERIENCE.md:64` (edge kind = category color + legend), `EXPERIENCE.md:136` (read/write/call as "first-class" edge kinds); `DESIGN.md:193` (sequence/flowchart marks reuse node/edge tokens).
**Floor:** WCAG 1.1.1, 1.3.1.

Every one of these is specified as a *visual* mark: guards shown on messages, cycles marked, `GO TO` distinguished, writes vs reads distinguished. The spine's own rule — "category color is never the only cue" (`EXPERIENCE.md:111`) — names shape/label/legend as the pairing, but shape and legend are still visual-only cues; nothing states these semantics enter the accessible name/description or the announcements. "Announces selection meaningfully" (`EXPERIENCE.md:113`) gives one node example and never covers edges, guards, cycles, fragments, or block kinds. A screen-reader Nomsa cannot learn that the TAXCALC call is guarded by `IF RUN-MODE = 'YR-END'` — the exact climax of Flow 2 (`EXPERIENCE.md:210`).

**Fix:** Extend the Accessibility Floor with an explicit rule: *every meaning-bearing mark has a text equivalent in the element's accessible name or description* — edge: "calls TAXCALC, conditional: IF RUN-MODE = 'YR-END'"; cycle: "PERFORM loop back to 2000-READ"; `GO TO` edge announced as "GO TO (unstructured jump)"; reach edges as "writes EMPFILE"; flowchart blocks as "decision block, lines 214–220". Add sequence/flowchart-specific announcement examples alongside the existing node example at `EXPERIENCE.md:113`.

---

## HIGH

### H1. `Tab` consumed by canvas = keyboard trap / unreachable chrome
**Location:** `EXPERIENCE.md:95` ("Arrow keys / `Tab` → move selection across nodes and edges"), `EXPERIENCE.md:114` ("`Tab` order follows the view's semantic structure"), `EXPERIENCE.md:116` (switcher and breadcrumb are "ordinary tab-stops").
**Floor:** WCAG 2.1.2 No Keyboard Trap, 2.4.3 Focus Order.

If `Tab` walks nodes and edges inside the canvas, a large graph (214 files, `EXPERIENCE.md:200`) puts hundreds of tab stops between the user and the header controls — the perspective tabs, breadcrumb, and rail toggle become effectively unreachable, and in the worst reading the canvas is a trap. This directly contradicts the "ordinary tab-stops" claim for the switcher.

**Fix:** Amend `EXPERIENCE.md:95` to the standard composite-widget pattern: the canvas is **one** tab stop; arrows (plus type-ahead find) move within it; `Tab`/`Shift+Tab` exit to the next chrome control. State the full focus ring explicitly: header controls → canvas → (rail when open) → and, in Code perspective, how focus moves between canvas and editor split (see M6).

### H2. The trace outcome — the core trust action — is visual-only
**Location:** `EXPERIENCE.md:64` (click edge → element + line light), `EXPERIENCE.md:82` (trace target treatment), `EXPERIENCE.md:94` (`F12`), `EXPERIENCE.md:203` (Flow 1 climax: "the edge and a line in the editor light together"); `DESIGN.md:144` (trace-active pair lighting).
**Floor:** WCAG 4.1.3 Status Messages, 1.3.1.

The spine's #1 trust check is expressed as a paired *lighting* of element and editor line. Nothing specifies an announcement or focus outcome: does `F12` move focus to the editor at the line? Does a click-trace (focus stays on canvas) announce "traced to PAYROLL.cbl:214"? The Accessibility Floor covers selection, drills, and perspective switches — but not the trace, which the spine itself calls "the core trust action" (`EXPERIENCE.md:94`). The line "a verifiability guarantee that only works with a pointer isn't a guarantee" (`EXPERIENCE.md:115`) applies equally: a guarantee that only works with *vision* isn't one either.

**Fix:** Add to the Accessibility Floor: `F12`/anchor-click moves focus into the editor at the highlighted line (VS Code then announces it natively); an in-canvas trace fires a live-region status ("Traced: CALL TAXCALC at PAYROLL.cbl:214") and the popover content is reachable from the focused element. Specify that the "stays visibly linked" state (`EXPERIENCE.md:82`) is also exposed as an `aria-*` state on the element.

### H3. Theme inheritance does not actually guarantee contrast for the compositions this product makes
**Location:** `DESIGN.md:23-33` (charts-* tokens for provenance + category ramp), `DESIGN.md:80-84` (unparsed foreground = `editorWarning-foreground` used as *text* color), `DESIGN.md:86-92` (edge stroke = `descriptionForeground`; provenance tick = charts-green), `DESIGN.md:139` ("that is what keeps the extension native... without a second palette"), `EXPERIENCE.md:110` ("honored automatically").
**Floor:** WCAG 1.4.3 Contrast (Minimum), 1.4.11 Non-text Contrast.

Delegating to the theme inherits the theme's *choices*, not a contrast *contract*. Specific breaks:
- **`charts-*` tokens carry no contrast guarantee** against `editorWidget-background` or `editor-background` — VS Code tunes them for chart marks on the editor surface in *default* themes only; thousands of custom themes set them arbitrarily (or not at all, falling back to defaults that may clash with a custom background). A charts-yellow category edge on a light custom theme can easily be <3:1 (1.4.11 fail on a meaning-bearing graphic).
- **`editorWarning-foreground` as label text** (`DESIGN.md:84`: unparsed node foreground) — warning hues are tuned for squiggles, not body text; frequently <4.5:1 (1.4.3 fail) exactly on the "honest hole" whose legibility is a brand commitment.
- **The provenance tick** is "a small dot/underline" (`DESIGN.md:187`) — a meaning-bearing graphic with no minimum size and no contrast floor. At dot size, even a compliant hue is hard to perceive; the mark that carries the entire trust thesis is the most fragile pixel on screen.
- **Edge base stroke** = `descriptionForeground` (`DESIGN.md:86`), a deliberately-muted token, at presumably 1px on a dense canvas.

**Fix:** Add a "contrast floor" clause to `DESIGN.md.Colors`: at render time, verify each composed pair (mark color × actual surface) against 3:1 (marks) / 4.5:1 (text); on failure, fall back in-system (swap to `foreground`/`editorWarning-border`, add a 1px `contrastBorder` outline, or thicken the mark) rather than hand-tuning hexes. Give the provenance tick a minimum rendered size (≥6×6px dot or 2px underline across the label width) and pair it with a non-color form (the tick *is* universal, so its form matters more than its hue — see also M4 for HC). Never rely on `editorWarning-foreground` alone for multi-word label text; reserve it for the border/badge and set label text in `foreground`.

### H4. Keybinding portability and conflicts: `Alt+←/→`, `F12`, `Esc`
**Location:** `EXPERIENCE.md:94` (`F12`), `EXPERIENCE.md:97` (`Alt+←`/`Alt+→` "VS Code's own Go Back/Forward keys"), `EXPERIENCE.md:99` (`Esc` layered retreat), `EXPERIENCE.md:124` (trail keys).
**Floor:** WCAG 2.1.1; plus the spine's own muscle-memory claim.

- **Webviews don't get VS Code keybindings.** Inside a webview the extension receives raw DOM key events; `F12`/`Alt+arrows` must be re-implemented and — more importantly — must *not* be advertised as "VS Code's own keys" without honoring the user's **remapped** bindings. A user who remapped Go Back (or uses a keyboard layout where `Alt+←` types a character) loses the trail with no discoverable alternative. No command-palette or menu fallback is specified for drill/back/trace.
- **`Alt+←` is not Go Back on macOS** (default is `Ctrl+-`); the muscle-memory rationale at `EXPERIENCE.md:97` silently breaks on one of the three named platforms (`EXPERIENCE.md:191`).
- **`Esc` collides with screen-reader conventions**: NVDA/JAWS use `Esc` to leave forms/focus mode; an SR user pressing `Esc` may drill out a grain instead of (or in addition to) switching SR mode — a destructive surprise given Esc's structural-retreat meaning (see also M3).

**Fix:** Specify that every primitive is also a **VS Code command** (contributable, remappable, palette-visible: "Comprehension View: Drill In / Drill Out / Go Back / Jump to Source / Toggle Rail / Open Structure View"), with webview keys resolved from the user's actual keybindings where feasible; document macOS defaults; make drill-out require an explicit key distinct from popover-dismiss when a popover is not open in SR-relevant modes, or at minimum announce each Esc stage (M3) so the layered behavior is perceivable.

---

## MEDIUM

### M1. Spine contradiction: trace highlight "fades" vs "stays until the user acts"
**Location:** `DESIGN.md:144` ("Lights the element and the editor line as a pair, **then fades**") vs `EXPERIENCE.md:82` ("stays visibly linked... **until the user acts again**").
**Floor:** WCAG 2.2.1 Timing Adjustable; consistency.
A timed fade on a meaning-bearing state is an accessibility failure for low-vision and SR users who reach the state late; the EXPERIENCE wording is the correct one. **Fix:** Amend `DESIGN.md:144` to match EXPERIENCE (persistent until next user action); any fade is decorative-only and disabled under reduced motion (M7).

### M2. Hover popover behavior vs WCAG 1.4.13 unspecified
**Location:** `EXPERIENCE.md:65` ("Opens on hover, pins on select, **dismisses on Esc or moving away**"); `DESIGN.md:98-104,166`.
**Floor:** WCAG 1.4.13 Content on Hover or Focus.
"Dismisses on moving away" as written means the pointer can never travel from element to popover to click the `file:line` chip. 1.4.13 requires hover content to be **hoverable**, **persistent**, and **dismissible without moving the pointer**. Keyboard-select-pins largely rescues this, but the hover path must comply too. **Fix:** Specify: popover remains while pointer is over element *or* popover; `Esc` dismisses without pointer movement; hover popover never occludes the hovered element's tick or label.

### M3. Esc stages, drill-out, and trail restores are not announced
**Location:** `EXPERIENCE.md:99` (three-stage Esc), `EXPERIENCE.md:97,124` (Back/Forward restores scope+perspective+selection), `EXPERIENCE.md:116` (announces drill *in* and perspective switch only).
**Floor:** WCAG 4.1.3; cognitive load.
The layered Esc is learnable *if perceivable*: a sighted user sees which layer consumed the keypress; an SR user hears nothing and cannot tell "popover closed" from "selection cleared" from "drilled out." Same for `Alt+←`: restoring "exact prior step" two grains and a perspective away is a large context switch with no specified announcement. The temporal-vs-structural distinction (`EXPERIENCE.md:124`) — the spine's subtlest model — is exactly the kind of thing that stays learnable only when every transition is narrated. **Fix:** Extend `EXPERIENCE.md:116`: announce each Esc stage ("Popover closed" / "Selection cleared" / "Drilled out to payroll — dependency map, 12 modules") and each trail restore ("Back: sequence, PAYROLL selected, system grain").

### M4. "High-contrast honored automatically" is an overclaim
**Location:** `EXPERIENCE.md:110`; `DESIGN.md:129` ("same light/dark/high-contrast the user already chose"); `DESIGN.md:17,79` (selection = `editor-selectionBackground` fill).
**Floor:** WCAG 1.4.11; VS Code HC conventions.
HC themes work by *removing* fills and relying on `contrastBorder`/`contrastActiveBorder`; several tokens the spine leans on (selection fills, hatch fills, charts hues) are muted or transparent in HC. Node-selected = focus-ring border survives; but category color, the green tick, and trace-active lighting may all flatten. Nothing commits to an HC-specific verification or to using `contrastBorder` where VS Code widgets would. **Fix:** Add an HC clause to `DESIGN.md.Colors`: in HC themes, every state distinction must survive on border/pattern/text alone (selection = double border weight; category = per-kind node shape or glyph — which C2's text equivalents also require; trace = thickened stroke + marker); QA pass on `Default High Contrast` light and dark named as a release gate.

### M5. No live-region strategy for state changes the spine calls "plainly stated"
**Location:** `EXPERIENCE.md:78` (parse progress), `EXPERIENCE.md:79` (hole count in header), `EXPERIENCE.md:100` (filter/collapse "says what it hid"), `EXPERIENCE.md:134,136` (collapse/depth statements), `EXPERIENCE.md:70` (continuity line "Sequence — same 12 modules").
**Floor:** WCAG 4.1.3.
All of these honesty statements are header *text*; none is specified as a status announcement, so an SR user gets the picture without the caveats — the exact dishonesty-by-omission the product refuses visually. **Fix:** One rule in the Accessibility Floor: every header state/continuity/hiding statement is a polite live-region update at the moment it changes ("3 regions unparsed — shown, unread"; "Collapsed subtree: 14 nodes hidden"; "Reach depth 2 — 3 further hops collapsed").

### M6. Code-perspective editor split: focus and sync behavior unspecified
**Location:** `EXPERIENCE.md:139` ("selection synced both ways"), `EXPERIENCE.md:170` (DESIGN: only split), `EXPERIENCE.md:213` (Flow 2 step 5).
**Floor:** WCAG 3.2.1/3.2.2 On Focus/Input, 2.4.3.
Two-way selection sync can steal focus or move the SR virtual cursor unexpectedly (editing code scrolls/re-selects the canvas; selecting a block yanks the editor). **Fix:** Specify: sync updates selection/scroll but **never moves keyboard focus** across the split; a deliberate key (`F12` canvas→editor; a named command editor→canvas) transfers focus; canvas announces sync-driven selection changes passively.

### M7. Zoom, reflow, density, and reduced motion
**Location:** `EXPERIENCE.md:188-189` (narrow groups, very large graphs, fit-to-canvas), `DESIGN.md:163` ("density is a feature"); no motion/zoom clause anywhere.
**Floor:** WCAG 1.4.4, 1.4.10, 1.4.12, 2.3.3/animation.
The graph itself is legitimately exempt from reflow (2D-layout exception), **provided** an alternative presents the content linearly — which is C1's structure view; the spine currently has no exemption-qualifying alternative. Popover, rail, header, and gallery are *not* exempt and must reflow at 400%/text-spacing overrides (monospace `file:line` chips truncating would break the copy-anchor primitive, `EXPERIENCE.md:102`). Fit-to-canvas on 214 nodes yields sub-legible labels by design; "collapse over pan" helps but no minimum label size or auto-collapse threshold is stated. No `prefers-reduced-motion` commitment exists despite pan/zoom/layout-toggle transitions and the trace fade (M1). **Fix:** Add to Responsive & Platform: chrome surfaces reflow at 400% without horizontal scroll and respect editor font-size/zoom; define a legibility floor (below N px effective label size, auto-collapse and say so); all viewport animation (fit, layout re-arrange, fades) is disabled or instant under `prefers-reduced-motion` — an easy commitment for a brand that already bans celebratory motion (`DESIGN.md:135`).

### M8. Molded tools inherit no accessibility contract
**Location:** `EXPERIENCE.md:143` (molded tools register as perspectives), `EXPERIENCE.md:167` ("provenance is a required argument of every drawing primitive").
**Floor:** whole-product AA.
The contract makes provenance structurally unfakeable — but says nothing about accessible names. A molded perspective drawing unlabeled marks silently breaks the SR floor for exactly the tools the product hopes proliferate. **Fix:** Mirror the provenance move: the drawing primitives take a required accessible-name/kind argument (or derive one from the required source span), so a molded tool *cannot* draw an unannouncable element. One sentence in Molding Handoff + the contract-docs requirement (feeds PRD OQ-2 alongside the gtView item).

---

## LOW

### L1. Glyphs in announced/labelled text: `↵`, `›`, `‹unparsed: 3 lines›`
**Location:** `EXPERIENCE.md:65` (drill preview "`↵ Flowchart · 9 blocks`"), `EXPERIENCE.md:71` + `DESIGN.md:192` (breadcrumb `›` separator), `DESIGN.md:185` (`‹unparsed: 3 lines›`).
Screen readers render these as "return symbol", "single right-pointing angle quotation mark", or silence, depending on verbosity. **Fix:** visual glyphs stay; accessible names read "Enter: open flowchart, 9 blocks", breadcrumb separators are `aria-hidden` with list semantics carrying order, unparsed label announced as "unparsed region, 3 lines".

### L2. Target sizes in a dense canvas
**Location:** `DESIGN.md:163` (density mandate), `EXPERIENCE.md:189` (fit-to-canvas defaults); tick dot `DESIGN.md:60,187`; anchor chips `DESIGN.md:93-97`.
**Floor:** WCAG 2.5.8 Target Size (Minimum), 24×24px.
Fit-to-canvas nodes, 1px edges as click-to-trace targets, and the tick dot can all land under 24px. Keyboard equivalence helps users but does not satisfy the SC by itself (the equivalent-control exception needs an on-screen equivalent — the rail rows and C1 structure view can be that equivalent if committed). **Fix:** give edges a fat invisible hit area (≥8px stroke hit target), chips/rows ≥24px height, and note the rail/structure view as the enlarged equivalent for sub-24px canvas marks.

---

## Summary counts
- Critical: 2 (C1 canvas structure model, C2 non-visual diagram semantics)
- High: 4 (H1 Tab trap, H2 trace announcement, H3 contrast delegation, H4 keybinding portability)
- Medium: 8 (M1–M8)
- Low: 2 (L1–L2)

The spine's stated floor (`EXPERIENCE.md:106-116`) is the right floor; the gap is that five of its seven bullets currently name outcomes with no committed mechanism. Fixing C1/C2 + H1/H2 makes the floor real; H3/M4 make "theme inheritance = accessibility" an earned claim instead of a delegation.
