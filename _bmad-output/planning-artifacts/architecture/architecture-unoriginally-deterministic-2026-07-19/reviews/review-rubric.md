# Rubric Review — ARCHITECTURE-SPINE.md

- **Target:** `_bmad-output/planning-artifacts/architecture/architecture-unoriginally-deterministic-2026-07-19/ARCHITECTURE-SPINE.md`
- **Reviewed against:** `.memlog.md` (same folder), `prds/prd-unoriginally-deterministic-2026-07-18/prd.md`, its `addendum.md`, `ux-designs/ux-unoriginally-deterministic-2026-07-19/EXPERIENCE.md`
- **Reviewer:** rubric walker
- **Date:** 2026-07-20
- **Severity = downstream impact** on epics/stories built from this spine.

---

## Overall

This is a genuinely good spine on the axis it cares most about: determinism. AD-1/2/3/4/7/14 form a tight, mutually-reinforcing chain, and the decision to enforce provenance at both layers (AD-4) and to make determinism falsifiable in CI (AD-14) are the two strongest things in the document. It is terse, it is a build substrate, and both diagrams are valid.

Where it is weak is the *other* half of the product — the navigation and discovery surface the UX spine owns. The spine fixes the **producer** contract thoroughly and the **consumer** contract barely: the vocabulary of node and edge kinds, the shape of a `scope`, the binding from a ViewDoc back to its snapshot, and how the extension discovers and runs a molded tool are all undecided *and* undeferred. Those are exactly the seams where two independently-built stories will diverge. Six high-severity findings, nine medium, six low.

---

## Per-checklist-item verdicts

| # | Item | Verdict |
|---|---|---|
| 1 | Fixes the real divergence points, misses none | **adequate** |
| 2 | Every Rule enforceable, Prevents follows | **adequate** |
| 3 | Nothing under Deferred hides a divergence | **adequate** |
| 4 | Named tech verified-current | **thin** |
| 5 | Covers FR-1..FR-10, nothing orphaned or mis-governed | **adequate** |
| 6 | Every owned dimension decided/deferred/open | **thin** |
| 7 | Terseness — build substrate, not prose | **strong** |
| 8 | Mermaid validity | **strong** |

Special question — AD-7 overriding a PRD §4 non-goal: **the override is authorized but dishonestly handled.** See H-5.

---

## Item 1 — Divergence points for epics/stories — *adequate*

Correctly fixed, and these are the right calls: purity boundary (AD-1), canonical node identity (AD-2 — correctly identified in the memlog as the highest-value invariant), single ordered content-addressed snapshot (AD-3), dual-layer provenance (AD-4), versioned ViewDoc (AD-5), topology-only producers (AD-6), zero-vscode core (AD-8), two-stage embedded parse (AD-9), closed honest-state vocabulary (AD-10), single output convention (AD-11), one-library-two-consumers (AD-13), CI gate (AD-14). That is a strong majority of the real surface.

Four real divergence points are missed. See **H-1, H-2, H-3, H-4**.

## Item 2 — Rule enforceability — *adequate*

Mechanically checkable and doing their stated job: **AD-1** (import/API ban, lintable), **AD-2** (minter-only, greppable), **AD-4** (signature + schema — the strongest AD in the document; a reviewer can test both halves), **AD-5**, **AD-6** ("no producer imports or calls a layout engine" is a lint rule), **AD-8**, **AD-9**, **AD-10** (closed union of exactly three — enforceable as a type), **AD-13**, **AD-14** (byte-compare, unambiguous).

Failing or weak: **AD-11** (M/H — see H-6, the Rule does not prevent its own stated divergence), **AD-12** (see M-4, vague), **AD-3**'s "a defined total order" (see M-9, defined where?), **AD-7**'s "by construction" (see M-5, overclaims relative to its own Deferred entry).

## Item 3 — Deferred section — *adequate*

