---
name: unoriginally-deterministic
description: Visual identity for a VS Code extension that draws deterministic, code-derived comprehension views. Inherits the user's active VS Code color theme; this DESIGN.md specifies only the brand-layer delta — the provenance, trace, unparsed and unresolved semantics — plus the contrast floor beneath that inheritance.
status: final
updated: 2026-07-20
colors:
  # THEME-INHERITED. All base surface/text/chrome tokens resolve to the user's
  # active VS Code color theme at render time (light / dark / high-contrast).
  # Values are the real mechanism: CSS custom properties VS Code injects into
  # every webview. Never hardcode a hex where a --vscode-* token exists.
  background: 'var(--vscode-editor-background)'
  foreground: 'var(--vscode-editor-foreground)'
  muted: 'var(--vscode-descriptionForeground)'
  border: 'var(--vscode-panel-border)'
  node-surface: 'var(--vscode-editorWidget-background)'
  node-border: 'var(--vscode-widget-border)'
  selection: 'var(--vscode-editor-selectionBackground)'
  focus-ring: 'var(--vscode-focusBorder)'
  link: 'var(--vscode-textLink-foreground)'
  link-active: 'var(--vscode-textLink-activeForeground)'
  popover-background: 'var(--vscode-editorHoverWidget-background)'
  popover-border: 'var(--vscode-editorHoverWidget-border)'
  popover-foreground: 'var(--vscode-editorHoverWidget-foreground)'
  # SEMANTIC BRAND LAYER — meaning-carrying, still theme-derived. Every binding
  # declares a FALLBACK CHAIN: not all themes define charts-*, and an undefined
  # custom property resolves to nothing — which would silently erase the very
  # marks that carry the trust model. Never let a meaning-bearing mark vanish.
  provenance: 'var(--vscode-charts-green, var(--vscode-testing-iconPassed, currentColor))'  # "carries a source span" [ASSUMPTION: green]
  trace-active: 'var(--vscode-editorLink-activeForeground, var(--vscode-textLink-activeForeground, currentColor))'
  unparsed: 'var(--vscode-editorWarning-foreground, var(--vscode-list-warningForeground, currentColor))'
  unresolved: 'var(--vscode-charts-orange, var(--vscode-editorWarning-foreground, currentColor))'  # dynamic dispatch: target not statically resolvable
  # GRAPH CATEGORY RAMP — for distinguishing node/edge kinds. Always paired with a
  # non-colour cue (line style / shape), so a flattened ramp degrades but never lies.
  cat-1: 'var(--vscode-charts-blue, var(--vscode-textLink-foreground, currentColor))'
  cat-2: 'var(--vscode-charts-purple, currentColor)'
  cat-3: 'var(--vscode-charts-orange, currentColor)'
  cat-4: 'var(--vscode-charts-yellow, currentColor)'
  cat-5: 'var(--vscode-charts-red, var(--vscode-errorForeground, currentColor))'
  cat-6: 'var(--vscode-charts-foreground, var(--vscode-foreground, currentColor))'
typography:
  # THEME-INHERITED. Two ramps, both from VS Code. UI font for chrome/labels;
  # editor (monospace) font for anything that is literally code.
  ui:
    fontFamily: 'var(--vscode-font-family)'
    fontSize: 'var(--vscode-font-size)'
    fontWeight: 'var(--vscode-font-weight)'
  ui-strong:
    fontFamily: 'var(--vscode-font-family)'
    fontWeight: '600'
  code:
    # Node labels that are identifiers, source-span coordinates (file:line),
    # primitive names, ERROR spans — anything the user could grep for.
    fontFamily: 'var(--vscode-editor-font-family)'
    fontSize: 'var(--vscode-editor-font-size)'
  wordmark:
    # The only deliberately-branded type moment. Lowercase, monospace, no styling
    # flourish — the "deterministic" wink is that it looks like a symbol name.
    fontFamily: 'var(--vscode-editor-font-family)'
    fontWeight: '400'
    letterSpacing: '0'
rounded:
  # VS Code chrome is nearly square. Match it — sharp reads "instrument," not "app."
  none: '0'
  sm: '2px'
  md: '3px'
  full: '9999px'   # dots/badges only
spacing:
  # 4-based scale, aligned to VS Code's own density. The editor is dense on purpose;
  # comprehension views must not feel roomier than the code they describe.
  '1': '4px'
  '2': '8px'
  '3': '12px'
  '4': '16px'
  '6': '24px'
  '8': '32px'
