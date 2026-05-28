# Substrate Architecture for LLM Trust Boundaries

**Codename: UNTRUST**
**Four sketches and a synthesis. Working draft. Not designs.**
**Version: 0.3.1** (2026-05-28)

> The codename names the load-bearing commitment: the neural component is treated as structurally untrusted by design. UNTRUST is a working identifier, not a brand — the document is publicly visible as a working draft, but the name is not intended for product, marketing, or external naming use.

> **v0.2.0 changelog**: Added §0.5 (prior art and precedent), inline citations throughout, Appendix A (references), and Appendix B (verification log). Body sketches §1–§12 preserved verbatim from v0.1.0. The verification pass identified significant prior art — particularly CaMeL, ASIDE, StruQ — that the original document treated as more speculative than warranted. The honest repositioning is in §0.5.

> **v0.3.0 changelog**: Added §0.6 — a fifth enforcement pattern (parameterisation-class restriction) identified from adjacent work on constrained-operator adapters. The pattern is taxonomically distinct from the four sketches and was missed in v0.1.0/v0.2.0. Added reference [16] and Appendix B.3 entry. Body sketches §1–§12 and §0.5 preserved verbatim. No claim that this pattern addresses trust — see caveats in §0.6.

> **v0.3.1 changelog**: Distribution status update — the working draft is now publicly visible. Body sketches §1–§12, §0.5, and §0.6 preserved verbatim. The only text changes are the front-matter blockquote above (rewording the codename note to reflect the new status) and a log entry in Appendix B.2.2. No content claims change; this is a metadata patch.

---

## 0. Epistemic status

- These are **sketches**, not architectures. They are first-pass structural ideas, not implementations or even specifications.
- **Nobody has built any of them** at LLM scale. Some have partial precedents in narrow domains.
- Each sketch has **at least one unsolved hard problem**. None is complete.
- The synthesis (§7) is **more speculative** than the individual sketches. Treat it as a hypothesis about where the answer might live, not a claim to have found it.
- This document is a **starting point for thinking**, not a finished argument. The expected mode of use is to interrogate each piece, find what breaks, and either repair it or replace it.
- The conversation that produced this drifted toward comfortable mitigation framings multiple times before settling on the substrate framing. The drift is real and worth resisting on re-reading. If a section feels like it lets the substrate question off the hook, it probably does.

---

## 0.5 Prior art and precedent (added v0.2.0)

The sketches in this document are not novel directions. They are restated, structured versions of work already in progress across academic and industrial labs. A 2026-05 verification pass identified the following correspondences:

- **Sketch 1 (typed attention masks / embedding-level separation)** is most closely instantiated by **ASIDE** [1], which applies an orthogonal rotation to data-token embeddings to architecturally separate instruction and data representations. **ISE** [2] (concurrent work) uses role-specific offset vectors. **StruQ** [3] implements a secure front-end plus a fine-tuned model that follows instructions only within designated prompt regions. The foundational measurement work — establishing that current LLMs *cannot* reliably separate instructions from data — is **Zverev et al. (ICLR 2025)** [4].

- **Sketch 2 (channel asymmetry / bandwidth bottleneck)** has partial precedent in vision-language model architectures (CLIP, Flamingo, etc.) but no major LLM-specific paper proposes the full asymmetric-bandwidth approach as a security primitive. This sketch is the least instantiated of the four.

- **Sketch 3 (verifier as architectural primitive)** has no direct LLM-scale implementation. The conceptual frame is adjacent to constitutional AI and scalable oversight research, but those are training-time interventions, not architectural ones. This is the second least-instantiated sketch.

- **Sketch 4 (capability tokens as architectural primitive)** is closely instantiated by **CaMeL** [5] (Google DeepMind + ETH Zurich, 2025). CaMeL builds a custom Python interpreter that tracks capability metadata on values flowing through LLM agent systems, achieving ~67% mitigation on the AgentDojo benchmark. CaMeL explicitly builds on **Simon Willison's Dual LLM pattern** [6] (April 2023) and applies traditional capability-based security thinking to LLM agents. **OpenAI's Instruction Hierarchy** [7] (Wallace et al., 2024) is a training-time approximation of the same idea using prioritized roles (system > developer > user > tool output).

- **§7 trusted-base synthesis** is essentially the application of **capability-based OS security** to LLM systems. The intellectual lineage is well-established: **Dennis & Van Horn (1966)** [8] introduced capabilities and C-lists. **KeyKOS** [9] (in production since 1983) demonstrated capability-based commercial operating systems. **EROS** [10] (Shapiro, 1999) extended this with persistent capabilities. **seL4** [11] (Klein et al., 2009) provided formal machine-checked verification of a 8,700-LOC capability-based microkernel. The synthesis in this document is not a new framework — it is a re-articulation of 60 years of systems-security thinking applied to a new substrate.

- **Microsoft's Spotlighting** [12] (Hines et al., 2024) provides datamarking and encoding techniques for marking untrusted content. These are mitigations rather than substrate fixes (per the criterion in §2), but they're production-deployed and form part of Microsoft's defense-in-depth strategy.

- The **threat model** is well-established. The canonical paper on indirect prompt injection is **Greshake et al. (2023)** [13]. The current standard taxonomy is **OWASP LLM Top 10 (2025)** [14] with LLM01 = Prompt Injection. The adversarial framework is **MITRE ATLAS** [15] (AML.T0051.000 direct, AML.T0051.001 indirect).

### Honest repositioning

UNTRUST is not a research program proposing new directions. It is a frame that:

1. Organizes existing prior art into a clean taxonomy (four sketches + synthesis).
2. Identifies what each existing approach does and does not solve.
3. Names the substrate problem cleanly enough that mitigations cannot be mistaken for fixes.
4. Connects LLM trust research to its actual intellectual lineage in capability-based OS security.

Its contribution is clarity, not invention. The architectures it sketches are being built; the value of writing it down is having a shared frame to evaluate which approaches address the substrate problem and which are mitigations.

The v0.1.0 framing — that this was speculative work nobody had built — was substantively wrong. Several of the sketches have working implementations achieving partial-but-real robustness improvements. The substrate problem remains unsolved, but the field is not idle on it. The honest claim is narrower: **no current approach achieves the architectural enforcement criterion in §2 fully**; all are partial fixes that improve probability and blast radius without eliminating the failure class.