- **COBOL grammar fork** — honest deferral, correctly argued as architecturally immaterial given AD-9. Sequencing caveat at L-6.
- **Layout engine** — honest, and unusually good: it names the fact that neither candidate documents determinism and points at AD-14 as the actual enforcement. That is the deferral doing its job.
- **Performance envelope** — credible. No budget can be honest pre-parse.
- **Supported VS Code / OS matrix** — mostly credible, but see L-3.
- **Operational envelope** — see Item 6.

No deferral outright hides a divergence. The gaps in this spine are *silences*, not bad deferrals.

## Item 4 — Named tech verified-current — *thin*

Checked live on 2026-07-20:

| Claim | Reality | Verdict |
|---|---|---|
| web-tree-sitter 0.26.11 | `latest` = **0.26.11** | correct |
| Node.js 24 LTS | v24 "Krypton" is the current Active LTS (v24.18.0); v26 is Current-not-LTS, v22 is Maintenance | correct |
| TypeScript 6.0 | npm `latest` = **7.0.2**. 6.0.x exists as the parallel JS-based branch (6.0.3 shipped); `beta` tag = 6.0.0-beta | **not current** |

See **M-1**. Also note: the memlog contains no entry recording any stack-version verification at all, so the header "Verified current 2026-07-19" is unsupported by the run's own record.

## Item 5 — Capability → Architecture Map — *adequate*

Sound: FR-1, FR-3, FR-4, FR-5 (modulo H-2/H-4), FR-6, FR-9, FR-10.

Problems: **FR-7 is mis-governed** (H-3), **FR-2 cites a governor that does not exist** (M-2), **FR-8's governor does not close** (M-3, L-4). Nothing is orphaned outright — every FR appears in the table — but three rows name governance the spine does not actually supply.

## Item 6 — Dimensional coverage — *thin*

**On the operational envelope specifically: the claim is credible, not a dodge — but it is incomplete.** The reasoning is sound and matches the PRD (§4 no hosted/SaaS, FR-6 no telemetry): there genuinely is no server, no tenancy, no infra provider, and "deployment" genuinely is two artifacts. Recording the assessment explicitly is the right move and the memlog shows it was reasoned about, not skipped. What it gets wrong is equating "no infrastructure" with "no operational dimension" — see **M-8**.

Whole dimensions left silent (not decided, not deferred, not an open question):

