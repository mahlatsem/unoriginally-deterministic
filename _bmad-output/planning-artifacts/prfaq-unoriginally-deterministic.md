---
title: "PRFAQ: unoriginally-deterministic"
status: "complete"
created: "2026-07-18"
updated: "2026-07-18"
stage: 5
inputs:
  - "https://medium.com/feenk/rewilding-software-engineering-ca3ad1e612d8 (feenk — moldable development thesis)"
  - "https://book.gtoolkit.com/ (Glamorous Toolkit — prior art, MIT OSS, Pharo/Smalltalk)"
---

# The code is the only source of truth — now you can see it

## unoriginally-deterministic is an open-source VS Code toolkit whose *deterministic* tools draw any system straight from its code — no hallucinated diagrams, no confident guesses, just what the code actually does every time you run it. Bring the AI assistant you already use to mold new tools that are every bit as trustworthy.

<!-- Press release forged interactively in Stage 2. Sections below locked incrementally. -->

**Worldwide, July 18, 2026** — Today the *unoriginally-deterministic* project ships its first open-source release: a VS Code toolkit that lets an engineer understand a codebase by *seeing* it — the real dependencies, call flows, and structure — drawn directly from the code itself. It starts with the hardest, most ordinary moment in software: you've been made responsible for a system you didn't write and can't fully read, and you have to change it anyway. Comprehension is the capability everything else grows from — you can't safely review, refactor, or redesign what you can't first see — and this release makes seeing a system as fast as reading a map.

Ask an engineer how they came to understand the last large codebase they owned, and the honest answer is: they didn't, not completely. They read the parts they had to, guessed at the rest, and built a mental model that was mostly hope. It isn't negligence — it's arithmetic. Reading code line by line is the slowest way there is to understand a system, and systems are now too large — and, with AI generating more of them faster than anyone can read, growing too fast — to read at all. The documentation describes what someone once intended, which may no longer be true. And when engineers ask an AI to explain the code, it answers confidently and is sometimes wrong, with no way to tell which time this is. So they change what they don't understand, and hope the tests catch it. And when the system you can't fully read is the one the business has quietly run on for decades — layers of code laid down by people long gone — hope stops being a plan.

With *unoriginally-deterministic*, understanding a system stops being a reading marathon and becomes something you look at. Point it at a repository and it draws the structure that's actually there — how modules really depend on each other, how a call flows across the system, where a change would ripple. Because the tools that draw these views are deterministic, the picture is faithful to the code by construction: it never quietly invents a connection that isn't there, and it never goes stale — run it again and it re-derives from the current code. It is also reproducible: the same code produces the same picture on your machine, your teammate's, and an auditor's, so a diagram becomes something you can point to as evidence, not just an illustration.

And it isn't limited to the views its authors imagined. Most tools show you what someone else decided you'd want to see. This one lets you build the view *you* need: when a question arises that only your codebase raises, you point the AI assistant you already use at the toolkit, and — because it's built to be extended exactly this way — the assistant writes a new, deterministic tool that plugs straight in and becomes a permanent part of your project's toolkit, versioned alongside the code. The toolkit itself ships no AI and asks for no keys; the confident guessing happens once, in your assistant, while the tool is being built, where a human can check it. From then on the analysis simply tells the truth, free, every time it runs. The tools that prove broadly useful graduate into a shared, community-maintained commons, so the toolkit grows sharper for everyone with every codebase it meets.