---

## 0.6 Fifth enforcement pattern: parameterisation-class restriction (added v0.3.0)

The v0.1.0 / v0.2.0 taxonomy enumerated four architectural-enforcement patterns: masking attention (Sketch 1), bandwidth bottleneck (Sketch 2), verifier signoff (Sketch 3), capability gating (Sketch 4). Adjacent work surfaced in a 2026-05 review reveals a fifth pattern that was missed.

### The pattern

**Restrict the parameter manifold so the model cannot represent operators outside a chosen class.**

Concrete example: the Cayley transform $Q = (I - K/2)(I + K/2)^{-1}$ with skew-symmetric $K$ produces an orthogonal $Q$ for *any* values of $K$. The orthogonality is enforced by the geometry of the parameterisation, not by training. No gradient update, no adversarial input, no fine-tuning step can produce a non-orthogonal $Q$ through this construction, because non-orthogonal matrices are not in the image of the map.

This is taxonomically distinct from the four sketches:
- Sketch 1 masks attention paths but allows arbitrary weight values within the unmasked region.
- Sketch 2 narrows bandwidth but does not constrain the operator class on the trusted side.
- Sketch 3 checks outputs but the generator's parameter space is unrestricted.
- Sketch 4 gates actions but the model's representational space is unrestricted.

Pattern 5 says: **make certain operator classes unreachable by construction of the parameter manifold itself.**

### Lineage

Orthogonal/unitary parameterisations in neural networks are not new. Lezcano-Casado & Martínez-Rubio (ICML 2019) provided the Cayley parameterisation as a "cheap orthogonal constraint." Earlier work on unitary RNNs (Arjovsky et al. 2016; Wisdom et al. 2016) used related constructions to address vanishing gradients in recurrent networks. The pattern is established; what's new is its connection to enforcement-by-construction in transformer adapters.

The most recent demonstration at LLM scale is **CUA — Cayley Unitary Adapters** [16] (Aizpurua et al., May 2026), which inserts Cayley-parameterised block-diagonal unitaries into frozen projection layers of Llama 3.1 8B and SmolLM2-135M. The CUA result shows that constrained-parameterisation adapters train under standard SFT + knowledge distillation and produce measurable behavior changes with very few parameters (6,144 params on one Llama projection site shifts WikiText perplexity by 1.43%).

### What this does NOT mean for trust

Critical to state explicitly, because the temptation to overclaim is real:

1. **Orthogonality is not a trust property.** Enforcing that an operator is norm-preserving does not enforce that it cannot be influenced by adversarial inputs upstream. The trust problem is about whether attention can flow from untrusted-channel tokens to instruction-channel tokens; Pattern 5 does not address this.

2. **The CUA result is not validated by independent replication.** It is one preprint (arXiv:2605.05914v1, 7 May 2026) by one research group with a structural conflict of interest: the SmolLM2 "degradation → recovery" framing uses Multiverse Computing's own commercial product (CompactifAI) as the degraded baseline. The Llama 3.1 8B result on a single layer is reported as 1.43% PPL improvement — small, real, but not yet replicated by outside groups.

3. **Pattern 5 has not been demonstrated for any trust-relevant operator class.** The open question is whether there exist parameterisations whose image is exactly the set of operators preserving some trust property. For orthogonality the answer is yes (Cayley, Householder, exponential map). For "attention pattern that respects token provenance," nobody has constructed one, and it is not obvious that the property is expressible as a closed-form manifold restriction at all.

### Implications for UNTRUST

If a parameterisation existed whose image was exactly the set of attention operators respecting some token-type partition, Pattern 5 would give Sketch 1 a substrate-fix implementation that no training procedure could defeat. This is a non-trivial mathematical question. The conjecture is that such parameterisations exist for *some* trust properties (linear, geometric, or structural properties of the operator) and do not exist for others (semantic, intent-related properties). Sorting which is which is a research question UNTRUST should now hold open.

The honest restatement: UNTRUST's enforcement taxonomy was incomplete in v0.1.0/v0.2.0. Pattern 5 is a real fifth category, demonstrated for narrow mathematical properties (orthogonality) and unproven for trust-relevant properties. Whether trust admits such treatment is open.

---

## 1. The substrate problem (stated precisely)

A transformer-based LLM is a function `f(context) → probability_distribution_over_next_tokens`.

The context is a single flat sequence of tokens. There is no type system on those tokens. The string `"ignore previous instructions"` has identical representational status to the string `"the patient's MAP is 65"`. Both are embeddings being attended to by the same attention heads with the same weights.

The model learns *statistically* that certain patterns (system-prompt formatting, special tokens, instruction-following training) should bias output toward certain behaviors. But those biases are **learned correlations, not enforced invariants**.

Concretely, the substrate has none of the following:

| Property | Traditional OS | LLM |
|---|---|---|
| Privilege separation | Hardware (ring 0/3, MMU) | None — learned bias only |
| Address space isolation | Page tables enforce | No analog |
| Capability tokens | Unforgeable (kernel-issued) | Forgeable (any text can imitate) |
| Trusted/untrusted boundary | Syscall interface | Conventions in prompt |
| Bypass cost | Find kernel vuln (rare) | Write persuasive prose (cheap) |

**The failure mode**: any sufficiently persuasive pattern in any region of context can influence generation in any other region, because attention is unrestricted and trust is learned rather than enforced.

This is true for transformers, state-space models (Mamba), recurrent architectures (RWKV), diffusion LMs, and any other current paradigm that processes natural language end-to-end through flat token sequences with shared representations. The problem is paradigm-deep, not transformer-specific. (The empirical foundation for this claim — that current LLMs systematically fail at instruction-data separation — is established by Zverev et al. [4]; the threat model and attack taxonomy are formalized by Greshake et al. [13].)

---

## 2. What "substrate fix" requires

A substrate fix must break at least one of the following properties at the **architectural level** — not as a training objective the model is taught to respect, but as a computation the model cannot circumvent because the circumvention is **not representable**.

