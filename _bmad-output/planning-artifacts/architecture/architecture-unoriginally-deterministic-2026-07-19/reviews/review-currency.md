# Technology-Currency Review — ARCHITECTURE-SPINE.md

- **Target:** `_bmad-output/planning-artifacts/architecture/architecture-unoriginally-deterministic-2026-07-19/ARCHITECTURE-SPINE.md`
- **Reviewed:** 2026-07-20
- **Method:** every external-world claim re-checked against live web sources, npm registry, GitHub API, and upstream grammar/algorithm source. No verdict below rests on model training data.
- **Overall verdict:** **Revise before build.** The spine's invariants (AD-1 … AD-14) survive intact — none of the findings threaten the architecture. But four of the outward-facing factual claims are wrong, stale, or stated far more confidently than the evidence supports, and two of them (the ELK seed claim and the EXEC SQL two-stage claim) are the kind of error that only surfaces at implementation time.

**Tally:** 21 discrete external claims examined — **12 confirmed**, **4 stale**, **4 overstated**, **1 unverifiable**.

---

## 1. Stack table — "Verified current 2026-07-19"

### 1.1 TypeScript 6.0 — **STALE**

The stack table asserts TypeScript 6.0 and stamps the whole table "Verified current 2026-07-19". On that date the current stable TypeScript was **7.0.2**, not 6.0.

Registry evidence (`registry.npmjs.org/typescript`):

```
6.0.3  2026-04-16
7.0.2  2026-07-08     <- dist-tag "latest"
next:  7.1.0-dev.20260719.1
```

TypeScript 7.0 (the Go-native "Project Corsa" compiler) shipped stable **2026-07-08**, eleven days before the spine was written and stamped as verified. `dist-tags.latest` is `7.0.2`; `6.0.0-beta` still sits on the `beta` tag.

Two separate problems:

- **The "verified current" label is factually wrong.** 6.0 was not current on 2026-07-19.
- **The version may still be the right choice, but the spine doesn't say so.** Microsoft's own 7.0 announcement recommends upgrading *through* 6.0, ships 7.0 **without a stable compiler API** (deferred to 7.1), publishes a `@typescript/typescript6` side-by-side compat package, and tells embedded-language tooling (Vue/Angular/Svelte) to stay on 6.0 until the 7.0 API stabilises. A project that plans to touch the compiler API — and FR-2 "TypeScript self-documentation" strongly implies a TS language profile reading the TS AST — has a real reason to pin 6.0.

**Verdict: stale as labelled; possibly correct as a decision.** Fix by changing the label from "Verified current" to the actual rationale: *"TypeScript 6.0 — deliberately one major behind. 7.0 (2026-07-08) ships no stable compiler API until 7.1; the FR-2 TypeScript profile depends on that API."* If FR-2 does **not** need the compiler API, there is no reason not to be on 7.0.2 and take the 8–12× build speedup.

Sources:
- https://registry.npmjs.org/-/package/typescript/dist-tags
- https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/
- https://www.theregister.com/devops/2026/07/09/speedier-type-checks-in-typescript-70-as-first-stable-go-release-ships/5268828

### 1.2 Node.js 24 LTS — **CONFIRMED**

As of July 2026: **Node 24 is Active LTS** (EOL 2028-04-30). Node 22 is Maintenance LTS (EOL 2027-04-30). Node 26 shipped May 2026 but is the **Current** line and does not enter LTS until **October 2026**.

Node 24 is therefore the correct choice today, and the reviewer's concern about Node 26 resolves in the spine's favour. One forward-looking note worth recording: Node 26 becomes Active LTS in ~3 months, and from Node 27 the release cadence goes annual with every major entering LTS. The spine should expect a Node 26 bump shortly after October 2026; AD-14's golden-file gate is the thing that will catch any behavioural drift from that bump.

Sources:
- https://nodejs.org/en/about/previous-releases
- https://nodejs.org/en/blog/announcements/evolving-the-nodejs-release-schedule
- https://endoflife.date/nodejs

### 1.3 web-tree-sitter 0.26.11 — **CONFIRMED**

`dist-tags.latest` = `0.26.11`, published **2026-07-12**. Current on the spine's stated verification date and still current at review time. This is the one stack row that was genuinely verified.

