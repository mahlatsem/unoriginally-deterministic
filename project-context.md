# Project Context — unoriginally-deterministic

Standing rules for BMad runs on this project, loaded automatically as `persistent_facts`.

These are not general advice. Each rule names a failure that actually happened during this project's planning runs, and each is stated so that following it is cheaper than not.

---

## Claim discipline

The recurring failure mode in this project's planning has been **confident interpolation**: filling a gap in evidence with a plausible answer, in output that reads identically to knowledge. It is invisible from the inside. These rules exist to make it visible in the artifact instead.

### 1. Verify at the moment you log, not at the moment you finalize

A memlog entry is the authority a document is later distilled from. A false claim entering the memlog **propagates and shapes later decisions** before any reviewer sees it — so catching it at Finalize means unwinding several layers, not fixing one line.

Any decision that names an external technology, version, capability, or limitation gets verified *before* the `memlog.py append` that records it.

> *What happened:* an unverified "the two-stage parse is practicable" entry drove the phasing decision, which drove the gate-slice contents. All three had to be unwound when the premise turned out to be wrong.

### 2. Never write a bare negative existence claim

"X does not exist" and "no library supports Y" are near-unfalsifiable by search and disproportionately damaging when wrong, because they close off options silently.

Write **"not found in: ⟨enumerated search terms and sources⟩"** instead. The enumeration lets any reader who knows more than the searcher spot the gap immediately.

> *What happened:* the architecture spine asserted "no current tree-sitter COBOL grammar emits `EXEC` regions," based on a search of five repositories that never included the term "enterprise." A grammar doing exactly that had existed the whole time. Had the spine listed the repositories searched, the omission would have been obvious on sight.

### 3. Quote the source when citing a requirement

When justifying a decision with "FR-6 requires…", "the protocol needs…", or "the spec says…", paste the sentence. If it cannot be pasted, the requirement is inferred — say so.

> *What happened:* an addendum line reading *"run however the user or Claude Code likes (terminal, npm script, etc.)"* was cited as "the terminal-runnable property SM-2 depends on," and used to justify an architectural option. Writing the sentence out would have exposed it as permissive on sight.

### 4. Apply the divergence test to your own wording, not just your own decisions

For every rule authored, name two implementations that both comply and check whether they still diverge. A rule can be correct in intent and unenforceable as written.

> *What happened:* an invariant fixed the node-id *format* while leaving canonicalization to each caller, and another required "a defined total order" without defining one. Both read as enforceable; neither was. Two compliant builders would still have produced incompatible output.

### 5. Grep, don't recall

Anything enumerable — cross-references, terminology consistency, upstream drift, contradictions between a diagram and the prose around it — is checked mechanically, never from memory.

> *What happened:* upstream drift was under-counted twice by eyeballing (two found, four actual), and two internal contradictions sat inside a single freshly-written document: an invariant declaring one transport while its own diagram drew another, and an absolute no-I/O rule contradicted by its own dependency arrow.

---

## What already works — keep it

The reviewer gate caught six of nine errors in the architecture run. The two configured `finalize_reviewers` lenses are high-yield and should not be trimmed for cost:

- The **currency lens** caught every interpolation error.
- The **adversarial lens** caught the systemic "owner but not algorithm" flaw that all four critical invariants shared.

The errors that reached the user were the ones no lens was pointed at. Prefer adding a lens over trusting a habit.

## Cross-document consistency

This project maintains four coupled documents: `prd.md` (+ `addendum.md`), `DESIGN.md`, `EXPERIENCE.md`, and `ARCHITECTURE-SPINE.md`. A decision in any one of them can supersede another.

- When a decision overrides a source document, record it in an explicit **Upstream Consequences** section and update the source in the same session. A spine that contradicts its own source is the condition three consecutive review passes kept detecting.
- Verify reconciliation by grep across all four, not by recalling which ones were touched.
