# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-document thinking project, not a codebase. No code, no build, no tests. The artifact is `UNTRUST.md` — a working draft on whether LLM trust boundaries can be enforced architecturally rather than statistically. `README.md` is the reading guide.

There is nothing to compile, run, or test. Operations on this repo are document edits, version bumps, and changelog entries.

## Document discipline

This is a **living note**, edited freely for clarity. (Through v1.x it followed a strict verbatim-preservation / additive-only discipline; v2.0.0 dropped that — the accumulated "(added vX)" markers, correction trail, and superseded-text archive had buried the ideas. History now lives in `CHANGELOG.md`, not inline.)

What remains load-bearing:

- **Edit for the reader.** Rewrite, tighten, and reorganize prose so it reads as a clean note. Do **not** re-introduce per-version markers, edit-history narration, or an inline correction trail — that is what `CHANGELOG.md` is for.
- **Keep the honesty apparatus.** References use `[N]` into the References section; anything verified-vs-asserted-vs-uncertain belongs in the "Sources & confidence" note at the end. Never let a mitigation read as a substrate fix.
- **`CHANGELOG.md` is the history.** Bump the version in the document header AND add a `CHANGELOG.md` entry. The header keeps only a pointer. Do not silently edit.

### Versioning semantics

- **Patch (x.y.Z)** — typos, link fixes, small clarifications. No claims change.
- **Minor (x.Y.0)** — new material: a section, references, or a sketch.
- **Major (X.0.0)** — restructure, retitle, scope change, or a change to this discipline. v1.0.0 was the bipartite Part I / Part II refactor; v2.0.0 was the readability rewrite that dropped verbatim preservation.

## Substance (so you don't have to re-read ~48 KB)

The document is **bipartite**, sorted by enforceability (title: _LLM Trust by Enforceability_).

**Part I — Substrate fix** (Class A; the original UNTRUST). Five enforcement patterns:

1. **Sketch 1** — typed attention masks with structural zeros (§4). Closest existing instantiation: ASIDE.
2. **Sketch 2** — channel asymmetry / bandwidth bottleneck (§5). Least instantiated of the four.
3. **Sketch 3** — verifier as first-class architectural primitive (§6). Second least instantiated.
4. **Sketch 4** — capability tokens as architectural primitives (§7). Closest existing instantiation: CaMeL.
5. **Pattern 5** — parameterisation-class restriction (§8). Demonstrated for orthogonality (Cayley), not yet for trust properties.

Prior art is §3. The **synthesis (§9–§10)** is the load-bearing claim: trust requires a trusted base outside the neural network — a "small trusted base + large untrusted neural component" architecture, drawing on capability-based OS security (Dennis & Van Horn 1966; KeyKOS; seL4). Cross-disciplinary inputs are §11.

The **criterion in §2** separates substrate fixes from mitigations: can the protection be defeated by sufficiently clever input within the training distribution? If yes → mitigation. If no → substrate fix.

**Bridge — §12 (the enforcement boundary).** States the precondition behind §2 — a property is architecturally enforceable only if it is intrinsic to the computation's internal structure, not defined by a relation to something outside it — and the three-class taxonomy: **A. structurally enforceable** (the §2 fix bar; Part I), **B. statistically guaranteeable** (e.g. conformal coverage under exchangeability — the class most likely to be mistaken for a fix, and is not one), **C. mitigation-only** (factual truth, honesty, intent-alignment, open-world OOD). §12 also extends Sketch 3 to deductive-reasoning validity, conditional on autoformalisation (the labeling-boundary analog), and it is what *defines* the Part I / Part II split.

**Part II — Non-substrate fix** (§13; Class B/C). The three adjacent clusters — hallucination/accuracy, alignment/honesty, robustness/OOD — mapped and classified, **not** substrate-fixed. §2 is retained as the cross-cluster discriminator; each cluster is tagged with its §12 class (hallucination, alignment → C; OOD → B bounded / C open world). **No Class C cluster gets a substrate fix** — the inevitability results [17][18] and the reference-dependence precondition (§12) foreclose one, so claiming a fix would make the document refute itself. The lanes stay separate (a single "fix" spanning both is the conflation §0 warns against).

## Editorial guardrails

The document opens with a warning that "the conversation that produced this drifted toward comfortable mitigation framings multiple times before settling on the substrate framing." Re-reading and editing must resist the same drift:

- Do not soften the §2 criterion. Mitigations are useful but are not substrate fixes; the document exists to hold that distinction.
- Do not add speculative claims without flagging them in the "Sources & confidence" note (verified vs. asserted vs. uncertain).
- Do not promote sketches to "designs" or "architectures" — the document calls them sketches deliberately.
- Do not remove the §0 epistemic status or §14 "what this does NOT address" — those sections exist to prevent overclaim.

## Distribution constraints

- The repo is **publicly visible** as a working draft. The codename **UNTRUST** is a working identifier, not branding — it names the load-bearing commitment, not a product. Do not use the name in marketing copy or as a project label outside this document.
- **No license is specified; all rights reserved by default.** Treat as readable working material. Do not assume permission for reuse, redistribution, or derivative work — explicit consent is required.
- If implementation work spins out from a sketch, it gets its own repo, scope, license, and naming — separate from this document.

## Adding a new section

If adding a new pattern, sketch, or analysis section (a minor version bump):

1. Insert it where it reads best, numbered into the existing §0–§16 sequence (under the right Part). Renumber neighbours if needed — there is no verbatim/additive constraint any more.
2. Mirror the sketch structure where applicable: mechanism, cost, hard problem, what it gets right, what it does not solve.
3. Cite prior art with `[N]` into the References section.
4. If a claim is unverified or is the author's framing rather than an established result, say so in the "Sources & confidence" note.
5. Add a `CHANGELOG.md` entry.

**Do not include `Co-Authored-By:` trailers in commit messages.** This applies to all assistant-generated commits, including those produced by Claude Code or any other AI tool. Commit attribution stays with the human author. Boilerplate trailers add noise to the history without conveying meaningful authorship and have been retroactively stripped from past commits.

**English-only requirement:**

- All Plans.md content must be in English (headers, table columns, task descriptions, status markers).
- No Japanese characters in Plans.md status markers (use `cc:done` instead of `cc:完了`, `cc:wip` instead of `cc:WIP`, etc).
- All harness output and documentation must be in English.
- This applies strictly to tracked files; commit to this constraint when editing Plans.md.