components:
  node:
    background: '{colors.node-surface}'
    border: '{colors.node-border}'
    foreground: '{colors.foreground}'
    radius: '{rounded.sm}'
    label-font: '{typography.code.fontFamily}'
  node-selected:
    border: '{colors.focus-ring}'
    background: '{colors.selection}'
  node-unparsed:
    # Honest hole. Dashed border + hatch fill. Reads "known unknown."
    # Label text uses `foreground` — editorWarning-* is tuned for squiggles, not
    # body text, and is frequently under 4.5:1. The hue rides the border/badge.
    border: '{colors.unparsed}'
    border-style: 'dashed'
    foreground: '{colors.foreground}'
    badge-color: '{colors.unparsed}'
  node-unresolved:
    # Dynamic-dispatch target. Parse SUCCEEDED — only the target is unknown,
    # so this must never be confusable with node-unparsed.
    border: '{colors.unresolved}'
    border-style: 'dashed'
    foreground: '{colors.foreground}'
    badge-color: '{colors.unresolved}'
  node-out-of-snapshot:
    # Statically named, not extracted. Neither a parse failure nor a dynamic
    # target — the boundary of what was looked at, drawn rather than implied.
    border: '{colors.muted}'
    border-style: 'dotted'
    foreground: '{colors.muted}'
  node-unresolved-inclusion:
    # A named include whose source could not be located. Deliberately the union
    # of two families: DOTTED like out-of-snapshot (named but absent) + HATCH
    # like unparsed (interior unknown) — because that is exactly what it is.
    # Distinct from both: the parser did not choke, and this is not the benign
    # edge of scope — an unbounded amount of THIS unit is simply missing.
    border: '{colors.unparsed}'
    border-style: 'dotted'
    fill: 'hatch'
    foreground: '{colors.foreground}'
    badge-color: '{colors.unparsed}'
  edge:
    stroke: '{colors.muted}'
    hit-area: '8px'        # invisible fat hit target; the visible stroke is 1px
  edge-traced:
    stroke: '{colors.trace-active}'
  edge-unparsed:
    # Crosses or touches an ERROR region: "control may flow here; the hole was not read."
    stroke: '{colors.unparsed}'
    stroke-style: 'dashed'
  edge-jump:
    # Unstructured jump (goto/fall-through). Distinguished by SHAPE, not colour alone.
    stroke: '{colors.cat-5}'
    stroke-style: 'dashed'
    arrowhead: 'open'
  provenance-tick:
    # The signature motif: a small source-anchored marker on every drawn element
    # confirming it carries a source span. Absent = the element could not exist.
    color: '{colors.provenance}'
    min-size: '6px'        # meaning-bearing mark: never smaller, at any zoom
  view-header:
    background: '{colors.node-surface}'
    border: '{colors.border}'
    foreground: '{colors.foreground}'
  legend-entry:
    # Colour swatch + line-style sample + label. The non-colour cue is mandatory.
    font: '{typography.ui.fontFamily}'
    foreground: '{colors.muted}'
  source-anchor:
    # The file:line chip. Appears in the anchor popover and in the provenance rail.
    foreground: '{colors.link}'
    font: '{typography.code.fontFamily}'
    radius: '{rounded.sm}'
  anchor-popover:
    # Primary provenance surface: attached to the element on hover/select.
    # Follows VS Code's own hover-widget conventions.
    background: '{colors.popover-background}'
    border: '{colors.popover-border}'
    foreground: '{colors.popover-foreground}'
    radius: '{rounded.md}'
  provenance-rail:
    # Secondary, opt-in. Only for auditing many elements at once.
    background: '{colors.node-surface}'
    border: '{colors.border}'
  perspective-tab:
    # Header tabs switching the projection of the current scope. Follows VS Code
    # panel-tab conventions: quiet inactive, underlined active.
    foreground: '{colors.muted}'
    active-foreground: '{colors.foreground}'
    active-border: '{colors.focus-ring}'
  breadcrumb:
    # The grain path (system › unit › callable). Segments are source identifiers,
    # so they set in the code ramp.
    font: '{typography.code.fontFamily}'
    foreground: '{colors.muted}'
    current-foreground: '{colors.foreground}'
---

## Brand & Style

unoriginally-deterministic is a VS Code extension whose tools draw comprehension views — dependency maps, call-flows, structure — straight from code, deterministically. The name is a deliberate wink: **unoriginal ideas, deterministic execution**. The visual identity follows the product's one non-negotiable value — *honesty* — and its one posture — *this is an instrument, not a product*.