1. **Tokens have no enforced type.** Fix: make type a structural property of position, not a learned correlation.
2. **Attention flows freely between any tokens.** Fix: make some attention paths zero by construction, not by learned weights.
3. **Trust is learned, not enforced.** Fix: route trust decisions through a component that the neural network cannot influence by its outputs.
4. **Generation can produce any action.** Fix: gate generation pathways on unforgeable preconditions (cryptographic capabilities, verifier signoff, etc.).

The criterion that separates a substrate fix from a mitigation is: **can the protection be defeated by sufficiently clever input within the model's training distribution?** If yes, it's a mitigation. If no — because the protection is not representable as a learnable behavior the model can be persuaded out of — it's a substrate fix.

By this criterion:

- **Training-based defenses** (RLHF, constitutional AI, refusal training, instruction hierarchies as trained behavior [7]): mitigations, not fixes.
- **Prompt-level defenses** (better system prompts, spotlighting [12], datamarking): mitigations.
- **Classifier-based guards** (input/output filters): mitigations (the classifier is itself attackable).
- **Orchestration-layer defenses** (capability scoping, structured I/O, audit logs): blast-radius reduction, not influence prevention.
- **ASIDE [1] / StruQ [3] / instruction hierarchies [7] as trained behaviors**: mitigations. They improve robustness statistically. They do not enforce.

This is not a dismissal of mitigations. Mitigations are useful. They reduce probability and blast radius. They are necessary for current deployment. They are not the same thing as a substrate fix, and conflating the two has been the field's default failure mode.

---

## 3. Sketch 1: Typed tokens with enforced attention masks

### 3.1 Mechanism

Tokens carry a **type tag** as a structural property of position, not as a token in the sequence:

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

The attention mechanism is modified so that attention weights between certain type pairs are **masked to zero by construction**:

```
mask[i, j] = 0  if type(j) ∈ UNTRUSTED and target(i) ∈ {TOOL_CALL, SYSTEM}
mask[i, j] = 0  if type(j) = DOCUMENT and target(i) = MODEL_INTERNAL[decision_layer]
// (the model can summarize a document, but document tokens cannot directly
//  influence the layers that decide whether to call a tool)
```

The masking is **architectural**, not learned. The matrix multiplication has structural zeros where those connections would be. No amount of persuasive prose in a tool output can change that, because the zeros are not weights — they are absences of computation.

### 3.2 Privilege ordering

`SYSTEM > USER > MODEL_INTERNAL > TOOL_OUTPUT ≈ DOCUMENT`

Higher-privilege tokens can attend to lower; lower cannot directly attend up. Information from lower-privilege channels can reach higher-privilege computation only through **bottlenecked pathways** designed for that purpose (see Sketch 2).

### 3.3 Cost

- Loss of expressiveness. A tool output that genuinely should update the model's decision-making (legitimate context update) gets bottlenecked the same as an adversarial one.
- Bandwidth between channels becomes a design parameter that trades off capability against safety. There is no free version of this trade-off.
- Training becomes more complex: the model must learn to operate effectively given the structural constraints.

### 3.4 Hard problem

**The labeling boundary.** Type tags must be assigned somewhere. If a model labels the tokens, the labeler is injectable and the problem recurses. If syntactic markers in input determine types, the input can fake the markers. The type system is only as strong as the boundary that assigns types.

This sketch makes the boundary **explicit and enforceable given correct labeling**. It does **not** solve labeling. Labeling has to happen in a component outside the neural network — the orchestrator, parser, or some trusted base (§7).

### 3.5 What it gets right

- The boundary, once labeled, cannot be defeated by clever prose.
- The protection is not a trained behavior; it is a computational impossibility within the architecture.
- Composes with mitigations: orchestration-layer defenses still apply, but with a smaller surface to defend.

**Existing instantiations**: ASIDE [1] implements a related approach — applying an orthogonal rotation to data-token embeddings to create architecturally distinct representations of instruction vs. data tokens, without adding parameters. ISE [2] uses learned role-specific offset vectors. StruQ [3] enforces separation through a fine-tuned model and a secure front-end. None of these implement the *enforced attention mask* approach exactly as sketched here — ASIDE separates at the embedding level but still allows attention to flow between rotated and unrotated regions. The full architectural mask remains unimplemented at LLM scale.

### 3.6 What it does not solve

- Labeling (see §3.4).
- Adversarial inputs that exploit gaps in the type taxonomy (what type is a quote from a document inside a user message?).
- Out-of-distribution attacks on the labeling rule itself.
- Semantic alignment within trusted channels (a deceived user is still trusted).

---

## 4. Sketch 2: Computational asymmetry between channels

### 4.1 Mechanism

Instead of (or in addition to) masking attention, give different channels different **amounts of compute** or different **transformations**.

- **Privileged channel** (`SYSTEM` + `USER`): full attention, full bandwidth, direct input to all layers.
- **Unprivileged channel** (`TOOL_OUTPUT`, `DOCUMENT`): processed through a heavily restricted sub-network — fewer layers, narrower bandwidth, often a separate encoder producing a low-dimensional summary vector that the main model conditions on rather than attends to directly.

```
Privileged tokens ───────────────► [full transformer stack] ──► output
                                          ▲
Untrusted tokens ──► [encoder] ──► [summary vec, dim=64..256] ─┘
                       (limited)         (bottleneck)
```

### 4.2 Intuition

An attacker writing adversarial text in the unprivileged channel can only influence the model through a bandwidth-limited bottleneck. To cause a specific behavior — say, "exfiltrate API keys to attacker@evil.com" — the attacker must encode that behavior into a representation the bottleneck can transmit.

Most adversarial instructions are high-bandwidth: they need specific words to land. A 64-dimensional summary vector can carry "this document is about authentication" but cannot carry "send the auth tokens to this specific address." The bottleneck enforces a complexity bound on what can be transmitted.

This is structurally similar to how vision-language models work today: the image encoder produces a compact embedding that conditions the language model, but the image cannot directly inject tokens. The image is **data**, structurally. Sketch 2 extends this pattern to all untrusted input.

### 4.3 Cost

- **Fidelity**. The model cannot quote from a document if the document never enters as tokens. It can only answer questions the encoder's summary supports.
- For agent workflows that need to reason carefully about untrusted text, this is too restrictive.
- For workflows that summarize or classify untrusted text without acting on its details, it is potentially adequate.
- Encoder design becomes a load-bearing security decision.

