# The Enforceability Discipline

**A conformance contract for AI trust claims.**
**Companion to *LLM Trust by Enforceability* (UNTRUST). Working draft.**
**Version: 0.2.0** (2026-06-04)

> This document externalizes one part of UNTRUST — the §12 enforceability cut — and turns it
> from a description into a discipline: a contract that any trust *claim* (a paper, a product, an
> eval, a filing) must honor to keep the substrate-fix / non-substrate-fix line intact.
> It is descriptive, not promotional. It does not argue anyone should adopt it; it states what
> faithfulness to the separation requires, so that departures are visible as departures.

> **Relationship to UNTRUST.** The argument here is a sharpening of UNTRUST §12, not a new result.
> Where this document and UNTRUST disagree, UNTRUST §12 is the source of record until reconciled.

---

## 0. Epistemic status

This is a **working draft of a discipline**, not a standard anyone has ratified. It inherits
UNTRUST's anti-pitch posture (§16): a conformance framework marketed as "the standard for
trustworthy AI" would invite exactly the overclaim it exists to prevent. So read it as a contract
proposal to be interrogated, broken, and repaired — not adopted.

Two standing cautions, both carried from UNTRUST:

- **The separation is a property of claims, not of code.** A document can hold the line cheaply,
  because it only makes claims. A *tool* relocates the same line into an adversarial market whose
  incentive gradient points at overclaim (UNTRUST §16). This discipline exists because that
  relocation, left ungoverned, erodes the line — not because the line is hard to *state*.
- **Distrust the comfortable reading.** If a clause here seems to let a mitigation pass as a fix,
  that is the drift UNTRUST §0 names. The contract is written to make the drift *visible*, not to
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
**and the slice of inputs the claim ranges over** — which is why the contract must be able to state
more than one class for one property (R6, B4).

## A2. The three classes, and where each bottoms out

Restated from UNTRUST §12, with the seam each class terminates in made explicit. The seam matters:
it is the point where "enforced" hands off to "assumed," and a claim that hides its seam is the most
common way a Class A mechanism is oversold.

