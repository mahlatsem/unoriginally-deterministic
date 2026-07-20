---
name: unoriginally-deterministic
type: roadmap
status: living
created: '2026-07-20'
updated: '2026-07-20'
scope: 'product capability sequenced beyond v1'
---

# Roadmap — unoriginally-deterministic

Capability committed in **direction** but deliberately not in **design**, sequenced after v1.

**What belongs here:** product capability past the v1 surface — things a user would eventually
do that v1 does not let them do.

**What does not:** decisions a planning document owes but hasn't made. Those stay in that
document's own *Deferred* section, where the context lives — the layout-engine selection,
grammar maturity exposure, performance budgets, and the supported VS Code / OS matrix all
belong to [ARCHITECTURE-SPINE.md](architecture/architecture-unoriginally-deterministic-2026-07-19/ARCHITECTURE-SPINE.md),
not here.

Nothing on this roadmap is scheduled, and nothing on it may pull work forward into v1. The
reliability gate (PRD §7 OQ-1) governs sequence for everything; a failed gate stops this
list along with the rest.

---

## Phase C — Change review

**Status:** direction fixed · mechanism deferred · nothing in v1 depends on it
**This document owns the item.** Its architecture-binding constraints are restated in
[ARCHITECTURE-SPINE.md](architecture/architecture-unoriginally-deterministic-2026-07-19/ARCHITECTURE-SPINE.md)
— Build Phasing (Phase C), *Deferred → Change review*, and AD-25 / AD-18 — because a builder
reading the spine alone must not miss them.

### Why

The reviewer's problem with AI-generated code is not *what changed* — the text diff answers
that well, and answers it better than a graph would. It is **what the change touched that the
reviewer would not have guessed**. A generated edit to one paragraph that turns out to write to
a table three programs depend on is exactly the finding a text diff cannot surface and an
extraction diff can.

This is also the honest answer to "can I share a view with my reviewer": no exported picture,
ever — both sides re-extract. See AD-25.

### Decided

- **Reviewed in VS Code first.** GitHub and other surfaces are a later rendering question, not
  an architecture one, so long as ViewDoc remains the exchange contract (AD-5). A reviewer
  invoking a diff is already AD-12's explicit user action on their own machine.
- **Compare what the PR compares** — base-vs-head as the forge computes it, i.e. against the
  merge base. Comparing against the base branch tip attributes everything else that landed
  meanwhile to this PR.
- **Never check out a ref under the reviewer.** The shell reads trees at a ref directly and
  supplies them to the core as values; AD-1 holds and the core never learns git exists.
- **Default projection is the changed set plus its blast radius**, not a full map with change
  decoration. A large view with the change highlighted still costs a full scan, which is the
  thing that does not scale. Blast radius is Phase B's bidirectional reach tracer applied to
  the changed nodes. Full-map decoration stays available as a secondary mode for small changes.
- **Change status is an axis, not a kind** — orthogonal to AD-16 and AD-17, which stay closed,
  and carrying a registered non-colour cue per AD-23.
- **One layout over the union graph.** Laying out each side independently moves shared nodes
  and renders everything as changed.
- **Differing parse status is not a difference** — already in force as AD-10's propagation
  clause, not waiting on this phase.

### Open

- Whether the head side is the working tree or the head commit when reviewing in-progress work.
  AD-3 hashes contents, so both are expressible.
- What a paired `SnapshotRef` means to a consumer — ordering, whether one view may mix
  single- and pair-derived elements, and how a pair serialises into a legible filename. The
  *seam* is cut (AD-18); the semantics are not.
- Whether molded tools may run against fetched branch code, and under what rule. AD-12 covers
  the local reviewer case and says nothing about this one. **Not decided** — deciding it early,
  without a concrete surface, is how a security-relevant default gets set by accident.

---

## Beyond Phase C — direction only

No design, no commitment; recorded so the reasoning isn't re-derived from scratch later.

- **Change review outside VS Code** (GitHub PR surface and similar). Blocked on nothing
  technical — it is a rendering surface over the same ViewDoc — but it reopens the execution
  question above in a genuinely hostile setting: CI running molded tools from a fork is
  untrusted code on your runner.
- **User-facing languages beyond COBOL** (PRD §4). TypeScript extraction already exists in v1
  but is scoped to self-documentation. AD-21 gives the seam; adding a language is a profile,
  not a model change.
- **Cross-language tracing (COBOL ↔ Java)** — explicitly out of v1 (PRD §4), and the half of
  the maintainer's real estate v1 deliberately does not solve.
- **Community contributions and the shared commons** (PRFAQ; PRD §7 OQ-4). No trigger defined.