Three commitments drive every visual decision:

1. **It belongs in the editor, not on top of it.** The extension inherits the user's active VS Code color theme wholesale. A view drawn from code should look like it was always part of the workspace — same background, same fonts, same density, same light/dark/high-contrast the user already chose. We add almost no color of our own. Imposing a brand palette on someone's editor would be the visual equivalent of the overclaim this product refuses to make.

2. **Trust is shown, never asserted.** The single custom motif is the **provenance tick** — a small, source-derived marker that rides on every drawn node and edge, confirming it traces to a real line. It is not decoration; it is the structural guarantee made visible (no source → no element → nothing to mark). The one thing we spend brand equity on is making *verifiability legible*.

3. **Holes are honest, not errors.** When the parser can't read a construct, the region renders as a visibly-marked **unparsed** node — dashed, hatched, in the theme's warning hue — never omitted, never guessed. It must read as *"known unknown, expected"*, not *"something broke."* A view with honest holes is the product working correctly.

The overall register is dry, dense, and instrument-grade. No gradients, no illustration, no celebratory motion, no marketing gloss. If a screen looks like it's trying to impress you, it's off-brand.

## Colors

Almost every color is inherited from the user's VS Code theme via `--vscode-*` tokens (`{colors.background}`, `{colors.foreground}`, `{colors.muted}`, `{colors.border}`, and the widget/selection/focus family). We never hardcode a hex where a theme token exists — that is what keeps the extension native across every theme and both light and dark without a second palette.

The brand layer is four **semantic** colors, and they are the entire thesis rendered:

- **Provenance (`{colors.provenance}`)** `[ASSUMPTION: green chosen for "grounded"]` — marks that a drawn element **carries a source span**. That is its whole claim: not that the element is correct, not that the span was read. Used only on the provenance tick. Never for generic success, never for chrome. Copy near it says "traces to" — never "verified".
- **Trace-active (`{colors.trace-active}`)** — links a drawn element to its exact source line during a click-to-source trace. Lights the element and the editor line as a pair and **persists until the user's next action**; it does not fade on a timer. A meaning-bearing state that expires on its own is a state the user can miss.
- **Unparsed (`{colors.unparsed}`)** — the honest-hole hue for tree-sitter ERROR regions. Deliberately *warning*, not *error/destructive*: an unparsed region is expected, correct behaviour, not a failure. Carried on the border and badge, never on multi-word label text.
- **Unresolved (`{colors.unresolved}`)** — an invocation whose target isn't statically resolvable (dynamic dispatch). Distinct from unparsed on purpose: **the parse succeeded**; only the target is unknown. These are different kinds of not-knowing and must never be confusable.

For distinguishing kinds of nodes or edges, use the **graph category ramp** (`{colors.cat-1}`…`{colors.cat-6}`). Category colour is meaning-bearing and must **always** be paired with a non-colour cue (line style, shape, legend entry).

### Contrast floor

Theme inheritance buys the user's *choices*, not a contrast *contract* — `charts-*` tokens carry no guarantee against any given surface, and thousands of custom themes set them arbitrarily or not at all. So inheritance is the default, and this is the floor beneath it:

- Every binding declares a **fallback chain** ending in `currentColor`, so an undefined token degrades instead of erasing the mark.
- At render time, verify each composed pair (mark colour × its actual surface) against **3:1 for marks** and **4.5:1 for text**. On failure, fall back *in-system* — swap to `{colors.foreground}`, add a `contrastBorder` outline, or thicken the mark. Never hand-tune a hex.
- The **provenance tick** has a minimum rendered size (`{components.provenance-tick}` `min-size`), at every zoom level. The mark carrying the entire trust thesis must not be the most fragile pixel on screen.
- **High-contrast themes** work by *removing* fills and leaning on borders. So every state distinction must survive on border, line style, or text alone: selection reads as a doubled border, category as line style or shape, trace as a thickened stroke. HC is a supported target to be checked, not a claim to be assumed.

Avoid: any brand hex outside the theme system; colour as pure decoration; red for unparsed regions (it says "broken"); using the same treatment for unparsed and unresolved; more than the six-category ramp in one view (past six the view is too dense — mold a narrower tool).

## Typography

Two ramps, both inherited from VS Code, split by a single rule: **if it's code, it's monospace.**