> "LLMs are not truth engines but coherence engines. They're remarkable at generating the tools that fetch the information we seek, and unreliable when asked for the information itself — so we let them build the tools, and we let the tools, which are deterministic, tell us the truth. Most of an engineer's energy goes not into writing code but into figuring out the knowledge encoded in the system. Give people thousands of small tools molded to fit their problem, and they no longer have to *read* the code to understand it. They can look — only when they want to — and trust what they see."
>
> — Tudor Girba, feenk — from ["Rewilding Software Engineering"](https://medium.com/feenk/rewilding-software-engineering-ca3ad1e612d8)

<!-- QUOTE-SOURCING NOTE (do not ship without resolving): "LLMs are not truth engines but coherence engines" is VERBATIM from the referenced essay "Rewilding Software Engineering" (Tudor Girba & Simon Wardley). The phrases "figuring out the knowledge encoded in the system" and "thousands of small tools molded to fit our problem" are also drawn from that essay. The connective sentences are composed for this PRFAQ. Acceptable for an internal planning artifact and clearly grounded in the source — but any PUBLISHED release must use Girba's exact, verified wording or obtain his sign-off before attributing this composed quote to him. -->

### How It Works

You're modernizing a platform that has grown for decades — 45 years of COBOL that still runs the business, wrapped in 10 years of Java that calls into it. Nobody alive understands how all of it fits together, and your job is to change it without breaking what the company runs on. You install *unoriginally-deterministic* from the VS Code marketplace and open the repo.

**1. Start from the commons.** The toolkit ships with a small, sharp starter set of ready-made views — a dependency map, a call-flow explorer, a "what does this module touch" tracer. You pick one and it renders in seconds, drawn from the code as it is right now. No setup, no molding, no tokens spent. This is the first ten minutes, and it's already more than you had.

**2. Read less, and only what matters.** You click a box and drop from the whole-system map down to the one function that actually matters, then back up. The view is faithful by construction — it's parsed from the source, so what you see is what's there. When you change code and re-run, the picture re-derives. It never drifts.

**3. Ask your own question.** You need to know what one COBOL program actually does — and the people who once knew are gone. No ready-made view answers "trace everything this program reads, writes, and calls, and show me where the Java invokes it." So you turn to the AI coding assistant you already use — and because the toolkit is built to be extended this way, it writes a new tool against clear extension points: a small, readable TypeScript analyzer that plugs straight in and answers exactly that.

**4. Check it two ways, cheaply.** You don't have to take the new tool on faith, and there are two independent ways to catch a lie. *Audit the lens:* the analyzer is short, readable TypeScript — far less to read than the system it inspects — so verifying the tool costs minutes, not hours. *Audit the output:* every element the view draws traces back to the exact source it came from — click an edge and land on the line that produced it — so you can verify the picture directly against the code. The one moment of AI fallibility is contained right here, in the open, where either check catches it.

**5. Keep it forever.** The tool is saved into the repo, versioned with the code, free to re-run by you or anyone on the team — no tokens, same answer every time. Next quarter's engineer inherits your *questions*, not just your code. And if the tool proves broadly useful, you send it upstream to the shared commons, and everyone's toolkit gets sharper.

> "I'm modernizing a platform that's part COBOL from before I was born and part Java from the twenty-tens, and no single person understands how they fit together. The truth is nobody remembers what half these programs actually do — finding out means an expensive specialist reading every line. I opened it in this thing and saw how the Java actually reaches into the COBOL — the real wiring, not the wiki's version. When I needed to know what a program really did, I read the twenty lines the generated tool wrote instead of paying someone to read the thousands it read for me. I changed it knowing what I'd hit. That hasn't happened to me before."
>
> — an engineer modernizing a decades-old core system *(illustrative persona)*

### How to Participate

*unoriginally-deterministic* is open source and free.

- **Try it in five minutes.** Install from the VS Code marketplace, open any repository, and run one of the starter views. No account, no configuration — the toolkit ships no AI of its own and runs entirely on your machine, with no tokens and no source leaving your repo.
- **Mold your own tools.** When the starter views don't answer your question, use whatever AI assistant you already work with — Claude, Copilot, Cursor, your own — to write a new tool against the toolkit's clear extension points. The toolkit bundles no model and asks for no keys; the one-time cost of building a tool lives in the assistant you already use. Once built, the tool runs free forever, like the rest.
- **Grow the commons.** The tools that prove broadly useful belong to everyone — open a pull request to the shared library. And because deterministic extraction needs a parser per language, the highest-leverage contribution you can make is a new language: help widen coverage beyond the first supported set.

Find the project, the roadmap, and the contribution guide at *[repository link]*.

## Customer FAQ

### Q1: Glamorous Toolkit already does this, and it's mature. Why does *unoriginally-deterministic* exist?

A: Because Glamorous Toolkit proved two things: that the idea works, and that it doesn't spread. After roughly ten years — free, MIT-licensed, backed by a company, a book, and a conference circuit — it sits at around 1,500 GitHub stars. Moldable development is one of the most under-adopted genuinely-good ideas in developer tooling, and the reason isn't the idea. It's the on-ramp: GT asks you to leave your editor, learn Pharo/Smalltalk, and mold your tools in a language almost nobody on your team writes.

We know that on-ramp is where people fall off, because we fell off it — the maintainers tried to get started with GT several times and bounced, and the forums are full of the same story. So *unoriginally-deterministic* keeps the validated idea and removes the thing that strangled it: you stay in VS Code, the tools are TypeScript, and you mold new ones with the AI assistant you already use — no new environment, no new language.

Two honest caveats, stated up front. First, this is not a moat. If feenk shipped a VS Code / TypeScript / AI-native on-ramp tomorrow, they might well do it better — and we'd happily shut this down. The goal is for the idea to finally cross the chasm, not for us to own it. Second, "lower the barrier and they will adopt" is a hypothesis, not a proven fact. The evidence we have — GT's numbers, and years of people stalling at setup — shows the *barrier* is real. Whether removing it converts people into sustained tool-builders is exactly what this project exists to test, in the open.

### Q2: My estate is 45 years of IBM Enterprise COBOL — copybooks, CICS, embedded SQL, dialect quirks. Open-source tree-sitter COBOL grammars only do partial COBOL85. What happens on *my* code?

A: The concern is smaller than it looks, because parsers are available and the toolkit is deliberately built on the simple, open-source ones. tree-sitter provides per-language grammars that compile to WASM and run in-process inside the extension — no JVM, no service, no heavy runtime — which is exactly the kind of simple dependency this toolkit commits to. COBOL has a main tree-sitter grammar, and others have already extended it with more capability; coverage grows the same way the toolkit itself does — openly and incrementally, by the people who need it.

Two things keep this honest. First, tree-sitter degrades gracefully by design: where it cannot parse a construct it emits explicit error nodes rather than failing the whole file — so "fail loud, mark the region unparsed, never fabricate an edge" is the natural default, not a bolt-on. You will sometimes see honest holes in the map; you will never see a confident fabrication where a hole should be. Second, the toolkit stays lean on purpose and sticks to simple open-source dependencies. If your estate genuinely needs commercial-grade dialect parsing, that is special tooling you bring and plug in — the toolkit gives you the extension point, not a bundled commercial parser.

### Q3: You say the tools never hallucinate because they're deterministic — but an AI *wrote* the tool. Haven't you just moved the hallucination up into the analyzer that draws the diagram?

A: Yes — and that is precisely the point, because it moves the fallibility somewhere you can contain it. When an AI explains your code, the mistake lives in the *answer*: the wrong claim and the confident tone arrive fused, there is nothing to inspect, and it is a fresh dice roll on every question. When an AI writes an analyzer, the fallibility lives in a small, static artifact you can check two independent ways — read the analyzer (short, far less than the system it inspects) or trace every element it draws back to source — and you only check it once. After that it runs deterministically, identically, forever, on any code, for free. The hallucination did not disappear. It moved from an invisible, per-use, unfixable place to a visible, one-time, checkable one.

### Q4: Molding is still work. Now I have to review AI-generated analyzer code for every non-standard question. Haven't you just moved my reading burden from the app code to the tool code?

A: Partly — and we won't pretend otherwise — but the two burdens are not the same size or frequency. Reading the tool is bounded: an analyzer that answers "what calls this program" is tens of lines whether your system is 50,000 lines or five million. Reading the system is unbounded and grows forever. You trade an unbounded, repeated cost for a bounded, one-time one — and the one-time cost is shared, because the tool is versioned in the repo and your teammates and next year's engineer inherit it without re-reading anything. And most days you don't mold at all: the starter commons answers the common questions with no code to review. Molding is only for the questions your codebase alone raises — and even then you review a page, once, not a system, repeatedly.

### Q5: I already have Sourcegraph for navigation and Copilot to explain things. Most days they're good enough. What's the one moment yours saves me and mine can't?

A: Keep them — this is not either/or, and for many tasks they are enough. The one moment we are built for that they structurally cannot serve: an answer about your code you can *trust without re-verifying* and *reuse for free*. Copilot's explanation is a coherence engine — right often, wrong sometimes, never labelled — so a careful engineer re-checks it, which erases the time it saved. Sourcegraph's navigation is deterministic but fixed: it answers the questions its authors built in (go-to-def, find-refs), not "trace every path that writes this file across COBOL and Java," and it won't let you mint a new deterministic question of your own. We are the tool for when you need an answer you can stake a change on, and the question you have isn't one someone else pre-built.

### Q6: The expensive mistakes in modernization are *semantic* — this paragraph rounds differently, this date routine has an edge case. Does your diagram stop me shipping *those*, or just help me feel oriented?

A: Honestly, no — not directly, and we deliberately do not claim it does. A structural view will not tell you a paragraph rounds the wrong way; that judgment is human and local. What it does is get you to the *right* paragraph — it collapses the "where do I even look, and what reaches this" that used to eat your hours, so your scarce attention lands on the handful of places the semantic risk actually lives instead of being spent skimming 200,000 lines to orient. It aims the reading; it does not replace the judgment. Anyone selling you a diagram that promises to catch your rounding bug is selling the coherence-engine lie in a new costume.

### Q7: Even a perfectly faithful dependency graph is a *compression* — it drops detail by definition. How do I know it didn't drop the exact thing that would've bitten me?

A: You don't, from the view alone — and this is the sharpest question here, because "faithful" and "dangerously incomplete" are not mutually exclusive. Two defenses. First, the view is never the final authority: every element traces back to the exact source, so the map is a fast index *into* the truth, not a replacement *for* it — when a node matters, you click through to the actual lines. Second, when a compression doesn't answer your question, you don't argue with it, you mold a new tool that keeps the detail this one dropped. That is the whole point of moldability: no single view is complete, so you build the one that surfaces exactly what you're worried about. A fixed tool that cannot be reshaped is where "incomplete" turns fatal; a moldable one turns "what did it drop?" into "then ask a different question."

### Q8: Zero users, open source, apparently one person. In six months the "commons" is a graveyard and you've moved on. Why build a multi-year modernization process on that?

A: Fair — and the honest answer is structural, not a reassurance. The artifacts outlive the project: the tools you build are plain TypeScript versioned in *your* repo. If this project vanished tomorrow, every tool you already molded keeps running — you own the code, you are not renting access to a service, and nothing shuts off. That is the opposite of the SaaS risk where a shutdown deletes your capability. The commons and the low molding barrier exist precisely to make value compound with contributors rather than depend on one maintainer. And we will not pretend a zero-user project is a safe bet — it is not, yet. The mitigation isn't a promise; it's the architecture: because what you build is yours and runs without us, adopting this costs you far less if we fail than adopting a hosted tool that can pull the rug.

## Internal FAQ

### Q1: The whole product rests on an arbitrary AI assistant reliably writing a *correct* deterministic tool against your API. LLMs write plausible-but-wrong code constantly. What makes that reliable?

A: The AI is not writing analysis from scratch, and that is the key. It composes over deterministic primitives that already exist and already work — tree-sitter and its language grammars, plus the toolkit's own vetted building blocks. Its job is narrowed from "write a correct program analysis" to "compose known-correct primitives to answer this question" — a far smaller, far more reliable target. And the output is never trusted blind: the generated tool is short, readable code the engineer reviews, and every element it draws traces back to source. Reliability comes from two directions at once — constrain what the AI has to invent, and make what it produces cheap to verify. The open risk we own: reliability scales directly with how well the toolkit narrows the AI's job. That narrowing is the core design work (see Q3), not a solved problem.

### Q2: Turning arbitrary analysis into a useful interactive visualization is the 80% GT spent a decade on. How much of that are you re-walking, solo?

A: We don't out-engineer GT's decade — we let AI write the view code too, over rendering primitives the toolkit provides. A tool's functional code, including how it renders, is AI-produced against a small set of vetted components (graph, tree, list, click-to-source). v1 is honest about the split: the toolkit ships a handful of hardcoded view types plus solid primitives; "the tool builds the tool," where AI composes novel views, is the direction, not the day-one claim. We are not rebuilding GT's polish — we are betting AI-composed views over good primitives reach *useful* far faster than *polished*.

### Q3: "Trace every element to source" is load-bearing for trust — but if an AI-generated tool skips provenance, the guarantee silently breaks. Can you enforce it, or is it a convention a bad generation violates?

A: This cannot be left to the AI's goodwill, so provenance is enforced structurally, not by convention. The drawing primitives *require* a source span to emit a node or an edge — you physically cannot draw a connection without passing the source it came from. A tool that skips provenance doesn't silently break the guarantee; it fails to produce anything. This is what "make it easy for the AI to get close" means concretely: the API is shaped so the correct, provenance-carrying path is the *only* path, and the easy thing the AI reaches for is the right thing. It is the single most important piece of design work in the project.

### Q4: You're one person facing five hard subsystems. What can you actually ship in three months — and what are you saying *no* to?

A: No hurry — this is a long-horizon build, and AI writing the tool's own code (Q2) is what makes solo feasible. The explicit no-list for v1: **no** broad language coverage (start with what the maintainer's own work needs); **no** general-purpose rendering framework (a few hardcoded views plus primitives); **no** enterprise features; **no** hosted anything; **no** polished contribution pipeline. v1 *is*: tree-sitter extraction, a small starter set of views the maintainer actually uses, provenance / trace-to-source, and the AI-molding workflow with human review. Everything else waits.