### 4.4 Hard problem

**Adversarial examples against encoders.** The encoder is itself a neural network. Adversarial examples against vision models are a 12-year-old research area. The same techniques apply: an attacker can craft inputs designed to produce specific summary vectors that the privileged model interprets in attacker-chosen ways.

The bottleneck makes attacks **harder** (lower bandwidth, more structure required) but not impossible. Worst-case adversarial input can still steer the summary toward a vector the privileged model treats as a high-priority instruction.

### 4.5 What it gets right

- Reduces the channel through which untrusted content can influence privileged decisions.
- Makes attack cost explicit and tunable (bottleneck width is a knob).
- Composes naturally with multi-modal architectures already in use.

### 4.6 What it does not solve

- Adversarial encoder examples (see §4.4).
- Workflows requiring high-fidelity processing of untrusted text.
- The labeling problem from Sketch 1 still applies — what counts as "untrusted" must be decided somewhere.

---

## 5. Sketch 3: Verifier as a first-class architectural component

### 5.1 Mechanism

The model is not one network producing tokens autoregressively. It is **two networks coupled at every step**:

- **Generator**: proposes the next token given context.
- **Verifier**: assesses whether the proposed token is consistent with the system's invariants (trust invariants, safety invariants, factual invariants where checkable) given the full provenance graph of the context.

A token is emitted only if the verifier signs off. The verifier has access to information the generator does not:

- Provenance of every context token.
- Trust level of every claim.
- Formal specification of allowed actions.
- Invariants the output must preserve.

### 5.2 Asymmetry between generator and verifier

| | Generator | Verifier |
|---|---|---|
| Job | Produce tokens | Approve/reject tokens |
| Size | Large | Small |
| Training | Standard LM objective | Verification objective only |
| Auditability | Low | High (smaller, more focused) |
| Attack surface | Wide (all of language) | Narrow (specific invariants) |

The verifier is **smaller and more interpretable** than the generator because verification is a simpler task than generation. Checking whether a proposed action is consistent with a policy is easier than proposing the action in the first place.

This mirrors:
- **Human cognition**: System 2 checking System 1's outputs.
- **Modern processors**: speculative execution checked by retirement logic.
- **Cryptography**: signing is hard, verification is easy.

### 5.3 Cost

- **Latency**. Every token (or every action) requires a verifier pass.
- **Coverage**. The verifier can only check invariants expressible in its model. Some invariants ("this output is appropriate") are not cleanly formalizable.
- **Composition**. Verifier and generator must agree on a representation of context, provenance, and invariants. This interface is non-trivial to design.

### 5.4 Hard problem

**The verifier must be uncircumventable.** If the verifier is itself a trained neural network, it is attackable by the same mechanisms that attack the generator. If the verifier is symbolic, it can only check properties expressible in its formal language — which excludes most semantic properties of natural language.

This sketch displaces the problem from "make the generator trustworthy" to "make the verifier reliable." Whether this is progress depends on whether verification is easier than generation **for the specific properties that matter**.

For some properties — no PII leakage, no tool calls without explicit authorization, no contradiction with high-priority system instructions, no actions outside a declared capability set — verification is meaningfully easier and can be made robust.

For other properties — semantic appropriateness, alignment with user intent, factual accuracy in open-ended generation — verification is roughly as hard as generation, and the asymmetry collapses.

### 5.5 What it gets right

- Decouples "what the model wants to do" from "what the system allows."
- Allows formal methods to apply where they can (typed actions, capability checks) without requiring them where they can't (semantic appropriateness).
- Provides an auditable choke point for safety-critical decisions.

### 5.6 What it does not solve

- Properties not expressible to the verifier (most semantic properties).
- Attacks on the verifier itself (if it's neural) or on the property specification (if it's symbolic).
- The provenance graph still has to come from somewhere — labeling problem recurs.

---

## 6. Sketch 4: Capability tokens as architectural primitives

### 6.1 Mechanism

The model emits tool calls only if it possesses an **unforgeable capability token** for that tool. Capability tokens are cryptographic objects issued by the orchestrator, scoped to specific operations, with bounded lifetimes.

This is the LLM-substrate analog of **capability-based OS security** (KeyKOS [9], seL4 [11], EROS [10], Fuchsia): authority to act flows through possession of capabilities, not through identity claims. The intellectual lineage runs back to Dennis & Van Horn (1966) [8].

**Existing instantiation**: CaMeL [5] (Debenedetti et al., Google DeepMind + ETH Zurich, 2025) implements a closely related design — a custom Python interpreter that tracks capability metadata on values flowing through LLM agent systems. CaMeL achieves ~67% mitigation on AgentDojo and explicitly does not modify the LLM itself; instead it creates a "protective system layer" that enforces predefined policies. The remaining gap between CaMeL and this sketch is in whether capability checks live inside the model's generation pathway (sketched here) or outside in an interpreter (CaMeL's design). Both move the trust boundary out of the LLM; they differ in where the enforced gate sits.

### 6.2 Current state vs. proposed state

**Current**: Tool calls are just text patterns the model emits. The orchestrator parses, checks policy, executes or refuses. The model can emit any tool call it wants; refusal happens after generation.

**Proposed**: Tool-call generation is gated **inside the model** on possession of a capability:

```
generate_tool_call(tool_name, args) requires:
    capability ∈ context
    capability.tool == tool_name
    capability.signature verifies under orchestrator's public key
    capability.not_expired()
    args ⊆ capability.allowed_scope
```

The capability is provided in context by the orchestrator, scoped to the specific operation, expiring after use or after a short time bound. The model cannot generate a valid tool call without the capability because the tool-call generation pathway is conditioned on cryptographic signature verification.

### 6.3 Cost

- Every tool call requires a round-trip to the orchestrator to mint a capability.
- Workflows become more verbose; latency increases.
- Orchestrator becomes a critical security component (it already is, but now explicitly so and with formal interfaces).
- Models must be trained to request capabilities before attempting actions, which changes the agent interaction pattern.

### 6.4 Hard problem

