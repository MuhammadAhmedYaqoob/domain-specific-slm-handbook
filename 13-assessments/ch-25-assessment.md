# CH-25 assessment: Dataset Design for Different Training Objectives

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [25.1 Chapter contract and evidence boundary](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md#251-chapter-contract-and-evidence-boundary)
- [25.14 Evaluation and benchmarking datasets](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md#2514-evaluation-and-benchmarking-datasets)
- [25.26 Review questions and engineering exercises](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md#2526-review-questions-and-engineering-exercises)
- [25.10 Preference-optimization datasets](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md#2510-preference-optimization-datasets)
- [25.19 Safety, privacy, rights, and expert review](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md#2519-safety-privacy-rights-and-expert-review)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **objective-specific schemas** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Objective-specific schemas preserve the fields and loss semantics needed by CPT, SFT, preference, distillation, calibration, pruning, and evaluation.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **objective-specific schemas** with **role-separated splits** using one condition where the distinction changes an engineering decision.

**Model answer:** Role-separated splits prevent one record, lineage, answer, or derived variant from serving incompatible training and evaluation roles.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **rare-case coverage** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Rare-case coverage deliberately includes consequential low-frequency cases; oversampling improves test power but must not be reported as real prevalence.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Given a corpus and target percentages, allocate CPT, SFT, preference, calibration, pruning, and evaluation records while respecting group-level leakage constraints.

**Worked example:** From 100,000 eligible records, an illustrative allocation might reserve 70,000 CPT, 15,000 SFT, 5,000 preference pairs, 2,000 quantization calibration, 2,000 pruning calibration, and 6,000 sealed evaluation records. These are disjoint by lineage, and percentages sum to 100%. Change them from task needs, not convention.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Critique reuse of evaluation or quantization-calibration data as training; identify exactly which conclusions become invalid. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** Reusing evaluation items in training invalidates generalization claims; reusing calibration data is acceptable only for its declared method and must not leak test answers.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Balance common tasks against rare safety cases without making prevalence claims from intentionally oversampled tests. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Create immutable lineage IDs first, then materialize objective views. Oversample rare hazards for learning/testing while reporting natural-distribution and risk-weighted results separately.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Create objective views for automotive diagnosis, pharmacy escalation, and general retention with shared lineage but separate roles.

**Model guidance:** High-risk objective views must encode abstention and qualified escalation. Evaluation safety cases remain sealed and non-compensatory.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A schema pack with example records, validators, group-aware split policy, coverage matrix, difficulty labels, and contamination tests.

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

**Deliverable:** Frozen dataset manifests for training and evaluation.

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
