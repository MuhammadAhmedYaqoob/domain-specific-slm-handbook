# CH-47 assessment: Proposed Production System

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [47.1 Problem, outcome, and invariants](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md#471-problem-outcome-and-invariants)
- [47.18 Cost and edge trade-offs](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md#4718-cost-and-edge-trade-offs)
- [47.9 Required architecture 7 — evaluation and release approvals](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md#479-required-architecture-7-evaluation-and-release-approvals)
- [47.21 Engineering exercises](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md#4721-engineering-exercises)
- [47.8 Required architecture 6 — cloud-to-edge distribution](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md#478-required-architecture-6-cloud-to-edge-distribution)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **separated architecture planes** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Separated planes distinguish data/training, evaluation/release, serving, update/distribution, and offline inference responsibilities and trust boundaries.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **separated architecture planes** with **immutable release contracts** using one condition where the distinction changes an engineering decision.

**Model answer:** Immutable release contracts bind exact data, model, tokenizer, runtime, evaluation, signatures, compatibility, and rollback identities.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **cloud-edge governance** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Cloud-edge governance keeps AWS and offline artifacts under one approval lineage while allowing distinct formats and operational controls.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Trace one domain release through data, training, evaluation, registry, AWS serving, edge export, activation, monitoring, and retraining; calculate storage/rollback capacity at key boundaries.

**Worked example:** If each edge release is 1.0 GiB and devices retain current, staged, and rollback copies, local capacity is 3.0 GiB plus temporary chunks. Ten retained registry versions need 10 GiB before metadata; lifecycle policy may archive older versions while preserving hashes and manifests.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Audit all seven architecture diagrams and manifests for trust boundaries, mutable identities, unowned transitions, and evidence-status inflation. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** Audit each architecture edge for owner, trust boundary, immutable identity, acceptance test, failure state, and rollback. A diagram arrow without these is not an operational contract.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Evolve a local prototype through multi-domain production while selecting only complexity justified at each stage. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Evolve from local manifests to managed services only when scale and audit needs justify complexity; preserve the same release invariants.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Enforce no-runtime-RAG, domain isolation, safety approval, signed delivery, offline operation, incident response, and rollback end to end.

**Model guidance:** No-runtime-RAG, signed promotion, domain isolation, offline operation, incident response, and rollback are hard invariants. R4 autonomous high-risk use is prohibited; consequential outputs require qualified escalation.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A capstone architecture package with component catalog, flows, state machines, RACI, manifests, threat/failure injection, cost envelope, and acceptance evidence map.

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

**Deliverable:** The final system design and prototype-to-production roadmap.

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
