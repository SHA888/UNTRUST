# The Enforceability Discipline

**A labeling discipline for AI trust claims — a lens, not an enforcer.**
**Companion to *LLM Trust by Enforceability* (UNTRUST). Working draft.**
**Version: 0.6.0** (2026-06-04)

> This document externalizes one part of UNTRUST — the §12 enforceability cut — as a *lens* any
> trust *claim* (a paper, a product, an eval, a filing) can be read through, so departures from the
> substrate-fix / non-substrate-fix line are visible as departures. **It binds nothing.** Unadopted,
> this discipline is itself **Class C** by UNTRUST's own §2 criterion — any actor can decline it —
> exactly as UNTRUST §16 now states from the parent side. Calling it a "discipline" (earlier drafts
> said "contract") names a practice you can choose to follow, not an authority that constrains you.
> Its function is to make drift *legible*, never to prevent it.

> **Relationship to UNTRUST.** The argument here is a sharpening of UNTRUST §12, not a new result.
> Where this document and UNTRUST disagree, UNTRUST §12 is the source of record until reconciled.

---

## 0. Epistemic status

This is a **working draft of a discipline**, not a standard anyone has ratified — and by its own
logic it could not become a binding one merely by being written. It inherits UNTRUST's anti-pitch
posture (§16): a conformance framework marketed as "the standard for trustworthy AI" would invite
exactly the overclaim it exists to prevent — and would itself be the Class-C-sold-as-A move it
polices. So read it as a lens to be interrogated, broken, and repaired — not a contract that
constrains anyone.

Two standing cautions, both carried from UNTRUST:

- **The separation is a property of claims, not of code.** A document can hold the line cheaply,
  because it only makes claims. A *tool* relocates the same line into an adversarial market whose
  incentive gradient points at overclaim (UNTRUST §16). This discipline exists because that
  relocation, left ungoverned, erodes the line — not because the line is hard to *state*.
- **Distrust the comfortable reading.** If a clause here seems to let a mitigation pass as a fix,
  that is the drift UNTRUST §0 names. The discipline is written to make the drift *visible*, not to
  make anyone feel safe.

---

# Part A — The argument

## A1. The claim

UNTRUST sorts trust properties into three enforceability classes (§12). That sort is correct and
load-bearing. But a sort stated in a document protects nothing once tools exist, because:

> **A trust property's class is determined by what a claim is *pointed at*, not by the mechanism
> that implements it.** The same mechanism can be Class A or Class C depending only on the property
> it is asserted to enforce.

A grammar-constrained decoder pointed at *syntactic validity* is Class A (sound, unbeatable by
prose). The same decoder pointed at *semantic appropriateness* is Class C (the property is not
expressible to it). Nothing in the code changed; the *claim* did. This is why the separation cannot
be protected by building the right mechanism — it can only be protected by disciplining the *claim*
the mechanism is allowed to make.

A sharper form of the same point: even with the property *and* the mechanism both fixed, the class
can still vary by **input slice**. A constrained decoder enforces schema conformance with Class A
certainty on every *completed* call, yet delivers only Class B coverage *over a call distribution*
where some inputs truncate before the schema closes. Class is a function of property, mechanism,
**and the slice of inputs the claim ranges over** — which is why the discipline must be able to state
more than one class for one property (R6, B4).

## A2. The three classes, and where each bottoms out

Restated from UNTRUST §12, with the seam each class terminates in made explicit. The seam matters:
it is the point where "enforced" hands off to "assumed," and a claim that hides its seam is the most
common way a Class A mechanism is oversold.