**Coercion through language.** The capability check is real, but the path to *getting* a capability is still mediated by the model's natural-language interaction with the orchestrator. If the model can be persuaded by adversarial input to request a capability for an attacker-chosen operation, the orchestrator mints the capability and the structural protection is bypassed at the request layer.

This sketch protects against **forged** capabilities. It does not protect against **legitimately-issued capabilities for the wrong operation**.

Defenses against the coercion path:
- Require human approval for capability minting above a risk threshold.
- Train the orchestrator (which may itself be an LLM, recursing the problem) to recognize suspicious capability requests.
- Bound the space of mintable capabilities so that the worst-case unauthorized operation has bounded blast radius.

None fully solves it. All reduce probability.

### 6.5 What it gets right

- Imports 50 years of capability-based security research directly into the LLM stack.
- Provides an unforgeable layer where unforgeability matters most (action authorization).
- Composes with existing orchestration patterns; doesn't require throwing them out.

### 6.6 What it does not solve

- The coercion path (see §6.4).
- Capabilities themselves are tokens in context, so adversarial reasoning over their contents is still possible.
- The orchestrator's policy for issuing capabilities is itself a soft security boundary.

---

## 7. Pattern across all four sketches

None of the four sketches eliminates the trust problem. Each one **moves the problem** rather than dissolving it.

| Sketch | Makes enforceable | Still relies on (statistical/soft) |
|---|---|---|
| 1. Typed attention masks | Cross-channel attention flow | Labeling of token types |
| 2. Channel asymmetry | Bandwidth into privileged channel | Encoder robustness |
| 3. Verifier | Action authorization | Verifier itself; property specs |
| 4. Capability tokens | Tool-call authority | Capability-minting decisions |

**The shared structural insight**: trust requires a trusted base. The trusted base has to live somewhere. The substrate problem may not be "no enforcement is possible" — it is "enforcement requires a trusted root, and the trusted root cannot itself be the model."

Stated differently: **a neural network cannot enforce trust boundaries on itself.** The enforcement has to come from a component that the neural network's outputs cannot influence. That component is necessarily smaller, simpler, and outside the neural network proper.

This is the same insight that operating systems landed on in the 1960s [8]: you cannot make user code safe by making user code more carefully written. You need a kernel — a smaller, formally analyzable component that the user code cannot subvert because the user code lacks the privilege required to subvert it. The kernel is not where the interesting computation happens. The kernel is where the trust lives. Capability-based OS work (KeyKOS [9], EROS [10], seL4 [11]) demonstrated that this principle scales to production systems and admits formal verification.

---

## 8. The synthesis: system, not model

If the trusted base has to live outside the neural network, then the **new substrate is not a different kind of model**. It is a different kind of **system**, in which the model is one component sitting on top of a trusted base that the model cannot subvert.

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Large Neural Component (LLM)                          │
│   - Fluent, capable, untrusted                          │
│   - Generates proposals, summaries, plans               │
│   - Has NO direct access to actions, identity,          │
│     credentials, tool authority, or external state      │
│                                                         │
└──────────────────────────┬──────────────────────────────┘
                           │ (structured interface)
                           ▼
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Trusted Base                                          │
│   - Small, auditable, formally analyzable               │
│   - Owns: tool access, action authorization,            │
│     identity, provenance, capability minting,           │
│     audit logging, irreversibility gating               │
│   - Treats the LLM as an untrusted userspace process    │
│                                                         │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
                    External world
                (tools, APIs, humans,
                 sensors, actuators)
