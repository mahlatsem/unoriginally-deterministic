---
title: "PRFAQ Distillate: unoriginally-deterministic"
type: llm-distillate
source: "prfaq-unoriginally-deterministic.md"
created: "2026-07-18"
purpose: "Token-efficient context for downstream PRD creation"
---

# PRFAQ Distillate — unoriginally-deterministic

Dense context for PRD creation. Each bullet stands alone.

## Concept in one line
- An open-source **VS Code + TypeScript** toolkit of **deterministic** tools that draw comprehension views (dependencies, call flows, structure) directly from code. AI is used to *build* the tools, never to *be* the answer. Lineage: feenk's **moldable development / Glamorous Toolkit**, ported from the niche Pharo/Smalltalk ecosystem to where developers already live.
- Core thesis (grounded in feenk essay "Rewilding Software Engineering", Girba & Wardley): **"LLMs are coherence engines, not truth engines."** So the non-determinism is spent once at *tool build* time (human-verifiable), and *analysis* is deterministic → never confidently wrong.
- Economic engine: a tool is built once (one-time AI cost, in the engineer's own assistant) and runs **free, on-machine, reproducibly, forever**; it re-derives from current code each run so it never goes stale. New question → new tool; same question on new code → free.

## Customer / problem / stakes
- **Customer (broad):** anyone who must understand software they didn't write. **Lead exemplar:** the legacy-**modernization** engineer — the maintainer's real reality: modernizing 45yr COBOL entangled with 10yr Java where no one holds the whole picture. (Banking is a sub-segment, NOT the frame. Earlier "bank engineer / PR review" framing was REJECTED as too narrow.)
- **Problem:** comprehension of what exists / what's changing has always been throttled by reading-effort; AI turned a chronic problem acute. Deepest truth: the knowledge of what legacy programs do is *lost* — recovering it today needs scarce SMEs reading every line (true for ALL languages).
- **Stakes:** you cannot safely modernize/review/refactor what you can't first see; comprehension is the ROOT capability everything else grows from.
- **Beachhead (v1):** existing-code **structure comprehension**. Roadmap on same engine: change/diff-for-review; design-level EventStorming-from-code; **drift detection** (design-as-implemented vs design-as-intended).

## Architecture & technical decisions (LOCKED)
- **Toolkit bundles NO AI.** Deterministic code end-to-end: runtime + starter tools + a moldable framework *designed to be extended by whatever AI assistant the engineer already uses* (Claude/Copilot/Cursor) — or by hand. Product's job = make analysis tools (AI-authored or hand-written) deterministic, durable, shareable — not to be the AI. AI just makes writing a tool faster; it's never the only way to write one.
- **Parsing:** simple OSS deps only, **tree-sitter-first** (WASM, in-process, no JVM). tree-sitter's ERROR nodes make "fail loud / honest holes / never fabricate" the DEFAULT. COBOL = base tree-sitter grammar + community extensions. Commercial/heavy dialect parsing = **user-supplied plug-in via extension point**, never bundled. (ProLeap/SonarQube exist as evidence robust enterprise-COBOL parsing is available if a user must BYO; JVM-backend-in-core was REJECTED as violating the lean/on-machine promise.)
- **Trust = STRUCTURAL, not convention:** provenance (source span) is a **REQUIRED argument of the drawing primitives** — no source → no node/edge. A lazy AI-generated tool can't silently break trace-to-source; it produces nothing. THE make-or-break design surface.
- **Reliability mechanism:** AI **composes over vetted deterministic primitives** (tree-sitter queries + toolkit building blocks); it does NOT author analysis from first principles. Reliability scales with how well the toolkit narrows this job — for AI *and* for a human writing the tool by hand. AI use is recommended, especially for speed, but the extension contract must be easy to build against either way.
- **Two orthogonal trust checks:** audit the (short) analyzer; OR trace every drawn element back to its exact source line.
- **Visualization:** v1 = a few hardcoded view types + rendering primitives. "Tool builds the tool" (AI composes novel views over primitives) = DIRECTION, not v1. Explicitly NOT rebuilding GT's decade of polish.
- **Ownership/sustainability:** molded tools are plain TypeScript **versioned in the user's own repo**, run without the project → anti-SaaS-rug-pull; low exposure if project stalls.

## v1 scope — the no-list
- **v1 IS:** (1) tree-sitter extraction; (2) small starter view set drawn from the maintainer's own recurring cross-project needs (dependency map, call-flow, cross-language reach); (3) provenance / trace-to-source; (4) AI-molding workflow + human review.
- **v1 is NOT:** broad language coverage; general-purpose rendering framework; enterprise features; hosted anything; polished contribution pipeline.

## Competitive intelligence (verified)
- **Glamorous Toolkit** = the real incumbent. Has adopted the *exact* positioning ("code reading is the bottleneck… AI generates too fast… deterministic contextual tools replace reading") AND already analyzes COBOL/Java/etc. ~10yr, MIT, company-backed, book, conference — yet **~1,500 GitHub stars**. → The idea is validated AND proven hard to spread. Adoption is THE risk; **accessibility (VS Code + TS + your own AI, no Pharo/Smalltalk) is the ONLY wedge.** Public HN evidence corroborates onboarding as the failure point.
- **CodeScene** = social/git-history behavioral analysis (hotspots, change coupling, code health). Adjacent, cleanly differentiable (they analyze how the team changes code; we analyze what the code is).
- **Sourcegraph Cody** = deterministic code-graph navigation BUT LLM-generated comprehension layer = the "confidently wrong" path this is positioned against.
- **COBOL parsing landscape:** tree-sitter COBOL is partial/COBOL85 but multiply-forked/extended (Neppord, yutaro-sakamoto, BloopAI, raresdolga, Arborium). Robust enterprise options exist for BYO (ProLeap MIT: copybooks, EXEC SQL, EXEC CICS, NIST-passing, used on banking/insurance COBOL; SonarQube supports IBM Enterprise dialects, commercial).

## Rejected framings (do not resurrect)
- "accurate map" / "always accurate" → overclaim. Claim is *fidelity to code*, not truth about intent.
- "Stop reading code" / "replace reading" → overclaim. Visualization *aims* scarce reading attention; correctness judgment stays human.
- Bank/PR-review over-anchoring → too narrow; problem is general comprehension, lead with modernization.
- "payments ledger" example → too cute; truer framing is "nobody remembers what these programs do."
- Confrontational "you approved that PR, did you read it?" headline → rejected for truth-framing.
- Bundled-AI / BYO-key-inside-product → rejected; toolkit ships no AI.
- ProLeap/JVM parser backend in core → rejected; simple OSS deps only.

## Accepted trade-offs (product ceiling)
- **Does NOT catch semantic/local bugs** (rounding, date edge cases). Aims human reading at the right place; correctness stays human. Accepted, not a gap — claiming otherwise = the coherence-engine lie.

## Open questions / unknowns (flagged, with how to resolve)
- **#1 (must de-risk first): molding reliability is unproven.** RESOLVE VIA: thin vertical slice — tree-sitter extraction + provenance-enforcing primitives + one view — then have **Claude Code (the builder's own assistant) generate a custom tool across several independent tasks; measure how many plug in and pass trust checks UNEDITED.** Go/no-go experiment before any broad build. Cross-assistant portability is not a v1 goal — toolkit is assistant-agnostic by design; other users may validate other assistants themselves.
- **Extension contract not yet designed** (the mechanism is known: constrained primitives + required provenance; the design is open). Largest + most important open engineering problem.
- **Visualization/rendering primitives + v1 view set** undefined. Largest undesigned surface.
- **Activation/retention:** no defined "aha" that converts a first-try into a returning second-mold.
- **Commons graduation:** path from personal project-specific tool → generalized shared tool (parameterization/de-specialization) unspecified.
- **"Striking enough" dogfood outcome** undefined — needs a measurable success bar for the adoption bet.

## Posture / strategy (consistent across FAQ)
- **Mission over ownership.** Happy to fold if a better-funded accessible version (GT ships VS Code/TS/AI on-ramp) or reliably-grounded agentic AI supersedes it. Success = the IDEA crossing the chasm, not market capture. Not a moat — a head start.
- **No-hurry, long-horizon, solo (nights/weekends).** AI writing the tool's own code is what makes solo feasible at pace — hand-writing a tool is always an option, AI just accelerates it.
- **Adoption = proof-by-dogfooding** (real COBOL/Java modernization) **+ outcome-sharing** (LinkedIn, moldable-dev communities). Deliberately organic, no growth engine. No one asked to switch — showcase outcomes, let enterprises self-select.
- **Sustainability:** small/reviewable tools → cheap PR review; optional paid support if adoption warrants; bus factor mitigated by user-owned tools running without the project.
- **Kill-signal to watch (Q8):** a mainstream AI assistant producing the SAME structural answer twice with verifiable source (reproducible + grounded, not merely plausible) → the deterministic-tool wedge is closing → compresses the build window → favors a fast thin prototype.

## Naming / doc-hygiene notes
- Name = **unoriginally-deterministic** (deliberate wink: unoriginal ideas, deterministic execution).
- Leader quote (Tudor Girba) and community quote are GROUNDED-BUT-COMPOSED / ILLUSTRATIVE — must be replaced with real, cleared quotes before any PUBLISHED release. "LLMs are not truth engines but coherence engines" is verbatim from the referenced essay.
- Press release contains a placeholder repository link and a "small, sharp starter set" (deliberately vague on exact count — keep honest, don't imply a mature library on day one).

## Verdict summary
- **Forged — with one crack to prototype first.** Sharp thinking, honest positioning, realistic solo scope. Strongest assets: the coherence-vs-truth reframe, structural provenance-enforced trust, and the evidenced accessibility wedge. Build the **reliability prototype** before any broad build: if Claude Code can reliably produce a working, provenance-carrying tool against the contract unedited → you have a product; if not → it's GT's adoption problem in a new editor.