One adjacent fact the spine should know: the Node-native `tree-sitter` binding is on **0.25.0** while `web-tree-sitter` is on 0.26.11 — the two lines have diverged. Since AD-3 hashes "grammar versions" into snapshot identity, the spine should be explicit that the hash covers the **web-tree-sitter runtime version and the grammar ABI version**, not just the grammar source revision. A web-tree-sitter minor bump that changes parser ABI will change output and must invalidate the snapshot.

Sources:
- https://registry.npmjs.org/-/package/web-tree-sitter/dist-tags
- https://www.npmjs.com/package/web-tree-sitter

---

## 2. Deferred technologies

### 2.1 "dagre and ELK.js are both layered with no seed, so both satisfy AD-7" — **OVERSTATED (the ELK half is wrong)**

This is the most consequential finding in the review. It is presented as a settled fact and it is half false.

**dagre — the claim holds.** Read the actual source: `dagre/lib/order/init-order.ts` assigns initial in-layer order by **DFS from rank-sorted nodes**, citing Gansner et al. There is no RNG anywhere in the ordering phase; the crossing-minimisation sweep in `order/index.ts` is a deterministic iterated barycenter with a `disableOptimalOrderHeuristic` escape. dagre is seed-free and deterministic **given input insertion order** — which is precisely why AD-3's canonical ordering is load-bearing and not merely tidy. Good architecture, correctly reasoned.

**ELK.js — the claim is false.** ELK Layered **explicitly supports a randomisation seed**. From `Layered.melk` in the upstream Eclipse ELK repo:

```
supports org.eclipse.elk.randomSeed = 1
```

And from the ELK option reference for `org.eclipse.elk.randomSeed`: *"If the value is 0, the seed shall be determined pseudo-randomly (e.g. from the system time)."* The option is listed as supported by ELK Layered alongside ELK Force, ELK Stress, and the other genuinely stochastic algorithms.

So: ELK Layered is a **seeded randomised algorithm that happens to default to a fixed non-zero seed (1)**, making it deterministic by default and non-deterministic if anyone ever sets the seed to 0. That is a materially different safety property from dagre's, and AD-7 as literally written — *"uses a layered algorithm with no random seed"* — would **exclude ELK.js**, contradicting the Deferred section that says both qualify.

Required fix, one of:
- (a) Amend AD-7 to *"uses a layered algorithm whose randomisation, if any, is pinned to a fixed non-zero seed"* and add "layout seed" to the pinned-configuration list alongside engine and version, and to the artifact's recorded provenance; or
- (b) Amend the Deferred section to state that dagre is seed-free and ELK requires an explicit `elk.randomSeed` pin, so the choice is not architecturally neutral.

The spine's follow-on sentence — *"neither documents a determinism guarantee, which is why AD-14's golden files are what actually enforces it"* — is sound and now carries more weight than the author intended.

**Package-currency sub-finding — STALE.** The spine names "dagre", and the npm package literally called `dagre` was **last published 0.8.5 on 2019-12-03** and is dormant. The maintained line is **`@dagrejs/dagre` 3.0.0** (2026-03-22), which is where the TypeScript source quoted above lives. `elkjs` is healthy at **0.12.0** (2026-07-17). The spine should name `@dagrejs/dagre`, not `dagre`, or a reader will pin a seven-year-old package.

Secondary note on elkjs: it is a GWT transpilation of the Java ELK. No JVM is needed at **runtime** (it is plain JS), which preserves the spine's no-JVM posture — but building or patching elkjs from source does require a JVM/Maven toolchain. Only relevant if the project ever needs to fork it.

Sources:
- https://github.com/eclipse-elk/elk/blob/master/plugins/org.eclipse.elk.alg.layered/src/org/eclipse/elk/alg/layered/Layered.melk
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-randomSeed.html
- https://github.com/dagrejs/dagre/blob/master/lib/order/init-order.ts
- https://registry.npmjs.org/-/package/dagre/dist-tags (0.8.5)
- https://registry.npmjs.org/-/package/@dagrejs/dagre/dist-tags (3.0.0)
- https://registry.npmjs.org/-/package/elkjs/dist-tags (0.12.0)

### 2.2 COBOL grammar candidates — "Live candidates: yutaro-sakamoto, Neppord, BloopAI, arborium-cobol" — **STALE / OVERSTATED**

Three of the four named candidates do not survive contact with the GitHub API. The word "live" is doing work the evidence does not support.