### Q5: A moldable framework invites an endless tail of "my generated tool broke," and you're a solo maintainer. How do you not drown — and is it honest to ask an enterprise to modernize on that bus factor?

A: Three honest points. One: the property that makes molding work also makes PRs cheap to review — tools are small and provenance makes them checkable, so reviewing a contribution is bounded work. Two: sustainability paths are open, not forced — if adoption warrants it, paid support is fine; if not, it stays a maintained side project. Three: on bus factor, we don't hide it — this is a solo project. The mitigation is architectural: because the tools you build are TypeScript in *your* repo and run without us, an enterprise's exposure if the project stalls is far smaller than adopting a hosted tool — you keep everything you built. No one is ever asked to switch; we showcase outcomes and let people decide, and they contribute through the normal open-source means.

### Q6: GT had talks, a book, and a company and still didn't cross. You have a repo. Where do the first 100 users come from?

A: The adoption strategy is proof, not promotion. First, dogfood it in real modernization work and let the outcomes be the argument — a genuine before/after on a real legacy estate. Second, share those outcomes where the audience already is (LinkedIn, and communities that care about moldable development). This is deliberately organic and slow — there is no growth engine, and given the no-hurry posture, that's acceptable. The bet: one honest, concrete "here's what I saw in 45 years of COBOL in ten minutes" is worth more than any volume of claims — the exact thing GT's abstract pitch never delivered for people.

