# LLM Trust by Enforceability

**Codename: UNTRUST**
**Substrate fixes vs. non-substrate fixes, sorted by what can actually be enforced. Working draft.**
**Version: 2.2.1** (2026-06-04)

> The codename names the load-bearing commitment: the neural component is treated as structurally untrusted by design. UNTRUST is a working identifier, not a brand — the document is publicly visible as a working draft, but the name is not intended for product, marketing, or external naming use.

> **Changelog**: what each version added, changed, or replaced is recorded in [CHANGELOG.md](CHANGELOG.md); earlier full text is recoverable from git history. Nothing here is rewritten silently.

---

## 0. Epistemic status

This note sorts LLM trust problems by **what can be enforced**, and splits in two. **Part I — Substrate fix** is the original UNTRUST argument: the narrow class of trust properties an architecture can enforce so that no within-distribution input can defeat them. **Part II — Non-substrate fix** covers the wider "trustworthy AI" problems — hallucination, alignment, robustness — where only mitigations or statistical guarantees exist. The bridge between them (§12) is the line that decides which is which.

A few things to keep in front of mind:

- The Part I designs are **sketches**, not architectures — first-pass structural ideas, not implementations or specifications.
- **No current approach is a complete substrate fix at LLM scale.** Several of the sketches have working partial implementations (ASIDE, StruQ, CaMeL) that improve robustness without meeting the §2 criterion fully. The early framing that "nobody has built any of this" was wrong; the accurate claim is narrower.
- Each sketch has at least one unsolved hard problem. None is complete.
- The synthesis (§10) is more speculative than the individual sketches — a hypothesis about where the answer lives, not a claim to have found it.
- Part II being in scope does **not** make it substrate-fixable. The note maps and classifies those problems honestly; it does not claim an architectural fix for any of them.
- This is a **starting point for thinking**, not a finished argument. Interrogate each piece, find what breaks, and repair or replace it.
- A standing caution: the thinking that produced this kept drifting toward comfortable mitigation framings before settling on the substrate one. If a section reads as though a mitigation were a substrate fix, that is the drift — distrust it.

---

# Part I — Substrate fix

## 1. The substrate problem

A transformer-based LLM is a function from a context to a probability distribution over the next token. The context is a single flat sequence of tokens with **no type system**. The string `"ignore previous instructions"` has identical representational status to `"the patient's MAP is 65"` — both are embeddings attended to by the same heads with the same weights.

The model learns *statistically* that certain patterns — system-prompt formatting, special tokens, instruction-following training — should bias output toward certain behaviours. But those biases are **learned correlations, not enforced invariants**. The substrate has none of the structural protections a traditional OS takes for granted:

| Property                   | Traditional OS              | LLM                              |
| -------------------------- | --------------------------- | -------------------------------- |
| Privilege separation       | Hardware (ring 0/3, MMU)    | None — learned bias only         |
| Address-space isolation    | Page tables enforce         | No analog                        |
| Capability tokens          | Unforgeable (kernel-issued) | Forgeable (any text can imitate) |
| Trusted/untrusted boundary | Syscall interface           | Conventions in prompt            |
| Bypass cost                | Find a kernel vuln (rare)   | Write persuasive prose (cheap)   |

**The failure mode**: any sufficiently persuasive pattern in any region of context can influence generation in any other region, because attention is unrestricted and trust is learned rather than enforced.

This is paradigm-deep, not transformer-specific — it holds for state-space models (Mamba), recurrent architectures (RWKV), diffusion LMs, and any paradigm that processes language end-to-end through flat token sequences with shared representations. The empirical foundation — that current LLMs systematically fail at instruction–data separation — is Zverev et al. [4]; the threat model and attack taxonomy are Greshake et al. [13].

---

## 2. What a substrate fix requires

A substrate fix must break at least one of the following at the **architectural level** — not as a training objective the model is taught to respect, but as a computation it cannot circumvent because the circumvention is **not representable**:

1. **Tokens have no enforced type.** → Make type a structural property of position, not a learned correlation.
2. **Attention flows freely between any tokens.** → Make some attention paths zero by construction, not by learned weights.
3. **Trust is learned, not enforced.** → Route trust decisions through a component the network cannot influence by its outputs.
4. **Generation can produce any action.** → Gate generation on unforgeable preconditions (cryptographic capabilities, verifier signoff).

The criterion that separates a fix from a mitigation: **can the protection be defeated by sufficiently clever input within the model's training distribution?** If yes, it is a mitigation. If no — because the protection is not a learnable behaviour the model can be persuaded out of — it is a substrate fix.

By this test, the following are **mitigations**, not fixes: training-based defences (RLHF, constitutional AI, refusal training, instruction hierarchies as trained behaviour [7]); prompt-level defences (better system prompts, spotlighting [12], datamarking); classifier-based guards (the classifier is itself attackable); orchestration-layer defences (capability scoping, structured I/O, audit logs — these reduce blast radius, not influence). ASIDE [1], StruQ [3], and instruction hierarchies [7] *as trained behaviours* improve robustness statistically; they do not enforce.

This is not a dismissal. Mitigations are useful, reduce probability and blast radius, and are necessary for current deployment. They are simply not substrate fixes — and conflating the two has been the field's default failure mode.

---

## 3. Prior art

The sketches are not novel directions. They are restated, structured versions of work already in progress:

- **Sketch 1 (typed attention masks / embedding separation)** is most closely instantiated by **ASIDE** [1], which applies an orthogonal rotation to data-token embeddings to separate instruction and data representations. **ISE** [2] uses role-specific offset vectors; **StruQ** [3] pairs a secure front-end with a fine-tuned model that follows instructions only in designated regions. The foundational measurement — that current LLMs *cannot* reliably separate instructions from data — is **Zverev et al.** [4].
- **Sketch 2 (channel asymmetry / bandwidth bottleneck)** has mechanism precedent in vision-language models (CLIP [24], Flamingo [25]) and an information-reduction echo in jailbreak defences (IBProtector [26], SecurityLingua [27]), but the latter are input-preprocessing mitigations, not an architectural channel split. This is the least instantiated sketch as a security primitive.
- **Sketch 3 (verifier as architectural primitive)** has no trust-invariant implementation at LLM scale. Production neural guards (Constitutional Classifiers [28][29]) are inference-time mitigations, and they are defeated by attacks like Boundary Point Jailbreaking [30] — exactly the weakness Sketch 3 names. The only decode-coupled verifier that meets the bar is grammar-constrained decoding [31][32] — sound, but for a *syntactic* property only.
- **Sketch 4 (capability tokens)** is closely instantiated by **CaMeL** [5] (Google DeepMind + ETH Zurich), a custom interpreter that tracks capability metadata on values in LLM agent systems (solves 77% of AgentDojo tasks with provable security, vs. 84% undefended). CaMeL builds on Willison's **Dual LLM pattern** [6]; OpenAI's **Instruction Hierarchy** [7] is a training-time approximation.
- **The trusted-base synthesis (§10)** is capability-based OS security applied to LLMs — a 60-year lineage: Dennis & Van Horn [8] (capabilities, C-lists), KeyKOS [9] (commercial, since 1983), EROS [10] (persistent capabilities), seL4 [11] (machine-checked verification of an 8,700-LOC microkernel).
- The **threat model** is well established: Greshake et al. [13] (indirect prompt injection), OWASP LLM Top 10 [14] (LLM01 = prompt injection), MITRE ATLAS [15]. Microsoft's Spotlighting [12] is a production mitigation.

The honest repositioning: UNTRUST's contribution is **clarity, not invention**. The architectures it sketches are being built; the value is a shared frame for telling substrate fixes apart from mitigations. No current approach meets the §2 criterion fully — all are partial fixes that improve probability and blast radius without eliminating the failure class.

---

## 4. Sketch 1 — Typed tokens with enforced attention masks

**Mechanism.** Tokens carry a **type tag** as a structural property of position, not as a token in the sequence:

```
TYPES = {
    SYSTEM,         // deployer-set, highest privilege
    USER,           // principal-set, high privilege
    MODEL_INTERNAL, // model's own generated reasoning
    TOOL_CALL,      // model's emitted actions
    TOOL_OUTPUT,    // returned data from tools (untrusted)
    DOCUMENT,       // retrieved/fetched content (untrusted)
}
```

Attention weights between certain type pairs are **masked to zero by construction**:

```
mask[i, j] = 0  if type(j) ∈ UNTRUSTED and target(i) ∈ {TOOL_CALL, SYSTEM}
mask[i, j] = 0  if type(j) = DOCUMENT and target(i) = MODEL_INTERNAL[decision_layer]
```

The mask is **architectural, not learned** — the matrix multiplication has structural zeros where those connections would be. No amount of persuasive prose in a tool output can change that, because the zeros are absences of computation, not weights.

**Privilege ordering** runs `SYSTEM > USER > MODEL_INTERNAL > TOOL_OUTPUT ≈ DOCUMENT`. Higher-privilege tokens may attend down; lower-privilege information reaches higher computation only through bottlenecked pathways (Sketch 2).

**Cost.** Loss of expressiveness — a tool output that legitimately *should* update a decision is bottlenecked the same as an adversarial one. Inter-channel bandwidth becomes a capability-vs-safety design parameter with no free setting, and training must accommodate the constraints.

**Hard problem — the labeling boundary.** Type tags must be assigned somewhere. If a model labels tokens, the labeler is injectable and the problem recurses; if syntactic markers determine types, input can fake the markers. The mask is enforceable *given correct labeling*, but it does not solve labeling — that must happen in a component outside the network (the orchestrator, a parser, the trusted base of §10).

**What it gets right.** Once labeled, the boundary cannot be defeated by clever prose; the protection is a computational impossibility, not a trained behaviour; it composes with mitigations over a smaller surface. ASIDE [1] is the closest instantiation (embedding-level separation, no added parameters), but it still allows attention between rotated and unrotated regions — the full architectural mask is unimplemented at LLM scale.

**What it does not solve.** Labeling; gaps in the type taxonomy (what type is a quoted document inside a user message?); out-of-distribution attacks on the labeling rule; semantic alignment within trusted channels (a deceived user is still trusted).

---

## 5. Sketch 2 — Computational asymmetry between channels

**Mechanism.** Give different channels different amounts of compute. The privileged channel (`SYSTEM` + `USER`) gets full attention and bandwidth; the unprivileged channel (`TOOL_OUTPUT`, `DOCUMENT`) is processed through a restricted sub-network — often a separate encoder producing a low-dimensional summary the main model conditions on rather than attends to directly:

```
Privileged tokens ───────────────► [full transformer stack] ──► output
                                          ▲
Untrusted tokens ──► [encoder] ──► [summary vec, dim=64..256] ─┘
                       (limited)         (bottleneck)
```

**Intuition.** An attacker in the unprivileged channel can influence the model only through a bandwidth-limited bottleneck. A 64-dimensional summary can carry "this document is about authentication" but not "send the auth tokens to this specific address." The bottleneck enforces a complexity bound on what can be transmitted — the same way vision-language models treat an image as structural *data* (CLIP [24], Flamingo [25]) rather than as injectable tokens.