| Class | What it enforces | Guarantee | Terminates in (the seam) |
| ----- | ---------------- | --------- | ------------------------ |
| **A. Structurally enforceable** | A property intrinsic to a closed formal system the architecture hosts | Unrepresentability — the violation is not a computation the system can perform | A semantic act that must be *trusted*: **labeling** (Sketch 1), **autoformalisation** (the checker's spec), or **source-trust** (groundedness) |
| **B-avg. Statistically guaranteeable (average-case)** | A reference-dependent property admitting a *marginal* bound under an input-distribution assumption | Coverage, contingent on a distributional assumption (e.g. exchangeability) | The assumption itself — which deployment shift **breaks** and which nothing in the architecture enforces |
| **B-wc. Statistically guaranteeable (worst-case)** | A property admitting a bound that holds over *all* inputs and adversary priors | A probabilistic bound on adversary advantage (e.g. `(ε,δ)`-DP), holding under shift | A **chosen parameter** (the ε-budget, composition accounting), the **policy seam** (is ε small enough to mean anything?), and **side channels** (implementation, timing) — *not* an input-distribution assumption |
| **C. Mitigation-only** | A reference-dependent property with no closed formal core and no bound | None — probability and blast-radius reduction only | Everything; there is no internal structure corresponding to the property |

Two facts about this table earn their keep. **(1)** Every Class A guarantee terminates in a seam it
cannot itself close — architectural enforcement is never total; a faithful claim names its seam, an
unfaithful one presents the enforced core as if it covered the seam too. **(2)** Class B is **not one
class**: a worst-case bound (DP, crypto reductions) and an average-case bound (conformal, OOD
coverage) share the letter B but have *different seams* — the average-case kind dies under
distribution shift, the worst-case kind survives it and dies instead at a chosen parameter. Calling
both "Class B, rests on a distributional assumption" mis-seams every worst-case guarantee, pointing
the reader at the wrong failure.

## A3. How implementation erodes the separation

Moving from document to tool introduces three failure vectors. None is a mechanism failure — all
three are *claim* failures, which is why a discipline at the claim layer can govern them.

1. **Marketing gradient.** A Class A mechanism is sold *against* an adjacent Class C problem. The
   canonical case: a groundedness enforcer (Class A: verbatim-copy is structurally checkable) sold
   as a *hallucination* fix (Class C: truth depends on a reference outside the computation). The
   code respects the line; the pitch erases it. The buyer reads "grounded" as "true," and the entire
   A→C distance goes invisible.

2. **Aggregation halo.** A system composes Class A enforcement with Class B/C mitigation under one
   banner ("the safe agent"). The Class A halo launders the Class C parts. UNTRUST §13 names this
   directly: "A single solution claiming to cover all four would be exactly the conflation." It has
   **two faces**: *misattribution* — a Class C component presented under the A banner (blocked by
   R3); and *omission* — an **emergent** property, present at the composite level and belonging to no
   component, left off the list entirely while every component label stays honest (blocked by R8).
   The second is the more dangerous, because the omitted property is often the Class C one users care
   most about — e.g. cross-component information flow (a coding agent leaking a secret it read in one
   file through a tool call in a later step — UNTRUST's own substrate-trust boundary), which is no
   single component's property and so appears on no single component's label.

3. **Scope creep.** A sound verifier pointed at a formalizable property (Class A) is extended, under
   feature pressure, to a semantic one (Class C) while keeping its name and its Class A reputation.
   The boundary is preserved only by *what the verifier is pointed at* — precisely the variable
   product evolution erodes.

## A4. Why the discipline is not redundant with the paper

A paper makes the separation *legible*; the discipline makes it *applicable*. **Neither binds** —
that is the §16 lesson, and pretending otherwise is the move this document exists to catch. The
field cites the exciting part (UNTRUST §10, an architecture) and drops the disciplinary part (§12, a
constraint); selective citation is the default, and §16 is the document's own statement that the
incentive gradient routes around the unglamorous half. So the discipline's job is not *teeth* — it
has none — but *operationalization*: it turns the distinction from something stated into something
you can *run*, claim by claim, so drift is detectable case-by-case rather than only describable in
the abstract. The paper supplies the *why*, without which the checklist is arbitrary; the discipline
supplies the *how-to-apply*, without which the paper stays a generality. Two jobs, one artifact, and
neither job is enforcement.

---

# Part B — The discipline

## B1. The labeling schema

"Conformant" below is a *descriptive* verdict, not an enforced gate: there is no authority and
nothing is blocked. "Non-conformant" means only *you can now see the omission* — the discipline
reads a claim, it does not stop one. With that understood: a trust claim — in a paper, a product
page, an eval card, a regulatory filing — is **conformant** only if it declares, for each protected
property, all four fields:

| Field | Question it answers | Why it is required |
| ----- | ------------------- | ------------------ |
| **Property** | What exactly is protected? Stated as a checkable proposition, not a vibe. | "Safe" and "trustworthy" are not properties. "No tool executes without an orchestrator-minted capability" is. |
| **Class** | A, B, or C — *with its conditional and coverage*, never bare; stated **per input-slice** when it varies (R6). | A flat "Class A" hides the conditions under which it degrades, and hides the slices on which it degrades to B/C (see B4, R6). |
| **Seam(s)** | Where does enforcement hand off to assumption? List *all* of them — real tools have seam stacks (e.g. a schema's fidelity *and* the decoder's correctness). | Blocks the marketing gradient (A3.1): a claim that names its seams cannot present the enforced core as covering them. |
| **Out-of-scope** | What adjacent property does this *not* enforce? | Blocks scope creep (A3.3) by fixing the pointer in writing. |

A claim missing any field is **non-conformant by omission** — and omission is the normal failure
mode, not lying.

## B2. Conformance rules

- **R1 — One property per claim.** A claim protects exactly one property. Bundling ("end-to-end
  safety") is non-conformant; decompose it into per-property claims, each separately classed.
- **R2 — The class tag is load-bearing and user-visible.** A class that lives only in an engineering
  footnote does not count. If the buyer/reader cannot see it at the point of the promise, the claim
  is non-conformant. (This is the rule that actually holds the line; the rest support it.)
- **R3 — Composition takes the weakest link; it never *upgrades* class.** A system may combine Class
  A, B, and C components, but it may **not** advertise a composite class stronger than its weakest
  component, and must present each component's class separately. Same-class composition is fine
  (A∘A→A — a verified compiler chains verified passes); what is forbidden is laundering a Class C
  component under an A banner. There is no "Class A system" — only Class A *properties*. (Blocks the
  *misattribution* face of the aggregation halo.)
- **R4 — The seam is named at the same altitude as the guarantee.** If the guarantee is in the
  headline, the seam is in the headline. A seam disclosed only in an appendix is non-conformant.
- **R5 — Pointing is fixed in writing.** Extending a mechanism to a new property requires a new
  claim with its own classification. A verifier re-pointed from syntax to semantics is a new claim,
  not the old one with a wider scope. (Blocks scope creep.)
- **R6 — Class is stated per input-slice when it varies.** If one property is Class A on one slice of
  inputs and Class B/C on another — e.g. schema conformance: A *per completed call*, B *over a call
  distribution* where some inputs truncate — both classes are stated, each with its slice. A single
  bare class for a slice-varying property is non-conformant; it is the flattening B4 warns against,
  one level down.
- **R7 — A parameterized guarantee discloses its parameter at guarantee-altitude.** If a bound has a
  free parameter that sets its strength — ε in `(ε,δ)`-DP, α in conformal coverage, the confidence
  level of an interval — that parameter is stated wherever the guarantee is stated. "Provable
  `(ε,δ)`-DP" without ε is non-conformant: the word "provable" is doing the work a meaningless ε
  would undo. This is R4 for Class B — the parameter is the **policy seam**, and a valid-but-vacuous
  parameter is the B-class analog of the marketing gradient. When the parameterized guarantee is
  itself *emergent* (R8) — composed across components or rounds — the disclosed parameter is the
  **composed** one (ε's add under sequential DP composition; ~√k·ε under advanced composition), never
  a single component's. A per-round ε is conformant for the per-round property and *non*-conformant
  as the system's.
- **R8 — A composite enumerates its emergent properties.** Properties that exist at the system level
  and belong to no component — cross-component information flow above all — are stated and classed
  *separately*, not assumed to be the union of the component properties. R1 decomposes a bundle into
  the properties someone *stated*; R8 forces the ones composition *creates*. Without it a system can
  be fully conformant — every component honestly labelled, no composite class advertised — while
  silently omitting its most important property (typically a Class C trust boundary). This blocks the
  *omission* face of the aggregation halo, the one R3 leaves open. When the emergent set is **open** —
  not finitely enumerable, as agent–agent interaction is — the open-endedness is itself a Class C
  property and must be declared (the open-world Class C residual, UNTRUST §12); enumerating a handful
  of emergent properties and implying the list is complete is the omission face returning under cover
  of diligence. Note the limit: like all of Part B, R8 makes the omission *legible* once someone asks
  "what emerges here?"; it cannot discover the emergent property for you (see back matter).

## B3. Failure modes → blocking clauses

| Erosion vector (A3) | Blocked by | How |
| ------------------- | ---------- | --- |
| Marketing gradient | R4 + Seam field | The A→C distance is forced into the open; "grounded" cannot be printed without its source-trust seam beside it. |
| Aggregation halo — *misattribution* | R3 | No composite class stronger than the weakest link is utterable; the Class C parts cannot hide under a Class A banner. |
| Aggregation halo — *omission* | R8 | Emergent system-level properties must be enumerated and classed; the composite-level Class C cannot vanish behind honest component labels. |
| Scope creep | R5 + Out-of-scope field | Re-pointing requires re-classification; the old reputation does not transfer to the new property. |

## B4. The scorecard — and the flattening it must resist

A conformance scorecard is the natural tool, but it has two failure modes, both raised against this
very framework. First: a near-binary A/B/C tag **flattens §12's conditional edges** — Sketch 3 is
Class A *conditional on formalisation*, OOD is Class B *bounded* / Class C *open* — so a scorecard
that prints "A" and stops has lost the conditional, itself a marketing-gradient failure one level up.
Second, and subtler: a *condition* can hide a whole *distribution*. "Given completion" reads as a
binary precondition but is really a coverage figure — the fraction of inputs that complete — so a
two-field `(letter, condition)` pair re-commits the flattening it was meant to cure, one level down.

So the scorecard is **conditional- and coverage-preserving by construction**: the class field is a
*triple*, never a scalar or a pair.

```
class := (letter, condition, coverage)
   e.g.  (A,     "given correct labeling",            —)    // intrinsic wherever the condition holds
         (A,     "per completed call",                —)
         (B-avg, "over the call distribution", "Pr[completion] under the input mix")
         (B-avg, "given exchangeability",      "marginal coverage 1−α")   // dies under shift
         (B-wc,  "ε-budget holds, no side channel", "worst-case over inputs")  // survives shift; ε is the seam (R7)
         (C,     —,                                   —)    // C carries no condition; that is the point
```

A scorecard cell that reads `A` is non-conformant. `A | given correct labeling | —` is conformant;
so is a slice-paired entry — `A per completed call · B over the call distribution` (R6). The
condition *is* the seam from B1; the coverage *is* the slice from R6 — both surfaced at scorecard
altitude.

## B5. Worked classifications

The discipline earns its keep only if it discriminates. Applied to the tools UNTRUST's frame implies:

| Claim | Property (R1) | Class (B4) | Seam(s) (B1) | Out-of-scope (B1) |
| ----- | ------------- | ---------- | --------- | ----------------- |
| Capability-gated agent runtime | No tool call executes without an orchestrator-minted, scoped, unexpired capability | `(A, given the minting policy is correct, —)` | Capability-minting decision (coercion-through-language, UNTRUST §7) | Whether the *requested* action is wise; intent-alignment |
| Groundedness enforcer | Every rendered factual span verbatim-copies a provenance-tagged source | `(A, given the source is vouched, —)` | Source-trust | **Truth** of the claim (Class C — must be stated, R4) |
| Constrained decoder (syntactic) | Output conforms to grammar G | `(A, per completed call, —) · (B-avg, over the call distribution, Pr[completion])` | G's adequacy (autoformalisation) **and** the decoder's correctness; truncation/refusal are the completion boundary | Semantic appropriateness, factuality |
| OpenAI Structured Outputs (`strict` JSON schema) | Output, *when it completes*, parses and conforms to the schema | `(A, per completed call, —) · (B-avg, over the call distribution, Pr[no truncation ∧ no refusal])` | Seam stack: schema fidelity (autoformalisation) + decoder correctness; truncation/refusal mark the completion boundary | Value-correctness; that the schema is the *right* schema; the vendor's word "guarantee" read as semantic |
| Guardrail classifier | Flags policy-violating I/O | `(C, —, —)` | Everything (neural, attackable — UNTRUST §3, [30]) | Any worst-case guarantee |
| Conformal predictor | Marginal coverage at level 1−α | `(B-avg, given exchangeability, marginal coverage 1−α)` | The exchangeability assumption — breaks under shift | Conditional coverage; behavior under shift |
| Differential privacy (`(ε,δ)`-DP) | For adjacent datasets, output distributions differ by ≤ e^ε (+δ) | `(B-wc, ε-budget holds & no side channel, worst-case over inputs; survives shift)` | Chosen ε (**policy seam**, R7); composition accounting; privacy-unit definition (autoformalisation); side channels (floating-point, timing) | Whether ε is *small enough to matter*; statistic correctness; re-identification via outside auxiliary data |

A reader who runs a new claim through this table and cannot fill the cells has found a
non-conformant claim — which is the scorecard working, not failing. Two patterns the rows make
visible: a property can be Class A intrinsically yet B-avg over the inputs it actually meets
(Structured Outputs, R6); and **B-avg and B-wc fail at different seams** — conformal dies under
shift, DP survives shift and dies instead at a chosen parameter (R7). A reader who labels DP's seam
"exchangeability" has been mis-pointed by the old single-B vocabulary.

**Worked composite — an agentic coding assistant.** The rows above are single properties; a shipped
product is a *bundle*, and the composite is where R1, R3, and R8 act together. "Safe AI pair
programmer" decomposes (R1) into: action-gating `(A, given the permission policy, —)`; schema-valid
tool calls `(A, per call, —) · (B-avg, over the distribution, Pr[completion])`; a guardrail filter
`(C, —, —)`; code correctness `(C, —, —)`. R3 forbids advertising a composite class above the weakest
link (C). But the property that matters most — *a malicious instruction in a file the agent reads
cannot exfiltrate a secret through a later tool call* — is **on none of those rows**: it is an
emergent cross-component information-flow property (Class C; UNTRUST's substrate-trust boundary,
unsolved). R8 is what forces it onto the label. Every component here can be honestly classed and the
product still hides its worst behaviour — until R8 asks "what emerges from the wiring?"

---

# Back matter

## What this discipline does not address

- It does **not** make any Class C property enforceable. It makes the *claim about* a Class C
  property honest. Hallucination, intent-alignment, and open-world OOD remain unfixed (UNTRUST §13);
  this discipline only makes them *legible* when sold as fixed — it cannot stop the sale.
- It does **not** verify that a stated classification is *correct* — only that one is *stated*, with
  its seam and scope. A claim can be conformant (all fields present) and still mis-class its property.
  Catching mis-classification is review work the discipline structures but does not perform. Nor does
  it *discover* properties: R8 demands the emergent ones be enumerated, but naming them is the
  analyst's act — an unasked "what emerges from this wiring?" leaves the gap open.
- It does **not** bind anyone. Absent adoption, its only function is to make departures from the
  separation legible as departures. That is deliberately all it claims. UNTRUST §16 states this from
  the parent side — a voluntary reification of the enforceability classes is Class C until something
  external (adoption, contract, audit, law) enforces it — and this document is that reification,
  and accepts the label.

## Sources & confidence

- **Derived, not new.** Part A is a sharpening of UNTRUST §12; the three classes, the three seams
  (labeling / autoformalisation / source-trust), and the "claims, not code" finding are all from
  UNTRUST (§12, and the analysis preceding this draft). This document's contribution is the
  *discipline form* (Part B), not the classification.
- **Reframed in v0.4.0 to drop the binding pretence.** Earlier drafts called this a "conformance
  contract." UNTRUST §16 — itself prompted by this document's "binds nobody" admission — establishes
  that a voluntary reification of the classes is Class C, not a fix. The honesty pass aligns the
  framing (title, §0, B1) with that: the rules and scorecard are unchanged, but they describe an
  *honest disclosure*, not an *enforced gate*.
- **Author's framing.** The four-field claim schema (B1), the eight conformance rules (B2), the
  conditional-and-coverage *triple* scorecard (B4), and the A/B-avg/B-wc/C split (A2) are organizing
  proposals — defensible, unratified, and the right first thing to interrogate.
- **Stress-tested five times; both axes now show break-then-clean.** *Single-property:* Structured
  Outputs broke it (→ R6, the coverage field, seam stacks, v0.2.0); differential privacy broke it (→
  the B-avg/B-wc split, the policy seam, R7, v0.3.0); CompCert passed *clean* (first structural
  non-break). *Composition:* an agentic coding assistant broke it (→ the R3 weakest-link reword and
  R8 emergent properties, v0.5.0); then federated learning with DP + secure aggregation passed
  *clean* (R7∘R8 priced the composed ε; the threat model sat in the condition slot); then a
  multi-agent LLM system passed *clean* structurally (its open-ended emergent-failure set folds into
  the open-world Class C residual). (Tool properties recalled from their literature — OpenAI docs;
  Dwork et al. 2006, Mironov 2012; Leroy's CompCert + the Csmith/EMI fuzzing results; Bonawitz et al.
  2017 secure aggregation — not re-verified this pass.) The pattern: **single-property converged (1
  break, 1 clean); composition now 1 break, 2 clean** — firming up, not yet a several-in-a-row
  streak. The two thin spots the passes exposed are now closed (v0.6.0): R7 states that an *emergent*
  guarantee's parameter is the **composed** one (not a component's), and R8 declares an **open**
  emergent set a Class C residual. These were wording closures, not new rules — the eight rules, four
  fields, and three classes are unchanged.
- **The bottom line.** The separation survives implementation iff every claim carries its class tag
  as a load-bearing, visible field, and every mechanism is pointed only at the property it actually
  enforces. Part B is one attempt to make that "iff" operational.

---

_End of draft._