### Q7: The commons needs contributors, who need users, who need a valuable commons. How do you cold-start that on day one?

A: The commons cold-starts from the maintainer's own recurring needs — the handful of tools wanted commonly across real projects (dependency map, call-flow, cross-language reach). That guarantees the starter set solves actual problems rather than speculative ones, and it makes "enough to make the first user glad" concrete: it's whatever already made the maintainer glad. The flywheel's first turn is the maintainer's own work; contributors arrive only after outcomes attract users.

### Q8: What kills this — the thing you don't say out loud? If AI assistants get reliably grounded on their own, "AI builds a deterministic tool" collapses into "AI just answers reliably," and your reason to exist evaporates.

A: We'll say it out loud: if agentic AI tools get reliably grounded — if they can answer "trace every path to this write" *reproducibly and verifiably*, not just plausibly — the deterministic-tool layer stops being necessary, and this should fold. That is a win for the idea, not a loss; if the agentic tools absorb this approach, great. The early signal to watch: the day a mainstream assistant produces the same structural answer twice, with source you can verify, the wedge is closing. Until then, "coherence engine" AI confidently guessing is exactly the gap this fills.

## The Verdict

**Overall: Forged — with one crack that must be prototyped before committing a year to it.**

This concept walked in as "GT-in-VS-Code" and walked out as something sharper and more honest: *a deterministic, moldable comprehension toolkit whose trust is enforced by architecture, whose reason to exist is validated by GT's own adoption failure, and whose scope is small enough for one person to ship.* It survived hard interrogation — two potential launch-blockers, an existential competitor, and a feasibility panel — and it got stronger at every turn because the maintainer refused to oversell. That honesty is the concept's single greatest asset; it's what makes the FAQ answers credible rather than defensive.

