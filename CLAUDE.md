# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-document thinking project, not a codebase. No code, no build, no tests. The artifact is `UNTRUST.md` — a working draft on whether LLM trust boundaries can be enforced architecturally rather than statistically. `README.md` is the reading guide.

There is nothing to compile, run, or test. Operations on this repo are document edits, version bumps, and verification logging.

## Document discipline (load-bearing)

The document follows strict versioning rules that override default "improve as you go" instincts:

- **Existing content is preserved verbatim across versions.** When revising, do not rewrite or "clean up" prior sections. If a prior claim was wrong, leave the original text and record the correction in Appendix B (Verification log). The sole exception is a **major bump**: v1.0.0 reframed the title, §0, and §12.5 non-additively, archiving the superseded text verbatim in B.6 rather than deleting it — the preservation ethos is kept even when the live text changes.
- **All corrections go in Appendix B**, not inline. The per-version corrections accumulate in B.2 / B.2.1 / … / B.2.7 (one subsection per version that made corrections); B.6 archives text superseded by the v1.0.0 reframe.
- **Additive changes only**, except for major version bumps. Minor/patch bumps insert new sections with a clear "(added vX.Y.Z)" marker and do not shift existing section numbers. The v1.0.0 **major** bump was the sole non-additive change: it renumbered and reordered every section into Part I / Part II (old→new map in B.2.7).
- **Inline citations use [N] referencing Appendix A.** When adding a new reference, append to Appendix A and update the verification log in B.1 with the verification status.

### Versioning semantics

- **Patch (0.x.y)** — typos, citation URL fixes, formatting, metadata, cross-reference/citation-linkage corrections. No content claims change. Examples: v0.3.1 (distribution metadata); v0.4.1 (prior-art citations [24]–[32] added via the appendix, body verbatim); v0.4.2 (cross-reference, stale-date, and citation-linkage fixes).
- **Minor (0.y.0)** — new sections, new references, new sketches. Body content preserved verbatim. Examples: v0.2.0 added §0.5 (now §3) + Appendix A/B; v0.3.0 added §0.6 (now §8) + reference [16]; v0.4.0 added §13 (now §12) + references [17]–[23]; v0.5.0 added §14 (now §13) + references [33]–[35].
- **Major (x.0.0)** — the only bump allowed to break verbatim preservation (renumber, reorder, rewrite framing). v1.0.0 refactored into Part I (substrate fix) / Part II (non-substrate fix), renumbered every section, and reframed the title/§0/§12.5 (superseded text in B.6). A future major bump may mark the document committing to a defensible position; v1.0.0 records a structural commitment, not that milestone.

Bump the version in the document header AND record the change in the changelog block immediately under the title. Do not silently edit.

## Substance (so you don't have to re-read ~125 KB)

The document is **bipartite**, sorted by enforceability (title: _LLM Trust by Enforceability_).

**Part I — Substrate fix** (Class A; the original UNTRUST). Five enforcement patterns:

1. **Sketch 1** — typed attention masks with structural zeros (§4). Closest existing instantiation: ASIDE.
2. **Sketch 2** — channel asymmetry / bandwidth bottleneck (§5). Least instantiated of the four.
3. **Sketch 3** — verifier as first-class architectural primitive (§6). Second least instantiated.
4. **Sketch 4** — capability tokens as architectural primitives (§7). Closest existing instantiation: CaMeL.
5. **Pattern 5** — parameterisation-class restriction (§8, added v0.3.0). Demonstrated for orthogonality (Cayley), not yet for trust properties.

Prior art is §3. The **synthesis (§9–§10)** is the load-bearing claim: trust requires a trusted base outside the neural network — a "small trusted base + large untrusted neural component" architecture, drawing on capability-based OS security (Dennis & Van Horn 1966; KeyKOS; seL4). Cross-disciplinary inputs are §11.

The **criterion in §2** separates substrate fixes from mitigations: can the protection be defeated by sufficiently clever input within the training distribution? If yes → mitigation. If no → substrate fix.

**Bridge — §12 (the enforcement boundary).** States the precondition behind §2 — a property is architecturally enforceable only if it is intrinsic to the computation's internal structure, not defined by a relation to something outside it — and the three-class taxonomy: **A. structurally enforceable** (the §2 fix bar; Part I), **B. statistically guaranteeable** (e.g. conformal coverage under exchangeability — the class most likely to be mistaken for a fix, and is not one), **C. mitigation-only** (factual truth, honesty, intent-alignment, open-world OOD). §12.3 extends Sketch 3 to deductive-reasoning validity, conditional on autoformalisation (the §4.4-analog seam). §12 is what *defines* the Part I / Part II split.

**Part II — Non-substrate fix** (§13; Class B/C). The three adjacent clusters — hallucination/accuracy, alignment/honesty, robustness/OOD — mapped and classified, **not** substrate-fixed. §2 is retained as the cross-cluster discriminator; each cluster is tagged with its §12.2 class (hallucination, alignment → C; OOD → B bounded / C open world). **No Class C cluster gets a substrate fix** — the inevitability results [17][18] and the §12.1 reference-dependence precondition foreclose one, so claiming a fix would make the document refute itself. §12.5 restates the boundary for the two categories; the lanes stay separate (a single "fix" spanning both is the conflation §0 warns against).

## Editorial guardrails

The document opens with a warning that "the conversation that produced this drifted toward comfortable mitigation framings multiple times before settling on the substrate framing." Re-reading and editing must resist the same drift:

- Do not soften the §2 criterion. Mitigations are useful but are not substrate fixes; the document exists to hold that distinction.
- Do not add speculative claims without a corresponding entry in Appendix B.3 (unverified claims) or B.1 (verified).
- Do not promote sketches to "designs" or "architectures" — the document calls them sketches deliberately.
- Do not remove the §0 epistemic status or §14 (was §10) "what this does NOT address" — those sections exist to prevent overclaim.

## Distribution constraints

- The repo is **publicly visible** as a working draft. The codename **UNTRUST** is a working identifier, not branding — it names the load-bearing commitment, not a product. Do not use the name in marketing copy or as a project label outside this document.
- **No license is specified; all rights reserved by default.** Treat as readable working material. Do not assume permission for reuse, redistribution, or derivative work — explicit consent is required.
- If implementation work spins out from a sketch, it gets its own repo, scope, license, and naming — separate from this document.

## Adding a new section

If adding a new pattern, sketch, or analysis section (a minor version bump):

1. Insert with a clear `## X.Y New section name (added vX.Y.Z)` heading.
2. Mirror the existing sketch structure where applicable: mechanism, cost, hard problem, what it gets right, what it does not solve.
3. Add inline citations `[N]` for any claim about prior art, with the reference appended to Appendix A.
4. Add a Verification log entry in Appendix B (B.1 for verified, B.3 for unverified, B.2.x for corrections).
5. Add the changelog block under the document title summarizing what changed and what was preserved.

**Do not include `Co-Authored-By:` trailers in commit messages.** This applies to all assistant-generated commits, including those produced by Claude Code or any other AI tool. Commit attribution stays with the human author. Boilerplate trailers add noise to the history without conveying meaningful authorship and have been retroactively stripped from past commits.

**English-only requirement:**

- All Plans.md content must be in English (headers, table columns, task descriptions, status markers).
- No Japanese characters in Plans.md status markers (use `cc:done` instead of `cc:完了`, `cc:wip` instead of `cc:WIP`, etc).
- All harness output and documentation must be in English.
- This applies strictly to tracked files; commit to this constraint when editing Plans.md.
