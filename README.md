# unoriginally-deterministic

> The code is the only source of truth — now you can see it.

An open-source VS Code toolkit whose **deterministic** tools draw comprehension views
— dependencies, call flows, structure — straight from a codebase. The views are
faithful to the code by construction: no hallucinated diagrams, reproducible on every
run, and every element traces back to its exact source. When you need a view that
doesn't exist yet, you use the AI assistant you already have to build a new
deterministic tool that plugs in and lives in your repo.

The core bet, in one line: **AI is a coherence engine, not a truth engine** — so let it
*build the lens*, never *be the answer*. The non-determinism is spent once, at tool-build
time, where a human can check it. From then on the analysis is deterministic — the same
answer, for free, every run.

## Why this exists

Reading code line by line is the slowest way to understand a system, and AI now
generates code faster than anyone can read it. Asking an AI to *explain* the code just
moves the trust problem — it answers confidently and is sometimes wrong, with no way to
tell which time this is. This toolkit takes the [moldable development][gt] idea proven by
[feenk's Glamorous Toolkit][gtbook] and removes the thing that strangled its adoption:
you stay in **VS Code**, the tools are **TypeScript**, and you mold new ones with
**whatever AI assistant you already use** — no new environment, no Smalltalk.

## Status

⚠️ **Early — concept stage.** This repository currently holds the **Working Backwards
PRFAQ** for the project, not an implementation. The immediate next step is a reliability
prototype: prove that different AI assistants can each generate a correct,
provenance-carrying tool against the toolkit's extension contract without hand-holding.

- 📄 [PRFAQ](_bmad-output/planning-artifacts/prfaq-unoriginally-deterministic.md) — the full Working Backwards write-up (press release + customer/internal FAQ + verdict)
- 📄 [Distillate](_bmad-output/planning-artifacts/prfaq-unoriginally-deterministic-distillate.md) — dense context for downstream planning

## Design principles

- **Deterministic by construction** — views are extracted mechanically (tree-sitter first); where a parser can't parse, the tool fails loud and marks the region unparsed rather than fabricating.
- **Trust is structural** — provenance (source span) is a required input to the drawing primitives; a tool literally cannot draw an edge it can't trace to source.
- **Ships no AI, asks for no keys** — the toolkit is deterministic code end to end; the AI lives in the assistant you already use.
- **You own your tools** — molded tools are plain TypeScript versioned in *your* repo; they keep running with or without this project.

## Acknowledgements

Built on the ideas of [moldable development][gt] and [feenk's Glamorous Toolkit][gtbook]
(Tudor Girba & Simon Wardley, *["Rewilding Software Engineering"][rewild]*). This project
is an accessibility-first re-imagining of those ideas for the ecosystem developers already
work in — and would happily fold if a better-funded, more accessible version arrived. The
point is for the idea to cross the chasm.

[gt]: https://gtoolkit.com/
[gtbook]: https://book.gtoolkit.com/
[rewild]: https://medium.com/feenk/rewilding-software-engineering-ca3ad1e612d8
