---
title: Adversarial review — ARCHITECTURE-SPINE.md
target: '{planning_artifacts}/architecture/architecture-unoriginally-deterministic-2026-07-19/ARCHITECTURE-SPINE.md'
method: divergence-pair construction (two units one level down, both AD-compliant, incompatible builds)
reviewed: 2026-07-20
verdict: structurally sound paradigm, leaky contract — 15 divergence pairs
---

# Adversarial Review — Architecture Spine

## Method

For each attack I construct a concrete **pair of units one level down** — two epics, two perspectives, two molded tools, two profiles, extraction vs renderer — where **both units obey every AD to the letter** and the two builds are nonetheless **incompatible**. A pair that survives is a hole: the spine's ADs did not constrain enough to make the second builder arrive where the first did.

The spine's paradigm choice (functional core / imperative shell) is correct and load-bearing, and AD-4's two-layer provenance enforcement is genuinely well-built — it is the only invariant in the document that attaches its guarantee to the product rather than the happy path. Most of the findings below are the same failure repeated: **the spine names an owner but not an algorithm, or names a format but not a vocabulary.** Both leave two compliant builders free to diverge.

**Findings: 15 divergence pairs — 7 critical, 5 high, 3 medium.**

Critical = a compliant build produces a *silently wrong answer* (an inbound reach that looks complete but is partial, an id that doesn't join, a hole rendered as a fact). These attack FR-4/FR-5 directly and are the failure modes the PRD says would break the business.

---

# CRITICAL

## C-1 — AD-2 owns the id *format*, not the id. The minter takes a string.

**The pair.** Two producers, both compliant.

- **Unit A — the COBOL file-declaration extractor.** Reads `SELECT TAX-RATES ASSIGN TO TAXRT01.` and calls the minter with the logical SELECT name: `mint('cobol', 'file', 'TAX-RATES')` → `cobol:file/TAX-RATES`.
- **Unit B — the stage-2 SQL reader (AD-9).** Reads `EXEC SQL SELECT … FROM payroll.tax_rates END-EXEC` and calls the minter with what the SQL text said: `mint('cobol','table','payroll.tax_rates')` → `cobol:table/payroll.tax_rates`.

Both obeyed AD-2 to the letter: neither hand-assembled a string, both went through the toolkit's minter, both produced `<lang>:<kind>/<canonical-name>`.

**Why this is the deepest hole in the spine.** AD-2's stated purpose is to prevent "two tools naming the same table differently." But AD-2 constrains only the *shape* of the output and the *provenance* of the function call. The minter's **input is unconstrained**, so canonicalization — the actual identity decision — is performed by the *caller*, one level down, by every producer independently. AD-2 as written is a naming convention wearing an invariant's clothes.

Concretely undecided, all of which two builders will decide differently:

| Question | Builder A | Builder B |
|---|---|---|
| Case | COBOL is case-insensitive → upper-fold | preserve source case |
| Hyphen vs underscore | `TAX-RATES` | `TAX_RATES` |
| SQL qualification | strip schema → `tax_rates` | keep → `payroll.tax_rates` |
| SQL folding | unquoted identifiers fold upper (DB2) | preserve as written |
| Logical vs physical file name | SELECT name `TAX-RATES` | ASSIGN/DDNAME `TAXRT01` |
| `PROGRAM-ID` quoted literal | strip quotes | keep |
| Unicode | raw bytes | NFC-normalized |

**Consequence.** `TAX-RATES` and `payroll.tax_rates` are two nodes. Inbound reach on either returns a **partial answer that looks complete** — verbatim the failure AD-2 was written to prevent, verbatim the failure FR-5 calls "the one failure mode with the power to break the running business," and it arrives *through* the invariant rather than around it. Worse: the view renders "No inbound references found statically" with a dynamic-dispatch count beside it, so the honest-state machinery makes the wrong answer look *more* trustworthy.

**Close it — tighten AD-2:**

> The minter does not accept a name. It accepts a **typed construct descriptor** — the parsed construct, its language profile, and its source span — and performs canonicalization itself. Canonicalization is a single published algorithm owned by `toolkit/model`: Unicode NFC; case folded per the profile's declared fold rule (COBOL: upper); the profile declares one **naming authority** per node kind (files are identified by their `ASSIGN`/external name, not the SELECT logical name; tables by their **unqualified, upper-folded** name with schema carried as an attribute, never as part of identity). No producer, bundled or molded, may pass a free-form string to the minter — the signature makes it impossible, the same way AD-4 makes a spanless draw impossible. The canonicalizer carries a version that participates in the AD-3 content hash and the AD-14 gate.

**Add the collision rule** (nothing in the spine handles it): two distinct constructs that canonicalize to the same id — two files declaring the same `PROGRAM-ID`, a file and a table with the same name in the same kind — must be **reported by extraction as a named conflict**, never silently merged and never silently last-write-wins. A silent merge here is a fabricated edge with real provenance on both ends.

---

## C-2 — Inclusion-derived identity is unanchored. Copybook expansion mints two different node graphs.

**The pair.** Two bundled perspectives, both compliant, over a copybook `PAYREC` defining paragraph `3000-TAX-DISPATCH`, included by `PAYROLL.cbl` and `BILLING.cbl`.

- **Unit A — Dependency map.** Anchors identity at the **expansion site**, following AD-2's own worked example `cobol:paragraph/PAYROLL#3000-TAX-DISPATCH`. Result: two nodes — `…/PAYROLL#3000-TAX-DISPATCH` and `…/BILLING#3000-TAX-DISPATCH`.
- **Unit B — Reach tracer.** Anchors identity at the **definition site**, reasoning that FR-1's dual-anchor rule means the definition line is the element's home and that a shared copybook item must be one node for "copybook-shared data items participate in reach" (FR-1) to be true. Result: one node — `cobol:paragraph/PAYREC#3000-TAX-DISPATCH`.

Both used the minter. Both produced the AD-2 format. Both carry dual anchors per the Consistency Conventions table. **The spine's convention row settles the *span* question and leaves the *identity* question wide open** — and dual anchors actively invite the mistake, because an element with two source anchors reads as though it may have two homes.

**Consequence.** The two perspectives produce node sets that cannot join. Selection preservation on perspective switch (FR-5, "selection preserved") silently fails — the selected id does not exist in the other's ViewDoc. FR-8 trail restore stores a selection that later cannot be found. Inbound reach on the copybook paragraph returns 1 caller or 2 depending on which lens asked. Cross-tool composition — AD-2's other stated purpose — is dead.

`COPY … REPLACING` sharpens it: if `PAYROLL` includes `PAYREC REPLACING ==TAX== BY ==LEVY==`, does the node canonicalize on the pre- or post-substitution name? Two builders, two answers, both defensible.

**Close it — new AD:**

> **AD-15 — Inclusion-derived identity is anchored at the definition site; expansion is an attribute, not an identity.** An element that arrives through textual inclusion is one node, identified by its **defining** compilation unit, regardless of how many units include it. Each inclusion produces an `included-in` edge from the including unit, carrying the inclusion-site anchor and any `REPLACING` substitution. The canonical name uses the **pre-substitution** identifier (the name as the copybook defines it); the substituted name is display metadata and appears in the popover per FR-1, never in the id. AD-2's `cobol:paragraph/PAYROLL#…` example is amended: the qualifier is the *defining* unit.

*(If the maintainer prefers the other branch — identity per expansion — that is equally implementable, but then FR-1's "copybook-shared data items participate in reach" needs an explicit merge rule and the spine must state it. What is not survivable is leaving it to the implementer.)*

---

## C-3 — Node **kind** vocabulary is unowned. AD-2 fixes the slot and not its contents.

**The pair.** Two language-profile epics.

- **Unit A — the COBOL profile epic** implements the EXPERIENCE language-profile table's "declared file, database table" as kinds `file` and `table`.
- **Unit B — the stage-2 SQL epic** (AD-9) needs to distinguish a table from a view from a cursor's underlying table and mints `sqltable`, `view`, `cursor`.

Both obeyed AD-2 exactly — `<lang>:<kind>/<name>` with a minter-produced id.

**Consequence.** `cobol:table/TAX_RATES` and `cobol:sqltable/TAX_RATES` are different nodes. This is C-1's failure re-entering through the kind slot instead of the name slot, and it is *more* likely, because the kind slot has no example-driven pressure toward convergence at all. Downstream: FR-10's registration declares "the grain and node kinds it applies to" — against an open vocabulary, a molded perspective's applicability predicate is unmatchable. The legend enumerates kinds; an open set means no legend can be specified. The grain model (system / component / unit) is defined over node kinds; an unknown kind has no grain, so drilling has undefined behavior.

**Close it — new AD:**

> **AD-16 — The node-kind vocabulary is closed and owned by `toolkit/model`.** Kinds are an enumeration in the model, each declaring exactly one grain and whether it is a leaf (data artifacts are leaves — EXPERIENCE). A language profile *maps its constructs onto* existing kinds; it may not introduce one. Adding a kind is a model change under AD-5 semver, reviewed once, not a profile's private decision. The minter rejects an unknown kind at the type level.

---

## C-4 — **Edge kind** vocabulary is entirely unowned, and the reach queries depend on it.

**The pair.** Two bundled perspectives.

- **Unit A — Dependency map.** EXPERIENCE says its edges are "**static reference** — one node naming another (invocation, inclusion, data access)". It emits one kind: `static-reference`, category-coloured by endpoint kind.
- **Unit B — Reach tracer.** EXPERIENCE says "Edge kind (read / write / invoke) is first-class because *writes* carry the risk." It emits `read`, `write`, `invoke`.

Both obey AD-6 ("a ViewDoc carries nodes, edges, spans and **kinds**") to the letter. AD-6 constrains what a ViewDoc *does not* carry (coordinates) and never constrains the kind vocabulary. Add a molded tool emitting `calls`, and a third emitting `invokes`, and every AD still holds.

**Consequence, three ways:**

1. **The legend cannot be specified.** EXPERIENCE requires a category legend "naming every kind present in the current view," each with a non-colour cue, and click-to-filter. With an open vocabulary the renderer cannot assign a stable colour or line style to a kind, so the same relationship renders differently in two views — and colour-plus-line-style is an *accessibility floor commitment*, not decoration.
2. **`read`/`write` typing is a product requirement that no invariant protects.** FR-5: "Data access is typed (`read`/`write`)." "What writes to TAX_RATES" — the question the PRD says a modernization estate asks most — is a **query over edge kind**. If the dependency map wrote `static-reference` for the same relationship, the reach tracer's inbound-write query misses it. Partial answer, complete appearance, again.
3. **Cross-tool composition is impossible.** Two molded tools' ViewDocs cannot be reasoned over jointly, which was half of AD-2's stated value.

**Close it — new AD:**

> **AD-17 — The edge-kind vocabulary is closed, owned by `toolkit/model`, and semantically typed.** v1 kinds: `invoke`, `include`, `read`, `write`, `jump` (unstructured), `sequence` (source-order message). Each declares its direction semantics, its legend cue (colour **and** line style), and whether it participates in outbound reach, inbound reach, or both. Producers select a kind; they never name one. **Pre-gate, molded tools may use only these kinds** (OQ-2's frozen-minimal constraint). Post-gate, an extension mechanism admits namespaced kinds `x:<toolId>/<name>` which render with a generic cue and **never participate in reach queries** — so an unknown kind can never make a reach answer look more complete than it is.

---

## C-5 — Two owners of the construct→kind mapping: the host profile and the stage-2 parser both classify.

**The pair.** AD-9 says stage 2 "reads that region's contents into the same node/edge model **through the AD-2 minter**." It does not say who decides *what kind* the resulting node is.

- **Unit A — the COBOL host profile** sees `SELECT TAXRT ASSIGN TO TAXRT01. FD TAXRT.` plus `READ TAXRT` and classifies TAXRT01 as `cobol:file/TAXRT01` with a `read` edge.
- **Unit B — the stage-2 SQL parser** sees `EXEC SQL SELECT … FROM TAXRT END-EXEC` and classifies TAXRT as `cobol:table/TAXRT` with a `read` edge.

Both went through the minter. Both obeyed AD-9's two-stage pattern. Neither is wrong in isolation — in a real estate the same physical artifact genuinely is reachable both ways, and a modernizer needs one node.

**Consequence.** One artifact, two node ids, two disjoint inbound sets. Selecting the table and tracing inbound — EXPERIENCE calls this "the canonical modernization question" — returns only the SQL half and reads as complete. This is a *structurally guaranteed* divergence, not a slip: the spine has literally two parsers, in two epics, both authorized to classify.

**Close it — tighten AD-9:**

> Stage-2 parsers **do not classify and do not mint**. A stage-2 parser yields *unclassified construct descriptors* (raw name, role, span) to the **host language profile**, which is the single owner of the construct→kind mapping and the sole caller of the minter (AD-2 as tightened by C-1). Where a host declaration and an embedded reference resolve to the same canonical artifact, the profile emits **one** node and both access edges; where they cannot be shown to be the same artifact, they are two nodes and the ambiguity is stated, never silently merged. A profile's mapping table is data, reviewed once, not scattered logic — which is also what makes adding a language a table change rather than a model change (EXPERIENCE, *Language Profiles*).

---

## C-6 — AD-3 says "a defined total order" and the spine never defines it.

**The pair.** Two implementers, both quoting AD-3.

- **Unit A — extraction.** Sorts by node id with JavaScript's default comparator (UTF-16 code-unit order). `#` (U+0023) sorts before `-` (U+002D) sorts before uppercase letters.
- **Unit B — a molded tool** (or the layout pre-pass, or a second perspective) sorts by `Array.prototype.sort` with `localeCompare` / `Intl.Collator`, reasoning that human-legible order is the "defined total order."

Both are total orders. Both are defined. Both are deterministic *on one machine*.

**Consequence, and it is worse than reordering.** `Intl.Collator` output depends on the ICU version and locale bundled with the Node build. AD-14 byte-compares laid-out output in CI. Golden files therefore pass on the maintainer's machine and fail on CI, or — far worse — pass in CI and produce a **different picture on a user's machine**, falsifying "same code, same picture" in front of the user's own eyes, which AD-7 exists to prevent. Every AD is obeyed throughout.

Two further undefined things inside the same words:

- **Order by what key?** Ordering nodes by id and ordering nodes by source span `(file, line, col)` are both total orders over the same set and produce different layered layouts. AD-7 feeds layout from this ordering, so the key choice *is* the picture.
- **No tiebreaker is specified, so the order is not actually total.** Two `CALL 'TAXCALC'` statements in one paragraph produce two edges identical in source, target, and kind. Sorting by `(source, target, kind)` leaves them tied; `sort` stability then depends on input order, which depends on traversal, which is the nondeterminism AD-3 was written to kill.

**Close it — tighten AD-3:**

> The total order is **one named comparator exported by `toolkit/model`**, and nothing else may sort a collection that escapes a producer. It compares **UTF-8 byte sequences, locale-independent** — `localeCompare`, `Intl.*`, and any locale-sensitive API are banned in `@ud/toolkit` and enforced by lint, in the same spirit as AD-8's `vscode` ban. Nodes sort by id. Edges sort by `(sourceId, targetId, kind, span.file, span.startLine, span.startCol, span.endLine, span.endCol)` — total by construction, no ties. Honest-state elements sort within their host collection by the same rule. The comparator carries a version that participates in the AD-3 content hash and fails the AD-14 gate on change.

---

## C-7 — A perspective can encounter a case extraction never classified, and AD-10 is *closed*.

**The pair.** `CALL 'TAXCALC'` — a literal, statically resolvable target — where `TAXCALC.cbl` is **not in the snapshot's file set** (outside the scoped folder, in another repo, or the Java side the PRD puts explicitly off the map).

AD-10 offers exactly three states and none fits: the parse succeeded (not *unparsed region*); the target **is** statically knowable (not *unresolved target*); nothing was `COPY`d (not *unresolvable inclusion*). Extraction, obeying a closed vocabulary it may not extend, assigns nothing. Now:

- **Unit A — Dependency map** draws `cobol:program/TAXCALC` as an ordinary node. It has an id and a span (the CALL site), so AD-4 passes, AD-2 passes, the render boundary passes. **The view asserts a program that was never read, with a provenance tick, indistinguishable from one that was.**
- **Unit B — Reach tracer** drops the edge, since it cannot resolve the endpoint into the snapshot. **A silent hole** — the precise thing FR-1 and EXPERIENCE ban.

Both perspectives obeyed AD-10 to the letter: neither invented a state, neither suppressed one that extraction assigned.

**Consequence.** Outbound reach either overclaims (A) or underclaims (B), and the disagreement is invisible. EXPERIENCE's *Boundary* section demands the cross-language edge be shown and marked — but with no state to mark it *with*, that requirement is unimplementable under AD-10 as written. AD-10's closure is correct in intent and **wrong in cardinality**: it closed the set before enumerating the cases.

**Close it — extend AD-10 to four extraction-assigned states:**

> **Out-of-snapshot reference** — the reference resolved to a named target, and that target is not in the extracted file set. It carries the reference-site span (AD-4 holds), renders visually distinct from both a read node and an unresolved target, and **transitive reach stops there and says so** — the same discipline dynamic dispatch already gets. This is the state that carries EXPERIENCE's cross-language boundary, out-of-scope folders, and missing modules. Extraction assigns it; a perspective that encounters an unclassifiable case **fails validation at the render boundary rather than choosing between A and B** — a producer must never be the one to decide whether an unknown becomes a fact or a hole.

Add the general rule, because a fifth case will arrive: *the honest-state vocabulary is closed but the closure is a model version under AD-5; a perspective encountering an unclassified condition is a bug in extraction, surfaced as such, never resolved locally.*

---

# HIGH

## H-1 — AD-2 has no render-boundary enforcement. AD-4's own lesson is not applied to identity.

**The pair.**

- **Unit A — a TypeScript molded tool** imports `@ud/toolkit` and calls the minter. Compliant.
- **Unit B — a Python script** (or a shell pipeline, or a hand-edited file) that writes valid ViewDoc JSON into the conventional output folder with plausible ids like `cobol:program/PAYROLL`. AD-11 makes the folder the exchange surface; AD-5 makes the renderer blind to its producer; AD-13's "no marketplace call" durability argument positively *encourages* producers the toolkit never saw. Unit B violates no AD except AD-2's unenforced sentence, and nothing detects it.

AD-4 states the principle exactly right — "the guarantee attaches to the product, not to the happy path" — and then applies it only to spans. AD-2 is left as a type-level convention on one of two entry paths, which is the same architecture AD-4 explicitly rejects.

**Consequence.** A hand-written id that differs by one canonicalization choice (C-1) renders as a real node and silently forks the graph. The highest-value invariant in the spine has the weakest enforcement.

**Close it — tighten AD-4 to cover identity:**

> Render-boundary validation checks ids as well as spans. Every node id must (a) parse as `<lang>:<kind>/<name>` with `kind` in the closed vocabulary (AD-16), (b) be byte-identical to what the canonicalizer produces for its own name, and (c) **resolve against the snapshot the ViewDoc names**. An id that does not resolve renders as `out-of-snapshot reference` (C-7) or is rejected — never as an ordinary node. This makes ids as unforgeable as spans, through the same mechanism, at the same boundary.

---

## H-2 — Scope and perspective have no identity, and FR-8's trail restore is defined over both.

**The pair.** `perspective(snapshot, scope) → ViewDoc`. The spine never says what a `scope` *is*.

- **Unit A — Dependency map** takes scope as a workspace-relative **folder path** (`payroll/`) — EXPERIENCE's "drawn from `payroll/` at 14:02" and "opens fit-to-canvas at whole-system scope" both read that way.
- **Unit B — Flowchart** takes scope as a **node id plus grain** (`cobol:paragraph/PAYROLL#3000-TAX-DISPATCH`, unit grain) — the only thing that makes sense for "control flow inside one callable."

Both comply. There is no AD on scope at all.

**Consequence.** FR-8 requires back/forward to "restore scope, perspective, *and* selection **together, as one restored step**," explicitly across perspective changes two grains apart. With two scope value shapes, a trail step is not a uniform value, so it cannot be compared, serialized, or restored across a perspective boundary — which is the trail's entire purpose. The breadcrumb (`payroll › PAYROLL.cbl › CALC-TAX`) mixes a folder segment and node segments and has no defined type, so "each segment click = drill out to that grain" is unimplementable uniformly.

**Perspective identity is missing too.** A trail step stores "perspective" as *what*? A display name collides across two molded tools in different files; a bundled perspective renamed at release breaks every stored trail; FR-10 registration needs a stable key to match context against; FR-7's gallery needs one to dedupe.

**Close it — new AD:**

> **AD-18 — Scope and perspective are minted values with canonical serializations.** A `Scope` is a closed model type `{ grain, root: NodeId | WorkspaceRoot, snapshotHash }`, minted by the toolkit and comparable by value. A `PerspectiveId` is minted: `bundled:<stable-slug>` for shipped perspectives (slug frozen across releases; display names may change freely), `molded:<workspace-relative-path>` for in-repo tools. A trail step is exactly `{ snapshotHash, scope, perspectiveId, selection: NodeId[] }` — a value, fully serializable, restorable without re-running anything, satisfying FR-8's "replays over the existing snapshot."

---

## H-3 — The shell's mutable state has no owner, and two extension features can write it concurrently.

The paradigm makes the core pure and then hands the shell **three pieces of mutable state — snapshot cache, view trail, current selection/scope — and assigns none of them an owner.** AD-1 governs the core; nothing governs the shell. This is the classic functional-core/imperative-shell failure: the discipline the core was spared reappears, unmanaged, on the other side of the boundary.

**The pair.** Two extension epics.

- **Unit A — FR-8 trail epic** pushes a step on every drill and perspective switch, and on Back writes `(scope, perspective, selection)` back into the live view.
- **Unit B — FR-7 gallery / AD-11 watcher epic.** A molded tool re-run drops a fresh ViewDoc into the watched folder; the watcher re-renders. Does that mutate selection? Push a trail step? Invalidate the cache? **No AD says**, so Unit B's author decides independently of Unit A's.

Both comply.

**Consequence.** A watcher event landing mid-drill mutates scope and selection underneath a trail push, producing a step that restores a selection which never existed in that ViewDoc. Trail steps reference a `snapshotHash` the cache may have evicted — FR-8 promises "retracing never re-runs extraction," so either the promise breaks or a restore silently re-parses and renders stale data as current.

**A second, sharper pair inside AD-11.** AD-11 fixes the output *folder* and never the *filename*. Molded tool `deps.ts` and bundled dependency map both write `dependency-map.json`; the watcher renders whichever landed last, attributing it to whichever the header names. **That defeats trust check #2 (audit the lens) at its root** — the user reads the wrong analyzer.

**Close it — new AD:**

> **AD-19 — One owner of shell state; ViewDoc filenames are minted.** All shell state (snapshot cache, trail, scope, selection) lives behind a single session-state owner in `@ud/extension`; the watcher, gallery, renderer, and trail all *dispatch* transitions and none writes directly. Transitions are serialized; a watcher event arriving during navigation is applied as an explicit, ordered transition that **states** what changed rather than silently replacing the view. Cache eviction may not evict a snapshot any live trail step references. ViewDoc filenames are derived by a toolkit function from `(producerId, scopeDigest, snapshotHash)` — two producers cannot collide, and a stale doc is identifiable as stale by name rather than by trust.

---

## H-4 — AD-5 says "semver" and never says what a reader does with a version it doesn't know.

**The pair.** AD-13's whole point is that the npm package and the extension version independently — a molded tool may be built against a *newer* toolkit than the installed extension, and FR-6's durability promise means one may be frozen against a much *older* one.

- **Unit A — a strict renderer** rejects any ViewDoc whose `schemaVersion` it does not exactly know.
- **Unit B — a lenient renderer** ignores unknown fields and renders what it recognizes.

Both are ordinary, defensible semver readings. Neither is constrained by AD-5.

**Consequence.** Unit A breaks FR-6 — the frozen molded tool, the exact artifact the versioning exists to protect, stops rendering. Unit B is worse: schema 2.1 adds a meaning-bearing field (a fourth honest state per C-7, or edge direction per C-4), the 2.0 renderer drops it, and the view renders **looking complete while omitting a hole** — fabrication by omission, through the stability contract itself.

**Close it — tighten AD-5:**

> Every ViewDoc carries `schemaVersion` and a `mustUnderstand` list naming the extensions a reader must comprehend to render it **honestly**. A renderer accepts same-major documents whose `mustUnderstand` entries it all knows; an unknown entry, or a major mismatch, means **refuse to render with a plain message naming the version gap** — never a partial render, because a partial render of an honest-state extension is precisely the confident omission this product exists to refuse. Honest states and edge kinds are always `mustUnderstand`. The renderer supports the current major and the one before it, stated in the README as the FR-6 window. AD-14's fixture corpus includes a **frozen ViewDoc at each supported schema version**, so the durability promise is gated rather than asserted.

---

## H-5 — AD-3's content hash may not cover file contents, and its path canonicalization is unowned.

AD-3: "content hash over grammar versions + **canonicalized file set** + extraction version."

**The pair.**

- **Unit A** reads "file set" literally — a set of paths — and hashes the sorted path list. Editing a file's contents without adding or removing a file leaves the hash **unchanged**.
- **Unit B** reads it as intended and hashes `(path, contentHash)` pairs.

**Consequence for Unit A.** AD-3's own promise — "staleness is detected by hash, never by timestamp" — inverts into staleness being *undetectable*. EXPERIENCE's stale-snapshot state never fires; a jump-to-source lands confidently on a moved line; FR-8 restores a stale step "as if current," the one thing FR-8 explicitly forbids.

**Second pair, inside "canonicalized."** Unit C hashes absolute paths; Unit D hashes workspace-relative POSIX paths. Both "canonicalized." Absolute paths make the hash **machine-dependent**, so AD-14's byte comparison fails between the maintainer's machine and CI for identical code — and the failure looks like a determinism bug in the layout engine, which is where the team will spend a week looking.

**Close it — tighten AD-3:**

> Snapshot identity is `hash(extractionVersion, canonicalizerVersion, comparatorVersion, [(workspaceRelativePosixPath, contentHash)] sorted by the AD-3 comparator, [(languageId, grammarId, grammarVersion)] sorted, [(languageId, profileVersion)] sorted)`. Paths are workspace-relative, POSIX-separated, NFC-normalized, symlinks unresolved. No absolute path, no timestamp, no machine identity may enter the hash. Machine-independence is the property being bought.

---

# MEDIUM

## M-1 — The "layout toggle" is a user-controlled input to the picture, and no AD reconciles it with the determinism claim.

AD-7 pins the engine and version; the memlog upgrades the product claim to "same code, same **picture**." But EXPERIENCE ships a **layout toggle** ("re-arrange the same deterministic graph; layout is cosmetic") and AD-7 pins a *version*, never a *configuration*.

**The pair.** Unit A calls dagre with default rank separation; Unit B, tuning density for the 214-node estate, sets `ranksep: 80`. Same engine, same pinned version, both AD-7-compliant, different pictures for identical code. And the user's own toggle means two users with the same code see different arrangements **by design** — while the copy claims the picture is reproducible.

**Consequence.** The strongest claim in the product is false in the one case a skeptical viewer will actually try: two machines, same repo, different-looking picture. AD-14 does not catch it — CI runs one preset.

**Close it:** layout parameters are **named, versioned presets owned by `toolkit/model`**, not free arguments. The toggle selects among presets by name. The rendered artifact records `(engine, engineVersion, presetId, presetVersion)` — AD-7 already records two of the four. The claim is scoped honestly in copy: *same code, same preset, same picture*. AD-14 gates every preset. Coordinates are rounded to a fixed integer grid before serialization, so byte comparison is stable across platform float formatting.

## M-2 — Presentation-level elision has no vocabulary, so perspectives will invent one — colliding with AD-10.

AD-10 closes the *extraction-assigned* states. But EXPERIENCE requires perspectives to state limits **extraction never assigned**: reach-depth truncation ("reach depth 2 — 3 further hops collapsed"), collapse-by-subtree ("says what it collapsed"), the legibility-floor auto-collapse, filter-hidden elements ("says what it hid"), and FR-8's stale-anchor notice.

**The pair.** Unit A (Reach tracer) renders truncation as a plain count node. Unit B (Dependency map) renders collapse using an honest-state-styled marker, reasoning it is a limit of knowledge. Both comply — AD-10 closes honest states and says nothing about elision.

**Consequence.** The user cannot tell *"the tool didn't show you"* from *"the code couldn't be read"* — two different kinds of not-knowing collapsed into one vague one, which is verbatim the reasoning AD-10 uses to justify separating unparsed from unresolved. Trustworthiness again varies by which tool drew the view.

**Close it:** a **second closed vocabulary — presentation elisions** (`depth-truncated`, `subtree-collapsed`, `legibility-collapsed`, `filter-hidden`, `stale-anchor`) — owned by the model, rendered from one legend, **visually distinct as a class from honest states**. A perspective needing anything outside both closed sets fails render-boundary validation rather than inventing.

## M-3 — Two undefined predicates: perspective availability, and AD-12's "interactions its ViewDoc can serve."

**Pair A (availability).** EXPERIENCE's availability rule turns on "can honestly derive *something*," predicate = derivability, not grain. Unit A implements a pure `canDerive(snapshot, scope)` on the perspective contract. Unit B runs the perspective and offers the tab if the ViewDoc is non-empty. Both satisfy the rule. Unit B costs a full run per candidate tab on every navigation and — under AD-11 — writes speculative ViewDocs into the watched folder, producing phantom renders and phantom gallery entries.

**Pair B (AD-12).** AD-12 says a molded view "supports exactly the interactions its ViewDoc can serve" without defining servability. Unit A infers drill-ability from whether a finer-grain node exists in the doc; Unit B requires an explicit declaration. A molded tool emitting nodes at two grains without declaring anything gets a drill affordance under A and not under B — a dead affordance or a missing one, both of which AD-12 exists to prevent.

**Close it:** `canDerive` is part of the perspective contract with the same purity constraints as `perspective` and is **never inferred from output**; a perspective with no `canDerive` (a pre-gate molded tool) is gallery-launchable only, never switcher-offered — consistent with AD-12's asymmetry. And a ViewDoc **declares its served interactions explicitly** (`drill`, `selectionSync`, `traceEdge`, `reachDirection`); the renderer offers exactly what is declared and nothing inferred.

---

# Summary table

| # | Sev | Hole | Diverging pair | Closing AD |
|---|---|---|---|---|
| C-1 | Critical | Canonical name undefined; minter takes a free string | file extractor vs SQL stage-2 | Tighten AD-2 — typed descriptor in, canonicalizer owns identity, collision rule |
| C-2 | Critical | Inclusion identity unanchored | Dependency map vs Reach tracer over one copybook | **AD-15** definition-site anchoring |
| C-3 | Critical | Node-kind vocabulary open | COBOL profile vs SQL profile | **AD-16** closed kind enum, grain-bearing |
| C-4 | Critical | Edge-kind vocabulary open | Dependency map vs Reach tracer | **AD-17** closed edge kinds + namespaced post-gate extension |
| C-5 | Critical | Two owners of construct→kind | host profile vs stage-2 parser | Tighten AD-9 — stage 2 never classifies or mints |
| C-6 | Critical | "A defined total order" never defined | code-unit sort vs `localeCompare` | Tighten AD-3 — one byte-order comparator, full sort key, no ties |
| C-7 | Critical | Perspective meets an unclassified case | draws it as fact vs drops it | Extend AD-10 — 4th state `out-of-snapshot reference` |
| H-1 | High | AD-2 unenforced at the render boundary | TS molded tool vs foreign JSON producer | Tighten AD-4 — validate ids as it validates spans |
| H-2 | High | Scope and perspective have no identity | folder-scope vs node-scope perspective | **AD-18** minted Scope + PerspectiveId |
| H-3 | High | Shell state unowned; ViewDoc filenames unowned | trail epic vs watcher/gallery epic | **AD-19** single state owner + minted filenames |
| H-4 | High | Version-skew behavior unspecified | strict renderer vs lenient renderer | Tighten AD-5 — `mustUnderstand`, refuse-don't-partial, N-1 window, gated |
| H-5 | High | Hash inputs ambiguous, paths uncanonicalized | path-set hash vs content hash; absolute vs relative | Tighten AD-3 — exact hash preimage, machine-independent |
| M-1 | Medium | Layout config and user toggle unpinned | default preset vs tuned preset | Named versioned presets; scope the claim to preset |
| M-2 | Medium | Presentation elision has no vocabulary | truncation-as-count vs truncation-as-honest-state | Second closed vocabulary, distinct as a class |
| M-3 | Medium | Availability and servability predicates undefined | `canDerive` vs run-and-see; inferred vs declared drill | Contract-level `canDerive`; declared interactions |

---

# Two closing observations

**The pattern.** Twelve of fifteen holes are the same shape: *the spine names an owner but not an algorithm, or a format but not a vocabulary.* An owner without an algorithm is not an invariant — it is a place to put the blame after two builders diverge. AD-2 and AD-3 are the two most load-bearing ADs in the document and both have this defect; AD-4 does not, which is why it is the strongest one here and the right template for fixing the others.

**Priority.** C-1, C-2, C-4, and C-5 all terminate in the same consequence — a split identity yielding an inbound reach result that is partial and *renders as complete*, dressed in a provenance tick and an honest-state count that make it look more trustworthy than a bare wrong answer would. The PRD names that outcome the one failure mode with the power to break the running business. They are also all in the **gate slice**: extraction, copybook expansion, and the dependency map. They must close before the first line of extraction is written, not at Phase B. C-6 and H-5 must close before AD-14's golden files are generated, or the corpus bakes in one implementer's arbitrary choice and the gate certifies it.