| Candidate | Reality | Verdict |
| --- | --- | --- |
| `yutaro-sakamoto/tree-sitter-cobol` | Not archived, 38 stars, last push **2025-10-14** (~9 months idle). README confirms the spine's parenthetical: rules derived from opensource-cobol, tested against the NIST COBOL85 suite. | **Confirmed** — the only genuinely viable candidate, and the spine's characterisation of it is accurate |
| `Neppord/tree-sitter-cobol` | **`archived: true`**. Last push **2020-08-02** — nearly six years dead, 12 stars. | **Stale** — not a live candidate under any reading |
| `BloopAI/tree-sitter-cobol` | Not archived but last push **2024-04-05**, 6 stars, no activity in over two years; upstream company's OSS activity has lapsed. | **Overstated** — dormant, not live |
| `arborium-cobol` | **Not an independent grammar at all.** It is a Rust crate in Amos Wenger's `bearcove/arborium` bundle that repackages the generated `parser.c` of `zharinov/tree-sitter-cobol` at a pinned commit — and the GitHub API reports `zharinov/tree-sitter-cobol` is itself a **fork of `yutaro-sakamoto/tree-sitter-cobol`** (0 stars, last push 2026-01-12). | **Overstated** — a downstream Rust repackaging of candidate #1, listed as a peer of it |

Net effect: the "four live candidates" are really **one maintained grammar (yutaro-sakamoto), one fork of it (zharinov, surfaced via arborium), one dormant repo, and one archived repo.** The deferred selection decision is far narrower than the spine implies, and the ERROR-node-density bake-off it describes has effectively one serious entrant.

Two consequences the spine does not acknowledge:

- **arborium is a Rust distribution channel.** It ships `parser.c` as Rust crates for a Rust host. It is not directly usable from a TypeScript/WASM pipeline; its only value here is as a pointer to the zharinov fork. Listing it beside JS-consumable grammars is a category error that could waste a spike.
- **No maintained COBOL grammar ships a prebuilt WASM artifact on npm.** The npm package `tree-sitter-cobol` is a stub at **0.0.1**; `@yutaro-sakamoto/tree-sitter-cobol` does not exist. The project must therefore run `tree-sitter build --wasm` itself (emscripten or Docker) and vendor the resulting `.wasm`. That is a real, unbudgeted build-toolchain dependency, and it interacts directly with AD-14: the golden-file gate must pin the grammar **commit SHA plus the emscripten/tree-sitter-cli version used to compile it**, because the same grammar source compiled by a different toolchain can produce a different `.wasm`.

Sources:
- https://api.github.com/repos/yutaro-sakamoto/tree-sitter-cobol
- https://api.github.com/repos/Neppord/tree-sitter-cobol (`"archived": true`)
- https://api.github.com/repos/BloopAI/tree-sitter-cobol
- https://api.github.com/repos/zharinov/tree-sitter-cobol (`"fork": true`, parent yutaro-sakamoto)
- https://docs.rs/arborium-cobol
- https://github.com/bearcove/arborium

---

## 3. web-tree-sitter / WASM in-process in a VS Code extension, no JVM — **CONFIRMED**

Current reality, verified against the upstream `lib/binding_web/README.md`: web-tree-sitter is distributed as `web-tree-sitter.js` + `web-tree-sitter.wasm`, initialised with `Parser.init()`, loadable via npm under webpack/Vite/Node/Deno. Grammars compile to standalone `.wasm` and are loaded in-process. VS Code extensions loading grammar WASM from the extension path at activation is an established pattern with multiple public precedents (e.g. `71/vscode-tree-sitter-api`). No JVM, no daemon, no subprocess. The spine's architectural bet here is sound.

**One caveat the spine should carry into its deferred performance envelope.** The tree-sitter maintainers state plainly that *executing `.wasm` in Node.js is considerably slower than the native Node bindings*. The spine defers NFR budgets on the grounds that "no honest budget exists before a real parse of the real corpus" — fair — but it should record that the WASM-vs-native gap is a **known, documented, order-of-magnitude-class cost**, not an unknown. If the first gate-slice run over the maintainer's estate misses budget, dropping to the native `tree-sitter` binding is the obvious escape hatch, and it costs the browser-side webview execution path. Naming that fork in the road now is cheaper than discovering it under time pressure.

Sources:
- https://github.com/tree-sitter/tree-sitter/blob/master/lib/binding_web/README.md
- https://github.com/71/vscode-tree-sitter-api