1. Node-kind and edge-kind vocabulary — **H-2**
2. Scope / grain addressing and perspective parameterisation — **H-4**
3. ViewDoc ↔ snapshot binding — **M-3**
4. Command surface (the UX spine's explicit cross-cutting requirement) — **M-7**
5. Webview trust boundary / untrusted ViewDoc rendering — **M-8**

## Item 7 — Terseness — *strong*

This is the spine's best axis. Every AD is Binds/Prevents/Rule with no padding; the Consistency Conventions table earns its keep entirely; the Structural Seed is exactly the right size. Only two mild trims (L-1, L-2). No section fails to earn its keep.

## Item 8 — Mermaid validity — *strong*

Both diagrams were extracted verbatim and run through the actual `mermaid` parser (v11, jsdom-backed):

```
d1.mmd OK {"diagramType":"flowchart-v2"}
d2.mmd OK {"diagramType":"flowchart-v2"}
```

Subgraph-with-quoted-title, the cylinder node `snap[("…")]`, the rhombus `val{"…"}`, the quoted edge label `-->|"element without span"|`, em dashes, and parentheses inside quoted labels all parse. No syntax findings.

Semantic completeness of diagram 1 is a separate issue — see **M-6**.

---

## Findings

**Counts: 6 high · 9 medium · 6 low (21 total).**

### HIGH

#### H-1 — AD-1's no-I/O rule is contradicted by grammar loading, and the spine never resolves it
**Location:** AD-1 (line ~72); Design Paradigm diagram edge `snap --> gram`; AD-3's hash input "grammar versions"; Stack row `web-tree-sitter`.

AD-1 says "nothing in `@ud/toolkit` performs I/O … Sources enter as values; results leave as values. All I/O lives in the extension." But `web-tree-sitter` requires loading a `.wasm` grammar file, and the paradigm diagram explicitly draws `snap --> gram`. Either extraction reads grammar files from disk (violating AD-1 on day one of the gate slice) or grammars must be injected as pre-loaded values — and the spine says neither. This is the first thing a builder implementing `extract()` will hit, and two stories can answer it differently. It also matters for AD-3: the snapshot hash is over "grammar versions", so how a grammar arrives and how its version is read must be fixed.

**Fix:** extend AD-1's Rule — "grammar modules enter `extract()` as already-instantiated values supplied by the caller, carrying a declared version string; the toolkit never resolves a grammar path. Grammar file resolution is shell work." Add `gram --> snap` direction accordingly or move `gram` outside the core boundary in the diagram.

#### H-2 — Node-kind and edge-kind vocabularies are unfixed, undeferred, and silent
**Location:** AD-2 (gives `<kind>` examples only), AD-6, AD-10; Capability map FR-5 row.

AD-10 correctly closes the *honest-state* vocabulary and explains exactly why a closed set matters ("perspectives inventing private ways to signal doubt"). The identical argument applies to kinds, and is not made. PRD FR-5 requires "data access is typed (`read` / `write`)"; EXPERIENCE.md requires a category legend naming "every kind present", filter-by-kind, colour-plus-line-style per kind, `invoke`/`read`/`write` as first-class in the Reach tracer, and unstructured-jump edges visually distinguished. All of that presumes a kind vocabulary the renderer knows. AD-2 constrains node ids to *contain* a kind but never enumerates or closes the set; nothing constrains edge kinds at all. Two producers emitting `write` vs `writes` vs `data-write`, or a molded tool inventing `cobol:copybook/` vs `cobol:program/` for the same artifact, silently breaks the legend, the filter, and — via AD-2's own stated harm — inbound reach.

**Fix:** new AD — "Node kinds and edge kinds are a closed vocabulary owned by the model package and versioned with the ViewDoc schema. Producers select from it; a kind not in the vocabulary fails schema validation at the render boundary (AD-4). Adding a kind is a ViewDoc schema change under semver (AD-5)." Enumerate the v1 sets (nodes: program, copybook/included-source, file, table, paragraph/section, callable, block; edges: invoke, include, read, write, unstructured-jump) or state the enumeration lives in the model package and is normative.

#### H-3 — FR-7 (view gallery) is mis-governed, and the "does the extension run molded tools?" question is left open
**Location:** Capability map row FR-7 (`extension` / AD-5, AD-11); AD-11; AD-12; AD-13.

The gallery must list molded tools *before any of them has run* — with "the question it answers", its origin, and, for a broken tool, its error and `file:line` (PRD FR-7 consequences; PRD OQ-2 names tool metadata and load-failure diagnostics as explicit contract requirements). AD-5 (ViewDoc) and AD-11 (watch an output folder) can only ever show the extension *output*, never the *tools*. Nothing in the spine governs tool discovery, tool metadata, or load diagnostics.

Worse, the underlying question is contradicted across the source set and unresolved here. EXPERIENCE.md: the gallery is a QuickPick that "runs on select", and molded tools "run identically to starters." The addendum: molded tools "run however the user or Claude Code likes (terminal, npm script)." AD-12: prevents "the extension executing arbitrary user code during navigation." AD-13: "no molded tool resolves primitives from the installed extension." So: does the extension load and execute a molded tool from the user's repo, or not? Both answers are defensible; leaving it open means the gallery story and the molding story will be built against different answers, and the FR-6/AD-13 guarantee is entangled with it.

**Fix:** decide it explicitly as an AD. Suggested shape — "A molded tool is discovered by a declared manifest/export at a conventional in-repo path carrying its question-string and grain applicability; the extension reads that manifest to populate the gallery without executing the tool. Executing a molded tool is an explicit, user-initiated run in a child Node process — never in the extension host, never during navigation. A tool whose manifest fails to load is listed with its error and `file:line`." Then re-map FR-7 to that AD plus AD-11.

#### H-4 — `scope` is undefined; the grain model and perspective parameters have no contract
**Location:** Design Paradigm (`perspective(snapshot, scope) → ViewDoc`); Structural Seed diagram; Capability map FR-5, FR-8, FR-10.

The spine's central core signature takes an argument it never specifies. EXPERIENCE.md builds its entire navigation model on it: three grains (system/component/unit), drilling moves scope down a grain, breadcrumb segments drill out, the trail restores "scope + perspective + selection" as one step (FR-8), the availability rule asks whether a perspective "can honestly derive something for this scope at this grain", and FR-10 requires a molded tool to *declare* the grain and node kinds it applies to. None of that is expressible without a fixed, serializable scope representation and a closed grain enum.

Related and equally silent: perspective *parameters*. EXPERIENCE.md has a user-controlled reach depth, a reach-direction toggle, and a Sequence entry-point selection — all of which change the ViewDoc. AD-5/AD-6 say nothing about how a perspective is parameterised or whether the ViewDoc records the parameters it was produced under (it must, for the trail and for the audit rail to be truthful).

**Fix:** new AD — "A scope is a serializable value `{ grain, nodeId }` over the closed grain enum system|component|unit, resolvable against a snapshot. Every perspective is a total function of (snapshot, scope, params); the ViewDoc records the scope and params it was produced under. A perspective declares the grains and node kinds it can derive for; the availability rule is that declaration plus a derivability check, not a hardcoded switch."

#### H-5 — AD-7's override of PRD §4 is authorized, but the spine hides it and leaves the copy rule contradicting itself
**Location:** AD-7; Consistency Conventions "Copy" row; PRD §4 last-but-two bullet; EXPERIENCE.md § Voice and Tone; `.memlog.md` entry 16.

The **override itself is legitimate.** PRD §4 explicitly hands architecture the fork: "Architecture inherits this as an explicit fork: adopt deterministic layout, or keep the narrower claim." AD-7 takes a branch the PRD authorized it to take. That is correct behaviour, and the memlog records it properly, including the upstream consequence: "UPSTREAM: PRD §4 non-goal 'Not pixel-stable view layout at v1' must be updated — architecture took the other branch."

**The handling is dishonest by omission.** That upstream note exists only in the memlog. The spine — the artifact epics will actually be written from — contains no marker anywhere that AD-7 resolves a PRD fork, no note that PRD §4 is now stale, and no reconciliation instruction. A reader with only the spine and the PRD sees AD-7 flatly contradicting a live non-goal in a document whose front matter still says `status: final`.

**It also creates a concrete, live inconsistency the spine does not notice.** AD-7's whole point (per the memlog) is that "product copy may claim 'same code, same PICTURE'." But:

- The spine's own Consistency Conventions "Copy" row governs only the "verified" ban. It does not update the determinism claim.
- EXPERIENCE.md — which the spine lists as a binding source — states the opposite as a rule: *"Determinism copy names structure, not pixels. … the spine picks structure."* Its copy table ships the literal string "Same code, same *structure* — re-run anytime, free."
- EXPERIENCE.md § Interaction Primitives also offers a user-facing **"Layout toggle → re-arrange the same deterministic graph; layout is cosmetic."* A user-selectable layout means the rendered picture is *not* a function of code alone, which directly undercuts AD-7's stated Prevents ("identical code producing differently-arranged pictures"). AD-7 says nothing about layout options.

So after AD-7, three live specs disagree on what the product may claim and whether layout is user-variable — and the spine, which caused the disagreement, is silent on all of it.

**Fix, three parts:**
1. Add an explicit line under AD-7: *"Supersedes PRD §4 'Not pixel-stable view layout at v1' — the PRD handed architecture this fork and this is the branch taken. PRD §4 and EXPERIENCE.md § Voice and Tone require an upstream edit before epics are written."*
2. Update the Consistency Conventions "Copy" row to state the determinism claim the product is now entitled to make, so the spine is internally consistent about it.
3. Resolve the layout toggle: either constrain it in AD-7 ("any user-selectable layout mode is itself a pinned deterministic function; the rendered artifact records the mode alongside engine and version") or state that the toggle is dropped.

#### H-6 — AD-11 does not name the folder, so it cannot prevent the divergence it claims to prevent
**Location:** AD-11 Rule; Consistency Conventions table.

AD-11's Prevents is "producers writing to divergent locations." Its Rule says "one conventional workspace-relative output folder" and never names it. A reviewer cannot mechanically tell compliance, and the extension-watcher story and the molded-tool-template story can each pick a plausible path (`.ud/views/` vs `views/` vs `.unoriginally-deterministic/out/`) while both fully satisfying the Rule as written. Same gap for the cache location, which the Rule requires to be "elsewhere" and git-ignored — the `.gitignore` entry has to name something.

**Fix:** name both paths in the Consistency Conventions table (e.g. output `.ud/views/`, cache `.ud/cache/`, and state that the cache path is the one added to `.gitignore`). One table row closes the finding.

---

### MEDIUM

#### M-1 — "TypeScript 6.0" is stated as verified-current; npm `latest` is 7.0.2
**Location:** Stack table, "Verified current 2026-07-19".

TypeScript 7.x is the current release line (`latest` = 7.0.2, `next` = 7.1.0-dev.20260719); 6.0.x is the parallel JS-based compatibility branch (6.0.3 shipped, and the `beta` dist-tag still points at 6.0.0-beta). Pinning 6.0 may well be the right call for a project that will consume the TS compiler API, but the spine presents it as *current* rather than as a deliberate branch choice, gives no rationale, and the memlog contains no stack-verification entry at all — so nothing in the run backs the "Verified current" header for any of the three rows.

**Fix:** either move to TypeScript 7.0 or keep 6.0 and add a half-line: `TypeScript | 6.0 (JS-branch; 7.x native port not adopted at v1)`. Add a memlog entry recording the verification so the header is supported.

#### M-2 — FR-2 is governed by a "language-profile boundary" that is not an AD
**Location:** Capability map row FR-2 ("AD-9, language-profile boundary"); Deferred, COBOL grammar fork bullet ("the language-profile boundary hold[s] whichever wins").

The spine invokes the language-profile boundary twice as a load-bearing governor and never defines it as an invariant. EXPERIENCE.md § Language Profiles specifies it precisely — a profile is the *only* place a language name belongs, adding a language means a column plus a grammar and touches neither the model nor the perspectives nor the trust rules. That is exactly an AD, and it is what actually protects FR-2 (and the whole "contribute a language" flywheel) from divergence. As written, FR-2's governance dangles.

**Fix:** promote it. New AD — "A language profile is the only place a language name appears: it maps generic concepts (compilation unit, callable, data artifact, static reference, unresolved target, included source) onto one grammar's constructs. No model type, perspective, trust rule, or renderer component may branch on language. Prevents: perspectives forking per language, and adding a language becoming a spine rewrite."

#### M-3 — A ViewDoc does not carry its snapshot identity
**Location:** AD-3, AD-5; Capability map FR-7/FR-8; EXPERIENCE.md § State Patterns "Stale snapshot" and "A view is a stated snapshot".

AD-3 makes snapshot identity a content hash and says "staleness is detected by hash, never by timestamp" — correct, and better than the UX spine's own timestamp framing. But nothing requires a ViewDoc to *carry* that hash. Without it the extension cannot render the header line EXPERIENCE.md requires ("drawn from `payroll/` at 14:02 · source changed since — re-run"), cannot warn on jump-to-source into a changed file, and cannot detect that a molded tool's output in the watched folder is stale. This is a producer/consumer contract gap, i.e. exactly a two-unit divergence.

**Fix:** add to AD-5's Rule (or the Consistency Conventions): "every ViewDoc carries the snapshot hash, the extraction version, the scope and params it was produced under, and the producing tool's identity. The extension compares that hash against the current source to decide staleness."

#### M-4 — AD-12's Rule is not mechanically checkable
**Location:** AD-12.

"A molded view supports exactly the interactions its ViewDoc can serve" — "can serve" is undefined, so a reviewer cannot tell compliance and two renderer stories can disagree about which affordances appear on a molded view. The intent (from the memlog: provenance, click-to-source, selection always work; drilling into an unemitted grain does not) is right; the Rule does not encode it.

**Fix:** make it declarative — "the renderer derives available affordances solely from what the ViewDoc contains: provenance, click-to-source and selection are always available because AD-4 guarantees spans; drill-in is offered only where the ViewDoc contains the target grain. Where an affordance is unavailable the UI states why rather than rendering it disabled or absent."

#### M-5 — AD-7 says "by construction"; its own Deferred entry says otherwise
**Location:** AD-7 title and Rule vs Deferred § "Layout engine".

The Deferred section states plainly that neither candidate engine "documents a determinism guarantee, which is why AD-14's golden files are what actually enforces it." That is the honest position — determinism here is *by verification*, not by construction. The AD-7 title claims the stronger thing, and this is a spine whose entire product thesis is refusing to overclaim.

**Fix:** retitle AD-7 "Layout is deterministic and verified as such", and add one clause: "the property is asserted by AD-14's byte-compare, not by an engine guarantee."

#### M-6 — The dependency diagram contradicts AD-2, AD-4 and AD-5
**Location:** Design Paradigm mermaid diagram (syntactically valid; semantically incomplete).

Under the stated reading ("an arrow means *may depend on*; the reverse is forbidden"), three required dependencies are absent:
- `render --> model` — AD-4 requires schema+provenance validation *at the render boundary* and AD-5 says the renderer consumes ViewDoc; it therefore must depend on the ViewDoc schema, which the diagram places in `model`.
- `layout --> model` — layout consumes ViewDoc for the same reason.
- `molded --> model` — AD-2 requires *every* producer of a node or edge endpoint to obtain ids from the minter, and the diagram places the minter in `model`. As drawn, molded tools cannot legally mint an id.

**Fix:** add the three edges. Also consider `query --> model`.

#### M-7 — The command-surface requirement is absent
**Location:** spine has no coverage; PRD §3.5 cross-cutting consequence; EXPERIENCE.md § Accessibility Floor.

Both driving specs state it as a hard cross-cutting rule: *every* navigation and trust action must be a named, contributable, remappable VS Code command, because webviews receive raw DOM key events and do not inherit editor keybindings, and platform defaults differ. That is a consistency convention every extension story must follow or the surface fragments. The spine does not mention commands anywhere.

**Fix:** one Consistency Conventions row — "Commands | every navigation and trust action is a contributed, remappable VS Code command; a webview key handler is a convenience binding onto that command, never the only route."

#### M-8 — The operational envelope is credible on infrastructure but silent on the webview trust boundary
**Location:** Deferred § "Operational envelope — assessed, near-empty."

The assessment is honest as far as it goes and I do not read it as a dodge — the PRD really does exclude servers, tenancy and telemetry, and stating "considered and empty" is better than omission. But "no infrastructure" is not "no operational surface". The architecture renders JSON that arrives from a watched folder in the user's repo — produced by user-authored tools, possibly checked in from elsewhere — inside a VS Code webview, and AD-11 has the extension watch that folder automatically. Node labels, tool names and `file:line` chips are all attacker-influenceable strings. Nothing in the spine addresses webview CSP, label sanitisation, or VS Code workspace-trust posture (should the watcher be live in an untrusted workspace at all?). AD-4's schema validation checks *shape*, not *content safety*. This is the one genuinely operational dimension the product does have.

**Fix:** either add it to the envelope bullet as decided ("the webview runs under a strict CSP with no remote resource loading; all ViewDoc-derived text is rendered as text, never markup; the output-folder watcher is disabled in untrusted workspaces") or record it as an open question. Do not leave it inside "near-empty".

#### M-9 — AD-3's "a defined total order" is never defined or located
**Location:** AD-3 Rule; Consistency Conventions "Ordering" row (which restates the same undefined phrase).

Both places say collections are "sorted by a defined total order" without saying what it is or who owns it. Compliance is therefore unfalsifiable, and two producers can each define their own order and both be conformant — which weakens the cross-producer half of the Prevents even though the within-run determinism half holds.

**Fix:** name it — "every collection is sorted ascending by node id (AD-2) using byte-wise comparison of the canonical URI; edges by (source id, target id, kind). The ordering function lives in the model package and is the only one used."

---

### LOW

#### L-1 — AD-8 largely duplicates AD-1
AD-1's Rule already ends "or imports `vscode`". AD-8's first sentence restates it. The load-bearing part of AD-8 is the *second* half ("a molded tool is a standalone Node module, not a plugin"), which is genuinely distinct. **Fix:** trim AD-8 to that clause and cross-reference AD-1, or fold it into AD-13.

#### L-2 — Design Paradigm ¶3 is memlog rationale
"This makes 'same code, same picture' true by construction rather than by discipline, and makes a molded tool exactly one pure function — the narrowest job an assistant can be given, which is what PRD SM-2's odds rest on." Accurate and well-argued, and it is `.memlog.md` entry 11 nearly verbatim. In the only section of the spine that carries prose, it is the one paragraph that earns no build decision. (It also contains the "same code, same picture" phrasing that M-5/H-5 flag as overclaimed.) **Fix:** cut to one clause or drop.

#### L-3 — The VS Code matrix deferral's justification is slightly false
"Same reason; no gate-slice work depends on it." The gate slice ships a webview panel, click-to-source, and a `.vsix` — all of which need an `engines.vscode` floor in `package.json` on the first extension story. **Fix:** state the floor (or "lowest version supporting the APIs used, set at first extension story") and keep only the OS matrix deferred.

#### L-4 — FR-8's governor collides with AD-11's cache policy
Capability map: "FR-8 … AD-3 — replay over the cached snapshot." AD-11: the cache is a derived artifact, git-ignored, reproducible from source. What the trail replays over after the cache is evicted, or after the source has changed under it, is unhandled — and PRD FR-8 explicitly requires "a restored step whose source anchors no longer exist reports that plainly." Largely resolved by M-3's fix (ViewDoc carries the snapshot hash), so treat as a consequence. **Fix:** state that trail steps store the snapshot hash and a missing-cache replay re-derives or reports staleness rather than rendering as current.

#### L-5 — The Stack table does not say which consumer each version binds
The toolkit runs on plain Node (AD-8), but the extension runs in VS Code's own bundled Node in the extension host — the maintainer does not choose it. "Node.js 24 LTS" is correct as the toolkit's target and is not something the extension can assert. **Fix:** annotate — `Node.js | 24 LTS (toolkit target; the extension host's runtime is VS Code's)`.

#### L-6 — AD-14's fixtures depend on two Deferred choices
Golden files pin both the grammar fork and the layout engine, both of which sit under Deferred. That is fine and the deferrals are honest, but the gate cannot be stood up until both land, and any later switch invalidates the entire corpus. **Fix:** one line in AD-14 or the Deferred entries noting that fixture generation is blocked on both selections and that a change to either is a deliberate fixture re-baseline, not a build failure to be worked around.

---

## What is strong and should not be touched

- **AD-4.** Enforcement at both layers, with the reason stated in one clause ("the guarantee attaches to the product, not to the happy path"). Exactly the right altitude, fully mechanical, and it is the invariant the whole product's trust claim rests on.
- **AD-10.** A closed vocabulary of exactly three, matching PRD FR-1's three honest states precisely, with ownership assigned to extraction and the inbound-reach corollary folded in. This is the model for what H-2 asks for on kinds.
- **AD-14.** Turning the headline claim into a falsifiable release gate is the single decision that most protects this product from its own marketing.
- **AD-2.** Correctly identified as the highest-value invariant; the "hand-assembled id string is a contract violation" clause makes it reviewable.
- **Terseness throughout.** The Consistency Conventions table in particular is doing a lot of work per line.
