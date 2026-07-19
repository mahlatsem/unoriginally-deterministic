# Input reconciliation: UX spines → PRD

**Date:** 2026-07-19
**Inputs:** `ux-designs/ux-unoriginally-deterministic-2026-07-19/EXPERIENCE.md`, `DESIGN.md`, `.memlog.md`
**Target:** `prd.md` (+ `addendum.md`) in this folder

The PRD's 2026-07-19 pass absorbed the *scope* decisions (OQ-3 closure, FR-5 expansion to six perspectives across three grains, copybook expansion, the third honesty state, §5.2 phasing) and the addendum absorbed the starter/molded asymmetry. What it did **not** absorb is a second class of spine output: decisions that constrain *what must be true of the product* — scope boundaries, claim limits, and capabilities — rather than how it looks.

Nine gaps below, ordered by importance. Each states the decision, where it lives in the spines, why it belongs in the PRD (or addendum) rather than only in the spines, and concrete insertion wording.

Explicitly **not** proposed: anything visual or behavioural (popover geometry, legend composition, `Esc` layering, node treatments, copy tables, flow choreography). Those correctly live in the spines only.

---

## 1. The accessibility floor is a v1 scope decision the PRD does not record

**Decision.** v1 commits to keyboard operability, theme fidelity, and non-colour-dependent meaning. It explicitly does **not** commit to assistive-technology / screen-reader support: no ARIA model for the canvas, no linearised structure alternative, no announcement contract. The justification is the PRD's own user model (§2.2 — sole v1 user; other engineers explicitly not an adoption or support target). The spine states it was recorded deliberately "because an unmet accessibility claim is exactly the overclaim this product refuses elsewhere," and flags that if v1 ever targets users beyond the maintainer, this is the first thing that must re-open.

**Where in the spines.** `EXPERIENCE.md` § Accessibility Floor, opening paragraph ("Scope of this floor — stated honestly") and the seven commitments following. Origin: `.memlog.md` line 24 (`override` — USER CORRECTION on accessibility), which also records five a11y-review findings *rejected* on this basis (C1 ARIA model + Structure view, C2 text equivalents, M3/M5 announcements, M8 accessible-name primitive).

**Why PRD, not spine-only.** This is the textbook shape of a PRD non-goal: a deliberate, justified, user-model-derived exclusion of a whole capability class, with a named re-open trigger. Today it is a scoping decision recorded *only* in a design document, which means (a) the architecture phase can't see it, (b) nothing in the PRD stops a later reader from assuming AA-with-SR-parity is implied, and (c) the re-open condition ("if v1 targets users beyond the maintainer") is invisible next to §2.2 and §5.3's rollout phases, which is exactly where it will bite — invited beta and public listing both widen the user set. The *mechanisms* (single tab stop with roving focus, focus-ring order, popover hover/dismiss rules) stay in the spine; only the commitment boundary moves up.

**Suggested insertion — §4 Non-Goals**, new bullet after "Not measuring or tracking other users":

> - **Not assistive-technology / screen-reader support.** v1 commits to keyboard operability, theme fidelity, and meaning that never depends on colour alone. It makes no commitment to AT/screen-reader support — no accessible-structure model for the view canvas, no text equivalents for meaning-bearing marks, no announcement contract. This follows from §2.2: the maintainer is the sole v1 user and other engineers are explicitly not an adoption or support target. It is recorded as a scope decision, not as a claim that the gap doesn't matter — an unmet accessibility claim would be the same overclaim this product refuses elsewhere. **Re-open trigger:** any move to a user set beyond the maintainer (§5.3 invited beta onward) must re-open this before it ships; retrofitting an accessible-structure model onto the canvas is not cheap. Behavioural floor: `EXPERIENCE.md` § Accessibility Floor.

**And §5.3 Rollout Plan**, appended to the existing sentence about graduation criteria:

> The accessibility floor (§4) is scoped to the sole-maintainer user model; re-opening it is a prerequisite for, not a consequence of, widening the user set — this should be one of OQ-6's graduation criteria.