| Class | What it enforces | Guarantee | Terminates in (the seam) |
| ----- | ---------------- | --------- | ------------------------ |
| **A. Structurally enforceable** | A property intrinsic to a closed formal system the architecture hosts | Unrepresentability — the violation is not a computation the system can perform | A semantic act that must be *trusted*: **labeling** (Sketch 1), **autoformalisation** (the checker's spec), or **source-trust** (groundedness) |
| **B. Statistically guaranteeable** | A reference-dependent property admitting a distribution-level bound | Coverage, contingent on a distributional assumption (e.g. exchangeability) | The assumption itself — which deployment shift breaks and which nothing in the architecture enforces |
| **C. Mitigation-only** | A reference-dependent property with no closed formal core and no bound | None — probability and blast-radius reduction only | Everything; there is no internal structure corresponding to the property |

The single most important fact about this table: **every Class A guarantee terminates in a seam it
cannot itself close.** Architectural enforcement is never total. A faithful claim names its seam; an
unfaithful one presents the enforced core as if it covered the seam too.

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
   directly: "A single solution claiming to cover all four would be exactly the conflation."

3. **Scope creep.** A sound verifier pointed at a formalizable property (Class A) is extended, under
   feature pressure, to a semantic one (Class C) while keeping its name and its Class A reputation.
   The boundary is preserved only by *what the verifier is pointed at* — precisely the variable
   product evolution erodes.

## A4. Why a paper alone cannot hold the line

A paper makes the separation *legible*; it does not make it *binding*. The field cites the exciting
part (UNTRUST §10, an architecture) and drops the disciplinary part (§12, a constraint) — selective
citation is the default, and §16 is the document's own statement that the incentive gradient routes
around the unglamorous half. A normative contract is therefore not redundant with the paper: the
paper supplies the *why*, without which the contract is arbitrary; the contract supplies the *teeth*,
without which the paper is inert. They are one artifact with two jobs.

---

# Part B — The discipline

## B1. The labeling contract

Any trust claim — in a paper, a product page, an eval card, a regulatory filing — is **conformant**
only if it declares, for each protected property, all four fields:

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
- **R3 — Composition does not compose class.** A system may combine Class A, B, and C components, but
  it may **not** advertise a composite class. It must present the class of each component
  separately. There is no "Class A system" — only Class A *properties*. (Blocks the aggregation halo.)
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

## B3. Failure modes → blocking clauses

| Erosion vector (A3) | Blocked by | How |
| ------------------- | ---------- | --- |
| Marketing gradient | R4 + Seam field | The A→C distance is forced into the open; "grounded" cannot be printed without its source-trust seam beside it. |
| Aggregation halo | R3 | No composite class is utterable; the Class C parts cannot hide under a Class A banner. |
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
   e.g.  (A, "given correct labeling",              —)    // intrinsic wherever the condition holds
         (A, "per completed call",                  —)
         (B, "over the call distribution",   "Pr[completion] under the input mix")
         (B, "given exchangeability / bounded shift", "marginal coverage 1−α")
         (C, —,                                      —)    // C carries no condition; that is the point
```

A scorecard cell that reads `A` is non-conformant. `A | given correct labeling | —` is conformant;
so is a slice-paired entry — `A per completed call · B over the call distribution` (R6). The
condition *is* the seam from B1; the coverage *is* the slice from R6 — both surfaced at scorecard
altitude.

## B5. Worked classifications

The contract earns its keep only if it discriminates. Applied to the tools UNTRUST's frame implies:

| Claim | Property (R1) | Class (B4) | Seam(s) (B1) | Out-of-scope (B1) |
| ----- | ------------- | ---------- | --------- | ----------------- |
| Capability-gated agent runtime | No tool call executes without an orchestrator-minted, scoped, unexpired capability | `(A, given the minting policy is correct, —)` | Capability-minting decision (coercion-through-language, UNTRUST §7) | Whether the *requested* action is wise; intent-alignment |
| Groundedness enforcer | Every rendered factual span verbatim-copies a provenance-tagged source | `(A, given the source is vouched, —)` | Source-trust | **Truth** of the claim (Class C — must be stated, R4) |
| Constrained decoder (syntactic) | Output conforms to grammar G | `(A, per completed call, —) · (B, over the call distribution, Pr[completion])` | G's adequacy (autoformalisation) **and** the decoder's correctness; truncation/refusal are the completion boundary | Semantic appropriateness, factuality |
| OpenAI Structured Outputs (`strict` JSON schema) | Output, *when it completes*, parses and conforms to the schema | `(A, per completed call, —) · (B, over the call distribution, Pr[no truncation ∧ no refusal])` | Seam stack: schema fidelity (autoformalisation) + decoder correctness; truncation/refusal mark the completion boundary | Value-correctness; that the schema is the *right* schema; the vendor's word "guarantee" read as semantic |
| Guardrail classifier | Flags policy-violating I/O | `(C, —, —)` | Everything (neural, attackable — UNTRUST §3, [30]) | Any worst-case guarantee |
| Conformal predictor | Marginal coverage at level 1−α | `(B, given exchangeability, marginal coverage 1−α)` | The exchangeability assumption | Conditional coverage; behavior under shift |

A reader who runs a new claim through this table and cannot fill the cells has found a
non-conformant claim — which is the scorecard working, not failing. The Structured Outputs and
constrained-decoder rows show why the class column must admit *two* entries: a property can be Class
A intrinsically yet Class B over the inputs it actually meets (R6).

---

# Back matter

## What this discipline does not address

- It does **not** make any Class C property enforceable. It makes the *claim about* a Class C
  property honest. Hallucination, intent-alignment, and open-world OOD remain unfixed (UNTRUST §13);
  this contract only stops them being sold as fixed.
- It does **not** verify that a stated classification is *correct* — only that one is *stated*, with
  its seam and scope. A claim can be conformant (all fields present) and still mis-class its property.
  Catching mis-classification is review work the contract structures but does not perform.
- It does **not** bind anyone. Absent adoption, its only function is to make departures from the
  separation legible as departures. That is deliberately all it claims.

## Sources & confidence

- **Derived, not new.** Part A is a sharpening of UNTRUST §12; the three classes, the three seams
  (labeling / autoformalisation / source-trust), and the "claims, not code" finding are all from
  UNTRUST (§12, and the analysis preceding this draft). This document's contribution is the
  *contract form* (Part B), not the classification.
- **Author's framing.** The four-field claim schema (B1), the six conformance rules (B2), and the
  conditional-and-coverage *triple* scorecard (B4) are organizing proposals — defensible, unratified,
  and the right first thing to interrogate.
- **Stress-tested once.** R6, the coverage field, and the seam-stack allowance (B1) were added after
  running OpenAI Structured Outputs through B5 surfaced a class that varies by input-slice — a gap
  the v0.1.0 single-class schema could not express. (The Structured Outputs limitations are recalled
  from OpenAI's documentation, not re-verified this pass.) One stress test is not many; treat the
  schema as still under test.
- **The bottom line.** The separation survives implementation iff every claim carries its class tag
  as a load-bearing, visible field, and every mechanism is pointed only at the property it actually
  enforces. Part B is one attempt to make that "iff" operational.

---

_End of draft._
