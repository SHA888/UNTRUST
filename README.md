# UNTRUST

Substrate architecture sketches for LLM trust boundaries. Working draft, publicly visible.

## What this is

A frame for thinking about whether trust boundaries inside LLM systems can be enforced architecturally rather than statistically. Four sketches plus a synthesis, with prior-art mapping and a verification log.

UNTRUST is not a research program, not a proposal, not a publishable artifact. It's a structured place to hold the substrate question against the gravity of comfortable mitigation answers.

## Contents

- `UNTRUST.md` — the document itself. Current version 0.3.1.
- `README.md` — this file.

## Reading order

First pass:
1. §0 — epistemic status. Calibrates expectations.
2. §0.5 — prior art. What's already built. Read before the sketches so they're not mistaken for novel proposals.
3. §1–§2 — the substrate problem and the criterion separating fixes from mitigations.
4. §0.6 — fifth enforcement pattern, added v0.3.0. Connects to §1–§2 but remains unvalidated for trust properties (see caveats in §0.6).

Second pass:
5. §3–§6 — the four sketches. Each has the same structure: mechanism, cost, hard problem, what it gets right, what it doesn't solve.
6. §7–§8 — pattern across sketches and the trusted-base synthesis.

Later passes:
7. §9–§12 — cross-disciplinary inputs, limits, open questions, notes on use.
8. Appendix A — references with full bibliographic info.
9. Appendix B — verification log. What was checked, what was corrected, what remains uncertain.

## Versioning

Semantic-version-shaped, applied to a thinking document:

- **Patch (0.x.y)** — clarifications, typos, citation fixes, distribution/metadata updates. No substantive content change.
- **Minor (0.y.0)** — additive: new sections, new references, new sketches. Existing content preserved verbatim. Examples: v0.2.0 added prior-art mapping; v0.3.0 added Pattern 5.
- **Major (x.0.0)** — reserved for the point where the document commits to a position defensible in writing. Not yet.

All updates are surgical. Existing content is preserved verbatim across versions; corrections go in the verification log so re-reading old sections doesn't require remembering what changed.

## Scope

**What UNTRUST addresses**: whether LLM systems can have architecturally enforced trust boundaries — i.e., boundaries that cannot be defeated by sufficiently clever input within the training distribution.

**What UNTRUST does NOT address**:
- Semantic alignment, deceived principals, side channels, supply chain, multi-agent dynamics — see §10.
- Mitigations (RLHF, constitutional AI, prompt hardening, classifier guards) except as contrast to substrate fixes.
- Specific deployment recipes — this is structural, not operational.

## Distribution

Publicly visible as a working draft (v0.3.1+). The codename UNTRUST is a working identifier, not branding — it names the load-bearing commitment (the neural component is treated as structurally untrusted by design) and is not intended for product, marketing, or external naming use. If any of the sketches becomes the basis for actual implementation work, that work gets its own scope, license, and naming — separate from this document.

## License

None specified; all rights reserved by default. The document is readable as a working draft. Reuse, redistribution, or derivative work requires explicit permission. A license decision will be made before any change that would meaningfully expand permitted use.
