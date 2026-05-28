# Changelog — UNTRUST

Version history for `UNTRUST.md` (_LLM Trust by Enforceability_). This file is the version ledger that previously lived in the document header; it was split out at v1.0.1. Entries are newest-first, and the prose of each pre-v1.0.1 entry is preserved verbatim from the header.

The document follows strict additive / verbatim-preservation rules — see `CLAUDE.md`.

> _Note: entries reference section numbers as they stood when each entry was written. Pre-v1.0.0 entries use the original numbering; the v1.0.0 refactor renumbered every section — see the old→new map in Appendix B.2.7 of `UNTRUST.md`._

## v1.0.2 — 2026-05-28

Consistency correction. §0's epistemic-status bullet ("Nobody has built any of them at LLM scale…") contradicted §3's honest repositioning, which has held since v0.2.0 that several sketches have working partial implementations and that the accurate claim is narrower — no current approach is a _complete_ substrate fix (none meets the §2 criterion fully). The v1.0.0 §0 rewrite had carried the stale bullet forward by oversight. Aligned the §0 bullet with §3; no position changed. Logged in B.2.8; the original §0 wording remains archived in B.6.

## v1.0.1 — 2026-05-28

Split the version history out of the `UNTRUST.md` header into this dedicated `CHANGELOG.md`. Organizational/metadata change only — no content claims, sections, references, or numbering changed. The document header now carries a one-line pointer here, and the per-version entries below are preserved verbatim from the header.

## v1.0.0 — 2026-05-28

First major bump, and the first non-additive change in the document's history. The material is refactored from one linear argument into an explicit **two-category** structure: **Part I — Substrate fix** (the original UNTRUST: the substrate problem §1, the §2 criterion, the four sketches, Pattern 5, the trusted-base synthesis), a **Bridge** (§12, the enforcement boundary / A–B–C taxonomy that defines the split), and **Part II — Non-substrate fix** (the v0.5.0 remit: hallucination, alignment, robustness/OOD). Every section was renumbered and reordered (old→new map in B.2.7); all section _content_ is preserved verbatim except the three reframed pieces — the title, §0, and §12.5 (was §13.5) — whose superseded text is archived verbatim in **B.6**. The title changes from "Substrate Architecture for LLM Trust Boundaries" to "LLM Trust by Enforceability". This fully realizes the scope reversal B.2.6 had deferred. No content claims, references, or sketches were added, and the §2 fix/mitigation criterion is unchanged. The document remains a working draft — this bump records a structural commitment, not the "defensible position" milestone x.0.0 is ultimately reserved for.

## v0.5.0 — 2026-05-28

Scope expansion. §14 (added) brings the three adjacent "trustworthy AI" clusters — hallucination/accuracy, alignment/honesty, robustness/OOD — into UNTRUST's remit, **reversing the scope decision §13.5 made in v0.4.0** (logged in B.2.6). The expansion is responsible, not a dissolution: §2's fix/mitigation criterion is retained and repurposed as the cross-cluster discriminator, and each newly-admitted cluster is tagged with its true enforceability class (§13.2) — all three are mitigation- or statistical-guarantee-class, and **none receives a substrate fix**, because the inevitability results [17][18] and the §13.1 precondition foreclose one. §14 carries the four-cluster comparison (failure mode / fix space / success criterion), the independence argument, and an explicit map onto §13.2's A/B/C classes. Added references [33]–[35] and Appendix B.2.6 / B.3 entries. Body §1–§13, §0.5, §0.6 preserved verbatim; the document title, §0, §10, and §13.5 are **not** rewritten — that non-additive reframe is deferred to the eventual major (v1.0.0) bump. Until then the title under-describes the remit, by design (additive-only discipline).

## v0.4.2 — 2026-05-28

