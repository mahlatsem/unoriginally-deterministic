# Addendum: unoriginally-deterministic

Technical-how and options-considered detail that doesn't belong in `prd.md` — captured here for the architecture phase. Source: party-mode architecture brainstorm, 2026-07-18 (installed roster: Winston/John/Sally/Amelia/Mary/Paige).

## Decision: v1 delivery shape

**A thin VS Code extension** — not a "fat" one. The extension's only jobs are (1) host a webview panel and (2) watch a conventional output folder for rendered-view JSON and refresh the webview accordingly. It does **not** own extraction, the primitive library, or molding logic.

- **Starter tools** (FR-5's perspectives — dependency map, sequence, reach tracer, call-flow, flowchart, code) are pre-written by the maintainer, bundled inside the extension package (compiled into the `.vsix`). Running one from the command palette is an in-process call from the extension's command handler straight into the bundled script — no subprocess needed. Output goes to the same conventional location a molded tool's output would.
- **Molded tools** (FR-6) live in the user's own repo, committed like any other source file, run however the user or Claude Code likes (terminal, npm script, etc.).
- **The webview/renderer is blind to which kind produced its input.** It only watches the output folder and renders whatever JSON-with-source-spans appears. The starter/molded distinction is entirely about *where the source lives* and *who executes it* — never about how it renders.
- **Asymmetry — RESOLVED 2026-07-19 by the UX pass:** starter tools silently update whenever the extension updates (Marketplace release); molded tools are frozen until the user or Claude Code re-mold them (this is the intended FR-6 guarantee). The UX spine now makes that difference legible in the gallery and the view header rather than leaving the user to infer it — see `EXPERIENCE.md` § Component Patterns. The point is not decoration: a frozen tool is the FR-6 ownership guarantee working correctly, and the UI should say so rather than let it read as staleness.
- **Commons graduation mechanism** (ties FR-6 to the starter set, post-v1 per the PRD's Non-Goals on the contribution pipeline): a molded tool that a user finds broadly useful gets PR'd upstream to the toolkit's own repo. If merged, it graduates from "living in one user's repo" to "bundled as a starter tool in the next extension release" — every user's extension then picks it up automatically on update.

## Open for architecture: extracting data artifacts

The PRD (FR-1, FR-5) requires that declared **files** and database **tables** be surfaced as first-class nodes, so that "what writes to this table?" is answerable. It deliberately does not say how. That mechanism is an architecture decision, and it carries one known risk worth stating before it's discovered late:

- **Files** are the easier half. COBOL declares them in `SELECT`/`ASSIGN` and `FD`, and the I-O verbs that touch them (`READ`, `WRITE`, `REWRITE`, `DELETE`) are ordinary statements the grammar handles. Mapping declaration → logical dataset → access site looks tractable with the base grammar.
- **Tables are harder, and may not be free.** Embedded SQL is a *second language nested inside* COBOL. The base tree-sitter COBOL grammar may treat `EXEC SQL … END-EXEC` as an opaque region — or emit an ERROR node — rather than parsing the statement well enough to name the tables it touches. If so, table-level lineage needs either a grammar extension that parses the embedded block, or a narrow secondary parse of just the `EXEC` regions, fed back into the same node/edge model.
- **The honest fallback already exists.** If a given `EXEC` block cannot be read, it is an unparsed region like any other — visible, marked, never guessed past. Table lineage degrades to "this program touches SQL we couldn't read" rather than silently claiming a program touches nothing. That keeps the product honest whichever way the parsing question lands, but it also means partial SQL coverage is a *visible* limitation rather than an invisible one.
- **Not to be confused with the dialect boundary.** The PRD's "no bundled commercial/heavy dialect parsing" non-goal is about proprietary COBOL dialects. Embedded SQL is not that — it is squarely inside the v1 comprehension need, so "BYO plug-in" is not the intended answer here unless architecture finds no workable path.

## Rejected: alternative primary architectures

- **Standard-format output via VS Code's built-in previewers** (Mermaid/DOT rendered by the existing markdown preview, no custom rendering code at all). Rejected once pixel-perfect-clickable was confirmed as a day-one requirement — Mermaid/DOT don't support precise click-to-source targets without extra tooling layered back on top, which erases the "near-zero rendering code" appeal that made the option attractive in the first place.
- **LSP-based (Language Server Protocol) architecture.** Genuinely strong for *local*, point-wise comprehension — hover, CodeLens ("Called by 3 · Calls 5"), Go to Definition/References, and diagnostics (a natural fit for FR-1's "unparsed construct" honesty requirement) are all native, free VS Code UI once you're speaking LSP, and it's cross-editor for any LSP client. Rejected as the *primary* architecture for two reasons: (1) LSP has no native "draw the whole-system graph" primitive — it's point-wise by protocol design, so the actual dependency-map/call-flow *picture* still needs a webview regardless; (2) LSP's capabilities are fixed by the protocol — extending it to answer a bespoke new question is a heavy lift on the server implementation itself, which doesn't fit the moldability thesis (a molded tool needs to be a thirty-line script, not a language-server patch). **Parked as a possible v2 complementary layer** — reusing LSP's native hover/CodeLens/diagnostics wiring underneath the already-locked design, not instead of it.

## Decision: primitive exposure — library, not MCP (for v1)

Key turn in the discussion: the AI's job when molding a tool is to learn the *toolkit's own libraries* (tree-sitter query helpers + drawing primitives) fast — not to reason about the target codebase's business logic. That reframing decided the primitive-exposure question:

- **Chosen:** a small, well-typed TypeScript library, with the extension contract effectively *being* its types + a handful of examples (taught to Claude Code via the accompanying skill). This is the well-trodden "AI reads an API + examples, writes calling code" pattern — not an exotic one — and it keeps the persisted, readable script that FR-3's audit-the-source trust check relies on as the default artifact, rather than something that only exists if someone remembers to ask for it.
- **Deferred, not rejected: MCP server.** Exposing the same primitives as MCP tools would make them reachable by any MCP-aware client, not just this one VS Code extension — a stronger "not trapped in one environment" story, and a more future-proof bet long-term. But it's a second technical surface (server lifecycle, schema-driven tool-calling instead of library-calling) and a different cognitive task for the AI. Revisit **after** OQ-1 (the reliability gate) proves the primitive surface is learnable at all by one assistant — don't pay the cross-protocol cost before proving the core bet.

## SM-2 reliability-gate test protocol

Confirmed by the maintainer: 5 independent molding tasks, at least 4 must pass both trust checks unedited. This is how that bar gets administered:

1. **Build the thin vertical slice first** — COBOL extraction **including copybook expansion**, the provenance-enforcing primitives, and **the dependency map only** (FR-1, FR-4, FR-5). This is the "gate slice" formalized in PRD §5.2. The experiment tests the extension contract (FR-3/FR-4), not the whole product — so the remaining five perspectives deliberately come *after* the gate. Copybook expansion is the one extraction feature pulled *inside* the slice: real COBOL is copybook-saturated, and a gate passed against non-expanded extraction might not transfer to the actual dogfooding target.
2. **Tasks are real, not synthetic** — each of the 5 is a genuine comprehension question pulled from actual dogfooding work (PRD §6, SM-1), not invented for the test.
3. **"Independent" = a fresh Claude Code session per task** — no memory of a previous task's generated code, nothing copied forward. Tests whether the contract's own docs/types/examples teach it cold every time, not whether one thread is quietly reusing a pattern it invented once.
4. **"Unedited" = no maintainer hand-fixing** a broken script before judging it. The AI may iterate/self-correct within its own session (that's normal usage); the line is "no human intervention," not "no iteration." Exactly where AI self-correction crosses into "editing" is still a judgment call for whoever runs the protocol — not resolved here.
5. **Pass requires both checks clean, not one:**
   - *Audit the source* — short/readable, and calls only approved primitives (no reaching around them into raw APIs).
   - *Trace to source* — every drawn node/edge traces to a source line that's actually correct, not merely present.
6. **Known limitation:** the maintainer is both prompter and judge — no independent reviewer. Accepted for a solo project; still far more rigorous than an unstructured impression.