---

## 2. The determinism claim's boundary — structure, not picture — is stated nowhere in the PRD

**Decision.** The guarantee is "same code, same **structure**," not "same code, same pixels." A graph layout may arrange identical structure differently between runs unless layout itself is made deterministic. The spine explicitly puts the choice to architecture: *either* architecture owns pixel-stable layout as an explicit requirement, *or* the copy claims structure — and the spine picks structure so the promise stays true either way.

**Where in the spines.** `EXPERIENCE.md` § Voice and Tone, final paragraph ("Determinism copy names structure, not pixels"). Related: § Interaction Primitives, "Layout toggle → re-arrange the same deterministic graph; layout is cosmetic, the derived structure is fixed."

**Why PRD, not spine-only.** This is the single most load-bearing claim in the product — it is the product's *name* — and the PRD currently asserts it unbounded. §1 Vision: "the tool runs deterministically, for free, forever, on-machine, re-deriving from current code." UJ-2: "the view re-derives correctly." Nothing anywhere bounds what "deterministically" covers. That leaves an unresolved fork the architecture phase will hit blind: pixel-stable layout is a real, non-trivial engineering requirement (seeded layout, stable node ordering, deterministic tie-breaking) that either is or isn't in scope. A spine microcopy note is not where a scope-bearing engineering requirement should be decided, and the spine itself says so by kicking the choice upward. Either the PRD states the claim's boundary, or it states pixel-stable layout as a requirement. It currently does neither.

**Suggested insertion — §3.3, FR-5 Consequences**, new bullet after "Switching perspective re-projects the existing extraction snapshot":

> - **Determinism is over derived structure, not over layout.** The same code re-derives the same nodes, edges, and provenance anchors, every run. Visual arrangement is not part of that guarantee: layout is cosmetic and user-togglable, and no product copy claims a stable picture. If pixel-stable layout is later wanted (e.g. for screenshot-diffing a view across code changes), it is an explicit additional architecture requirement — deterministic seeding and stable ordering — not something that falls out of deterministic extraction. Copy claims structure; see `EXPERIENCE.md` § Voice and Tone.

**And §8 Assumptions Index**, new entry:

> - Pixel-stable layout is **not** a v1 requirement; the determinism claim is bounded to derived structure (§3.3, FR-5). Flagged by the UX spine as an architecture-phase fork — if architecture chooses to own stable layout, the claim can be widened.

---

## 3. Every interaction must also be a remappable VS Code command — a capability requirement, currently absent

**Decision.** Every interaction primitive is *also* contributed as a VS Code command — remappable, visible in the Command Palette ("Comprehension View: Drill In / Drill Out / Jump to Source / Go Back / Toggle Rail"). Rationale is mechanical, not stylistic: webviews receive raw DOM key events and do **not** inherit editor keybindings, so a default key is a convenience and never the only route. A user who has remapped bindings, or is on a platform where the default differs (`Alt+←` is not Go Back on macOS), must retain a working path.

**Where in the spines.** `EXPERIENCE.md` § Accessibility Floor, bullet "Every primitive is also a VS Code command." Origin: `.memlog.md` line 25 (a11y finding H4, keybinding portability — explicitly KEPT).

**Why PRD, not spine-only.** This is a capability and a contract-surface requirement, not a behaviour. It says every trust action has a second, keybinding-independent invocation path; it fixes a real cross-platform correctness problem; and it grows the extension contract (a molded tool that registers as a perspective, per gap 5, presumably contributes commands too). It also has direct architecture consequences — command contribution points in `package.json`, a command-to-primitive dispatch layer, and the webview↔extension-host messaging to back it. None of that is visible to the architecture phase from the PRD today. The specific command names and the key defaults stay in the spine.

**Suggested insertion — §3.3, FR-5 Consequences**, new bullet:

> - **Every interaction is also a remappable VS Code command.** Each view interaction (drill in, drill out, jump to source, trace, switch perspective, back/forward, toggle rail) is contributed as a named, remappable command visible in the Command Palette, in addition to any default key. Webviews receive raw DOM key events and do not inherit editor keybindings, so a default key is a convenience, never the only route — a remapped or differently-platformed user keeps a working path to every trust action. Command naming and key defaults: `EXPERIENCE.md` § Accessibility Floor / Interaction Primitives.

---

## 4. "Audit the lens" is a product capability, and FR-3 currently promises something the toolkit cannot enforce

**Two coupled findings.**

**(a) Missing capability.** The PRD's §9 glossary defines trust check (a) as "reading its source (short, bounded)" but no FR requires the product to *make that possible*. The spine does: when the active perspective is a molded tool, the view header names it and offers one-click "view analyzer source" — and the spine argues this must be **ambient, not buried in a command**, because nothing else on the canvas distinguishes an unaudited molded lens from a vetted starter. Without this, one of the two named trust checks has no product surface behind it.

**(b) An unenforceable PRD claim the spine corrects.** PRD §3.2, FR-3 Consequences currently reads: *"A molded tool's own source is short enough to be read end-to-end as one of the two trust checks."* The spine explicitly refuses this framing: "the audit view states the tool's actual size rather than promising a length the toolkit cannot enforce." The toolkit does not author molded tools and cannot bound their length — the honest guarantee is that the size is *stated*, so the user can judge whether the audit is affordable. As written, FR-3 is a testable consequence that cannot be tested and a promise of exactly the kind the product refuses to make elsewhere.

**Where in the spines.** `EXPERIENCE.md` § Provenance & Trust Model, item 2 ("Audit the lens"); § Component Patterns, View header row; Flow 3 step 5.

**Why PRD.** (a) is a capability that operationalises a PRD-level concept (§9 trust checks) and is a pass criterion of SM-2 ("audit the source"). (b) is a defect in an existing PRD consequence, not new content.

**Suggested insertion — §3.2, FR-4 Consequences**, new bullet:

> - **The lens is identifiable and auditable.** When a rendered view was produced by a molded tool rather than a bundled starter, the product names the tool in the view and provides one-click access to its source — ambient, not buried behind a command, since nothing else on the canvas distinguishes an unaudited molded lens from a vetted starter. This is what makes trust check (a) (§9) performable rather than merely defined.

**Suggested edit — §3.2, FR-3 Consequences**, replace the existing "short enough to be read end-to-end" bullet with:

> - A molded tool's source is the object of one of the two trust checks (§9), so the product surfaces it directly and **states its actual size** rather than promising a length the toolkit cannot enforce — the contract makes the short path the easy one, but the user judges whether the audit is affordable.

---

## 5. OQ-2 has known inputs from the UX run that it does not list — including a hard sequencing constraint against SM-2

**Decision, two parts.**

**(a) Contract-surface requirements the UX run generated.** The spines name several things the extension contract must be able to express, all currently invisible to OQ-2:
- A molded tool can **register as a perspective**, declaring its grain and the node kinds it applies to, and then appear in the switcher contextually — not merely as a gallery entry.
- Every drawing primitive requires provenance, and elements derived through inclusion carry **both** anchors (inclusion site + definition line) plus any substitution — so provenance is a structured value, not a single span.
- A tool declares a one-line "what question it answers" for the gallery.
- A malformed tool or an invalid perspective declaration must surface as an **unloadable entry with its error and `file:line`**, never be silently dropped — implying the contract has a validation/diagnostic story.
- Primitives are also contributed as remappable commands (gap 3).

**(b) A sequencing constraint on the gate.** The spine is explicit that "every contract feature added before the reliability gate raises what an assistant must pattern-match unedited (PRD SM-2), so the gate runs against the *minimal* contract — draw primitives plus required provenance — and registration lands as a post-gate contract extension. The UX target stands; the commitment waits."

**Where in the spines.** `EXPERIENCE.md` § Views & Perspectives, "Extensible by molding — the `gtView` lesson (Phase B)"; § State Patterns, "Molded tool won't load"; § Component Patterns, Gallery item / Perspective switcher rows. Origin: `.memlog.md` line 22 (adoption 2 of the GT lessons, "requirement fed to extension-contract design (PRD OQ-2)").