- **UI ramp (`{typography.ui}`)** — VS Code's UI font, for chrome, panel titles, legends, prose, buttons, empty-state copy.
- **Code ramp (`{typography.code}`)** — VS Code's editor (monospace) font, for anything the user could select and grep: node labels that are identifiers, source-span coordinates (`PAYROLL.cbl:214`), primitive/API names, and unparsed spans. Monospace here is not stylistic — it signals "this is a literal string from your code," reinforcing fidelity.
- **Wordmark (`{typography.wordmark}`)** — the one branded moment: `unoriginally-deterministic` set lowercase in the editor monospace, unstyled, so it reads like a symbol name. Appears in the view header and the marketplace card, nowhere else.

No custom fonts ship. The extension has no font opinion the user's theme doesn't already hold.

## Layout & Spacing

A 4-based scale (`{spacing.1}`…`{spacing.8}` — 4/8/12/16/24/32) aligned to VS Code's own density. Comprehension views must never feel roomier than the code they describe — density is a feature; it keeps the whole-system picture on one screen so the user can drop into the one part that matters.

- **View surface:** a VS Code Webview panel. Fills the editor group; the graph/tree canvas is the primary region.
- **Anchor popover:** the primary provenance surface — attached to the element on hover/select, sized to its content, following VS Code's own hover-widget geometry. Costs the canvas nothing.
- **Provenance rail:** a slim, opt-in right-side rail listing source anchors across many elements. **Off by default and never auto-opens** — it is an audit surface, not a companion pane. The canvas owns the space unless the user asks otherwise.
- **View header:** a compact top strip (wordmark, perspective switcher, grain breadcrumb, category legend, layout/filter controls, rail toggle, honest counts). Never a heavy toolbar. Spec'd as `{components.view-header}`.

Single-canvas layout; no multi-pane dashboards. One canvas shows one scope through one perspective at a time — perspectives switch *in place* (EXPERIENCE.md owns the model). The only split is the Code perspective, which opens VS Code's **native editor** beside the canvas; the toolkit never re-renders source inside a webview.

## Elevation & Depth

Inherited from VS Code's widget conventions — flat surfaces, a single subtle `{colors.border}` to separate node from canvas and rail from canvas. No drop shadows as hierarchy, no floating cards, no layered z-depth beyond VS Code's own hover/popover. Depth in this product is *semantic* (drilling from system → module → line), never decorative.

## Shapes

Near-square, matching VS Code chrome: `{rounded.sm}` (2px) on nodes and chips, `{rounded.md}` (3px) on the few panels that need it, square (`{rounded.none}`) canvas edges. `{rounded.full}` is reserved for the provenance tick dot and count badges only. Sharpness reads "instrument"; rounded corners would read "consumer app" and undercut the trust posture.

## Components

Base interactive controls (buttons, inputs, dropdowns, checkboxes, tooltips, panel chrome) inherit VS Code Webview UI conventions and theme tokens as-is `[ASSUMPTION: aligning to @vscode/webview-ui-toolkit or equivalent theme-token styling]`. The contract: don't restyle VS Code's controls. Custom components are only the ones that render *the code-derived view itself*:

- **Node (`{components.node}`)** — a drawn element at any grain (compilation unit, callable, flow block). Monospace label = the source identifier. Carries a **provenance tick**. Selected state uses `{components.node-selected}` (focus-ring border + selection fill).
- **Unparsed node (`{components.node-unparsed}`)** — a tree-sitter ERROR region. Dashed `{colors.unparsed}` border, hatch fill, badge in the unparsed hue, label reading e.g. `‹unparsed: 3 lines›` set in `{colors.foreground}` for legibility. Rendered in place, never omitted. Carries the provenance tick — it is *located* — but the hatch and dash must dominate, because located and read are different claims.
- **Out-of-snapshot node (`{components.node-out-of-snapshot}`)** — a target the source names but extraction did not cover. Dotted, muted, labelled with the name it was given. Must read as *the edge of what was looked at* — not as a failure (unparsed) and not as an unknowable (unresolved). Three different kinds of not-having-it, three distinguishable treatments.
- **Unresolved-target node (`{components.node-unresolved}`)** — a dynamic call target, labelled `‹dynamic: WS-PROG-NAME›`. Dashed `{colors.unresolved}` border. **Must be visually distinct from an unparsed node**: different hue, different hatch (or none). Confusing "we couldn't read this" with "this isn't knowable until runtime" would collapse two honest states into one vague one.
- **Unresolved-inclusion node (`{components.node-unresolved-inclusion}`)** — an include whose source could not be located, labelled with the member it named, e.g. `‹copybook not found: CUSTREC›`. Dotted border **and** hatch fill: it is genuinely the union of the two neighbouring states — *named but absent* like out-of-snapshot, *interior unknown* like unparsed — and the treatment says so rather than picking a side. It is the one honest state whose local mark **cannot** convey its own severity: what is missing is unbounded and invisible, since nothing renders where the copybook's paragraphs and data items would have been. It therefore always travels with a view-level count in the header (`EXPERIENCE.md` § What the picture must never over-resolve). The node says *where*; only the header says *how much you cannot see*.
- **Edge (`{components.edge}`)** — a drawn relationship. 1px stroke `{colors.muted}`, with an `8px` invisible hit area so a click-to-trace target is reachable. Category colour when kind matters, always with a line style. Variants: `{components.edge-traced}` while tracing, `{components.edge-unparsed}` when it touches a hole, `{components.edge-jump}` for the unstructured jump (dashed + open arrowhead — distinguished by shape, not colour alone).
- **Provenance tick (`{components.provenance-tick}`)** — the signature mark. A small `{colors.provenance}` dot/underline on every drawn element confirming it carries a source span, at or above `min-size` in every theme and zoom. Its *absence is impossible by construction* — that's the point. What it does **not** claim: correctness, or that the span was read.
- **View header (`{components.view-header}`)** — the top strip: wordmark, perspective tabs, breadcrumb, legend, controls. One `{colors.border}` separating it from the canvas, no elevation. Carries the honest counts (unparsed, unresolved, collapsed) in `{colors.muted}` and the draw timestamp.
- **Legend entry (`{components.legend-entry}`)** — colour swatch **plus** a line-style sample plus the label. The non-colour cue is not optional; it is what keeps the view readable when the ramp flattens.
- **Sequence & flowchart marks** — cycle badge, guard label (set in `{typography.code}` on the message), loop fragment. All reuse the node/edge tokens above; no perspective introduces a second visual vocabulary.
- **Source anchor (`{components.source-anchor}`)** — the `file:line` chip (monospace, link-colored). Click = jump to that exact line in the editor. Appears inside both provenance surfaces below.
- **Anchor popover (`{components.anchor-popover}`)** — the **primary** provenance surface. Attached to the element on hover/select, styled to VS Code's own hover widget so it reads as native editor behavior (the go-to-definition muscle memory). Holds the element's source anchor(s) and nothing else.
- **Provenance rail (`{components.provenance-rail}`)** — the **secondary, opt-in** audit surface. A slim right column listing anchors across many elements at once, separated from the canvas by a single `{colors.border}`. Never auto-opens; toggled from the view header.
- **Perspective switcher (`{components.perspective-tab}`)** — header tabs switching the projection of the current scope. Quiet `{colors.muted}` inactive, `{colors.foreground}` + `{colors.focus-ring}` underline active — VS Code panel-tab geometry, no pill or fill. Only valid perspectives render; there is no disabled-tab state.
- **Breadcrumb (`{components.breadcrumb}`)** — the grain path (`payroll › PAYROLL.cbl › CALC-TAX`). Segments set in `{typography.code}` because they are source identifiers; ancestors in `{colors.muted}`, current grain in `{colors.foreground}`. Separator `›` in the UI ramp.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Inherit every base token from the user's VS Code theme | Hardcode a hex where a `--vscode-*` token exists |
| Give every token binding a fallback chain ending in `currentColor` | Let an undefined theme token silently erase a meaning-bearing mark |
| Spend brand colour only on provenance / trace / unparsed / unresolved | Add a decorative brand palette on top of the editor |
| Render unparsed regions visibly (dashed, hatched, warning hue) | Omit, hide, or red-flag an unparsed region |
| Draw *unresolved targets* distinctly from *unparsed regions* | Collapse two different kinds of not-knowing into one treatment |
| Say "traces to `file:line`" | Say "verified" anywhere near the provenance tick |
| Put source identifiers and `file:line` in monospace | Set code-derived labels in the UI (sans) font |
| Keep the provenance tick on every drawn element, at min-size | Draw a node or edge that can't show its source |
| Verify composed pairs against the contrast floor, falling back in-system | Assume theme inheritance guarantees legibility |
| Stay dense and flat, matching VS Code | Add shadows, gradients, rounded cards, or celebratory motion |
| Pair category colour with a line style or shape | Rely on colour alone to distinguish node/edge kinds |