---

## 4. AD-9 two-stage parse — "a second parser reads that region's contents" — **OVERSTATED**, on two counts

AD-9 is written as though stage 1 and stage 2 are both solved. Neither is, and the spine's own Deferred section does not flag either as an open question.

### 4.1 Stage 1 — the leading COBOL grammar cannot find `EXEC SQL` at all

AD-9 states: *"stage 1 — the host grammar locates the embedded region (`EXEC SQL … END-EXEC`)."*

I downloaded `yutaro-sakamoto/tree-sitter-cobol`'s `grammar.js` (3,783 lines) and grepped it. **Zero occurrences of `exec`, case-insensitive.** There is no `exec_sql_statement`, no `exec_block`, no raw-region rule of any kind. The best-maintained candidate grammar — the one the spine implicitly favours — **has no EXEC SQL support whatsoever**. An `EXEC SQL` block fed to it will not produce a locatable region; it will produce ERROR nodes.

The same grep confirms the good news for copybooks: `copy_statement` with a `replacing_clause` is present (lines 1455–1470), which supports FR-1's copybook handling and the spine's dual-anchor / substitution convention.

So AD-9 stage 1 is not a matter of picking a grammar — **it requires writing a grammar rule and upstreaming or forking it.** That work is invisible in the Deferred section, which frames the grammar decision as "architecturally immaterial… AD-9 holds whichever wins." AD-9 holds as an *invariant*, yes — but the sentence reads as reassurance that the grammar choice carries no work, and it carries a grammar patch.

### 4.2 Stage 2 — tree-sitter-sql exists, but does not speak embedded SQL

**The parser exists and is healthy.** `DerekStride/tree-sitter-sql` is the de-facto tree-sitter SQL grammar: 243 stars, last push 2026-03-02, published as `@derekstride/tree-sitter-sql` **0.3.11**. So the *existence* half of AD-9 stage 2 is **confirmed**.

**Its viability for this specific job is not.** I read the grammar source. `grammar/expressions.js` defines:

```js
parameter: $ => /\?|(\$[0-9]+)/,
_tsql_parameter: $ => seq('@', $._identifier),
```

That is `?` (ODBC/JDBC), `$1` (Postgres), and `@name` (T-SQL). **There is no colon-prefixed host-variable production.** Embedded SQL in COBOL is written almost entirely in host variables — `SELECT SALARY INTO :WS-SALARY FROM PAYROLL WHERE ID = :WS-EMP-ID`, frequently with indicator variables `:WS-VAL:WS-NULL`. The README describes the grammar as "general/permissive" with references to Postgres, SQLite and Phoenix — no DB2, no SQL/DS, no embedded-SQL preprocessor dialect.

Concretely: the most common statement shape in the target corpus will very likely hit ERROR nodes in the stage-2 parser. AD-10 means those degrade honestly rather than lying — the architecture holds — but the practical outcome is *"data lineage silently omitting SQL-touched tables"*, which is the exact failure AD-9 exists to prevent, arriving as a flood of honest-state noise instead.

Two further integration notes: DerekStride does **not** commit generated parser files to `main` (they live on a `gh-pages` branch, though the npm package carries them), and no prebuilt WASM is published — same self-build requirement as the COBOL grammar, and the same AD-14 pinning obligation.

### 4.3 "EXEC-block mining is an established technique" — **UNVERIFIABLE as stated**

I could not find a citation for this phrasing, and I want to state that plainly rather than wave it through.

