# Edge-Case Path Trace — PRD: unoriginally-deterministic (2026-07-18)

Method: mechanical walk of every branch/state transition implied by FR-1..FR-5, their Consequences bullets, Non-Goals, MVP Scope, Success Metrics, and Open Questions. Only genuinely silent gaps are listed — items already deferred to an Open Question, tagged `[ASSUMPTION]`, or otherwise explicitly addressed are excluded.

## FR-1 — COBOL Tree-Sitter Extraction

1. **Whole-file parse failure vs. per-construct ERROR node.** FR-1 defines behavior only for a construct the grammar can't parse (visible ERROR node). No behavior is specified for total extraction failure at the file level — e.g. a non-COBOL/binary file, wrong encoding, or an empty file — leaving open whether this crashes the pipeline, produces an empty tree, or is silently skipped.
2. **Multi-file COBOL programs (COPY/copybooks).** COBOL programs routinely span a main source file plus `COPY`-included copybooks. FR-1 only describes parsing "COBOL source files" with no statement on whether copybook resolution/inclusion is part of the queryable representation or whether each file is extracted in isolation — a gap with direct consequences for FR-4's dependency-map/call-flow views.
3. **Competing community grammar extensions.** FR-1 allows "base tree-sitter grammar + community extensions" but gives no precedence/selection rule if multiple community dialect extensions are installed simultaneously (which one applies, or how conflicts are resolved).

## FR-1 / FR-4 boundary

4. **Downstream handling of ERROR nodes in a rendered view.** FR-1 fixes the parse-level behavior (visible ERROR node, never fabricated structure) but nothing specifies what a starter view (FR-4) does when it encounters such a node during rendering — silently omit the affected element, render a visible gap/warning, or abort the whole view. This is exactly the case where a real COBOL file is parseable overall but the community grammar doesn't cover one specific dialect construct.

## FR-2 — Compose a Molded Tool Over Primitives

5. **Bypassing primitives entirely.** FR-3's provenance enforcement is scoped to calls into toolkit "drawing primitives." Nothing in FR-2 prevents a molded tool from skipping those primitives altogether (e.g. writing output via raw VS Code APIs or its own rendering) — the structural guarantee ("no source, no node") only holds for tools that actually route through the primitive layer.
6. **"Short enough to read end-to-end" has no enforcement.** Unlike FR-3's structural provenance check, this trust-check property (Consequences, FR-2) is asserted with no defined threshold, linter, or gate — no stated behavior for a legitimately useful tool whose source naturally exceeds "one sitting."
7. **Local-only guarantee doesn't bind the molded tool's own code.** FR-2's "no source code ... transmitted off the user's machine" describes the toolkit's own extraction/molding process. Once a molded tool exists as independent, arbitrary TypeScript (FR-5), nothing constrains its runtime code from making outbound network calls — the guarantee has no enforcement at the molded-tool layer.

## FR-3 — Structurally Enforced Provenance

8. **Presence-only enforcement, not correctness.** The primitive requires *a* source span argument, but nothing validates that the span is accurate, current, or genuinely corresponds to the drawn element — a call can supply a fabricated, mismatched, or stale span and still pass the structural gate, undermining the trust model it exists to guarantee.
9. **Multi-location provenance (edges).** A relationship (e.g. a call-flow edge between caller and callee) inherently references two source locations, but FR-3 specifies "a source span argument" (singular) with no rule for how edges satisfy this — one endpoint, both, or a combined range.
10. **Stale/broken click-to-source targets.** No behavior is defined for when a recorded span's target no longer exists at click time (file renamed, moved, deleted, or edited so the line shifted/disappeared between render and click) — "click-to-source or equivalent" has no stated fallback.

## FR-2/FR-5 — Extension-Contract Versioning

11. **Molded tool generated against an older extension-contract version.** The Assumptions Index explicitly excludes molded-tool-level versioning from its "non-blocking" toolkit-versioning assumption ("Toolkit-level (**not** molded-tool-level) versioning/deprecation policy is undefined"). No path is defined for what happens to an already-generated molded tool when the toolkit's extension-contract API later changes signature — silent compile failure, silent misbehavior, or required manual migration is unaddressed.
12. **Grammar/extension version drift without source-code change.** UJ-2/FR-5 promise re-running a molded tool "re-derives the view with no re-molding needed," but this is only stated with respect to *code* changing. No behavior is defined for the toolkit's own tree-sitter grammar or community extension being upgraded between runs against unchanged COBOL source — silent drift in view output with no code change is unaddressed.

## FR-5 — Molded Tools Run Independent of the Toolkit

13. **Runtime dependency on the toolkit itself.** FR-5 states a molded tool "continues to run without any dependency on the toolkit project's continued existence," but a molded tool likely still imports the toolkit's primitives/runtime library (npm package, or the toolkit's own VS Code extension) to execute drawing/query calls. No statement addresses what happens if that package is unpublished/removed, or if the toolkit's VS Code extension is uninstalled/disabled in the environment where the molded tool is run.

## FR-4 — Starter View Over Real COBOL

14. **COBOL file discovery/identification.** No mechanism is specified for how the toolkit determines which files in a workspace are "COBOL" (extension-based detection vs. content sniffing, fixed-format vs. free-format source) — behavior on non-standard extensions (`.cbl`/`.cob`/`.cpy`/no extension) is unaddressed.
15. **Empty-workspace state.** FR-4 asserts the starter view "renders on an unmolded repo with no setup," but no behavior is given for a workspace containing zero COBOL files — what the view shows (nothing, an empty-state message, an error) is unaddressed.

## Success Metrics / Reliability Gate (SM-2, OQ-1)

16. **Definition of "unedited."** SM-2 requires molded tools to pass both trust checks "unedited," but no threshold defines what counts as an edit — whether a human hand-fixing an AI-generated tool after generation (even a trivial one-line fix) disqualifies it from the reliability gate is unaddressed.
17. **No pass/fail bar for trust check (a).** Trust check (b) (provenance trace) is objectively verifiable; trust check (a) ("reading its source") has no defined criterion for what constitutes a pass (e.g. max length, comprehensibility bar), leaving gate evaluation partly subjective.
18. **No fallback if the reliability gate fails.** OQ-1 states the gate "must resolve before any broad build proceeds" as a blocking condition, but no contingency is defined for a negative result — redesigning the extension contract, restricting v1 to hand-authored tools only, or abandoning the broader build are all unaddressed.
19. **No reliability check for hand-authored tools.** SM-2's reliability gate validates only Claude-Code-generated tools. No analogous verification concept exists for hand-authored molded tools before they're trusted/run, leaving an asymmetry between the two authoring paths the PRD treats as equally valid (§4.2).

## Rollout Plan (§6.2)

20. **No contingency for a break during invited beta.** The three-phase rollout (dogfood → invited beta → public listing) has no defined process for a regression or trust-model violation (e.g. a provenance bug) discovered during invited beta — no rollback, pause, or communication plan to beta participants before progressing further.
21. **No graduation criteria between phases.** Beyond sequential ordering, no gate/criteria are defined for when "invited beta" is considered successful enough to proceed to "public listing" (distinct from OQ-4, which only concerns opening community contributions post-launch).