**Why PRD + addendum.** §5.2 Phase B already lists "molded-tool perspective registration" as a line item, so the *deferral* is captured — but only as a delivery-order fact. The *reason* is a constraint on how SM-2 is administered: the gate must be run against a frozen minimal contract, and any contract growth before the gate invalidates the transfer of its result. That is a protocol constraint (addendum, alongside the existing SM-2 protocol) and a rationale note in §5.2 (PRD). The contract-capability list belongs on OQ-2 in the PRD, because OQ-2's whole content is "the design isn't done" — knowing what it must eventually express is the most useful thing the UX run produced for it.

**Suggested insertion — §7, OQ-2**, replace the current one-line entry with:

> 2. **OQ-2:** Detailed extension-contract design (exact API shape, conventions, templates) — the mechanism (constrained primitives + required provenance) is known; the design itself is not yet done. The UX run identified requirements the eventual design must accommodate, without settling the shape: (a) provenance is structured, not a single span — an element derived through inclusion carries both the inclusion site and the definition line plus any substitution (§3.1, FR-1); (b) a molded tool can register itself **as a perspective**, declaring its grain and applicable node kinds, so it appears contextually in the switcher rather than only in the gallery (Phase B, §5.2); (c) a tool declares a one-line statement of the question it answers; (d) an invalid or malformed tool surfaces as an unloadable entry with its error and `file:line` rather than being silently dropped, so the contract needs a validation/diagnostic story; (e) every primitive is contributable as a remappable VS Code command (§3.3, FR-5). **Constraint from SM-2:** the reliability gate runs against the *minimal* contract — draw primitives plus required provenance — because every feature added before the gate raises what an assistant must pattern-match unedited. (b)–(d) are therefore post-gate contract extensions.

**Suggested insertion — §5.2 phasing table**, appended to the Phase B rationale cell:

> Molded-tool perspective registration is deferred for a second reason beyond build cost: it grows the contract surface the SM-2 gate is measuring. The gate result only transfers if the contract it was run against is the contract the assistant faces.

**Suggested insertion — `addendum.md`, SM-2 protocol**, new numbered step after step 1:

> 1b. **The contract is frozen minimal for the duration of the gate** — draw primitives plus required provenance, nothing more. Perspective registration, command contribution, and gallery metadata (PRD OQ-2) are post-gate extensions. A gate passed against a larger contract measures a different thing than a gate passed against the one the assistant will actually face, and each added feature raises the pattern-matching load the 4-of-5 bar is testing.

---

## 6. The language-agnostic model is a v1 requirement; the PRD still reads COBOL-hardcoded in the places that matter

**Decision.** The viewing model is language-agnostic, and this was a *correction* applied late in the run: the spine had been written in COBOL nouns and was refactored so that all normative text is generic (grains = compilation unit / named callable / control flow within a callable; edges = static reference, invocation, unstructured jump), the three honesty states are restated as **general categories** (dynamic dispatch / parse-failure region / included-generated source) with COBOL as one instance, and a **Language Profiles** table is the only place a language name is allowed to appear. The stated payoff: adding a language is one column plus a grammar, with no edit to the model, the perspectives, the trust rules, or any component. The memlog frames this as protecting the PRFAQ's "contribute a language" flywheel — a first-order product dimension.

**Where in the spines.** `EXPERIENCE.md` § Views & Perspectives ("Grains are language-agnostic") and § Language Profiles. Origin: `.memlog.md` line 28 (`override` — USER CORRECTION, "the views are going to be agnostic of the language", with the full refactor recorded).

**Why PRD, partially.** The PRD *does* now say, in §3.3, "The model is language-agnostic; concrete constructs live in the UX spine's language profiles" — that half is landed. What is missing is the requirement form and the residue:

- **The additivity requirement is not stated.** "A new language is added as a profile (construct mapping + grammar) without modifying the model, perspectives, or trust rules" is a testable architectural constraint with real design consequences — it forbids language-keyed perspective code. It is currently only an aspiration in a design doc.
- **§3.1, FR-1's consequences are written as COBOL-specific facts** where the spine has made them general categories. "Copybook expansion" and "`CALL` on an identifier — dynamic dispatch" read as COBOL features rather than as COBOL's instances of *included/generated source* and *unresolved targets*. The treatment is defined once, centrally; FR-1 currently implies it's defined per language.
- **§3.1's heading and §5.1 read as "COBOL + a TypeScript exception"** rather than "two profiles, COBOL first." That framing is defensible for scope (COBOL is genuinely the only user-facing target — §4 is right) but it obscures that the *machinery* is profile-driven.

**Suggested insertion — §3.3, FR-5 Consequences**, new bullet:

> - **Language support is additive via a profile.** No perspective, trust rule, or rendering primitive is keyed to a specific language. A language is supported by supplying a tree-sitter grammar plus a profile mapping generic concepts (compilation unit, named callable, control flow, static reference, invocation, unstructured jump, dispatch, unresolved target, included source) to that language's constructs. Adding one must not require modifying the viewing model, the perspectives, or the honesty states. v1 ships two profiles — COBOL (user comprehension) and TypeScript (self-documentation, FR-2). Profile contents: `EXPERIENCE.md` § Language Profiles.

**Suggested edit — §3.1, FR-1 Consequences**, reframe the two honesty bullets to name the general category first (wording, not scope change):

> - **Included/generated source is resolved during extraction (COBOL instance: copybook expansion).** Where the source a reader sees isn't the source the compiler saw, v1 resolves the inclusion — for COBOL, `COPY` including `COPY … REPLACING` — so a callable referenced in one unit but defined in an included file resolves correctly and shared data items participate in reach. In real legacy estates this is the primary reach mechanism, not an edge case…

> - **Unresolved targets are a distinct honest state (COBOL instance: `CALL` on an identifier).** An invocation whose target is not statically resolvable — dynamic dispatch in any language — is neither resolved by heuristic nor silently dropped…

**And §4 Non-Goals**, minor clarification to the "Not broad language coverage" bullet — append:

> The *model* is language-agnostic and language support is additive by profile (§3.3, FR-5); the v1 restriction is on which profiles ship, not on the machinery.

---

## 7. Theme inheritance and the legibility floor for meaning-bearing marks

**Decision.** The extension inherits the user's active VS Code theme wholesale (light / dark / high-contrast / custom) and imposes no brand palette — framed not as taste but as posture: "imposing a brand palette on someone's editor would be the visual equivalent of the overclaim this product refuses to make." Inheritance alone does not guarantee legibility, so a floor sits beneath it: every token binding declares a fallback chain ending in `currentColor` so an undefined theme token degrades rather than silently erasing a meaning-bearing mark; composed pairs are verified at render time (3:1 marks, 4.5:1 text) with in-system fallback on failure; the provenance tick has a minimum rendered size at every zoom; and high-contrast is a supported target to be *checked*, not assumed, since HC themes work by removing fills.

**Where in the spines.** `DESIGN.md` frontmatter (fallback chains on every semantic binding) and § Colors → Contrast floor; `EXPERIENCE.md` § Accessibility Floor, "Theme fidelity, with a floor" and "A legibility floor over density"; § Responsive & Platform, "Themes: every VS Code theme … is a supported platform." Origin: `.memlog.md` lines 7, 12, 25 (a11y finding H3, KEPT).

**Why PRD, narrowly.** The ratios, token names, and fallback chains are unambiguously spine content and should stay there. Two things are not:

1. **Theme inheritance as a product constraint** — "the extension imposes no palette of its own and adopts the user's theme" is a scope/posture commitment with architectural teeth (it forbids a bundled design system with fixed colours, and it makes every custom theme a supported surface). It is currently invisible in the PRD.
2. **The mark-legibility floor as a trust requirement** — this is not cosmetics. The provenance tick is the visible form of FR-4's structural guarantee. A tick erased by an undefined theme token, or shrunk below legibility at zoom, is a *provenance failure with a visual cause*: the guarantee holds in the data and fails at the surface. That deserves a line in FR-4, whose entire subject is "the guarantee cannot be silently skipped."

Note on placement: §8 currently defers "supported VS Code versions/OS" NFRs to architecture. Supported *themes* is adjacent but different — it is a claim surface, not a compatibility matrix, and the spine's honest-limits discipline applies ("HC is a supported target to be checked, not a claim to be assumed").

**Suggested insertion — §3.2, FR-4 Consequences**, new bullet:

> - **The provenance guarantee must survive rendering.** The toolkit imposes no palette: all base styling inherits the user's active VS Code theme (light / dark / high-contrast / custom). Because inheritance carries no legibility contract, meaning-bearing marks — above all the provenance tick and the unparsed/unresolved distinction — must never be erased by an undefined theme token or rendered below legibility at any zoom. A mark that carries the trust model cannot be the most fragile pixel on screen. Contrast floor, fallback chains, and minimum sizes: `DESIGN.md` § Colors → Contrast floor.

**And §8 Assumptions Index**, new entry:

> - Supported themes: every VS Code theme including high-contrast is treated as a supported surface (`EXPERIENCE.md` § Responsive & Platform). High-contrast is a target to be *verified*, not an assumed claim — it works by removing fills, so state distinctions must survive on border, line style, or text alone.

---

## 8. A view is a stated snapshot — the anti-staleness guarantee is a trust requirement, not a behaviour

**Decision.** Extraction happens once per run; the view always states what it was drawn from and when. When underlying files change, the view does **not** silently re-derive mid-reasoning *and does not quietly go on lying*: the header states that source has changed, and a jump-to-source into a changed file says so rather than landing confidently on a line that may have moved. Re-running is always the deliberate act producing a new snapshot. Trail steps whose anchors no longer exist after a re-derive restore with the same stale notice rather than silently rendering.

**Where in the spines.** `EXPERIENCE.md` § Views & Perspectives, "A view is a stated snapshot"; § State Patterns, "Stale snapshot" and "Re-run on changed code".

**Why PRD.** The PRD has half of this. FR-5 says "Switching perspective re-projects the existing extraction snapshot: no re-parse, selection preserved" — the performance half. FR-6 says re-running re-derives with no re-molding — the durability half. Neither covers the failure mode in between, which is a **provenance-correctness** issue and therefore FR-4 territory: a stale view's anchors can point at lines that have moved, meaning click-to-source lands on a line that does not contain what the drawn element claimed. That is precisely the surprise FR-1's dual-provenance bullet already guards against for inclusion — the same guarantee, a different cause. It is a testable capability, not a visual treatment.

**Suggested insertion — §3.2, FR-4 Consequences**, new bullet:

> - **A view never claims to be current when it isn't.** Every rendered view is a stated snapshot: it names what it was drawn from and when. When source has changed since the draw, the view says so rather than continuing to present anchors that may no longer point where they did, and a trace into a changed file states the risk instead of landing confidently on a moved line. Views never silently re-derive mid-reasoning; re-running is always the user's deliberate act (UJ-2).

---

## 9. Honest limits: the boundary is stated, never silently returned smaller — and there is no export/share surface in v1

**Two smaller decisions with the same root discipline.**

**(a) Stated limits.** The spine converts several PRD *exclusions* into product *behaviour requirements*: cross-language reach is out of v1 (PRD §4), and the view must **say so** at the boundary rather than "quietly returning less than the question wanted" — "an honest edge of the map, marked as such, is the same discipline as an honest hole." Same rule for depth truncation ("reach depth 2 — 3 further hops collapsed"), collapse ("says what it collapsed"), filtering ("says what it hid"), thin derivation ("2 of 14 calls have static targets"), and absent perspectives (overflow lists them with a one-line reason). Flow 3 makes the product point explicitly: "an honest 'no' she can plan around beat a confident guess she'd have had to re-check."