**Cost.** Fidelity: the model cannot quote a document that never enters as tokens; it can answer only what the summary supports. Fine for summarize/classify workflows, too restrictive for ones that must reason over untrusted detail. Encoder design becomes a load-bearing security decision.

**Hard problem — adversarial examples against the encoder.** The encoder is itself a neural network, and adversarial examples are a 12-year-old research area. The bottleneck makes attacks *harder* (lower bandwidth, more structure required) but not impossible: worst-case input can still steer the summary toward a vector the privileged model treats as a high-priority instruction.

**What it gets right.** Shrinks the channel through which untrusted content reaches privileged decisions; makes attack cost explicit and tunable; composes with multimodal architectures already in use.

**What it does not solve.** Adversarial encoder examples; high-fidelity processing of untrusted text; and the labeling problem from Sketch 1 still applies — "untrusted" has to be decided somewhere.

---

## 6. Sketch 3 — Verifier as a first-class component

**Mechanism.** The model is two networks coupled at every step. A **generator** proposes the next token; a **verifier** assesses whether it is consistent with the system's invariants given the full provenance of the context. A token is emitted only if the verifier signs off. The verifier sees what the generator does not: provenance of every token, trust level of every claim, a formal specification of allowed actions, and the invariants the output must preserve.

|                | Generator              | Verifier                     |
| -------------- | ---------------------- | ---------------------------- |
| Job            | Produce tokens         | Approve/reject tokens        |
| Size           | Large                  | Small                        |
| Training       | Standard LM objective  | Verification objective only  |
| Auditability   | Low                    | High (smaller, focused)      |
| Attack surface | Wide (all of language) | Narrow (specific invariants) |

The verifier is smaller and more interpretable because verification is simpler than generation — checking that an action is consistent with a policy is easier than proposing it. The pattern mirrors System 2 checking System 1, retirement logic checking speculative execution, and cheap signature verification against expensive signing.