### Forged in steel
- **The core reframe.** "AI is a coherence engine, not a truth engine — so let it *build the lens*, never *be the answer*." Grounded verbatim in feenk's own writing, and it cleanly separates this from every "ask Copilot to explain your code" tool. This is a genuinely differentiated, defensible thesis.
- **The trust architecture.** Provenance as a *required argument* of the drawing primitives (no source → no edge) plus two orthogonal, cheap verification paths (audit the short analyzer / trace every element to source). This turns "trust me" into structure. It's the strongest piece of engineering thinking in the document.
- **The accessibility wedge, validated.** GT at ~1,500 stars in ~10 years, plus public HN evidence of people bouncing off Pharo onboarding, makes "the on-ramp the idea never had" a real, evidenced reason to exist — not a hunch.
- **Honesty as positioning.** No "stop reading code," no "accurate," an accepted semantic ceiling (Q6), fail-loud parsing, no bundled AI, own-your-tools sustainability. The concept promises exactly what it can deliver, which is rare and powerful.
- **The beachhead.** Legacy modernization (45yr COBOL + 10yr Java) is funded, high-stakes, comprehension-bottlenecked, and the maintainer's own lived reality — which makes proof-by-dogfooding credible.

### Needs more heat
- **The visualization layer (Q2).** "AI composes views over rendering primitives" is a direction, not a design. v1's actual view set, primitives, and layout approach are undefined. This is the largest chunk of undesigned surface area.
- **The extension contract (Q1/Q3).** The *mechanism* is identified (constrained primitives + required provenance) but not *designed*. This is simultaneously the best insight and the biggest open engineering problem.
- **The activation moment (Q6).** "Dogfood + LinkedIn" is an honest start but has no answer for what converts a curious first-try into a returning second-mold. Define the "aha" that pulls people back.
- **Commons graduation (Q7).** Cold-starting from the maintainer's own needs is sound, but the path by which a personal, project-specific tool becomes a generalized *shared* one (parameterization, de-specialization) is unspecified.

### Cracks in the foundation
- **THE crack — molding reliability is unproven (Q1/Q3).** Everything rests on the toolkit narrowing the AI's job enough that arbitrary assistants reliably emit correct, plug-in tools. Until a prototype demonstrates that, it's a hope. **What it takes to address:** build a thin vertical slice — tree-sitter extraction + provenance-enforcing primitives + one view — then have 3 different AI assistants each generate a custom tool and measure how many plug in and pass the trust checks *unedited*. That is the go/no-go experiment, and it should come before any broad build.
- **Adoption is the same wall that stopped GT.** Accessibility is a real wedge but unproven to *convert*. The concept honestly frames this as the falsifiable bet — but it remains the existential risk. **Mitigation:** decide in advance what a "striking enough" dogfood outcome looks like, so success is measurable rather than hoped for.
- **Obsolescence from below (Q8).** Agentic AI getting reliably grounded could erode the wedge mid-build. Not a deal-breaker (the maintainer is mission-aligned to fold), but it *compresses the window* — which argues strongly for a fast, thin prototype over a long, broad build.
- **Buyer/supplier mismatch.** The most serious buyer (an enterprise modernizing critical COBOL) is the worst fit for the least-durable supplier (a solo nights-and-weekends project). Own-your-tools softens it, but the tension is real. **Mitigation:** target individual engineers and small teams first; let enterprise pull happen only after the tools have proven durable in the wild.