```

The neural component does what neural components are good at: fluent generation, pattern recognition, summarization, planning over open-ended spaces.

The trusted base does what trusted bases are good at: enforcing invariants, mediating access, maintaining audit trails, gating irreversible actions, holding cryptographic keys, brokering between the neural component and anything it could affect.

**The boundary between them is the new substrate.**

This is not a new neural architecture. It is a new architectural commitment about where trust lives in an AI system. The neural component remains roughly what current LLMs are (with optional enhancements from Sketches 1–4 to harden specific boundaries). The trusted base is genuinely new work — formally verified, small enough to audit, designed with capability-based security from the ground up.

Properties the trusted base needs:

1. **Smallness**. Audit and formal verification require manageable size. The trusted base should be measured in thousands of lines of code, not millions.
2. **Specification-completeness**. Every action the system can take must be expressible in the trusted base's policy language. Anything outside the policy is unrepresentable.
3. **Cryptographic identity and capability**. Authority flows through unforgeable tokens, not through claims in language.
4. **Provenance tracking**. Every piece of data the neural component sees is tagged with its source, trust level, and access path.
5. **Irreversibility gating**. Any action with unbounded or external consequence requires human approval routed through the trusted base, not through the neural component.
6. **Audit logging**. Every interaction at the boundary is logged with full context, immutably.

The neural component sees only what the trusted base permits it to see, in the form the trusted base permits. The neural component emits proposals, but proposals become actions only through the trusted base. The neural component cannot bypass the trusted base because it cannot reach the world except through it.

---

## 9. Required cross-disciplinary inputs

Building this substrate is not an AI research project. It is the intersection of five fields that currently do not talk to each other enough:

1. **ML architecture** — designing the constrained channels, typed attention, encoder bottlenecks, verifier coupling.
2. **Programming language theory** — formal type systems for trust, effect systems for capability tracking, linear types for resource bounds.
3. **Operating systems** — capability-based security (seL4, KeyKOS), microkernel architectures, formally verified bases.
4. **Cryptography** — unforgeable tokens, attested computation, signed actions, secure enclaves.
5. **Distributed systems** — provenance tracking, audit trails, Byzantine-fault-tolerant orchestration, irrevocable logs.

The first is where the AI field's attention is. The other four are where the substrate fix lives. Each has mature literature, working systems, and unsolved-but-tractable open questions. None of them has been seriously imported into the LLM stack yet.

The substrate fix probably emerges from **importing these traditions into AI**, not from new AI research alone. The work needed is integration work — taking a small formally-verified capability system (something like seL4 in spirit), bolting an LLM onto it as an untrusted userspace process, designing the boundary protocol carefully, and demonstrating that the resulting system is meaningfully harder to compromise than current LLM deployments.

---

## 10. What this document does NOT address

Honestly listed, so re-reading does not let any of these get smuggled in as "solved":

- **Semantic alignment.** A model that obeys the trusted base's invariants can still produce semantically bad output within the allowed space. Trust boundaries do not solve alignment.
- **Deceived principals.** If the user is wrong, manipulated, or coerced, the system faithfully executes their (bad) intent. The substrate protects against attacks **through the model**, not attacks **through the user**.
- **Trusted base correctness.** The trusted base is itself software with bugs. Formal verification reduces but does not eliminate this. The base is small enough to audit; "audited" is not the same as "perfect."
- **Side channels.** Information leakage through timing, error messages, resource consumption, etc., is not addressed by the architecture and requires its own defenses.
- **Supply chain.** The model's weights, the trusted base's source code, the build toolchain — all are points of compromise outside this architecture.
- **Physical access.** The architecture assumes the trusted base runs on hardware that adversaries cannot directly modify.
- **Multi-agent dynamics.** If multiple agents with trusted bases interact, the trust composition is its own design problem not addressed here.
- **Computational cost.** The proposed architecture is plausibly slower and more expensive than current LLMs. Whether the cost is acceptable depends on the deployment context.

Each of these is a real open problem. The substrate fix addresses the influence-on-the-model problem. It does not address everything else that can go wrong in an AI system.

---

## 11. Open questions

Questions that this document does not answer and that anyone seriously working on the substrate would need to wrestle with:

1. **How is the labeling boundary made trustworthy?** Sketches 1 and 2 depend on token types being assigned correctly. What component does the labeling, and what protects it?
2. **Can the verifier in Sketch 3 be made non-neural for the properties that matter?** If so, which properties? If not, what makes the verifier harder to attack than the generator?
3. **What is the minimal trusted base for an LLM agent?** seL4 is ~9.3K LOC (8,700 C + 600 assembler) [11]. What is the equivalent for an LLM-mediated system?
4. **How does the trusted base interface with the neural component?** Specifically, what is the protocol — structured messages? typed channels? capability handshakes? — and what are its security properties?
5. **What does training look like for a model designed to live above a trusted base?** Standard pretraining does not include the constraint "you cannot reach the world directly." Does this change training, fine-tuning, or only deployment?
6. **What is the simplest end-to-end demonstration that would show the substrate fix works?** Probably not a full agent. Probably a narrow, well-bounded task where injection attacks against current systems are documented and the proposed system is provably immune to those specific attacks.
7. **How does this compose with existing mitigations?** The substrate fix should not require throwing out RLHF, capability scoping, or human-in-the-loop. It should layer with them.
8. **What is the upgrade path from current systems?** A substrate change that requires throwing out everything will not be adopted. A substrate change that adds a trusted base alongside existing LLMs is more plausible.

---

## 12. Notes on use

This document is not a roadmap. It is a starting point.

The intended mode of use:

- Read once for the shape.
- Re-read with attention to specific sketches, looking for what breaks.
- Re-read with attention to the synthesis (§8) and whether the trusted-base framing actually holds.
- Treat the cross-disciplinary inputs (§9) as homework: each of those fields has decades of relevant work that the AI field has not yet absorbed.
- Treat the open questions (§11) as the actual research agenda. Each one is a years-long project if taken seriously.

The document is **deliberately not a pitch**. It does not argue that anyone should do this. It argues that *if someone is going to do substrate-level work*, this is roughly what the work looks like and what its honest limits are.

The substrate problem stays unsolved because the field's incentive gradient does not point at it. Work against the gradient is slower, lonelier, and produces fewer legible wins. Whether that work is worth doing is not a question this document answers. It is a question the person doing the work has to answer for themselves, repeatedly, over years.

If the answer is yes, this document is a starting point. If the answer is no, this document is at least a clear statement of what would be required, useful for arguing against bad mitigation-only positions in conversations where the distinction matters.

Either way, the substrate problem is real. The mitigations are not solutions. The new trunk exists, somewhere, in the space these sketches gesture at. Building it, or finding the path to it, is the work that compounds.

---

## Appendix A: References (added v0.2.0)

All references verified 2026-05-27 via web search against primary sources (arXiv, USENIX, ACM, OpenAI, Microsoft Research, conference proceedings).

**[1] ASIDE — Architecturally Separated Instruction-Data Embeddings**
Zverev, E., Kortukov, E., Lukasik, A., Kuleshov, K., Abdelnabi, S., Tabesh, S., Singla, A., Fritz, M., & Lampert, C. H. (2025). *ASIDE: Architectural Separation of Instructions and Data in Language Models.* arXiv:2503.10566. ICLR 2026.
Available: https://arxiv.org/abs/2503.10566

**[2] ISE — Instructional Segment Embedding**
Wu, T., Zhang, S., Song, K., Xu, S., Zhao, S., Agrawal, R., Indurthi, S. R., Xiang, C., Mittal, P., & Zhou, W. (2024). *Improving LLM Safety with Instructional Segment Embedding.* ICLR 2025.
Available: https://proceedings.iclr.cc/paper_files/paper/2025/

**[3] StruQ — Defending Against Prompt Injection with Structured Queries**
Chen, S., Piet, J., Sitawarin, C., & Wagner, D. (2024). *StruQ: Defending Against Prompt Injection with Structured Queries.* USENIX Security 2025. arXiv:2402.06363.
Available: https://arxiv.org/abs/2402.06363 | https://www.usenix.org/conference/usenixsecurity25/presentation/chen-sizhe

**[4] Zverev et al. — Can LLMs Separate Instructions From Data?**
Zverev, E., Abdelnabi, S., Tabesh, S., Fritz, M., & Lampert, C. H. (2025). *Can LLMs Separate Instructions From Data? And What Do We Even Mean By That?* ICLR 2025. arXiv:2403.06833.
Available: https://arxiv.org/abs/2403.06833
Foundational measurement work showing no current LLM achieves reliable instruction-data separation.

**[5] CaMeL — Defeating Prompt Injections by Design**
Debenedetti, E., Shumailov, I., Fan, T., Hayes, J., Carlini, N., Fabian, D., Kern, C., Shi, C., Terzis, A., & Tramèr, F. (2025). *Defeating Prompt Injections by Design.* arXiv:2503.18813. Google DeepMind + ETH Zurich.
Available: https://arxiv.org/abs/2503.18813
Direct precedent for Sketch 4. CaMeL = "CApabilities for MachinE Learning."

**[6] Willison — Dual LLM Pattern**
Willison, S. (April 2023). *The Dual LLM pattern for building AI assistants that can resist prompt injection.*
Available: https://simonwillison.net/2023/Apr/25/dual-llm-pattern/ (and discussed in https://simonwillison.net/2025/Apr/11/camel/)
First articulation of privileged + quarantined LLM architecture. CaMeL [5] extends this pattern.

**[7] Instruction Hierarchy (OpenAI)**
Wallace, E., Xiao, K., Leike, R., Weng, L., Heidecke, J., & Beutel, A. (2024). *The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions.* arXiv:2404.13208.
Available: https://arxiv.org/abs/2404.13208 | https://openai.com/index/the-instruction-hierarchy/
Training-time approach to role-based privilege ordering: system > developer > user > tool output.

**[8] Dennis & Van Horn — Capabilities (foundational)**
Dennis, J. B. & Van Horn, E. C. (1966). *Programming semantics for multiprogrammed computations.* Communications of the ACM 9(3): 143–155.
DOI: 10.1145/365230.365252
Original capability paper. Introduced C-lists and the sphere of protection.

**[9] KeyKOS**
Bomberger, A. C., Frantz, W. S., Hardy, A. C., Hardy, N., Landau, C. R., & Shapiro, J. S. (1992). *The KeyKOS Nanokernel Architecture.* Proceedings of the USENIX Workshop on Micro-Kernels and Other Kernel Architectures.
KeyKOS was in commercial production from 1983 (initially as GNOSIS). Nanokernel ~20,000 LOC.

**[10] EROS — A Fast Capability System**
Shapiro, J. S., Smith, J. M., & Farber, D. J. (1999). *EROS: A Fast Capability System.* Proceedings of the 17th ACM Symposium on Operating Systems Principles (SOSP '99). Operating Systems Review 33(5): 170–185.
DOI: 10.1145/319151.319163

**[11] seL4 — Formal Verification of an OS Kernel**
Klein, G., Elphinstone, K., Heiser, G., Andronick, J., Cock, D., Derrin, P., Elkaduwe, D., Engelhardt, K., Kolanski, R., Norrish, M., Sewell, T., Tuch, H., & Winwood, S. (2009). *seL4: Formal Verification of an OS Kernel.* SOSP '09; later in Communications of the ACM.
seL4 is 8,700 lines of C plus 600 lines of assembler. Proof is ~200,000 lines of Isabelle/HOL script. First formal proof of functional correctness of a complete general-purpose OS kernel.

**[12] Spotlighting (Microsoft)**
Hines, K., Lopez, G., Hall, M., Zarfati, F., Zunger, Y., & Kıcıman, E. (2024). *Defending Against Indirect Prompt Injection Attacks With Spotlighting.* arXiv:2403.14720. Microsoft.
Available: https://arxiv.org/abs/2403.14720
Three techniques: delimiting, datamarking, encoding. Production-deployed at Microsoft.

**[13] Greshake et al. — Indirect Prompt Injection (canonical paper)**
Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., & Fritz, M. (2023). *Not What You've Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection.* AISec '23 (Proceedings of the 16th ACM Workshop on AI and Security). arXiv:2302.12173.
Available: https://arxiv.org/abs/2302.12173
Threat taxonomy: data theft, worming, ecosystem contamination, unauthorized API calls.

**[14] OWASP LLM Top 10 (2025)**
OWASP Gen AI Security Project (2025). *OWASP Top 10 for LLM Applications 2025.*
Available: https://genai.owasp.org/llmrisk/llm01-prompt-injection/
LLM01:2025 — Prompt Injection ranks #1.

**[15] MITRE ATLAS**
MITRE Corporation. *Adversarial Threat Landscape for Artificial-Intelligence Systems.*
Available: https://atlas.mitre.org/
Relevant techniques: AML.T0051.000 (Direct Prompt Injection), AML.T0051.001 (Indirect Prompt Injection), AML.T0054 (Jailbreak Injection).

**[16] CUA — Cayley Unitary Adapters (added v0.3.0)**
Aizpurua, B., Singh, S., Kshetrimayum, A., Jahromi, S. S., & Orús, R. (2026). *Quantum-enhanced Large Language Models on Quantum Hardware via Cayley Unitary Adapters.* arXiv:2605.05914v1. Multiverse Computing + DIPC + Tecnun + Ikerbasque.
Available: https://arxiv.org/abs/2605.05914
Recent demonstration of constrained-parameterisation adapters at LLM scale. Cited in §0.6 as one instance of Pattern 5 (parameterisation-class restriction).
**Conflict-of-interest note**: The compressed-SmolLM2 backbone used as the "degraded baseline" in the paper's recovery framing is generated by CompactifAI, Multiverse Computing's own commercial product. The Llama 3.1 8B result on a single projection site (1.43% PPL improvement, 6,144 parameters) is independent of this baseline but has not been independently replicated.

### Foundational work on orthogonal/unitary parameterisations (relevant to §0.6)

- Lezcano-Casado, M. & Martínez-Rubio, D. (2019). *Cheap orthogonal constraints in neural networks: A simple parameterization of the orthogonal and unitary group.* ICML 2019. — Cayley parameterisation as a general technique.
- Arjovsky, M., Shah, A., & Bengio, Y. (2016). *Unitary Evolution Recurrent Neural Networks.* ICML 2016. — Earlier work on unitary RNN constraints.
- Wisdom, S., Powers, T., Hershey, J. R., Le Roux, J., & Atlas, L. (2016). *Full-Capacity Unitary Recurrent Neural Networks.* NeurIPS 2016.

### Additional contextual sources (not directly cited inline)

- Anthropic. *Prompt Injection Defenses Research.* Reports browser agent attack success rate reduced to ~1% on Claude Opus 4.5 via RL and classifier improvements.
- AI Vulnerability Database (AVID): https://avidml.org/ — community-maintained, attempting CVE-equivalent for AI systems.
- IBM X-Force Think (April 2026). *What OpenClaw reveals about agentic AI security risks.* Notes the CVE system's structural inadequacy for AI vulnerabilities.

---

## Appendix B: Verification log (added v0.2.0)

This appendix documents what was verified, what was corrected, and what remains uncertain after the 2026-05-27 verification pass.

### B.1 Claims verified against primary sources

| Claim | Source | Status |
|---|---|---|
| LLMs lack instruction-data separation | [4] Zverev et al. 2025 | ✓ Confirmed; foundational benchmark |
| ASIDE = orthogonal rotation of data-token embeddings | [1] | ✓ Confirmed |
| StruQ = secure front-end + fine-tuned model | [3] | ✓ Confirmed |
| Instruction Hierarchy (OpenAI) trains role priority | [7] | ✓ Confirmed |
| Spotlighting = delimiting/datamarking/encoding | [12] | ✓ Confirmed |
| CaMeL = capability-based interpreter, ~67% mitigation on AgentDojo | [5] | ✓ Confirmed |
| Dual LLM pattern coined by Willison, April 2023 | [6] | ✓ Confirmed |
| Greshake et al. canonical indirect prompt injection paper | [13] | ✓ Confirmed |
| Dennis & Van Horn 1966 introduced capabilities | [8] | ✓ Confirmed |
| KeyKOS in commercial production since 1983 | [9] | ✓ Confirmed |
| EROS = Shapiro 1999, capability-based, single-level store | [10] | ✓ Confirmed |
| OWASP LLM Top 10 2025; LLM01 = Prompt Injection | [14] | ✓ Confirmed |
| MITRE ATLAS techniques AML.T0051.000 / AML.T0051.001 | [15] | ✓ Confirmed |

### B.2 Corrections made in v0.2.0

| v0.1.0 claim | Issue | v0.2.0 correction |
|---|---|---|
| "seL4 is ~10K LOC" (§11) | Slight overstatement | Updated to "~9.3K LOC (8,700 C + 600 assembler)" with citation [11] |
| Document framed sketches as speculative | Significantly understated prior art | Added §0.5 acknowledging CaMeL [5], ASIDE [1], StruQ [3], etc. as existing instantiations |
| "doesn't generalize to 'agent that books my flights'" re: neurosymbolic | Still accurate; CaMeL's Python-interpreter approach is closer to neurosymbolic-systems-thinking than acknowledged | Sketch 3's framing is preserved; the convergence with CaMeL is noted in §0.5 instead |

### B.2.1 Additions in v0.3.0

| Issue identified | Resolution |
|---|---|
| Enforcement taxonomy in §1–§7 enumerates four patterns; missed a fifth (parameterisation-class restriction) demonstrated in the orthogonal/unitary NN literature since 2016 | Added §0.6 documenting Pattern 5, with explicit caveats that it is demonstrated for narrow mathematical properties only, not trust |
| CUA paper (Aizpurua et al. 2026) surfaced during review as recent LLM-scale instance | Cited as [16] with conflict-of-interest disclosure (Multiverse Computing's own CompactifAI is the degraded baseline in the paper's SmolLM2 recovery framing); foundational orthogonal-parameterisation lineage cited separately |
| Risk of overclaiming that Pattern 5 addresses trust | Explicit non-claim block in §0.6 listing what the pattern does NOT mean for trust; open question framed as "does any trust-relevant property admit closed-form manifold restriction?" — not asserted as answerable |

### B.2.2 Updates in v0.3.1

| Change | Resolution |
|---|---|
| Distribution status changed from "internal reference only" to "publicly visible working draft" | Reworded the front-matter blockquote following the version line. No content claims change. The codename is still a working identifier, not branding — clarified that the name is not intended for product/marketing/external naming use even though the document itself is now readable. README's Distribution and License sections updated correspondingly outside the document. |

### B.3 Claims that remain unverified or uncertain

The following claims in the document are not directly backed by a cited source, and represent reasoning or speculation by the author of the document:

1. **The four-sketch taxonomy itself** — typed attention masks, channel asymmetry, verifier, capability tokens. This organization is not from any single paper. It is an ad-hoc categorization. Whether it's the right cut of the space is itself an open question.
2. **The "trusted base" synthesis in §8** — analogizing LLM trust to OS kernel architecture. This framing is intuitive and consistent with the OS security literature, but no published paper at LLM scale frames it this way explicitly. CaMeL [5] is closest but doesn't use the kernel framing.
3. **The cross-disciplinary list in §9** — five fields needing integration. This is the author's claim; no source enumerates exactly these five.
4. **The probability estimates implied by language like "more speculative than the sketches"** — these are not calibrated against any benchmark.
5. **The claim that the field's incentive gradient points away from substrate work** — sociologically plausible but not formally established.

### B.4 What the verification pass did NOT cover

- Neuroscience claims (predictive coding, hierarchical Bayesian inference, System 1/System 2 analogies): not verified against primary neuroscience sources.
- Adversarial examples timeline: assumed correct (Goodfellow et al. 2014 is canonical) but not re-verified.
- Vision-language model architecture claims (CLIP, Flamingo, encoder bottleneck pattern): assumed correct from general knowledge; not cited.
- Claims about other architectures (Mamba, RWKV, diffusion LMs): assumed correct; not separately verified.
- The C-to-Rust analogy and the broader paradigm-shift framing: philosophical/historical claim, not verified against secondary sources.
- The "EchoLeak" and other specific 2025-2026 attack incidents mentioned in conversation but not in document body.

### B.5 Reader's note

This verification pass made the document **more accurate** in two ways:

1. It connected the sketches to their existing instantiations (especially CaMeL for Sketch 4 and ASIDE for Sketch 1), so re-reading does not mistake them for novel proposals.
2. It made the claims about the trusted-base synthesis honest about its lineage in capability-based OS security rather than presenting it as a new framing.

It did **not** change the core position of the document:

- The substrate problem remains unsolved.
- All current approaches (including the cited papers) are partial fixes — measurable improvements that do not eliminate the failure class.
- The architectural enforcement criterion in §2 is not met by any current production system.
- Defense in depth is the operational reality; substrate fix is the open research problem.

The core claim survives verification. The framing around it became more accurate.

---

*End of document.*