**Cost.** Latency (every token or action requires a verifier pass); coverage (only invariants expressible in the verifier's model can be checked — "this output is appropriate" is not cleanly formalizable); composition (generator and verifier must share a representation of context, provenance, and invariants).

**Hard problem — the verifier must be uncircumventable.** A *neural* verifier is attackable by the same means as the generator; a *symbolic* verifier can only check what its formal language expresses. The sketch displaces "make the generator trustworthy" to "make the verifier reliable," which is progress only where verification is genuinely easier: no PII leakage, no tool calls without authorization, no contradiction with high-priority instructions, no actions outside a declared capability set. For semantic appropriateness, intent-alignment, or open-ended factual accuracy, verification is as hard as generation and the asymmetry collapses.

**What it gets right.** Decouples what the model wants to do from what the system allows; lets formal methods apply where they can without forcing them where they can't; gives an auditable choke point.

**What it does not solve.** Properties not expressible to the verifier; attacks on the verifier (if neural) or on the spec (if symbolic); and the provenance graph still has to come from somewhere — the labeling problem recurs.

---

## 7. Sketch 4 — Capability tokens as architectural primitives

**Mechanism.** The model emits a tool call only if it holds an **unforgeable capability token** for that tool — a cryptographic object issued by the orchestrator, scoped to a specific operation, with a bounded lifetime. This is the LLM analog of capability-based OS security (KeyKOS [9], seL4 [11], EROS [10]): authority flows through possession, not identity claims, a lineage running back to Dennis & Van Horn [8].

**CaMeL** [5] is the closest instantiation — a custom interpreter tracking capability metadata on values, enforcing policy in a protective system layer without modifying the LLM. The difference from this sketch is *where the gate sits*: inside the generation pathway (sketched here) vs. outside in an interpreter (CaMeL). Both move the trust boundary out of the model.

Today, tool calls are just text the orchestrator parses and may refuse *after* generation. The proposal gates generation **inside the model** on the capability:

```
generate_tool_call(tool_name, args) requires:
    capability ∈ context
    capability.tool == tool_name
    capability.signature verifies under orchestrator's public key
    capability.not_expired()
    args ⊆ capability.allowed_scope
```

**Cost.** A round-trip to mint each capability; more verbose, higher-latency workflows; the orchestrator becomes an explicit critical component with formal interfaces; models must be trained to request capabilities before acting.

**Hard problem — coercion through language.** The check is real, but *getting* a capability is still mediated by natural-language interaction with the orchestrator. If adversarial input persuades the model to request a capability for an attacker-chosen operation, the orchestrator mints it and the protection is bypassed at the request layer. The sketch defeats **forged** capabilities, not **legitimately-issued capabilities for the wrong operation**. Partial defences — human approval above a risk threshold, an orchestrator trained to flag suspicious requests (which recurses), bounding mintable capabilities so worst-case blast radius is small — reduce probability without closing it.

**What it gets right.** Imports 60 years of capability security where unforgeability matters most (action authorization); composes with existing orchestration.

**What it does not solve.** The coercion path; adversarial reasoning over capability contents (they are still tokens in context); the orchestrator's issuing policy, which remains a soft boundary.

---

## 8. Pattern 5 — Parameterisation-class restriction

A fifth enforcement pattern, distinct from the four sketches: **restrict the parameter manifold so the model cannot represent operators outside a chosen class.**

The canonical example is the Cayley transform $Q = (I - K/2)(I + K/2)^{-1}$ with skew-symmetric $K$, which produces an orthogonal $Q$ for *any* $K$. Orthogonality is enforced by the geometry of the parameterisation, not by training — no gradient step, adversarial input, or fine-tune can produce a non-orthogonal $Q$ this way, because non-orthogonal matrices are not in the image of the map. Where the sketches mask paths, narrow bandwidth, check outputs, or gate actions, Pattern 5 makes whole operator classes **unreachable by construction**.

The lineage is established (Lezcano-Casado & Martínez-Rubio 2019; unitary RNNs, Arjovsky et al. 2016, Wisdom et al. 2016); what is new is the connection to enforcement-by-construction in transformer adapters. The most recent LLM-scale demonstration is **CUA** [16], inserting Cayley-parameterised unitaries into frozen projection layers of Llama 3.1 8B and SmolLM2.

What this does **not** mean for trust:

- **Orthogonality is not a trust property.** Norm-preservation says nothing about whether attention can flow from untrusted to privileged tokens.
- **The one cited demonstration is unreplicated**, and [16] carries a structural conflict of interest (its degraded baseline is the authors' own commercial product); the headline single-layer result is a 1.43% perplexity change.
- **No trust-relevant operator class has been shown to admit such a parameterisation.** For orthogonality the construction exists; for "attention that respects token provenance," nobody has one, and it is not obvious the property is even a closed-form manifold restriction.

If such a parameterisation existed, it would give Sketch 1 a substrate-fix implementation no training could defeat. Whether *some* trust properties (linear, geometric, structural) admit this while others (semantic, intent-related) do not is the open question Pattern 5 leaves on the table.

---

## 9. The pattern across the sketches

None of the sketches eliminates the trust problem. Each **moves** it:

| Sketch                   | Makes enforceable                 | Still relies on (soft)          |
| ------------------------ | --------------------------------- | ------------------------------- |
| 1. Typed attention masks | Cross-channel attention flow      | Labeling of token types         |
| 2. Channel asymmetry     | Bandwidth into privileged channel | Encoder robustness              |
| 3. Verifier              | Action authorization              | Verifier itself; property specs |
| 4. Capability tokens     | Tool-call authority               | Capability-minting decisions    |

The shared insight: **trust requires a trusted base, and the trusted base cannot be the model.** A neural network cannot enforce trust boundaries on itself; the enforcement must come from a component its outputs cannot influence — necessarily smaller, simpler, and outside the network.

This is what operating systems learned in the 1960s [8]: you do not make user code safe by writing it more carefully, you add a kernel — a small, analyzable component the user code lacks the privilege to subvert. The kernel is not where the interesting computation happens; it is where the trust lives. Capability-based OS work (KeyKOS [9], EROS [10], seL4 [11]) showed the principle scales and admits formal verification.

---

## 10. The synthesis: system, not model

If the trusted base lives outside the network, the new substrate is not a different kind of *model* — it is a different kind of *system*, with the model as one untrusted component on top of a base it cannot subvert.

```
┌─────────────────────────────────────────────────────────┐
│   Large Neural Component (LLM)                          │
│   - Fluent, capable, untrusted                          │
│   - Generates proposals, summaries, plans               │
│   - NO direct access to actions, identity,              │
│     credentials, tool authority, or external state      │
└──────────────────────────┬──────────────────────────────┘
                           │ (structured interface)
                           ▼
┌─────────────────────────────────────────────────────────┐
│   Trusted Base                                          │
│   - Small, auditable, formally analyzable               │
│   - Owns: tool access, action authorization, identity,  │
│     provenance, capability minting, audit, irreversibility gating │
│   - Treats the LLM as an untrusted userspace process    │
└──────────────────────────┬──────────────────────────────┘
                           ▼
                    External world (tools, APIs, humans, sensors)
```

The neural component does what neural components are good at — fluent generation, pattern recognition, summarization, open-ended planning. The trusted base does what trusted bases are good at — enforcing invariants, mediating access, holding keys, gating irreversible actions, maintaining audit trails. **The boundary between them is the new substrate.**

This is not a new neural architecture; it is a commitment about where trust lives. The model stays roughly what current LLMs are (optionally hardened by Sketches 1–4); the trusted base is genuinely new work — formally verified, small enough to audit, capability-based from the ground up. It needs: **smallness** (thousands of LOC, not millions), **specification-completeness** (anything outside the policy is unrepresentable), **cryptographic identity and capability**, **provenance tracking**, **irreversibility gating** (unbounded actions require human approval through the base), and **immutable audit logging**. The model sees only what the base permits, emits proposals that become actions only through the base, and cannot reach the world except through it.

---

## 11. Cross-disciplinary inputs

Building this substrate is integration work across five fields that do not talk to each other enough:

1. **ML architecture** — constrained channels, typed attention, encoder bottlenecks, verifier coupling.
2. **Programming-language theory** — type systems for trust, effect systems for capabilities, linear types for resource bounds.
3. **Operating systems** — capability-based security (seL4, KeyKOS), microkernels, formally verified bases.
4. **Cryptography** — unforgeable tokens, attested computation, signed actions, enclaves.
5. **Distributed systems** — provenance, audit trails, Byzantine-tolerant orchestration, irrevocable logs.

The AI field's attention is on the first; the substrate fix lives mostly in the other four. The likely path is importing those traditions into AI — bolting an LLM onto a small formally-verified capability base as an untrusted userspace process, designing the boundary protocol carefully, and showing the result is meaningfully harder to compromise than current deployments.

---

# Bridge — The enforcement boundary

## 12. Why some properties can be enforced and others cannot

The §2 criterion carries an unstated precondition: **a property is architecturally enforceable only if it is a function of the computation's internal structure alone — not of any relation to something outside the computation.**

The trust-boundary property satisfies this. "No information flows from untrusted-channel positions to privileged-operation positions" is a statement about attention paths; both relata are positions inside the forward pass; information flow is structural. That is *why* it can be masked to zero. When a property is instead defined by a relation to something outside the computation — the world's ground truth, the model's latent state, the principal's latent intent, the unbounded complement of the training distribution — there is no internal structure whose presence corresponds to the property. The §2 bar then cannot be *met* because it cannot be *posed*. This is stronger than "unsolved": the question is not well-formed.

Sorting by this precondition gives three classes, not two:

| Class                              | Definition                                                  | Guarantee                                                  | Members                                                                          |
| ---------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **A. Structurally enforceable**    | Intrinsic to a closed formal system the architecture hosts | Unrepresentability — the §2 fix bar                       | Trust-boundary information flow; deductive-reasoning validity; verbatim-copy groundedness |
| **B-avg. Statistically guaranteeable (average-case)** | Reference-dependent; admits a *marginal* bound under an input-distribution assumption | Coverage contingent on a distributional assumption — **dies under shift** | Calibration (conformal coverage under exchangeability [23]); bounded, known OOD |
| **B-wc. Statistically guaranteeable (worst-case)** | Reference-dependent; admits a bound holding over *all* inputs and adversary priors | A bound on adversary advantage at a **chosen parameter** — **survives shift** | Differential privacy (ε,δ) [36]; randomized-smoothing robustness certificates [37] |
| **C. Mitigation-only**             | Reference-dependent, no closed formal core, no bound       | None; probability- and blast-radius reduction only        | Factual truth [17][18]; honesty/non-deception [20][21]; intent-alignment; open-world OOD |

The line that matters is **A versus {B, C}** — only Class A meets the §2 bar. **Class B is the one most likely to be mistaken for a fix and is not one** — and it is itself two species with different seams. **B-avg** (conformal, bounded OOD): a conformal guarantee is real, distribution-free, and finite-sample, but it is marginal rather than conditional and rests on exchangeability — precisely the assumption distribution shift breaks and that nothing in the architecture enforces. **B-wc** (differential privacy [36], randomized-smoothing certificates [37]): a bound that instead holds worst-case over inputs and *survives* shift, failing not at a distributional assumption but at a **chosen parameter** — an ε or a confidence level that can be technically valid yet vacuous. Neither species crosses into Class A: a bounded violation is still a representable computation, not an unrepresentable one. Factual truth sits firmly in Class C: its elimination is foreclosed by computability-theoretic [18] and statistical [17] inevitability results (with [19] arguing the bound is too loose to predict practical error rates — so "unenforceable," not "uniformly wrong").

**Deductive validity is the one place Sketch 3 reaches Class A.** "The chain is truth-preserving" — unlike "the chain looks plausible" — is, in a formalisable domain, intrinsic to a formal system and checkable by a **sound, deterministic** verifier (a proof checker, an SMT solver, program execution). A sound checker cannot be argued out of soundness by prose. But the enforcement is **conditional**: it holds only *relative to a formalisation*, and the natural-language → formal translation (autoformalisation) is a semantic act the checker cannot verify — a faithful checker over a wrong formalisation yields a validly-derived falsehood. It also covers only the formalisable fraction; for defeasible or natural-language argument, process-reward models [22] and neural verifiers revert to Class C. (That valid reasoning is not free is well-evidenced: stated chains are often unfaithful to the computation behind the answer — hint use was verbalised ~25%/~39% of the time across two frontier models [20] — and models rationalise bias-induced answers without disclosing the bias [21].)

This exposes a pattern shared by **every** Class A enforcement here: each bottoms out in an un-enforceable judgment at the edge — Sketch 1's **labeling**, the checker's **autoformalisation**, and, for **groundedness** (output that verbatim-copies a source — the copy itself is structurally checkable), the **source-trust** that the source must be vouched for outside the copy mechanism. Architectural enforcement is never total; it terminates in a semantic act that must be *trusted*, not enforced. That is the same seam as the trusted base in §9 — the location where "enforced" hands off to "assumed."

So the boundary **sorts** the space: Class A is Part I (substrate fixes); Class B/C is Part II (non-substrate). Admitting Part II is not conflating it with Part I — the lanes stay separate, and the §2 criterion is unchanged, now doing classification across the whole space rather than guarding one corner of it.

---

# Part II — Non-substrate fix

## 13. The wider trustworthy-AI clusters

Beyond substrate trust lie the problems usually bundled under "trustworthy AI." This note brings them in only to **map and classify** them — not to claim a fix. Three of the four clusters are mitigation- or statistical-guarantee-class; saying so plainly, and keeping the fix-spaces in separate lanes, is the whole point. A single solution claiming to cover all four would be exactly the conflation §0 warns against.

| Problem cluster                 | Failure mode                                | Fix space                                                                                          | Success criterion                |
| ------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------- |
| Substrate trust (UNTRUST)       | Adversarial influence through input channels | Architectural enforcement: the four sketches (§4–§7) + Pattern 5 (§8)                             | Holds against worst-case input   |
| Hallucination / accuracy        | Wrong in the absence of any adversary       | Training-data quality, retrieval augmentation [33], verification loops, calibration               | Statistical reduction of error   |
| Alignment / honesty             | Behaviour diverges from principal intent    | RLHF [34], constitutional AI [35], character training, interpretability                            | Behaviour matches stated intent  |
| Robustness / OOD                | Degrades outside the training distribution  | Wider training, ensembles, uncertainty quantification, conformal prediction [23], randomized-smoothing certificates [37] | Graceful degradation, not confident error |

**The clusters are independent.** A model can be perfectly trustworthy in the UNTRUST sense and still hallucinate constantly; it can be perfectly accurate on clean input and catastrophically injectable. The clusters have different failure modes (adversary present vs. absent), different fix spaces (architecture vs. data vs. preference training vs. coverage), and different success criteria (worst-case guarantee vs. error rate vs. intent-match vs. graceful degradation). A mitigation for one does not touch the others.

Mapped onto the enforceability classes of §12: **substrate trust → Class A**; **hallucination → Class C**; **alignment/honesty → Class C**; **robustness/OOD → B-avg** (bounded shift) / **B-wc** (worst-case, survives shift) / **Class C** (open world). Only substrate trust is Class A — one cell of the table. The per-cluster reading (with why each lands where it does):

- **Hallucination / accuracy (Class C).** The model is simply wrong on a clean input. The fix space is statistical — retrieval grounding [33], verification loops, calibration, better data — and the success criterion is a lower error rate, never zero. There is no substrate fix, and not for lack of trying: hallucination is inevitable by statistical [17] and computability-theoretic [18] argument, and "is this true?" depends on a ground truth outside the computation. What UNTRUST-style thinking *does* buy is narrow: a sound symbolic checker (§12) enforces deductive *validity* over the formalisable fraction — a guarantee about a chain, not about a claim's truth.
- **Alignment / honesty (Class C).** Value divergence with no adversary required. The fix space is preference- and feedback-based — RLHF [34], constitutional AI [35], interpretability — aiming at intent-match. There is no substrate fix, because intent is a relation to a latent principal state outside the computation; there is no internal structure that corresponds to "aligned." Worse, the model's own stated reasoning is an unreliable window [20][21], so honesty cannot be read off the output.
- **Robustness / OOD (B-avg / B-wc / C).** Degradation outside the training distribution, and the one Part II cluster where *both* B-species appear. Coverage methods — conformal prediction [23] — give a real distribution-free guarantee *where the shift is known and bounded* (**B-avg**), but rest on exchangeability, which shift breaks. Per-input certificates — randomized smoothing [37] — instead bound an adversarial neighbourhood worst-case and survive shift (**B-wc**), at the cost of a chosen sampling budget and coverage only out to the certified radius. In the open world the complement of the training distribution is unbounded, no guarantee is available, and only mitigations remain (Class C). The honest target *beyond the certified radius* is graceful degradation, not a global worst-case bound.

What stays true across the expansion: the §2 criterion still discriminates (now as a cross-cluster test); no Class C cluster receives a substrate fix; and the lanes stay separate. A reader must not leave this part thinking hallucination or intent-alignment is architecturally enforceable — the mapping above says the opposite.

---

# Back matter

## 14. What this note does not address

Listed so nothing gets smuggled in as "solved":

- **Semantic alignment** — a model that obeys the trusted base's invariants can still produce bad output within the allowed space.
- **Deceived principals** — if the user is wrong, manipulated, or coerced, the system faithfully executes their intent. The substrate protects against attacks *through the model*, not *through the user*.
- **Trusted-base correctness** — the base is software with bugs; formal verification reduces but does not eliminate them. "Audited" is not "perfect."
- **Side channels** — timing, error messages, resource use; needs its own defences.
- **Supply chain** — weights, source, build toolchain are all compromise points outside this architecture.
- **Physical access** — the base is assumed to run on hardware adversaries cannot modify.
- **Multi-agent dynamics** — trust composition across interacting agents is its own problem.
- **Computational cost** — the architecture is plausibly slower and more expensive; whether that is acceptable is deployment-dependent.

The substrate fix addresses influence-on-the-model. It does not address everything else that can go wrong.

---

## 15. Open questions

1. **How is the labeling boundary made trustworthy?** Sketches 1 and 2 depend on correct type assignment — by what component, protected how?
2. **Can Sketch 3's verifier be made non-neural for the properties that matter?** If not, what makes it harder to attack than the generator?
3. **What is the minimal trusted base for an LLM agent?** seL4 is ~9.3K LOC [11]; what is the LLM-mediated equivalent?
4. **What is the base↔model protocol**, and what are its security properties?
5. **What does training look like for a model designed to live above a trusted base?** Pretraining does not include "you cannot reach the world directly."
6. **What is the simplest end-to-end demonstration** — likely a narrow task where documented injection attacks are provably impossible against the proposed system? (An implementation probe of exactly this is under way — **KEEP**, [github.com/SHA888/KEEP](https://github.com/SHA888/KEEP) — a capability-token trusted base over a narrow AgentDojo scenario. Spin-out: separate repo, scope, and license.)
7. **How does it compose with existing mitigations?** It should layer with RLHF, capability scoping, and human-in-the-loop, not replace them.
8. **What is the upgrade path?** A change requiring a full rebuild will not be adopted; adding a trusted base alongside existing LLMs is more plausible.

---

## 16. Notes on use

This is not a roadmap; it is a starting point. Read once for the shape; re-read each sketch looking for what breaks; test whether the trusted-base synthesis (§10) actually holds; treat the cross-disciplinary inputs (§11) as homework and the open questions (§15) as the real, years-long agenda.

It is deliberately not a pitch. It does not argue anyone should do this — it argues that *if* someone does substrate-level work, this is roughly what it looks like and what its honest limits are. The substrate problem stays unsolved because the field's incentive gradient does not point at it; whether the work is worth doing is a question the person doing it must answer repeatedly, over years.

One use to refuse: do not mistake this frame for something that can *enforce* itself. The classes are a lens — they classify, clarify, and discriminate, and that is all they bind. Reify the lens into a voluntary discipline — a conformance standard, a labeling contract, a checklist — and the discipline inherits no enforcement from the classification it rests on: any actor can decline it, so by the §2 criterion it is defeasible within the distribution, which makes it **Class C — a mitigation, not a fix**. This is the §9 lesson turned on the frame itself: enforcement requires a trusted base outside the thing being governed, and a document is not one. A standard built on UNTRUST is useful exactly as far as something external — adoption, contract, audit, law — chooses to enforce it, and not one step further. Applying these classes is sound; claiming that the act of applying them is itself a fix is the drift §0 warns against. (The `ENFORCEABILITY-DISCIPLINE.md` companion is one such reification, and labels itself Class C accordingly.)

Either way: the substrate problem is real, the mitigations are not solutions, and the path worth finding is the one that compounds.

---

## References

**[1] ASIDE — Architecturally Separated Instruction-Data Embeddings.** Zverev, E., et al. (2025). arXiv:2503.10566 (ICLR 2026). Orthogonal rotation of data-token embeddings; closest instantiation of Sketch 1.
**[2] ISE — Instructional Segment Embedding.** Wu, T., et al. (2024). ICLR 2025. Role-specific offset vectors.
**[3] StruQ — Defending Against Prompt Injection with Structured Queries.** Chen, S., Piet, J., Sitawarin, C., & Wagner, D. (2024). USENIX Security 2025. arXiv:2402.06363. Secure front-end + fine-tuned model.
**[4] Can LLMs Separate Instructions From Data?** Zverev, E., et al. (2025). ICLR 2025. arXiv:2403.06833. Foundational measurement: no current LLM separates reliably.
**[5] CaMeL — Defeating Prompt Injections by Design.** Debenedetti, E., et al. (2025). arXiv:2503.18813 (Google DeepMind + ETH Zurich). Capability-tracking interpreter; solves 77% of AgentDojo tasks with provable security (vs. 84% undefended); closest instantiation of Sketch 4.
**[6] The Dual LLM Pattern.** Willison, S. (April 2023). https://simonwillison.net/2023/Apr/25/dual-llm-pattern/. Privileged + quarantined LLM architecture.
**[7] The Instruction Hierarchy.** Wallace, E., et al. (2024). arXiv:2404.13208 (OpenAI). Training-time role priority; a mitigation, not a fix.
**[8] Programming Semantics for Multiprogrammed Computations.** Dennis, J. B. & Van Horn, E. C. (1966). CACM 9(3):143–155. The original capability paper.
**[9] The KeyKOS Nanokernel Architecture.** Bomberger, A. C., et al. (1992). USENIX. Capability-based OS in commercial production since 1983.
**[10] EROS — A Fast Capability System.** Shapiro, J. S., Smith, J. M., & Farber, D. J. (1999). SOSP '99. Persistent capabilities.
**[11] seL4 — Formal Verification of an OS Kernel.** Klein, G., et al. (2009). SOSP '09. 8,700 LOC C + 600 assembler; ~200K-line Isabelle/HOL proof.
**[12] Spotlighting.** Hines, K., et al. (2024). arXiv:2403.14720 (Microsoft). Delimiting/datamarking/encoding; production mitigation.
**[13] Not What You've Signed Up For (Indirect Prompt Injection).** Greshake, K., et al. (2023). AISec '23. arXiv:2302.12173. The canonical threat paper.
**[14] OWASP Top 10 for LLM Applications 2025.** OWASP Gen AI Security Project. LLM01 = Prompt Injection.
**[15] MITRE ATLAS.** https://atlas.mitre.org/. AML.T0051.000 (direct), AML.T0051.001 (indirect), AML.T0054 (jailbreak).
**[16] CUA — Cayley Unitary Adapters.** Aizpurua, B., et al. (2026). arXiv:2605.05914 (Multiverse Computing et al.). LLM-scale parameterisation-class adapters. *Conflict of interest*: the degraded baseline is the authors' own commercial product; the single-layer Llama 3.1 8B result (1.43% PPL) is unreplicated.
**[17] Why Language Models Hallucinate.** Kalai, A. T., Nachum, O., Vempala, S. S., & Zhang, E. (2025). arXiv:2509.04664. Hallucination as binary-classification error; evals reward guessing over abstention.
**[18] Hallucination is Inevitable.** Xu, Z., Jain, S., & Kankanhalli, M. (2024). arXiv:2401.11817. Computability-theoretic inevitability.
**[19] Hallucinations Are Inevitable but Can Be Made Statistically Negligible.** Suzuki, A., et al. (2025). arXiv:2502.12187. The innate bound is too loose to predict practical error rates.
**[20] Reasoning Models Don't Always Say What They Think.** Chen, Y., et al. (2025). arXiv:2505.05410 (Anthropic). Hint use verbalised ~25%/~39% across two frontier models.
**[21] Language Models Don't Always Say What They Think.** Turpin, M., Michael, J., Perez, E., & Bowman, S. R. (2023). NeurIPS 2023. arXiv:2305.04388. CoT rationalises biased answers.
**[22] Let's Verify Step by Step.** Lightman, H., et al. (2023). arXiv:2305.20050 (OpenAI). A learned process-reward verifier — Class C, attackable.
**[23] Conformal prediction.** Vovk, Gammerman & Shafer (2005); Angelopoulos & Bates (2023). Marginal, finite-sample, distribution-free coverage under exchangeability — basis for B-avg.
**[24] CLIP.** Radford, A., et al. (2021). ICML 2021. arXiv:2103.00020. VLM encoder-bottleneck precedent (mechanism only).
**[25] Flamingo.** Alayrac, J.-B., et al. (2022). NeurIPS 2022. arXiv:2204.14198. Perceiver Resampler → fixed small latent set; concrete bottleneck mechanism.
**[26] IBProtector — Protecting Your LLMs with Information Bottleneck.** Liu, Z., Wang, Z., Xu, L., et al. (2024). NeurIPS 2024. arXiv:2404.13968. Information-bottleneck jailbreak defence; input-preprocessing mitigation, not channel asymmetry.
**[27] SecurityLingua — Security-Aware Prompt Compression.** Li, Y., Ahn, S., Jiang, H., et al. (2025). arXiv:2506.12707. Compression-based jailbreak defence; mitigation class.
**[28] Constitutional Classifiers.** Sharma, M., et al. (2025). arXiv:2501.18837 (Anthropic). Neural I/O guard; jailbreak success 86% → 4.4%; a mitigation.
**[29] Constitutional Classifiers++.** Cunningham, H., Wei, J., Wang, Z., et al. (2026). arXiv:2601.04603 (Anthropic). Successor that cuts the classifier's added compute from ~23.7% to ~1%; still a neural I/O guard.
**[30] Boundary Point Jailbreaking of Black-Box LLMs.** Davies, X., Giglemiani, G., Lau, E., Winsor, E., Irving, G., & Gal, Y. (2026). arXiv:2602.15001 (UK AISI + Oxford OATML). Defeats Constitutional Classifiers and a GPT-5 input classifier — corroborates Sketch 3's hard problem.
**[31] Grammar-Constrained Decoding.** Geng, S., et al. (2023). EMNLP 2023. arXiv:2305.13971. Masks grammar-violating tokens — genuine enforcement, but of a *syntactic* property only.
**[32] Grammar-Aligned Decoding.** Park, K., et al. (2024). arXiv:2405.21047. Greedy grammar masking distorts the output distribution — the gate is sound, not free.
**[33] Retrieval-Augmented Generation.** Lewis, P., et al. (2020). NeurIPS 2020. arXiv:2005.11401. Grounding in retrieved documents; hallucination mitigation.
**[34] InstructGPT / RLHF.** Ouyang, L., et al. (2022). arXiv:2203.02155. (Foundational: Christiano et al. 2017, arXiv:1706.03741.) Canonical alignment fix-space entry; training-time mitigation.
**[35] Constitutional AI.** Bai, Y., et al. (2022). arXiv:2212.08073 (Anthropic). RLAIF against a written constitution; distinct from Constitutional *Classifiers* [28][29]; mitigation class.
**[36] Differential Privacy.** Dwork, C., McSherry, F., Nissim, K., & Smith, A. (2006). TCC 2006. Calibrating noise to sensitivity; a worst-case bound over adjacent datasets (B-wc). *Implementation seam*: floating-point breaks the guarantee (Mironov, CCS 2012).
**[37] Certified Adversarial Robustness via Randomized Smoothing.** Cohen, J., Rosenfeld, E., & Kolter, J. Z. (2019). ICML 2019. Per-input certified radius via a high-probability bound — a B-wc robustness certificate.

*Foundational work on orthogonal/unitary parameterisations (Pattern 5):* Lezcano-Casado & Martínez-Rubio (ICML 2019); Arjovsky et al. (ICML 2016); Wisdom et al. (NeurIPS 2016).

*Additional context (not cited inline):* Anthropic prompt-injection defences (browser-agent attack success reduced to ~1% on Claude Opus 4.5); AI Vulnerability Database (avidml.org); IBM X-Force Think (April 2026) on agentic-AI security risk.

---

## Sources & confidence

The point of this section is the same as the document's: keep what is *verified* apart from what is *asserted*, so neither is mistaken for the other.

- **Re-checked against arXiv (2026-05-28).** The recent and higher-risk references were confirmed to exist with matching titles and authors — including the 2026-dated CUA [16], Constitutional Classifiers++ [29], and Boundary Point Jailbreaking [30] (which does defeat both Constitutional Classifiers and a GPT-5 input classifier), plus IBProtector [26], SecurityLingua [27], the hallucination results [17][19], and the 86% → 4.4% Constitutional Classifiers figure [28]. This pass also **corrected a factual error**: CaMeL [5] solves 77% of AgentDojo tasks with provable security (vs. 84% undefended) — earlier versions misstated it as "~67% mitigation."
- **Recognised from the literature, not individually re-pulled this pass.** The remaining well-known references match published work but were not each re-fetched; a few in-paper figures (e.g. [16]'s 1.43% PPL) are reported from the papers rather than line-checked. **[36] specifically** is a TCC 2006 conference proceedings paper with no arXiv ID; it predates arXiv-first norms and is not verifiable by URL — its claims are accepted on canonical standing in the cryptography literature, not via a live arXiv check.
- **Author's framing, not from any single source.** The four-sketch taxonomy, the trusted-base synthesis (§10), the five-field cross-disciplinary claim (§11), the three-class enforceability cut (§12), and the four-cluster map (§13) are organizing frames — defensible but not established; whether they are the right cut of the space is open.
- **The B-avg / B-wc split (§12, §13) is a refinement, not an established cut.** It subdivides Class B to separate average-case bounds (conformal — dies under shift) from worst-case bounds (DP, randomized smoothing — survive shift, fail at a chosen parameter). The A-vs-{B,C} line, the §2 criterion, and scope are unchanged. DP/smoothing properties are recalled from [36][37], not re-derived here.
- **The bottom line.** No current approach meets the §2 criterion fully; every existing defence is partial; and only Class A admits an architectural fix at all.

---

_End of document._