**Bottom line:** The thinking is sharp, the positioning is honest, and the scope is realistic. The idea deserves to be built — but build the *reliability prototype* first. If three different AI assistants can each produce a working, provenance-carrying tool against your contract without hand-holding, you have a product. If they can't, you have GT's adoption problem in a new editor, and better to know in a month than a year.

<!-- coaching-notes-stage-4 -->
**#1 FEASIBILITY RISK (de-risk first):** Molding reliability (Q1/Q3) is unproven until built. The core bet — "the toolkit narrows the AI's job (compose vetted primitives, not write analysis from scratch) well enough that arbitrary AI assistants reliably emit correct, plug-in tools" — is validated only by building the extension contract and watching real AI sessions produce real tools without hand-fixing. This is the thing to prototype before anything else.

**KEY BUILDABILITY INSIGHT (the session's best):** Trust is enforced STRUCTURALLY, not by convention — **provenance (source span) is a REQUIRED argument of the drawing primitives.** No source → no node/edge emitted. A lazy AI-generated tool can't silently break the trace-to-source guarantee; it just produces nothing. This is what makes "make it easy for the AI to get close" concrete: the correct provenance-carrying path is the ONLY path. This is the make-or-break design surface and the answer to Q3(customer), Q7(customer), Q1/Q3(internal) simultaneously.

**Architecture / feasibility decisions locked:**
- AI composes over deterministic primitives (tree-sitter grammars + toolkit building blocks); it does NOT author analysis from first principles. This is the reliability mechanism.
- Visualization: v1 ships a few hardcoded view types + rendering primitives; "tool builds the tool" (AI composes novel views over primitives) is DIRECTION, not v1. Explicitly NOT rebuilding GT's decade of polish — betting AI-composed-useful beats hand-built-polished.
- Simple OSS deps only, tree-sitter-first (from Stage 3). Commercial parsing = user BYO plug-in.

**Solo v1 scope (the no-list):** v1 = (1) tree-sitter extraction, (2) small starter view set from maintainer's own recurring needs, (3) provenance / trace-to-source, (4) AI-molding workflow + human review. NO to: broad language coverage, general rendering framework, enterprise features, hosting, polished contribution pipeline.

**Posture (consistent across FAQ):** Mission-over-ownership — happy to fold if a better-funded accessible version (Q1-cust) or reliably-grounded agentic AI (Q8-int) supersedes it; success = the IDEA crossing the chasm. No-hurry, long-horizon, nights/weekends solo build. Adoption = proof-by-dogfooding (real COBOL/Java modernization work) + outcome-sharing (LinkedIn / moldable-dev communities), deliberately organic, no growth engine. Commons cold-starts from maintainer's own cross-project needs. Sustainability mitigated architecturally: user owns TS tools in own repo, they run without the project → low rug-pull exposure; optional paid support if adoption warrants.

**Kill-signal to watch (Q8):** a mainstream AI assistant producing the SAME structural answer twice with verifiable source (reproducible + grounded, not merely plausible) → the deterministic-tool wedge is closing.

<!-- coaching-notes-stage-3 -->
**Launch-blockers cleared:**
- **Q1 (GT already exists):** Answered on accessibility. Validated with public evidence — GT ~1,500 stars in ~10y; HN threads document people bouncing off Pharo/GT onboarding ("couldn't even tell how to use it," "convoluted and complex"). Positioning: *unoriginally-deterministic is the on-ramp the idea never had* (VS Code + TS + your own AI assistant). Owner accepts it's a head-start not a moat, and would happily be superseded by a better-funded accessible version — success metric is the IDEA crossing the chasm, not market capture. "Lower the barrier → adoption" framed honestly as the falsifiable bet the project exists to test.
- **Q2 (COBOL parsing reality):** Answered. ARCHITECTURE PRINCIPLE LOCKED BY OWNER: toolkit sticks to **simple open-source dependencies, tree-sitter-first** (WASM, in-process, no JVM — natively fits the extension). COBOL has a base tree-sitter grammar + community extensions; coverage grows openly. tree-sitter's ERROR nodes make "fail loud / honest holes / never fabricate" the DEFAULT. Commercial/heavy dialect parsing = **user-supplied special tooling via extension point**, NOT bundled. (Rejected earlier ProLeap/JVM-backend framing — violated the lean/on-machine promise. ProLeap/SonarQube noted only as evidence robust enterprise-COBOL parsing exists if a user must BYO.)

**Accepted trade-off (confirmed ceiling of the promise):**
- **Q6:** Product does NOT catch semantic/local bugs (rounding, date edge cases). It aims human reading at the right place; correctness judgment stays human. Accepted, not a gap to close — claiming otherwise would be the "coherence-engine lie" the product is positioned against.

**Key answer moves banked:**
- Q3: hallucination isn't removed, it's *relocated* from invisible/per-use/unfixable (AI answer) to visible/one-time/checkable (short analyzer, audited once then frozen).
- Q4: molding burden is bounded + one-time + repo-shared vs reading which is unbounded + repeated; most days = zero molding (starter commons).
- Q5: explicitly NOT either/or vs Sourcegraph/Copilot; the wedge = a trustworthy-without-re-verifying, reusable answer to a not-pre-built question.
- Q7: trace-to-source (map indexes truth, isn't a substitute) + molding (drop-a-detail → ask a different question).
- Q8: sustainability answered via ARCHITECTURE — user owns the TS tools in their own repo; they run without the project; nothing shuts off. Own-your-tools = the anti-SaaS-rug-pull.

**Competitive intel (verified, for Internal FAQ + Verdict):** CodeScene = social/git-history behavioral analysis (adjacent, differentiable). Sourcegraph Cody = deterministic code-graph nav BUT LLM comprehension layer (the confidently-wrong path). GT = the real incumbent, has adopted the exact positioning + supports COBOL, ~10y, ~1,500 stars → adoption is THE risk, accessibility is the only wedge.

**Requirement signals surfaced:** (1) extension contract / API quality is the make-or-break design surface (arbitrary LLM must emit correct plug-in tools); (2) trace-every-element-to-source is load-bearing for trust AND for Q7 — it's a hard requirement, not a nice-to-have; (3) tools are versioned-in-repo TS artifacts (ownership = the sustainability story); (4) starter commons must deliver day-one value with zero molding.

<!-- coaching-notes-stage-1 -->
**Concept type:** Open-source project (developer tool / VS Code extension). Rationale: user explicitly chose OSS. This calibrates all downstream FAQ toward adoption, contributor sustainability, and stakeholder value — NOT unit economics.

**Core concept:** Use non-deterministic AI to generate *deterministic*, reusable, token-free tools that extract visual views directly from code. Lineage: feenk's moldable development / Glamorous Toolkit, ported from the niche Pharo/Smalltalk ecosystem into the mainstream one (VS Code + TypeScript). Economic engine: a tool is written once (one-time AI/token capital cost) and runs free on any code forever; it does NOT break when code changes because it re-derives the view every run. New question → new tool; same question on new code → free.

**Assumptions challenged and resolved:**
- Challenged "no token cost": user correctly rebutted — reusable deterministic tools amortize the build cost; token spend is one-time capital, not per-use tax. Conceded.
- Challenged scope of visual comprehension: settled that visualization serves *structural* comprehension and *aims* scarce human reading attention down to the exact changed line; it does NOT replace human semantic/correctness judgment. This is the load-bearing, honest claim — press release must NOT say "stop reading code."
- Challenged determinism-on-context: prose context docs have no grammar, so deterministic extraction breaks and reintroduces hallucination/token cost. User chose **Path B** (accept non-determinism for context *if* needed) AND cleverly collapsed it back to code by making design-level views (EventStorming: events/policies/flows) *deterministic tools that read the code*. Caveat banked: code yields design-as-implemented, not design-as-intended — the *drift* between them is itself a high-value review signal.

**Beachhead (v1) decision:** Existing-code **structure comprehension** — visualize what a codebase *is* (architecture, dependencies, call flows) before touching it. Chosen over (a) change/diff-for-review and (b) design-level EventStorming board, which become roadmap on the same extraction engine. Known risk of this choice: no forcing event (a PR forces review; nothing forces comprehension) — press release must anchor a vivid trigger moment (onboarding to a strange codebase, inheriting legacy, incident on an unfamiliar subsystem).

**Prior-art / adversarial note for later stages:** Glamorous Toolkit is also MIT OSS and brilliant, yet moldable development hasn't crossed the chasm in ~10 years. OSS is not an adoption strategy — it IS the survival mechanism. Internal FAQ must answer: why does this break through where GT hasn't? (Working hypothesis: environment friction — GT requires living in Pharo; this lives where developers already are.)

**Customer / problem / stakes (locked):**
- Customer: bank engineer; code review mandated by compliance; drowning as AI multiplies code faster than anyone can read.
- Problem: comprehension of what exists and what's changing has always been throttled by reading-effort; AI turned a chronic problem acute.
- Stakes: review becomes theater → unread code ships → in a regulated bank that is regulatory + financial risk, not just tech debt.

<!-- coaching-notes-stage-2 -->
**MAJOR CUSTOMER REFRAME (supersedes Stage 1's "bank engineer"):** The customer is *anyone who must understand software they didn't write*. The sharp, real lead exemplar is the **legacy-modernization engineer** — drawn from the user's actual reality: modernizing 45 years of COBOL entangled with 10 years of Java, where no single person holds the whole picture. This scenario also resolves the Stage-1 worry that comprehension "has no forcing event" — **modernization IS the forcing event** (funded, high-stakes, comprehension-bottlenecked). Banking stays a strong sub-segment, not the frame.

**Deepest truth surfaced (load-bearing for FAQ):** The knowledge of what legacy programs actually do is *lost* — not documented, not in anyone's head. Recovering it today requires scarce, expensive SMEs to read every line, and this is true for ALL languages, not just COBOL. The product's core value: deterministically recover "what this program does" from the code itself, removing the SME-line-reading bottleneck.

**Rejected framings (do not resurrect):**
- "accurate map" / "always accurate" → overclaim; replaced with *fidelity to code* ("what's really there, not what you assume"). The tool is faithful to code, NOT truth about intent.
- "Stop reading code" / "replace reading" → overclaim; the honest claim is visualization *aims* scarce reading attention, correctness judgment stays human.
- Bank/PR-review over-anchoring → too narrow; the problem is general comprehension.
- The "payments ledger" example → too cute/specific; replaced with the truer "nobody remembers what these programs do."
- Confrontational "you approved that PR, did you read it?" headline → rejected in favor of truth-framing (points at tool value, not reader shame).

**Differentiators established:** (1) AI builds the lens once (verifiable), the deterministic lens tells the truth every run — vs "coherence engine" AI that's confidently wrong; (2) **moldability** — user authors new persistent, project-versioned tools on the fly; the moat AND the OSS flywheel (broadly-useful tools graduate to a shared commons); (3) **reproducibility** — same code → same picture on any machine, so a diagram is auditable evidence; (4) two orthogonal, cheap trust checks — audit the (short) analyzer, OR trace every drawn element back to its exact source line.

**Competitive landscape to verify/position against in FAQ:** Glamorous Toolkit (prior art, MIT OSS, Pharo — the thing being ported to mainstream VS Code/TS; ~10yr and hasn't crossed the chasm → adoption is THE risk). Also position against CodeScene, Sourcegraph, IDE call-hierarchies, and "just ask Copilot to explain it" (the confidently-wrong path). NOTE: these competitive claims still need real-world verification before Stage 3/4.

**Parked technical / feasibility facts (for Internal FAQ, Stage 4):**
- Deterministic extraction needs a **parser per language**; **tree-sitter** is the intended grammar layer (has COBOL, Java, + dozens more; grammars are shareable → makes "contribute a language" flywheel realistic). Multi-language coverage is now a FIRST-ORDER product dimension.
- **ARCHITECTURE DECISION — the toolkit bundles NO AI.** It is deterministic code end to end: a runtime + starter tools + a *moldable framework designed to be extended by whatever AI assistant the engineer already uses* (Claude Code / Copilot / Cursor / etc.). The product's job is to make AI-authored analysis tools deterministic, durable, and shareable — NOT to be the AI. Benefits: removes model/key dependency and liability (nothing AI running in a regulated repo), rides existing AI-assistant adoption, more durable than any bundled model (AI churns; a clean extension contract + deterministic primitives endure). This supersedes the earlier "BYO-key inside the product" framing. "Free, every run" = RUNNING tools; the one-time build cost lives entirely in the engineer's own assistant, outside the product.
- **NEW load-bearing design surface (Internal FAQ):** because the toolkit does NOT bundle AI, "*easy* to mold with any AI" rests entirely on the quality of the **extension contract** — the API, conventions, templates, and deterministic primitives an arbitrary LLM can pattern-match to emit a correct tool that plugs in. This is now the make-or-break engineering work. Trust checks (audit-the-analyzer / trace-every-element-to-source) remain the safety net.
- On-machine / no source leaves repo → critical for proprietary/regulated code adoption.
- Roadmap surfaces on same engine: change/diff-for-review; design-level EventStorming-from-code; **drift detection** (design-as-implemented vs design-as-intended) = high-value review signal.

**Open items still to resolve:** exact size of day-one starter commons (kept honest as "small, sharp starter set"); repository link placeholder; leader/community quotes flagged as grounded-but-composed / illustrative — must be replaced with real, cleared quotes before any PUBLISHED release.

