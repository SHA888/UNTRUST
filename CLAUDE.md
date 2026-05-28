# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-document thinking project, not a codebase. No code, no build, no tests. The artifact is `UNTRUST.md` — a working draft on whether LLM trust boundaries can be enforced architecturally rather than statistically. `README.md` is the reading guide.

There is nothing to compile, run, or test. Operations on this repo are document edits, version bumps, and verification logging.

## Document discipline (load-bearing)

The document follows strict versioning rules that override default "improve as you go" instincts:

- **Existing content is preserved verbatim across versions.** When revising, do not rewrite or "clean up" prior sections. If a prior claim was wrong, leave the original text and record the correction in Appendix B (Verification log).
- **All corrections go in Appendix B**, not inline. A reader re-reading §3 should see the v0.1.0 text; the v0.2.0/v0.3.0 corrections live in B.2 / B.2.1.
- **Additive changes only**, except for major version bumps. New sections (e.g., §0.5 in v0.2.0, §0.6 in v0.3.0) are inserted with a clear "(added vX.Y.Z)" marker. Existing section numbers do not shift.
- **Inline citations use [N] referencing Appendix A.** When adding a new reference, append to Appendix A and update the verification log in B.1 with the verification status.

### Versioning semantics

- **Patch (0.x.y)** — typos, citation URL fixes, formatting. No content claims change.
- **Minor (0.y.0)** — new sections, new references, new sketches. Body content preserved verbatim. Examples: v0.2.0 added §0.5 + Appendix A/B; v0.3.0 added §0.6 + reference [16].
- **Major (x.0.0)** — reserved for the point where the document commits to a defensible position. Not yet reached.

Bump the version in the document header AND record the change in the changelog block immediately under the title. Do not silently edit.

## Substance (so you don't have to re-read 55 KB)

The document organizes LLM trust enforcement into **five architectural patterns**:

1. **Sketch 1** — typed attention masks with structural zeros (§3). Closest existing instantiation: ASIDE.
2. **Sketch 2** — channel asymmetry / bandwidth bottleneck (§4). Least instantiated of the four.
3. **Sketch 3** — verifier as first-class architectural primitive (§5). Second least instantiated.
4. **Sketch 4** — capability tokens as architectural primitives (§6). Closest existing instantiation: CaMeL.
5. **Pattern 5** — parameterisation-class restriction (§0.6, added v0.3.0). Demonstrated for orthogonality (Cayley), not yet for trust properties.

The **synthesis (§7–§8)** is the load-bearing claim: trust requires a trusted base outside the neural network. The substrate fix is a "small trusted base + large untrusted neural component" architecture, drawing on capability-based OS security (Dennis & Van Horn 1966; KeyKOS; seL4).

The **criterion in §2** separates substrate fixes from mitigations: can the protection be defeated by sufficiently clever input within the training distribution? If yes → mitigation. If no → substrate fix.

## Editorial guardrails

The document opens with a warning that "the conversation that produced this drifted toward comfortable mitigation framings multiple times before settling on the substrate framing." Re-reading and editing must resist the same drift:

- Do not soften the §2 criterion. Mitigations are useful but are not substrate fixes; the document exists to hold that distinction.
- Do not add speculative claims without a corresponding entry in Appendix B.3 (unverified claims) or B.1 (verified).
- Do not promote sketches to "designs" or "architectures" — the document calls them sketches deliberately.
- Do not remove the §0 epistemic status or §10 "what this does NOT address" — those sections exist to prevent overclaim.

## Distribution constraints

- Codename **UNTRUST** is for internal reference and searchability only. Not for publishing, marketing, external sharing, or product naming.
- No license is specified; treat as private working material. A license decision precedes any further distribution.
- If implementation work spins out from a sketch, it gets its own repo, scope, and naming — separate from this document.

## Adding a new section

If adding a new pattern, sketch, or analysis section (a minor version bump):

1. Insert with a clear `## X.Y New section name (added vX.Y.Z)` heading.
2. Mirror the existing sketch structure where applicable: mechanism, cost, hard problem, what it gets right, what it does not solve.
3. Add inline citations `[N]` for any claim about prior art, with the reference appended to Appendix A.
4. Add a Verification log entry in Appendix B (B.1 for verified, B.3 for unverified, B.2.x for corrections).
5. Add the changelog block under the document title summarizing what changed and what was preserved.