Internal-consistency patch from a 2026-05-28 cross-check of numbering, headings, cross-references, and citation linkage. Four discrepancies fixed, none touching a content claim: (1) Appendix A's references-preamble date — "All references verified 2026-05-27" was true only for [1]–[15]; scoped accordingly, with [16]–[32] pointing to their own dates in B.1 / B.2.4. (2) Appendix A entries [24]–[32] relabelled "Cited in" → "Supports", since the verbatim body (§0.5 / §4 / §5) predates them and they are linked via the changelog and Appendix B, not inline brackets. (3) B.4's VLM-citation bullet now points to its v0.4.1 closure ([24][25], B.2.4). (4) Added B.2.5 logging the §0 "synthesis (§7)" cross-reference (the trusted-base insight is stated in §7; the system synthesis is §8 — §0's verbatim text is preserved, the correction is logged). Body §1–§12, §0.5, §0.6, and §13 preserved verbatim.

## v0.4.1 — 2026-05-28

Citation/verification patch for Sketch 2 (§4 / §0.5) and Sketch 3 (§5 / §0.5), re-verified against 2024–2026 primary sources. Added references [24]–[32] (CLIP, Flamingo, IBProtector, SecurityLingua, Constitutional Classifiers + 2026 successor, Boundary Point Jailbreaking, grammar-constrained / grammar-aligned decoding) and Appendix B.1 rows + new subsection B.2.4. **No body-claim changes**: §0.5's assessments — Sketch 2 "least instantiated," Sketch 3 "second least-instantiated, no direct LLM-scale implementation" — were re-verified and _survive_; the patch supplies their evidence rather than correcting them. §0.5, §0.6, §13, and body §1–§12 preserved verbatim. Notably, §5.4's prediction (a neural verifier is attackable like the generator) is now empirically corroborated by 2026 attacks on production neural guards [30].

## v0.4.0 — 2026-05-28

Added §13 — the precondition behind the §2 criterion (architectural enforcement requires a property intrinsic to the computation's internal structure), a three-class enforceability taxonomy (structurally enforceable / statistically guaranteeable / mitigation-only), the extension of Sketch 3 to deductive-reasoning validity (stated conditionally, with autoformalisation as the §3.4-analog seam), the semantic-seam pattern shared across Class A enforcements, and an explicit non-collapse block (§13.5). Added references [17]–[23] and Appendix B.1 / B.2.3 / B.3 entries. Body sketches §1–§12, §0.5, and §0.6 preserved verbatim. This is an additive minor bump that sharpens the enforcement boundary; it does **not** expand scope to the wider "trustworthy AI" cluster, which remains out of scope per §10 and §13.5. The categorisation locates that cluster relative to the §2 bar without absorbing it.

## v0.3.1 — 2026-05-28

Distribution status update — the working draft is now publicly visible. Body sketches §1–§12, §0.5, and §0.6 preserved verbatim. The only text changes are the front-matter blockquote above (rewording the codename note to reflect the new status) and a log entry in Appendix B.2.2. No content claims change; this is a metadata patch.

## v0.3.0 — 2026-05-28

Added §0.6 — a fifth enforcement pattern (parameterisation-class restriction) identified from adjacent work on constrained-operator adapters. The pattern is taxonomically distinct from the four sketches and was missed in v0.1.0/v0.2.0. Added reference [16] and Appendix B.3 entry. Body sketches §1–§12 and §0.5 preserved verbatim. No claim that this pattern addresses trust — see caveats in §0.6.

## v0.2.0 — 2026-05-27

Added §0.5 (prior art and precedent), inline citations throughout, Appendix A (references), and Appendix B (verification log). Body sketches §1–§12 preserved verbatim from v0.1.0. The verification pass identified significant prior art — particularly CaMeL, ASIDE, StruQ — that the original document treated as more speculative than warranted. The honest repositioning is in §0.5.

## v0.1.0 — 2026-05

Initial working draft: the four sketches (typed attention masks, channel asymmetry, verifier signoff, capability tokens) and the trusted-base synthesis, the epistemic status (§0), the substrate problem, and the §2 fix/mitigation criterion. No prior-art mapping, citations, or appendices yet — those arrived in v0.2.0.
