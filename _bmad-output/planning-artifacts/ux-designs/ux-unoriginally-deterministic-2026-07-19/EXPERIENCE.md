---
name: unoriginally-deterministic
status: final
sources:
  - {planning_artifacts}/prds/prd-unoriginally-deterministic-2026-07-18/prd.md
  - {planning_artifacts}/prfaq-unoriginally-deterministic.md
  - {planning_artifacts}/prfaq-unoriginally-deterministic-distillate.md
updated: 2026-07-20
---

# unoriginally-deterministic — Experience Spine

> VS Code extension. Deterministic, code-derived comprehension views — one **scope** projected through switchable **perspectives** across three grains — with structurally-enforced provenance. Paired with `DESIGN.md` (visual identity — theme-inherited). This spine owns *how it works*; the two spines win on conflict with any mock or import. Reconciled with the PRD and the architecture spine on 2026-07-19.

## Foundation

**Form-factor:** a VS Code extension. Comprehension views render inside **VS Code Webview** panels in the editor group; the real code opens in VS Code's native editor. There is no separate app, no window, no web surface — the product lives entirely where the developer already is (the whole accessibility wedge: no Pharo, no new environment, per PRFAQ Q1).

**UI system:** VS Code itself. Chrome, controls, theming, and density inherit from VS Code / Webview UI conventions; `DESIGN.md` is the visual-identity reference and names the only brand-layer delta (provenance / unparsed / trace semantics). This spine specifies behavior.

**Runtime posture — load-bearing, shapes every interaction:**
- **No AI at run time.** The toolkit bundles no model and asks for no keys (PRD FR-4 / §4). Nothing in this UI ever shows an "AI thinking," a token cost, or a confidence score. Views are computed deterministically from tree-sitter extraction, on-machine. No source leaves the repo.
- **Zero-friction start.** Starter views run on install with no account, no config, no molding, no tokens (PRD FR-5). The first surface a user meets already works.
- **Molding is out-of-band.** Authoring a *new* tool happens in the engineer's own AI assistant (Claude Code, or by hand) — outside this UI. See *Molding Handoff*. The toolkit's job at molding time is to expose a clear extension contract, not to host the assistant.
- **Sharing is out-of-product.** Showcasing a dogfood outcome (PRD UJ-3 / SM-3) happens by screenshot or by publishing the view's own repo-committed tool — there is no export, share, or publish affordance in v1, and no tracking of any kind. "Copy anchor" yields a literal `file:line`; that is the extent of it.

## Information Architecture

| Surface | Reached from | Purpose |
|---|---|---|
| Commands | Command Palette (`⇧⌘P`), editor/explorer context menu | Entry points, one per starter view: "Open dependency map", "Open call-flow", "Trace module reach", plus "Run tool…" and "How to mold a tool". |
| View gallery | First run / "Run tool…" command | Native QuickPick listing starter perspectives and in-repo molded tools; runs on select. |
| View canvas | Gallery select / a command | **Primary surface.** One scope projected through the active perspective; the header carries the perspective switcher and grain breadcrumb — see *Views & Perspectives*. |
| Anchor popover | Hover / select an element | **Primary provenance surface.** Attached to the element; shows its source anchor(s). Always available, costs no canvas. |
| Provenance rail | Header toggle (opt-in) | **Audit surface.** Slim right rail listing anchors across many elements at once — for verifying a whole view, not a single element. Off by default; never auto-opens. |
| Native editor | `F12` / source anchor / Code perspective | VS Code's own editor, scrolled to and highlighting the exact source line. The trust destination — not our surface, but the end of every trace. In the Code perspective it opens as a split beside the canvas, selection-synced. |
| Molded tools (in-repo) | View gallery / the repo's own tool folder | Tools the user (or their assistant) authored, versioned as plain TS in *their* repo. Run identically to starters. |
| Contract & primitives docs | "How to mold a tool" command / README | The extension contract read as code + docs — the molding on-ramp. See *Molding Handoff*. |

A view is one **scope** seen through switchable **perspectives** (see *Views & Perspectives*); there is still no multi-pane dashboard — the only split is the native editor in the Code perspective. Drilling is *semantic* (system → component → unit), not spatial.

→ Composition reference: [`mockups/dependency-map.html`](mockups/dependency-map.html) — system grain, Dependency perspective: header with perspective tabs and breadcrumb, category legend, provenance ticks, one unparsed node, pinned anchor popover. [`mockups/unit-flowchart.html`](mockups/unit-flowchart.html) — unit grain, Flowchart perspective with the native editor split and selection sync. **The spines win on conflict** — the mocks illustrate composition, they do not decide behaviour. (Both were rendered before the review fixes above; where they show a claim the spine has since corrected — "verified" copy, undistinguished dynamic calls — the spine governs.)

## Voice and Tone

Microcopy. Brand voice and aesthetic posture live in `DESIGN.md.Brand & Style`. The register is **honest, dry, technically literal, never hyped** — the product's entire positioning is refusing to overclaim, so the copy must never oversell either.

| Do | Don't |
|---|---|
| "Drawn from PAYROLL.cbl as it is now." | "Your codebase, visualized! ✨" |
| "3 regions couldn't be parsed — shown below, unread." | "Some errors occurred." |
| "Every element traces to source. Click any node to see where." | "AI-powered insights into your code." |
| "No molded tools yet. Run a starter view, or mold one with your assistant." | "Get started in seconds — no code required!" |
| "Same code, same picture — re-run anytime, free." | "Refreshing your analysis…" (implies nondeterminism) |
| "2 of 14 calls have static targets — the rest shown unresolved." | "14 calls found." (hides how much is actually known) |
| "No inbound references found statically. 12 dynamic call sites in scope could reach this." | "Unused." / "No callers — safe to remove." |
| Name unparsed regions plainly; they're expected. | Frame an honest hole as a failure, warning, or user error. |

Never say "accurate," "always correct," "verified," "everything," or "understands your code." The honest claim is *fidelity to code* — "what's actually there," "drawn from the source," "traces to `file:line`." (Rejected framings, per distillate: "accurate map," "stop reading code.")

