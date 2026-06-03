# UNTRUST

**LLM Trust by Enforceability** sorted by what can actually be enforced: _substrate fixes_ (Part I) vs. _non-substrate fixes_ (Part II). Working draft, publicly visible.

## What this is

A frame for thinking about LLM trust, organized by enforceability. **Part I — Substrate fix** asks whether trust boundaries can be enforced architecturally rather than statistically: four sketches plus a trusted-base synthesis, with prior-art mapping. A **bridge** (§12) states the enforcement boundary and a three-class taxonomy (A structurally enforceable / B statistically guaranteeable / C mitigation-only). **Part II — Non-substrate fix** maps the wider trustworthy-AI clusters (hallucination, alignment, robustness/OOD) that fall on the non-enforceable side. Backed by a references list and a short sources-and-confidence note.

UNTRUST is not a research program, not a proposal, not a publishable artifact. It's a structured place to hold the substrate question against the gravity of comfortable mitigation answers.

## Contents

- `UNTRUST.md` — the document itself (_LLM Trust by Enforceability_). Current version 2.2.0.
- `ENFORCEABILITY-DISCIPLINE.md` — companion labeling discipline (a lens, not an enforcer). Turns the §12 enforceability cut into a per-claim check for trust *claims*: a paper-shaped argument (the separation is a property of claims, not code) plus a four-field labeling schema that keeps Class A/B/C distinct under implementation pressure. Binds nothing — unadopted, it is itself Class C by UNTRUST's §2 criterion (see UNTRUST §16). Working draft, v0.5.0.
- `CHANGELOG.md` — version history.
- `README.md` — this file.

## Reading order

1. §0 — epistemic status. Calibrates expectations; explains the two categories.

**Part I — Substrate fix:**

2. §1–§2 — the substrate problem and the criterion separating fixes from mitigations.
3. §3 — prior art. What's already built. Read before the sketches so they're not mistaken for novel proposals.
4. §4–§7 — the four sketches. Each has the same structure: mechanism, cost, hard problem, what it gets right, what it doesn't solve.
5. §8 — Pattern 5 (parameterisation-class restriction). Distinct from the sketches; unvalidated for trust properties.
6. §9–§10 — pattern across sketches and the trusted-base synthesis.
7. §11 — required cross-disciplinary inputs.

**Bridge:**

8. §12 — the enforcement boundary: the precondition behind §2 and the three-class enforceability taxonomy. This is what defines the split between the two categories.

**Part II — Non-substrate fix:**

9. §13 — the wider trustworthy-AI clusters: the four problem clusters, their independence, and their map onto §12's classes.

**Back matter:** §14 limits, §15 open questions, §16 notes on use; then the References and a short Sources & confidence note.

## Versioning

Semantic-version-shaped, applied to a thinking document:

- **Patch (x.y.Z)** — typos, link fixes, small clarifications.
- **Minor (x.Y.0)** — new material: a section, references, or a sketch.
- **Major (X.0.0)** — restructure, retitle, scope change, or a change to the editing discipline. v1.0.0 refactored into Part I / Part II; v2.0.0 rewrote the document as a clean note and dropped the earlier verbatim-preservation discipline (the per-version markers and correction trail that had buried the ideas).

Full version history is in `CHANGELOG.md`.

## Scope

**What UNTRUST addresses**: LLM trust, sorted by enforceability into two categories. **Part I (substrate fix)** is the core: whether trust boundaries can be enforced architecturally — boundaries that cannot be defeated by sufficiently clever input within the training distribution (Class A). **Part II (non-substrate fix)** maps the adjacent clusters — hallucination/accuracy, alignment/honesty, robustness/OOD — which are Class B (statistical guarantee where bounded) or Class C (mitigation-only). Both categories live in one document; the §2 criterion and the §12 enforcement boundary keep them sorted.

**What UNTRUST does NOT address**:

- A substrate fix for Part II. The non-substrate clusters are mapped and classified, never claimed as architecturally enforceable; the §2 line and the clusters' independence hold.
- Deceived principals, side channels, supply chain, multi-agent dynamics, computational cost — see §14.
- Specific deployment recipes — this is structural, not operational.

## Distribution

Publicly visible as a working draft (v2.0.1+). The codename UNTRUST is a working identifier, not branding — it names the load-bearing commitment (the neural component is treated as structurally untrusted by design) and is not intended for product, marketing, or external naming use. If any of the sketches becomes the basis for actual implementation work, that work gets its own scope, license, and naming — separate from this document.

## License

None specified; all rights reserved by default. The document is readable as a working draft. Reuse, redistribution, or derivative work requires explicit permission. A license decision will be made before any change that would meaningfully expand permitted use.