What **is** confirmable: commercial COBOL static-analysis platforms routinely parse embedded SQL as part of building program/data metadata models, and SQL-parse-then-resolve lineage extraction is well-established practice on the SQL side (DataHub's SQLGlot-based column-level lineage is a documented public example). So the *concept* — locate the embedded region, parse its contents with a SQL parser, fold the results into one graph — is real and commercially proven.

What I could **not** confirm: any published account of doing this with **tree-sitter specifically**, via a two-grammar pipeline, for COBOL EXEC SQL. Note also that tree-sitter has a first-class name and mechanism for exactly this pattern — **language injection**, driven by `injections.scm` queries — which AD-9 reinvents under bespoke terminology without referencing it. Aligning AD-9's vocabulary with tree-sitter's injection model would let the project inherit a documented mechanism rather than describe a novel one, and would make the rule legible to anyone who knows tree-sitter.

**Verdict: the technique is real at the industry level; the tree-sitter instantiation of it is unattested and should be treated as a spike, not a given.**

Sources:
- https://github.com/yutaro-sakamoto/tree-sitter-cobol/blob/main/grammar.js (no `exec` token)
- https://api.github.com/repos/DerekStride/tree-sitter-sql
- https://github.com/DerekStride/tree-sitter-sql/blob/main/grammar/expressions.js
- https://github.com/DerekStride/tree-sitter-sql/blob/main/README.md
- https://datahub.com/blog/extracting-column-level-lineage-from-sql/
- https://www.in-com.com/blog/top-cobol-static-code-analysis-solutions-for-mission-critical-systems/

---

## 5. Other outward-facing assertions

| # | Claim (location) | Finding | Verdict |
| --- | --- | --- | --- |
| 5.1 | yutaro-sakamoto is "NIST COBOL85-tested" (Deferred) | README states rules are based on opensource-cobol and tested with the NIST COBOL85 test suite. Accurate as written. | **Confirmed** |
| 5.2 | "Deployment is exactly two artifacts: a Marketplace extension listing and an npm package" (Deferred) | Both channels exist and behave as described; AD-13's "no toolkit-hosted registry" is satisfiable by ordinary npm dependency resolution. Nothing here depends on unverified externals. | **Confirmed** |
| 5.3 | AD-13 "the extension bundles the same package" | Standard, supported VS Code packaging (`vsce` bundles `node_modules`). No external risk. | **Confirmed** |
| 5.4 | AD-3 "content hash over grammar versions" | Sound, but under-specified against §1.3 and §2.2 above: the hash must cover web-tree-sitter runtime version, grammar commit SHA, **and** the WASM build toolchain version. As written a reader will hash only the grammar version and get false cache hits across a toolchain bump. | **Overstated** (incomplete, not wrong) |
| 5.5 | AD-14 "byte-compared in CI" for laid-out output | Achievable with dagre. With ELK it additionally requires pinning `elk.randomSeed`, per §2.1. Also note byte-comparison of SVG/laid-out output is sensitive to float formatting across Node majors — relevant given the likely Node 24→26 bump in October 2026. Worth an explicit tolerance or normalisation policy. | **Overstated** |
| 5.6 | AD-7 "force-directed engines are excluded" | Correct and well-reasoned as a principle. The flaw is only in believing layered ⇒ seedless (§2.1). | **Confirmed** as principle |
| 5.7 | Stack table header "Verified current 2026-07-19" applied to all three rows | Two of three rows verified out; one (TypeScript) was already a major version behind on the stated date. A blanket verification stamp over a table where one row fails is the pattern most likely to prevent re-checking later. Recommend per-row dates. | **Overstated** |

---

## Recommended edits, in priority order

1. **AD-7 / Deferred §Layout engine** — correct the ELK seed claim. Either amend AD-7 to "randomisation pinned to a fixed non-zero seed" and add the seed to the recorded provenance, or state that dagre is seed-free and ELK needs an explicit `elk.randomSeed` pin. Also rename `dagre` → `@dagrejs/dagre` (the bare `dagre` package has been dead since 2019).
2. **AD-9 / Deferred §COBOL grammar fork** — record that no candidate grammar currently parses `EXEC SQL`, so stage 1 requires a grammar patch; and that `@derekstride/tree-sitter-sql` lacks colon-prefixed host-variable support, so stage 2 needs a dialect extension or an alternative. Downgrade "EXEC-block mining is an established technique" to "established in commercial COBOL analysers; unattested with tree-sitter — spike required."
3. **Deferred §COBOL grammar fork** — fix the candidate list: Neppord is archived (2020), BloopAI is dormant (2024), arborium-cobol is a Rust repackage of a fork of yutaro-sakamoto. Add that no candidate ships prebuilt WASM, so the project owns the WASM build step.
4. **Stack table** — replace the blanket "Verified current 2026-07-19" with per-row verification dates, and replace the bare "TypeScript 6.0" with the actual rationale for staying one major behind 7.0.2 (compiler-API stability), or move to 7.0.2 if FR-2 does not need that API.
5. **AD-3 / AD-14** — extend the content hash and the golden-file pin to cover the WASM build toolchain and, if ELK is chosen, the layout seed. Add a float-formatting normalisation policy for byte-compared laid-out output ahead of the Node 26 LTS bump in October 2026.
