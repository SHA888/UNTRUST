# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-document thinking project, not a codebase. No code, no build, no tests. The artifact is `UNTRUST.md` — a working draft on whether LLM trust boundaries can be enforced architecturally rather than statistically. `README.md` is the reading guide.

There is nothing to compile, run, or test. Operations on this repo are document edits, version bumps, and verification logging.

## Document discipline (load-bearing)

The document follows strict versioning rules that override default "improve as you go" instincts:

- **Existing content is preserved verbatim across versions.** When revising, do not rewrite or "clean up" prior sections. If a prior claim was wrong, leave the original text and record the correction in Appendix B (Verification log).
- **All corrections go in Appendix B**, not inline. A reader re-reading §3 should see the v0.1.0 text; the per-version corrections accumulate in B.2 / B.2.1 / … / B.2.5 (one subsection per version that made corrections).
- **Additive changes only**, except for major version bumps. New sections (e.g., §0.5 in v0.2.0, §0.6 in v0.3.0) are inserted with a clear "(added vX.Y.Z)" marker. Existing section numbers do not shift.
- **Inline citations use [N] referencing Appendix A.** When adding a new reference, append to Appendix A and update the verification log in B.1 with the verification status.

### Versioning semantics

- **Patch (0.x.y)** — typos, citation URL fixes, formatting, metadata, cross-reference/citation-linkage corrections. No content claims change. Examples: v0.3.1 (distribution metadata); v0.4.1 (prior-art citations [24]–[32] added via the appendix, body verbatim); v0.4.2 (cross-reference, stale-date, and citation-linkage fixes).
- **Minor (0.y.0)** — new sections, new references, new sketches. Body content preserved verbatim. Examples: v0.2.0 added §0.5 + Appendix A/B; v0.3.0 added §0.6 + reference [16]; v0.4.0 added §13 + references [17]–[23]; v0.5.0 added §14 (wider-remit map) + references [33]–[35].
- **Major (x.0.0)** — reserved for the point where the document commits to a defensible position. Not yet reached.

Bump the version in the document header AND record the change in the changelog block immediately under the title. Do not silently edit.

## Substance (so you don't have to re-read ~117 KB)

The document organizes LLM trust enforcement into **five architectural patterns**:

1. **Sketch 1** — typed attention masks with structural zeros (§3). Closest existing instantiation: ASIDE.
2. **Sketch 2** — channel asymmetry / bandwidth bottleneck (§4). Least instantiated of the four.
3. **Sketch 3** — verifier as first-class architectural primitive (§5). Second least instantiated.
4. **Sketch 4** — capability tokens as architectural primitives (§6). Closest existing instantiation: CaMeL.
5. **Pattern 5** — parameterisation-class restriction (§0.6, added v0.3.0). Demonstrated for orthogonality (Cayley), not yet for trust properties.

The **synthesis (§7–§8)** is the load-bearing claim: trust requires a trusted base outside the neural network. The substrate fix is a "small trusted base + large untrusted neural component" architecture, drawing on capability-based OS security (Dennis & Van Horn 1966; KeyKOS; seL4).

The **criterion in §2** separates substrate fixes from mitigations: can the protection be defeated by sufficiently clever input within the training distribution? If yes → mitigation. If no → substrate fix.

**§13 (the enforcement boundary, added v0.4.0)** sharpens §2 without widening scope. It states the precondition behind the criterion — a property is architecturally enforceable only if it is intrinsic to the computation's internal structure, not defined by a relation to something outside the computation — and adds a three-class enforceability taxonomy: **A. structurally enforceable** (meets the §2 fix bar; trust-boundary info-flow lives here), **B. statistically guaranteeable** (e.g. conformal coverage under exchangeability — the class most likely to be mistaken for a fix, and is not one), **C. mitigation-only** (factual truth, honesty, intent-alignment, open-world OOD). §13.3 extends Sketch 3 to deductive-reasoning validity, conditional on autoformalisation (the §3.4-analog seam). §13.5 is an explicit non-collapse block: naming the wider "trustworthy AI" cluster here locates it relative to the §2 bar but does **not** bring it into scope.

**§14 (the wider trustworthy-AI remit, added v0.5.0)** reverses §13.5's scope decision: it brings the three adjacent clusters — hallucination/accuracy, alignment/honesty, robustness/OOD — into the document's remit, but as _mapped and classified_ clusters, **not** substrate-fixed ones. The expansion is responsible, not a dissolution: §2 is retained and repurposed as the cross-cluster discriminator, and each cluster is tagged with its §13.2 class (substrate trust → A; hallucination, alignment → C; OOD → B bounded / C open world). **No Class C cluster gets a substrate fix** — the inevitability results [17][18] and the §13.1 reference-dependence precondition foreclose one, so claiming a fix would make the document refute itself. The reversal is logged in B.2.6; the non-additive consequences (the title, §0, §10, and §13.5 still describe a substrate-trust-only document) are deferred to the eventual major (v1.0.0) bump, so the title currently under-describes the remit by design.

## Editorial guardrails

The document opens with a warning that "the conversation that produced this drifted toward comfortable mitigation framings multiple times before settling on the substrate framing." Re-reading and editing must resist the same drift:

- Do not soften the §2 criterion. Mitigations are useful but are not substrate fixes; the document exists to hold that distinction.
- Do not add speculative claims without a corresponding entry in Appendix B.3 (unverified claims) or B.1 (verified).
- Do not promote sketches to "designs" or "architectures" — the document calls them sketches deliberately.
- Do not remove the §0 epistemic status or §10 "what this does NOT address" — those sections exist to prevent overclaim.

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
