# CH-39 assessment: Motorcycle and Truck Mechanic SLMs

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [39.0 Chapter contract and evidence status](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md#390-chapter-contract-and-evidence-status)
- [39.1 Why “mechanic” is not one domain](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md#391-why-mechanic-is-not-one-domain)
- [39.14 Review questions and engineering exercises](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md#3914-review-questions-and-engineering-exercises)
- [39.5 Candidate model organizations](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md#395-candidate-model-organizations)
- [39.11 Safety and operational limits](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md#3911-safety-and-operational-limits)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **shared mechanic ontology** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** A shared mechanic ontology represents genuinely common systems and concepts while preserving vehicle-class-specific meanings.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **shared mechanic ontology** with **domain-specific adapters** using one condition where the distinction changes an engineering decision.

**Model answer:** Domain-specific adapters specialize a shared base without duplicating all weights, but routing and cross-domain interference must be tested.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **safety boundary separation** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Safety-boundary separation prevents motorcycle procedures, heavy-truck pneumatic systems, or automotive assumptions from being applied across incompatible equipment.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Calculate storage and training cost for shared-base adapters versus independent models, then design an interference test with motorcycle and truck strata.

**Worked example:** If two independent 1.2 GiB models require 2.4 GiB and one shared 1.2 GiB base plus two 80 MiB adapters requires 1.356 GiB, storage saving is 1.044 GiB. Add adapter training and validation cost; storage alone does not settle the decision.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Audit transfer-learning evidence for domain overlap; reject conclusions based only on generic mechanic vocabulary. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** Generic “mechanic” benchmark gains do not prove transfer between motorcycles and heavy trucks. Require subsystem- and hazard-stratified results and confusion tests.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Choose shared foundation, adapters, or independent models for motorcycles and heavy trucks under mobile switching and update constraints. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Use a shared foundation only for verified overlap; isolate adapters or models for pneumatic brakes, heavy loads, emissions, and vehicle-specific controls.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Separate hydraulic brakes, pneumatic brakes, emissions, high-voltage, load, torque, and service-document requirements.

**Model guidance:** Brake, load, high-voltage, hydraulic/pneumatic, and emissions procedures require vehicle-specific context and qualified-technician escalation.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A paired-domain ontology, source/data partition, adapter experiment, cross-domain confusion suite, and safe diagnostic response contract.

### Walkthrough

1. **Freeze the question and inputs.** Record immutable model, tokenizer, data, code, environment, device/runtime, and
   license identities; state intended use and prohibited use.
2. **Build the smallest discriminating artifact.** Include a normal case, boundary case, and expected failure. Preserve
   raw outputs and unsuccessful attempts rather than keeping only the preferred result.
3. **Measure against acceptance gates.** Separate quality, safety, memory, latency, energy/cost, and operational
   evidence. Do not average a critical failure into a passing score.
4. **Challenge the result.** Test an alternative explanation, one changed assumption, rollback/cleanup, and the
   condition that would reverse the decision.
5. **Package the evidence.** Submit the protocol, manifests, hashes, commands or code, raw results, analysis, failure
   record, decision, and honest status label.

**Completion test:** Another engineer can reproduce the calculation or run without guessing an identity, unit,
threshold, or excluded failure.

## 7. Milestone handoff — 5 points

**Deliverable:** A transfer decision with evidence for what may and may not be shared.

Link the prior artifact rather than copying it. The handoff names its version, evidence status, unresolved risks, owner,
and the next gate. A prose summary or screenshot alone does not establish execution.

## Chapter-specific rubric

| Dimension | Points | Full-credit evidence | Partial-credit signal | No-credit / revision trigger |
| --- | ---: | --- | --- | --- |
| Retrieval and concepts | 20 | Mechanisms, distinctions, boundaries, and consequences are correct | Mostly correct but one boundary is vague | Terms are listed without mechanism or are conflated |
| Mechanism and calculation | 15 | Formula, substitutions, units, exclusions, and checks are correct | Correct method with a minor arithmetic or unit issue | Unsupported number, hidden assumption, or dimension error |
| Evidence evaluation | 15 | Source entails a bounded claim; limitations and needed test are explicit | Relevant source but weak entailment analysis | Fabricated source/result or vendor claim treated as reproduction |
| Engineering decision | 15 | Constraints, alternatives, rejection, gates, and reversal criterion align | Plausible choice with incomplete comparison | Unqualified “best” claim or ignored hard constraint |
| Domain/edge/AWS/safety | 10 | Target-specific transfer preserves offline and escalation requirements | Covers targets but misses one operational boundary | Unsafe recommendation, runtime RAG, or missing critical escalation |
| Engineering artifact | 20 | Reproducible identities, tests, raw evidence, failures, and rollback | Useful design with one material reproducibility gap | Template-only submission or inflated execution status |
| Milestone handoff | 5 | Versioned, linked, owned, and ready for the next gate | Deliverable exists but unresolved work is unclear | Untraceable or copied artifact |

**Pass rule:** 75/100 overall and at least half credit in every row. Fabricated evidence, concealed leakage or failed
runs, prohibited runtime RAG, false `EXECUTED`/`REPRODUCED` status, or an unbounded safety-critical recommendation is
non-compensatory and requires resubmission.

## Further practice

- Explain the chapter’s three key mechanisms aloud in two minutes without notes.
- Change one assumption in the worked example and predict the direction of the result before recalculating.
- Replace the evidence source with a weaker source and identify exactly which claim becomes unsupported.
- Revisit the milestone after the next chapter and record one assumption that changed.

## Governance cross-links

- [Evidence and citation standard](../00-governance/evidence-and-citation-standard.md)
- [Reproducibility standard](../00-governance/reproducibility-standard.md)
- [Safety and risk standard](../00-governance/safety-and-risk-standard.md)
- [No-runtime-RAG standard](../00-governance/no-runtime-rag-standard.md)
- [Assessment index](README.md)
