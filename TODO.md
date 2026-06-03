# UNTRUST — TODO

Atomic task list for the UNTRUST thinking project. Status: `[x]` done · `[~]` in progress · `[ ]` open.
UNTRUST is a single-document project; "tasks" here are document, research, and coordination work —
not code. Keep the §0 epistemic discipline and the §2 fix/mitigation line: do not let any task drift
a mitigation into a fix.

## Status

The document is stable at **v2.2.1**. The companion *enforceability discipline* reached convergence
(five stress tests; both axes break-then-clean) at **v0.6.0**. The build fork was taken: **KEEP**
([github.com/SHA888/KEEP](https://github.com/SHA888/KEEP)) is the §15.6 demonstration probe. UNTRUST's
forward motion is now (a) maintenance, (b) absorbing KEEP findings, (c) the long §15 research agenda.

## Maintenance (atomic, near-term)

- [ ] Cross-reference audit after the v2.1.0–v2.2.1 edits: every `§N` pointer and `[N]` ref resolves
- [ ] Re-verify the newer references against arXiv (the Sources & confidence cadence): [16][29][30], plus the new [36][37]
- [ ] Confirm §12 ↔ §13 consistency after the B-avg/B-wc split (no remaining single-"Class B" phrasing)
- [ ] Confirm companion ↔ §12/§16 stay consistent (companion is Class C, lens-not-enforcer)

## Companion convergence (optional — removes the selection-bias caveat)

- [ ] Run convergence stress tests with **externally chosen** tools (not author-picked) to kill selection bias
- [ ] Add a clean Class C and a clean B-avg single-property tool to the streak
- [ ] If a test breaks the schema, fold the fix; if several pass clean, record "converged" explicitly

## Feedback loop from KEEP (update as phases land)

- [ ] When KEEP P3 measures the minimal trusted base, update §15.3 (minimal trusted base) with real LOC/structure
- [ ] When KEEP defines the base↔model protocol, sketch an answer into §15.4
- [ ] Fold KEEP's empirical coercion-path findings back into §7's "hard problem" note
- [ ] Keep the §15.6 KEEP pointer current as the probe advances (P0 → P3)

## Long research agenda (the real §15 backlog — years-long, not near-term)

- [ ] §15.1 — how the labeling boundary is made trustworthy (Sketches 1, 2)
- [ ] §15.2 — can Sketch 3's verifier be made non-neural for the properties that matter
- [ ] §15.5 — what training looks like for a model designed to live above a trusted base
- [ ] §15.7 — composition with existing mitigations (RLHF, capability scoping, human-in-the-loop)
- [ ] §15.8 — the upgrade path (a trusted base alongside existing LLMs, not a full rebuild)

## Decisions (owner: human)

- [ ] License decision (currently all-rights-reserved working draft)
- [ ] Whether to externalize the companion as a citable paper (the earlier fork option not taken)
- [ ] Whether the B-avg/B-wc split warrants a deeper §13 treatment or stays a §12 refinement