**Determinism copy may claim the picture.** The architecture phase resolved this fork in favour of reproducible layout (spine AD-7, AD-14): canonical ordering plus a layout engine configured as a pure function of its input, protected by a golden-file release gate. So "same code, same picture" is honest — **per layout selection**, since the layout toggle chooses among layouts, each individually reproducible. This is what lets a rendered view be pointed at as evidence rather than an illustration.

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md.Components`.

| Component | Use | Behavioral rules |
|---|---|---|
| Node | View canvas | Hover or select → **anchor popover** with its source anchor. `Enter` / double-click jumps to the source line in the editor. Every node carries a provenance tick (`{components.provenance-tick}`); a node with no source span cannot be drawn, so none ever renders without it. |
| Unparsed node | View canvas | A tree-sitter ERROR region rendered in place (`{components.node-unparsed}`). Carries the provenance tick (it is *located*), but its *unread* status dominates visually. Its popover shows the unparsed span and offers "jump to unread source." Never omitted, never auto-fixed, never counted as a normal node. Edges touching it inherit the unparsed treatment; no edge is drawn *around* it. |
| Unresolved-target node | View canvas | A dynamic call whose target isn't static (`{components.node-unresolved}`): terminal node labelled `‹dynamic: WS-PROG-NAME›`, reached by a dashed edge, own legend entry. Popover shows the `CALL` line and the identifier. Distinct from unparsed — the parse succeeded; only the target is unknown — and the two must never look alike. |
| Edge | View canvas | Represents a relationship (call, dependency, data reach). Click traces it: the edge and its source line light (`{components.edge-traced}` + `{colors.trace-active}`) and the popover shows the exact origin; the link persists until the user acts again. Kind is shown by category color **plus** line style and a legend entry, never color alone. |
| Category legend | View header | Always visible when more than one kind is drawn; names every kind present in the current view, including `unparsed` and `unresolved`. Each entry shows the colour *and* the non-colour cue (line style / shape). Click filters the view to that kind and says what it hid; click again restores. |
| Anchor popover | Any drawn element | Primary provenance surface (`{components.anchor-popover}`). Opens on hover, pins on select, dismisses on `Esc` or moving away. Styled to VS Code's hover widget so it reads as native go-to-definition behavior. Holds the source anchor(s) plus a one-line drill preview (`↵ Flowchart · 9 blocks`) — GT's peek-before-spawn affordance, and nothing else. |
| Provenance rail | View canvas (opt-in) | Audit surface (`{components.provenance-rail}`). Toggled from the header; lists anchors for all currently-shown (or selected) elements so a user can verify a whole view in one pass. Each row is a source anchor. Selecting a row highlights its element on canvas and vice versa. |
| Provenance tick | Every drawn element | Not interactive on its own — it's the always-present proof mark. Its presence is invariant; its meaning is "this element exists because a real source span backed it." |
| Source anchor | Popover + rail | `file:line` chip (monospace). Click = jump-to-source. Copy = yields the literal `file:line` for sharing/grepping. |
| View header | View canvas top strip | Wordmark + perspective switcher + grain breadcrumb + category legend + layout/filter controls + rail toggle. States plainly what the view was drawn from and when, plus the honest counts (unparsed regions, unresolved targets, collapsed subtrees). When the active perspective is a molded tool, names it with one-click "view analyzer source". |
| View gallery | "Run tool…" command | Native VS Code **QuickPick** — the low-cost, posture-consistent choice; no webview, no custom chrome to spec. Lists starter perspectives and in-repo molded tools together, each with a one-line "what question it answers". Also where the Sequence entry-point ask surfaces when nothing is selected. |
| Perspective switcher | View header | Tabs for the perspectives that can honestly derive something here (bundled and molded alike) — the rest are absent, not disabled, with an overflow naming them and why. Switching re-projects the existing snapshot: no re-parse, selection preserved, and the header states continuity plainly *and truthfully* ("Sequence — 7 of 12 modules reached from PAYROLL") so the change is legible and the compression is admitted in the same breath. Switching to Sequence uses the selected program as entry point, or asks via QuickPick if nothing is selected. |
| Breadcrumb | View header | The grain path (`payroll › PAYROLL.cbl › CALC-TAX`), segments in monospace since they're source identifiers. Each segment click = drill out to that grain, keeping the perspective if valid there. |
| Gallery item | View gallery | One row per perspective or molded tool, each with a one-line "what question it answers." Unloadable ones show their error rather than vanishing. Each row states **where it lives and how it changes** — "ships with the toolkit · updates on release" vs "lives in your repo · frozen until you re-mold". These are the two halves of the same ownership guarantee (PRD FR-6), and the difference is something the user should be able to *feel*, not infer. |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Extracting | View canvas | Deterministic parse progress (e.g. "Parsing 214 files…"), never an indeterminate "AI thinking" spinner. Resolves to the drawn view. |
| Honest holes present | View canvas | Unparsed regions render in place; header notes the count plainly ("3 regions unparsed — shown, unread"). The view is *correct*, not degraded. |
| Fully unparseable file | View canvas | Show the file with all regions marked unparsed — never a blank canvas, never a fabricated structure. "This file couldn't be parsed. Nothing here is guessed." |
| Unresolved inclusion | View canvas + header | A `COPY` member that couldn't be located renders at its inclusion site as `‹copybook not found: CUSTREC›` (dotted + hatched), **and** the header states the count with its consequence ("2 copybooks unresolved — paragraphs and data items they define are absent from this view"). The header half is not optional: what's missing has no location to render at. |
| Empty — no molded tools | View gallery | "No molded tools in this repo yet. Run a starter view, or mold one with your assistant." Links to the starter set and the contract docs. Not framed as a deficiency. |
| Trace target | Editor | Jumping to source highlights the exact line(s); the originating element stays visibly linked (`{colors.trace-active}`) until the user acts again. |
| Re-run on changed code | View canvas | Re-derives from current code — no diff prompt, no "re-analyze?" step. Same command, new snapshot (PRD UJ-2). |
| Nothing to draw | View canvas | Scope contains no sources in any supported language: "No supported sources found under ⟨scope⟩. Nothing here is guessed." Plus the list of languages that *are* supported, so the limit is legible. Never a blank canvas. |
| Stale snapshot — *view already open* | View canvas + editor | Files changed since the view was drawn: header states it ("drawn 14:02 · source changed since — re-run to re-derive"). **No auto-re-derive mid-reasoning** — the canvas is never yanked out from under a train of thought. A jump-to-source into a changed file says so rather than landing confidently on a line that may have moved. |
| Stale snapshot — *view being opened* | View canvas | A view whose snapshot already fails to match before anything is drawn — restored from a previous session, pulled from git, copied in from elsewhere — is **not opened with a notice; it is re-derived**. Bundled perspectives re-derive immediately ("re-deriving from current code…"); a molded tool asks first, since running repo code needs the user's say-so. Nothing stale ever reaches the canvas, because there is no reasoning in progress to protect yet — only a picture whose truth has expired. Refusal is stated, never a blank canvas. |
| No inbound references | View canvas (Reach, inbound) | "No inbound references found statically." Always paired with the unresolvable-dispatch count in scope ("12 dynamic call sites could reach this"). Never rendered as "unused", never accompanied by a removal affordance. |
| Thin derivation | View canvas | Perspective applies but resolves poorly: renders and states it ("2 of 14 calls have static targets — the rest shown unresolved"). Never pre-filtered away. |
| Molded tool errors at run | View canvas | The tool's own failure surfaces as a plain error with its `file:line`; the toolkit does not fabricate a partial view to cover it. |
| Molded tool won't load | View gallery | Malformed tool or invalid perspective declaration: listed as unloadable with its error and `file:line`. Never silently dropped from the list. |

## Interaction Primitives

**Keyboard-operable throughout** — the audience is developers in an editor; parity with VS Code's own keyboard surface is expected.

- **Hover element** → anchor popover with its source anchor(s).
- **Click node** → select + pin the popover.
- **Click edge** → trace to source (light element + line, show origin in the popover).
- **`Enter` / double-click** → **drill in** one grain on the selected element (system → component → unit). At unit grain, the default perspective is Flowchart.
- **`F12`** → **jump to source** — the core trust action, at every grain on any element. Moves focus into the editor at the exact line; no sync contract, no split. Mirrors go-to-definition so the trust check rides existing muscle memory. Clicking a source anchor does the same.
- **Arrow keys** → move selection across nodes and edges within the canvas (which is a single tab stop — see *Accessibility Floor*); keyboard selection opens the popover exactly as hover does.
- **Perspective switch** → header tabs, keyboard-reachable; re-projects the existing snapshot, never re-parses. The **Code** perspective — distinct from `F12` — is what opens the selection-synced editor split.
- **Trail Back / Forward** → restore the exact prior step (scope + perspective + selection). Structural retreat (`Esc`) and temporal retreat (Back) are distinct and both always available. Note the seam honestly: the trail applies **with canvas focus**; once focus is in the editor, the same gesture is VS Code's own navigation history. Two histories, one gesture — designed and documented rather than discovered.
- **Toggle reach direction** → outbound / inbound / both, on the Reach tracer. Re-projects the existing snapshot; the header restates which question is on screen.
- **Toggle rail** → opt into the audit surface when verifying many elements at once.
- **`Esc`** → layered retreat: dismiss the popover if open → else clear selection → else **drill out** one grain → at system grain, nothing further. Breadcrumb segments drill out directly.
- **Filter / collapse** → narrow the view by category or subtree; never fabricates, only hides (and says what it hid).
- **Layout toggle** → re-arrange the same deterministic graph; layout is cosmetic, the derived structure is fixed.
- **Copy anchor** → yields literal `file:line`.

**Banned everywhere:** any run-time AI affordance (explain-this-with-AI, confidence %, token counter); drawing a node/edge without a source span; silently dropping an unparsed region; **resolving a dynamic `CALL` target by heuristic**; **drawing a verified-style edge whose derivation crosses an ERROR region**; a "refresh" that implies the picture could differ for the same code; offering a perspective the extraction can't honestly derive (the state-machine trap — see *Views & Perspectives*); the word "verified" anywhere near the provenance tick; presenting an inbound result as complete, or deriving a dead-code/safe-to-delete claim from inbound emptiness.

## Accessibility Floor

Behavioral. Visual contrast rules live in `DESIGN.md`.

**Scope of this floor — stated honestly.** v1 commits to **keyboard operability, theme fidelity, and non-colour-dependent meaning**. It does **not** commit to assistive-technology / screen-reader support: no ARIA model for the canvas, no linearised structure alternative, no announcement contract. That is a deliberate v1 scoping call, justified by the PRD's user model (§2.2 — the maintainer is the sole v1 user; other engineers are explicitly not a v1 adoption or support target), not a claim that the gap doesn't matter. It is recorded here rather than left implicit **because an unmet accessibility claim is exactly the overclaim this product refuses elsewhere**. If v1 ever targets users beyond the maintainer, this floor is the first thing that must be re-opened — the canvas would need an accessible-structure model and text equivalents for every meaning-bearing mark, and neither is cheap to retrofit.

What v1 *does* commit to:

- **Fully keyboard-operable.** Every element is reachable and every trust action performable without a mouse: drill, drill out, trace, jump to source, switch perspective, toggle the rail. The keyboard surface is the product for a developer audience, not an accommodation bolted on.
- **The canvas is a single tab stop.** Arrow keys (plus type-ahead find) move *within* the graph; `Tab` / `Shift+Tab` leave it for the next chrome control. A 214-node graph must never put hundreds of tab stops between the user and the perspective tabs — the standard composite-widget pattern, not a per-node tab ring. Focus ring order: header controls → canvas → rail (when open) → editor split (when in Code perspective).
- **Every primitive is also a VS Code command** — contributable, remappable, and visible in the Command Palette ("Comprehension View: Drill In / Drill Out / Jump to Source / Go Back / Toggle Rail"). Webviews receive raw DOM key events and do not inherit editor keybindings, so a default key is a convenience, never the only route; a user who has remapped their bindings, or whose platform differs (`Alt+←` is *not* Go Back on macOS), keeps a working path.
- **Focus is always visible** (`{colors.focus-ring}`), including on canvas elements.
- **Provenance is never hover-only.** The anchor popover opens on keyboard selection exactly as on hover — a verifiability guarantee that only works with a pointer isn't a guarantee. The popover stays open while the pointer is over the element *or* the popover itself (so the `file:line` chip is reachable), and `Esc` dismisses it without moving the pointer.
- **Colour is never the only cue.** Node and edge kinds carry a shape, line style, label, or legend entry alongside colour — so the view survives colour-blindness and low-quality themes alike.
- **Theme fidelity, with a floor.** The extension inherits the user's light / dark / high-contrast choice rather than fighting it. Inheritance alone does not guarantee legibility, so `DESIGN.md` sets a contrast floor and fallbacks for the marks that carry meaning (see *Colors*). High-contrast themes work by removing fills, so every state distinction must survive on border, line style, or text alone.
- **Motion is minimal and optional.** No celebratory motion exists to begin with; all viewport animation (fit, layout re-arrange, transitions) is instant under `prefers-reduced-motion`.
- **A legibility floor over density.** Fit-to-canvas must not render labels below a readable size — below the threshold the view auto-collapses and *says* what it collapsed, rather than showing an unreadable picture.

## Views & Perspectives

*Product-specific — the core viewing model. Everything below draws from the same tree-sitter extraction, carries provenance on every element, and runs with no molding, no account, no tokens (PRD FR-5). Chosen from the maintainer's own recurring cross-project needs, which is what cold-starts the commons honestly (PRFAQ Internal Q7).*

> **Supersedes PRD OQ-3 and expands FR-5.** The PRD commits "at least one working starter view" and deliberately leaves the starter set open (OQ-3). This spine closes that question with six perspectives across three grains. **The spine wins until the PRD is updated**, and the PRD update should run before architecture consumes this pair. The phase split below keeps the reliability gate (PRD OQ-1 / SM-2) reachable without building all of it first.

**Phasing.** The *model* — grains, perspectives, the switcher, the availability rule, provenance — is committed in full, so nothing below needs redesign later. The *surface* ships in two phases, because the PRD gates everything on the molding-reliability experiment (OQ-1) and that gate needs extraction, provenance-enforcing primitives, one view, and click-to-source — not six renderers.

**PRD §5.2 owns the authoritative tiering; this table mirrors it.** Three tiers, not two — the gate fires *inside* Phase A:

| Tier | Contains | Why |
|---|---|---|
| **Gate slice** *(subset of Phase A)* | Dependency map only · anchor popover · jump-to-source · unparsed + unresolved rendering · extraction incl. inclusion resolution **and data artifacts (files and tables)** | Exactly what the reliability gate runs against — one perspective, not six. Inclusion resolution is inside the slice because a gate passed against unresolved includes might not transfer to the real estate. |
| **Phase A remainder** *(post-gate)* | Unit flowchart · view gallery | Completes a standalone, honest product. The flowchart is what proves the grain model and the switcher. |
| **Phase B** *(post-gate)* | Sequence · Reach tracer · Call-flow · trail history · provenance rail · Code-perspective selection sync · molded-perspective registration | Each is a substantial build (bespoke layout engines, data-flow extraction, bidirectional editor sync). None is needed to answer OQ-1, and a failed gate should not have been preceded by them. |

Phase A must stand alone as a complete, honest product — not a demo with gaps.

**The model: one scope, many perspectives.** A view is not a fixed picture. It is a *scope* (what code you're looking at) seen through a *perspective* (how that scope is projected). Switching perspective re-projects the **same extracted structure** — no re-parse, no scope change, selection preserved. Drilling moves the scope **down a grain** (`Enter`/double-click); `Esc` or the breadcrumb moves back up. The feel is turning the system over in your hands, not opening rival tools.

**A view is a stated snapshot.** Extraction happens once per run; the header always names what was drawn and when ("drawn from `payroll/` at 14:02"). *That* is what "no re-parse on perspective switch" means — switching projects the existing snapshot, which is why it's instant and why selection survives. When the underlying files change, the view does **not** silently re-derive mid-reasoning and does not quietly go on lying: the header states it ("source changed since draw — re-run to re-derive"), and a jump-to-source into a changed file says so rather than landing confidently on a line that may have moved. Re-running is always the deliberate act that produces a new snapshot (PRD UJ-2).

**The trail: temporal, not spatial (Phase B).** Every drill and perspective switch pushes a step onto the view trail; Back / Forward restore the exact prior step — scope, perspective, *and* selection. `Esc` retreats *structurally* (up a grain); Back retreats *temporally* (to where your reasoning was, even if that was a different perspective two grains down). Restoring a step whose snapshot has since expired — the common case across sessions — re-derives first rather than reopening the old picture; if the step's anchor is absent from the new snapshot, that is stated rather than silently rendering something adjacent. The trail is history behind ordinary navigation keys, so the canvas always shows exactly one thing — see *Inspiration & Anti-patterns* for what this deliberately avoids.

**Grains are language-agnostic.** Every language the toolkit supports maps onto the same three grains; only the concrete constructs differ, and those live in *Language Profiles* below. Nothing in this section is keyed to a particular language — that is what lets a new language arrive as a profile rather than a spine rewrite (the "contribute a language" flywheel is a first-order product dimension, PRFAQ).

**Data artifacts are nodes, not edge decorations.** A file or a database table is a first-class system-grain node, selectable like any program. This is what makes *"what depends on this table?"* a question the product can answer at all — and in a legacy estate it is often the question that matters most, because a program can be rewritten while corrupted data cannot be un-corrupted. Two consequences:

- **Data nodes are leaves.** There is no code inside a table, so they have no component grain; drilling is simply unavailable there (the availability rule already covers this) and the natural action on a data node is **inbound reach** — *what touches this*.
- **Access is typed on the edge, and the target is real.** `read` / `write` edges point *at* the data node they name. An edge kind called "write" with nothing on the far end would be a claim about a relationship the view can't show.

| Grain | A node is… | Perspectives (v1) |
|---|---|---|
| **System** | a **compilation unit** (program, included source) **or a data artifact** (file, table) — the things that depend on each other | **Dependency map** · **Sequence diagram** · **Reach tracer** (bidirectional) |
| **Component** | a **named callable** within one unit | **Call-flow** |
| **Unit** | the **control flow inside one callable** — its branches, loops, calls, exits | **Flowchart** · **Code** (native editor split) |

**Per-perspective behavior:**

- **Dependency map** (system) — the orientation perspective, and the one a user meets first. Opens fit-to-canvas at whole-system scope so the shape of the estate is visible before drilling. Nodes: compilation units, the sources they include, and the data artifacts they declare. Edges are **typed by kind** — `invoke`, `include`, `read`, `write` (the closed set; architecture AD-17) — each category-coloured *and* carrying a non-colour cue in the legend, so a program-to-program call is never confusable with a program-to-table write. "Static reference" is prose for the family, never a kind a producer may emit. Collapse-by-subtree for density; never silently truncates (it says what it collapsed).
- **Sequence diagram** (system) — the *order* perspective on the same units: lifelines are units, messages are **invocations in source order** from a chosen entry point. Conditional invocations carry their guard condition visibly; statically evident loops are marked as fragments. This is the sequence **as written in source, not a runtime trace** — the header says so plainly. Switching to it with a unit selected uses that as the entry point; with nothing selected, it asks. Every message traces to its invocation site.
- **Reach tracer** (system) — the blast-radius perspective, most directly serving "I have to change this without breaking the business." **Bidirectional**, because that question has two halves and only one of them points outward:
  - **Outbound** — what this node **statically** reads, writes, and invokes, including the files and tables it touches. Answers *"once this is invoked, what else does it reach."* Edge kind (read / write / invoke) is first-class because *writes* carry the risk.
  - **Inbound** — what statically reaches this node. Answers *"what will I break by changing this."* Works from a **data artifact** as readily as from a program: selecting a table and tracing inbound is how you learn everything that writes to it. The canonical modernization question, and the one a static outbound trace can never answer.
  - **Both** — the union, with direction distinguished visually and in the legend. Never a merged undirected blur; the two answers carry different consequences.

  Direction is a header toggle on one perspective, not three perspectives. Depth is user-controlled and stated in either direction ("reach depth 2 — 3 further hops collapsed"), never silently bounded, and the same header breath reports unresolved targets and holes. Never "everything it touches" — unearnable wherever dynamic dispatch or parse holes exist, and the rejected-framings list bans exactly that completeness claim.
- **Call-flow** (component) — the static control-flow perspective inside one unit: its callables in **declared and invocation order, as written** — not a runtime trace, and the header says so with the same disclaimer Sequence carries. Where a language permits fall-through or unstructured jumps, static order does not equal execution order — which is precisely why **unstructured-jump edges are visually distinguished** as the thing that breaks a reader's mental model. Cycles are marked explicitly rather than broken or hidden.
- **Flowchart** (unit) — the control-flow graph of one callable: decision, loop, statement-block, call and exit blocks, unstructured jumps distinguished, every block spanning real lines. This is the **universal** unit perspective — control flow is mechanically derivable from the AST for any callable in any grammar, which is exactly why it's the bundled one and why it anchors Phase A.
- **Code** (unit) — the source itself, via a **native editor split**: VS Code's real editor opens beside the canvas, scrolled to the unit's span, selection synced both ways. The toolkit never re-renders source in a webview — the editor owns code, always current, with every editor feature intact. Trivially language-agnostic, which is part of why it's the right answer.

### What the picture must never over-resolve

Every language has constructs where a drawn edge is less certain than it looks. These are **four general categories, not four language quirks** — each language profile names its own instances, but the treatment is defined once, here. Each gets a *visible* state, because a uniform-looking view whose parts are differently trustworthy is the fabrication this product exists to refuse, arriving by omission instead of invention.

- **Unresolved targets** — *dynamic dispatch.* An invocation whose target isn't statically knowable (a call through a variable, a function pointer, reflection, a dynamic import, a virtual call with many candidates). Drawing a resolved edge is a guess; dropping it is a silent hole. So it renders as a third first-class state: a dashed edge to a distinct **`‹dynamic: …›`** terminal node carrying the dispatch expression, its own legend entry, popover showing the invocation site. In Sequence it is a message to an unresolved lifeline, marked. In Reach, transitive reach *stops* there and says so ("reach ends at 2 dynamic targets — not statically resolvable"), the same discipline as depth truncation. This is **not** a parse hole: the parse succeeded, only the target is unknown, and the two must never look alike.
- **Inbound answers are never complete** — *the reverse face of dynamic dispatch, and the sharpest trap in the product.* Outbound is self-limiting: when a unit invokes something unresolvable, the unknown sits visibly on the path and the unresolved-target state covers it. Inbound has no such tell. **"Nothing reaches this" can simply be false** — any dynamic-dispatch site in the estate might resolve to this unit at runtime, and no static extraction can rule it out. So an inbound result is never a bare count and never a bare empty state: it always reports the unresolvable-dispatch population that *could* reach the target, in the same breath as the ones that provably do — *"4 static callers, plus 12 dynamic call sites whose targets aren't statically resolvable — any could reach this."* An empty inbound result reads as **"none found statically,"** never as "unused." **Banned:** any dead-code, disposal, or safe-to-remove affordance derived from inbound emptiness. A view that implies a program is safe to delete, when a dynamic call reaches it, is the confidently-wrong failure this product exists to refuse — and unlike a wrong edge, that one takes the business down.
- **Out-of-snapshot references** — *named, but not extracted.* A target the source names statically that is absent from the extracted set: a called program outside the current scope, or missing from disk. Distinct from a dynamic target — the destination *is* known — and distinct from a parse hole, since nothing failed to parse. It renders as a marked boundary node carrying the reference site, so the map shows its own edge rather than stopping silently. Without this state one perspective would draw the reference and another drop it, both defensibly.
- **Holes inside a rendered structure** — *parse-failure regions.* Unparsed regions are specified as nodes, but *edges and order across* a hole are the subtler lie. A flowchart drawing `A → ‹unparsed› → B` asserts fall-through the parser never verified — the hole could hold a jump or an exit. So: edges entering or leaving an unparsed block render in the unparsed treatment (dashed, warning hue) meaning *"control may flow here; the hole was not read"*, and an edge is **never** drawn *around* a hole. In Sequence and Call-flow a hole renders as a marked gap in the order, so "in source order" is visibly interrupted rather than silently bridged. This one is grammar-universal: tree-sitter's ERROR node is the mechanism in every language.
- **Included and generated source** — *text that arrives from elsewhere.* Wherever the source a reader sees isn't the source the compiler saw (textual includes, macro expansion, codegen, aliasing), provenance gets harder, not easier. **v1 resolves inclusion during extraction** so references across an include boundary resolve properly and shared declarations participate in reach — in most legacy estates this is the *main* reach mechanism, not an edge case. It carries a hard rule: **an element derived through inclusion carries both anchors** — the inclusion site *and* the definition line — and where the inclusion renamed or substituted an identifier, the popover states that plainly. The click-through must never land on a line that doesn't say what the node said; that surprise would break the core trust gesture more thoroughly than not resolving inclusion at all.

  **When the inclusion cannot be resolved at all** — the `COPY` member is not on any configured path, or is missing from disk — extraction marks the inclusion site and continues; it never omits the reference, guesses at a member, or abandons the file. It renders as its own state (`{components.node-unresolved-inclusion}`): dotted **and** hatched, labelled with the member it named, provenance tick pointing at the `COPY` line. The union treatment is deliberate — this state sits between its two neighbours and belongs to neither. It is *not* a parse hole (nothing failed to parse) and *not* out-of-snapshot (that is the benign edge of what you chose to look at; this is a piece of the program in front of you, gone).

  **This is the one state whose element mark cannot carry its own weight**, so it is also the one that must always appear in the header. Every other honest state renders *where the problem is*: an unparsed region occupies its lines, a dynamic target sits at the end of its edge. An unresolved copybook renders at the `COPY` site — but what is missing is everything the copybook *would have defined*, which has no location to render at. Paragraphs that should exist simply don't; reach through them silently doesn't happen; an inbound query returns fewer callers with no visible gap. So the view header always states the count and its consequence — *"2 copybooks unresolved — paragraphs and data items they define are absent from this view"* — carried as **thinness** (architecture AD-22), never left to the canvas alone. In a copybook-saturated estate this is routine, not exceptional, and a silently thin view is the failure this product exists to refuse.

**Not bundled — state machine.** A state-machine perspective is only truthful where the code *explicitly* encodes one (a dispatch construct switching on a declared state variable); inferring states from arbitrary code would be fabrication, the exact thing this product refuses. It ships as the first-listed **moldable-tool candidate** in the contract docs, not as a v1 perspective.

**Extensible by molding — the `gtView` lesson (Phase B).** A molded tool can register itself **as a perspective** — declaring its grain and the node kinds it applies to — and then appears in the switcher exactly like a bundled one whenever the context matches, not merely as a gallery entry. Same provenance rules, same availability rule. The state machine above is the canonical example: molded for the one program that really *is* a state machine, it becomes a perspective tab on that program. **Timing matters:** every contract feature added before the reliability gate raises what an assistant must pattern-match unedited (PRD SM-2), so the gate runs against the *minimal* contract — draw primitives plus required provenance — and registration lands as a post-gate contract extension. The UX target stands; the commitment waits (feeds PRD OQ-2).

**Availability rule:** the switcher shows a perspective when it can honestly derive *something* for this scope at this grain — never a disabled tab, never an empty canvas. The predicate is derivability, not merely grain and node kind. Thinness, though, is shown rather than hidden: a perspective that applies but resolves poorly renders and *says so* ("2 of 14 calls have static targets — the rest shown unresolved"). Pre-filtering on quality would quietly decide for the user what they're allowed to see; stating the thinness lets them judge it. Where a perspective is genuinely absent, the switcher's overflow lists it with a one-line reason — legible limits, consistent with the Boundary rule below.

### Language Profiles

Everything above is language-agnostic. A **language profile** is the thin mapping from those generic concepts to one grammar's concrete constructs — and it is the *only* place a language name belongs. Supporting a new language means adding a column here and a tree-sitter grammar, not touching the model, the perspectives, the trust rules, or any component.

| Generic concept | **COBOL** (v1 — user comprehension) | **TypeScript** (v1 — self-documentation, PRD FR-2) |
|---|---|---|
| System node — compilation unit | program, copybook | module / source file |
| System node — data artifact (leaf) | declared file, database table | *(rarely applicable — the toolkit's own source declares none)* |
| Component node — named callable | section, paragraph | function, method, class member |
| Unit — control flow within a callable | statements inside a paragraph | statements inside a function |
| Inclusion edge (`include`) | `COPY`, `COPY … REPLACING` | `import`, re-export, path alias |
| Data access edge (`read` / `write`) | file `SELECT`/`ASSIGN`/`FD` + I-O verbs; embedded-SQL table references | *(rarely applicable)* |
| Invocation edge (`invoke`) — also the sequence message | `CALL 'LIT'`, `PERFORM` | function / method call |
| Unstructured jump | `GO TO`, `PERFORM THRU` fall-through | *(none — the distinction simply doesn't render)* |
| Loop construct | `PERFORM UNTIL` / `VARYING` | `for`, `while`, `do` |
| Dispatch construct | `EVALUATE` | `switch`, conditional chains |
| **Unresolved target** | `CALL WS-PROG-NAME` (identifier form) | dynamic `import()`, call through a variable, high-fan-out interface dispatch |
| **Included / generated source** | `COPY … REPLACING` | path aliases, barrel re-exports, generated `.d.ts` |

Two things this table makes obvious, and both are deliberate. **Absence is fine:** TypeScript has no unstructured jump, so that treatment simply never renders for TS — a profile is allowed to leave rows empty, and the availability rule already handles a perspective that can't derive anything. **Every language populates the honesty rows:** the "unresolved target" and "included source" rows are never empty in a real grammar, which is the argument for defining their treatment centrally rather than per language.

**Boundary — cross-language reach is out of v1** (PRD §4). Each profile is extracted independently; where a unit in one language is invoked from another, the view shows the boundary of the language it knows and stops there. It does not guess at, stub, or imply the other side, and it *says so* when asked rather than quietly returning less than the question wanted. An honest edge of the map, marked as such, is the same discipline as an honest hole. (Running the same perspectives over the toolkit's own TypeScript is the leading dogfood-showcase candidate — PRD OQ-5.)

## Provenance & Trust Model

*Product-specific — the heart of the experience. Behavioral spec of the PRD's structurally-enforced provenance (FR-4) and two trust checks (§9).*

Every drawn element exists **because** a source span backed it; there is no code path that draws an unsourced node or edge (PRD FR-4).

**What the provenance tick claims — and what it doesn't.** The tick means exactly one thing: *this element carries a source span you can jump to*. It does **not** mean the element is correct, and it does not mean the span was read and understood. Three consequences the UI must honour:

- Copy never says "verified" — it says **"traces to `PAYROLL.cbl:214`"**. "Verified" would promise a judgement the toolkit cannot make.
- **Unparsed nodes carry the tick too** — an ERROR region has a span, so it is locatable — but their *unread* status dominates visually. Located and read are different claims, and the view must not let a green mark blur them.
- The tick is a *structural* guarantee, not a semantic one: a molded tool can emit a wrong edge whose provenance is a real but irrelevant line. It passes the structural check and renders identically to a starter view. Which is why trust check #2 below is ambient, not buried.

The UI's job is to make the guarantee *usable*, two independent ways (PRFAQ "check it two ways, cheaply"):

1. **Trace the output.** Any element → its exact source line, one click. This is the primary, always-available trust action, and it is served at **two grains**: the **anchor popover** for checking *this one element I doubt* (the common case, zero friction, zero canvas cost), and the opt-in **provenance rail** for sweeping *every element in this view* when the stakes justify a full audit. Spot-check and full-audit are different jobs; conflating them into one surface would make the cheap check expensive. The map is a fast index *into* the truth, never a replacement *for* it — when a node matters, the user clicks through to the real lines (PRFAQ Q7).
2. **Audit the lens.** When the active perspective is a **molded** tool, the view header names it and offers one-click "view analyzer source" — "Drawn by `tools/reach.ts` — read it." Ambient, not buried in a command, because nothing else on the canvas distinguishes an unaudited molded lens from a vetted starter. Reading the analyzer is bounded work against the unbounded system it inspects (PRFAQ Q4); the audit view states the tool's actual size rather than promising a length the toolkit cannot enforce.

**Honest holes are a first-class state, not an error.** Where tree-sitter emits an ERROR node, the region renders visibly unparsed and is counted plainly. The contract with the user: *you will sometimes see honest holes; you will never see a confident fabrication where a hole should be* (PRFAQ Q2). Any UI that would hide, soften, or fabricate over a hole is a trust-model break.

## Molding Handoff

*Product-specific — the boundary between the toolkit and the engineer's own AI assistant. The toolkit ships no AI (PRD FR-3/§4).*

Molding a new tool is **not** an in-app wizard. The experience is a clean handoff:

1. The user hits a question no starter view answers.
2. "How to mold a tool" surfaces the **extension contract** — the primitives (tree-sitter queries + drawing primitives) and conventions, written to be pattern-matched by an AI assistant *and* read cold by a human (PRD FR-3). Provenance is a **required argument** of every drawing primitive, so the correct, source-carrying path is the only path (PRFAQ Internal Q3).
3. The user points their own assistant (Claude Code) at that contract, in their own tooling. The exchange of code context happens in their assistant, outside our control — exactly as any AI-assisted coding task (PRD FR-3 consequence).
4. The assistant writes a small plain-TS tool that **plugs straight in** — it appears in the View gallery and runs identically to a starter (PRD FR-6).
5. The user verifies it two ways (*Provenance & Trust Model*), commits it to *their own repo*, and re-runs it free forever (PRD FR-6, UJ-2).

**Frozen is a feature, and the UI says so.** Bundled perspectives move forward with each toolkit release; a molded tool stays exactly as authored until its owner changes it. That asymmetry *is* the ownership guarantee — the tool keeps working even if this project stalls (PRD FR-6) — so the gallery states each tool's provenance and update behaviour plainly rather than letting "frozen" read as "stale". A user who wants a molded tool updated re-molds it; nothing updates their repo behind their back.

The toolkit never assumes AI: hand-authoring against the same contract is a first-class path; AI is the recommended accelerator, not a dependency.

## Inspiration & Anti-patterns

- **Lifted from Glamorous Toolkit:** moldable, code-derived views; "look, only when you want to, and trust what you see." Three specific inspector patterns carried over: **contextual views per object** (the `<gtView>` tabs → our perspective switcher, including molded-tool registration), **the preserved reasoning trail** (GT's pager keeps every drill so you can trace cause back through effect → our temporal Back/Forward), and **peek-before-spawn** (GT's pane-preview triangle → our popover drill preview). What we deliberately *don't* lift: the environment cost (Pharo/Smalltalk, leaving your editor) — the exact on-ramp that stalled GT (PRFAQ Q1). We stay in VS Code.
- **Rejected — GT's Miller-columns pane cascade.** GT preserves the trail *spatially*: each drill spawns a live pane to the right. Public evidence says this is where newcomers drown — "the interface looks completely different without clear indication of what changed," panes crowd out navigation, "half the windows and symbols just make no sense." We preserve the same trail *temporally* (Back/Forward history) on a single canvas with an explicit breadcrumb and announced transitions. The insight survives; the disorientation doesn't.
- **Lifted from VS Code itself:** density, theming, keyboard-first operation, `F12`/`Alt+←` muscle memory, editor-style breadcrumbs. The extension should feel like a native editor capability.
- **Rejected — AI at run time / "explain this":** the confidently-wrong path this product is positioned *against* (PRFAQ Q5). No run-time model, ever.
- **Rejected — hiding or auto-fixing unparsed regions:** silence would be the fabrication we refuse. Holes are shown.
- **Rejected — "accurate / complete map" framing in UI:** every view is a compression; the answer to "what did it drop?" is "mold a different tool," not a promise of completeness (PRFAQ Q7).
- **Rejected — polished consumer-app chrome:** no celebratory motion, gradients, or dashboards. This is an instrument.

## Responsive & Platform

Not a responsive-web product; the surface is a VS Code Webview at whatever size the user's editor group is.

- **Narrow editor group / split view:** the canvas stays usable. The anchor popover is unaffected (it's element-attached and self-sizing). The opt-in rail, when enabled, becomes an overlay rather than a column that would starve the canvas.
- **Very large graphs:** default to the whole-system view fit-to-canvas, then let the user drill; dense graphs favor collapse-by-subtree over infinite pan. Never silently truncate — say what's collapsed.
- **Themes:** every VS Code theme (light, dark, high-contrast, custom) is a supported "platform"; the extension inherits all of them (see `DESIGN.md`).
- **OS:** wherever VS Code runs (macOS / Windows / Linux). No OS-specific surface; parsing is in-process WASM (no JVM, no service) so platform behavior is uniform (PRD FR-1).

## Key Flows

Protagonist: **Nomsa**, a modernization engineer who has just been made responsible for a 45-year-old COBOL platform entangled with 10-year-old Java. Nobody left understands how it fits together, and she has to change it without breaking what the business runs on. *(She embodies the maintainer / sole v1 user.)*

### Flow 1 — The first ten minutes, no molding (Tuesday, 9:10am)

1. Nomsa installs the extension from the marketplace and opens the COBOL repo. No account, no config, nothing to set up.
2. In a `PAYROLL.cbl` she's never read, she right-clicks → **Open dependency map**. A parse-progress line ("Parsing 214 files…") resolves into a graph, drawn from the code as it is right now. No tokens spent.
3. She sees three nodes rendered as **unparsed** — dashed, marked `‹unparsed: 12 lines›`. The header reads "3 regions unparsed — shown, unread." She exhales: it's not hiding anything from her.
4. One edge claims `PAYROLL` reaches a module she didn't expect. She distrusts it. She hovers it — the anchor popover names the source on the spot — and clicks to trace.
5. **Climax:** the edge and a line in the editor light together — `PAYROLL.cbl:214`, a `CALL` she'd have taken an hour to find by reading. It's real, it's exactly where the picture said, and she didn't have to take anything on faith. Ten minutes in, she already knows more than she did — and she knows she can *trust* it. She keeps the view open and starts planning her change around what it showed.

*Failure:* the whole file is unparseable → the canvas shows the file with every region marked unparsed and "Nothing here is guessed," never a blank or a fabricated shape.

### Flow 2 — Turning the system over in her hands (Wednesday, 11:30am)

1. Nomsa reopens the dependency map on the payroll folder. Yesterday's unexpected `TAXCALC` connection still nags her — she now knows *that* payroll reaches it; she needs to know *how* and *when*.
2. With `PAYROLL` selected, she switches the header perspective from **Dependency** to **Sequence**. Same scope, same modules — re-projected as call order from `PAYROLL`'s entry, `CALL`s in source order, and the `TAXCALC` message carrying its guard: `IF RUN-MODE = 'YR-END'`. So it only fires at year-end. That's why nobody mentioned it.
3. She drills in (`Enter`) — component grain: the call-flow of `PAYROLL`'s sections and paragraphs. The breadcrumb reads `payroll › PAYROLL.cbl`. One paragraph, `3000-TAX-DISPATCH`, owns the call.
4. She drills again into the paragraph — unit grain: a **flowchart** of its branches and loops, each block carrying a provenance tick. The `GO TO` edge out of the error branch is marked distinctly; she'd never have spotted it skimming.
5. On the dispatch block she hits `F12`: the **native editor splits** beside the canvas, scrolled to the exact lines, selection synced. The flowchart and the real code sit side by side.
6. **Climax:** dependency → sequence → call-flow → flowchart → real code: five gestures, no re-parse, no new tool, every hop verifiable to source. The same extracted truth, turned over in her hands until the question dissolved. `Alt+←` retraces her exact steps — flowchart, call-flow, sequence — landing back on the system map with her selection intact, and she writes her change plan with the guard condition in it.

*Failure:* she tries a perspective that doesn't apply (Sequence at unit grain) → she can't — the switcher only shows perspectives valid at the current grain. There is never an empty canvas or a dead tab.

### Flow 3 — Ask her own question, mold a tool (Thursday, 2:00pm)

1. Nomsa needs something no starter answers: *"list every paragraph that MOVEs into a `TAX-RATE` field, grouped by the copybook that defines it."* She'd also love to know where the Java invokes all this — but she already knows the answer to that one, because the toolkit told her plainly the first time she asked: COBOL side only, Java is off the map (see *Boundary*). An honest "no" she can plan around beat a confident guess she'd have had to re-check.
2. She opens **How to mold a tool**. The extension contract lays out the primitives and conventions — short, readable, provenance required on every draw.
3. She points **Claude Code** (her own assistant, her own tooling) at the contract and describes the question. It writes a small plain-TS analyzer.
4. The tool appears in the **View gallery** and runs like any starter. The new view draws the reach she asked for.
5. She doesn't take it on faith. The header names the lens — "Drawn by `tools/tax-rate-writers.ts` — read it." **Audit the lens:** she reads it end-to-end, far less than the thousands of lines it read for her. **Audit the output:** this is a tool she's never run before, so she opts into the **provenance rail** and sweeps every anchor in one pass rather than spot-checking. Two she clicks through — one lands inside a copybook, and the popover names the inclusion (`COPY PAYREC REPLACING` at `PAYROLL.cbl:88`) so the renamed field doesn't surprise her.
6. **Climax:** satisfied both ways, she commits the tool into *her own repo*, versioned with the code. She makes her change knowing what she'll hit — and next quarter's engineer will inherit her *question*, not just her code, and re-run it for free. "That hasn't happened to me before."

*Failure:* the molded tool errors at run → a plain error with its `file:line`; the toolkit does not fabricate a partial view to paper over it, and she re-reads or re-molds that one tool.

### Flow 4 — Months later, no re-molding (following quarter)

1. The COBOL has changed under a later modernization pass. Nomsa re-runs the same molded tool with the same command.
2. **Climax:** the view **re-derives** from the current code — no re-molding, no re-review of the tool, no tokens. The picture is faithful to the code as it is *today*, not as it was when she built the tool. The investment didn't go stale; it just kept working (PRD UJ-2).

*Failure:* the changed code breaks the tool (a construct it never expected) → a plain error with its `file:line`, no fabricated partial view. She re-reads the analyzer or re-molds that one tool — the other tools in the repo are untouched.