**(b) No export/share affordance.** "Sharing is out-of-product. Showcasing a dogfood outcome (PRD UJ-3 / SM-3) happens by screenshot or by publishing the view's own repo-committed tool — there is no export, share, or publish affordance in v1, and no tracking of any kind. 'Copy anchor' yields a literal `file:line`; that is the extent of it."

**Where in the spines.** (a) `EXPERIENCE.md` § Views & Perspectives → Boundary + Availability rule; § State Patterns, "Thin derivation" / "Nothing to draw"; § Interaction Primitives, filter/collapse. (b) § Foundation, "Sharing is out-of-product."

**Why PRD.** (a) is a generalisable product requirement that the PRD only expresses in one instance — FR-5's "a perspective… renders and states its own thinness rather than being silently withheld." The general rule (every limit the product hits is stated, never silently applied) is the same trust principle as fail-loud parsing and belongs alongside it, especially since it converts §4's non-goals from paper exclusions into observable behaviour. (b) is a genuine MVP scope statement — SM-3 depends on showcasing, and the PRD does not currently say *how* showcasing happens or that no product surface supports it; a reader could reasonably assume an export feature is implied by SM-3. The PRD's existing telemetry non-goal is adjacent but distinct (no tracking ≠ no export).

**Suggested insertion — §3.3, FR-5 Consequences**, new bullet:

> - **Every limit the product hits is stated, never silently applied.** Depth truncation, subtree collapse, category filtering, thin derivation, an unsupported language, and the cross-language boundary (§4) are all rendered *and named* — the view says what it collapsed, hid, could not resolve, or does not know. Returning quietly less than the question asked for is the same failure as fabricating: a limit the user can see is a limit they can plan around.

**And §5.4 Out of Scope for MVP**, replacing the current single line:

> Every v1 exclusion is captured in §4 (Non-Goals). One additional MVP-surface note: there is **no export, share, or publish affordance** in v1. Showcasing a dogfood outcome (SM-3, UJ-3) happens by screenshot or by publishing the molded tool from the user's own repo; "copy anchor" yields a literal `file:line` and that is the full extent of the sharing surface.

---

## Deliberately not proposed

Checked against the spines and judged correctly spine-resident, or already landed:

- **Already in PRD/addendum:** OQ-3 closure and the six-perspective/three-grain model (§3.3); copybook expansion inside the gate slice (§3.1, §5.2, addendum); dual provenance anchors for inclusion (FR-1); the unresolved-target third honesty state (FR-1); Phase A/B sequencing and "Phase A must stand alone" (§5.2); the starter/molded update asymmetry and its gallery legibility (addendum, 2026-07-19); state-machine as moldable candidate rather than bundled perspective (§3.3 Out of Scope); zero-friction first run (FR-5); no run-time AI (FR-4).
- **Spine-only, correctly:** all component visuals and token bindings; the dual provenance-surface split (popover = spot-check, rail = full audit) and the rail's opt-in/never-auto-open rule; `Esc` layering and the structural-vs-temporal retreat distinction; the trail's two-histories seam; canvas-as-single-tab-stop and focus-ring order; peek-before-spawn drill preview; per-perspective behavioural specs; the microcopy do/don't table and banned phrasings; the GT Miller-columns rejection; Nomsa's flows.
- **Borderline, left in the spines:** the "canvas is a single tab stop" pattern (mechanism serving gap 1's keyboard commitment, not a separate requirement); reduced-motion (behavioural, and gap 1's floor covers the commitment); the QuickPick choice for the gallery (implementation, and the addendum's thin-extension decision already implies it).

## Suggested application order

Gaps 1, 2, 3 are the ones that change what architecture must build or bound what the product may claim — apply first. Gap 5 should be applied before architecture consumes OQ-2. Gaps 4 and 6 are corrections to existing PRD text as much as additions. Gaps 7, 8, 9 are single-bullet additions to existing consequence lists.
