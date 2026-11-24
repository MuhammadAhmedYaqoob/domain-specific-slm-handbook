# CH-14 assessment: Additional Model Compression Techniques

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [14.1 Status, scope, and evidence boundary](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md#141-status-scope-and-evidence-boundary)
- [14.19 Review questions and exercises](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md#1419-review-questions-and-exercises)
- [14.8 Weight sharing and parameter tying](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md#148-weight-sharing-and-parameter-tying)
- [14.15 Resource and priority synthesis](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md#1415-resource-and-priority-synthesis)
- [14.21 Honest completion record](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md#1421-honest-completion-record)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **low-rank factorization** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Low-rank factorization replaces a matrix with thinner factors to reduce parameters and multiply cost when useful rank is small.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **low-rank factorization** with **architecture shrinking** using one condition where the distinction changes an engineering decision.

**Model answer:** Architecture shrinking removes depth, width, vocabulary, or components. It changes the executable graph and usually requires recovery or retraining.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **weight sharing and coding** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Weight sharing reduces distinct parameters; coding compresses stored symbols. Neither guarantees lower runtime memory or latency without runtime support.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Factor a m×n matrix at rank r and calculate parameter break-even; separately estimate vocabulary/embedding savings and explain why entropy coding changes storage but not dense compute.

**Worked example:** An m×n matrix has mn parameters; rank-r factors have r(m+n). Compression begins when r<mn/(m+n). For m=n=4096, break-even r<2048. At r=256, factors hold 2,097,152 values versus 16,777,216: 8× fewer before metadata.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Classify evidence for early exit, tensor decomposition, model merging, and memory mapping as mature, experimental, or target-dependent. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** Early-exit evidence may be mature for classifiers but experimental for autoregressive SLMs; model merging evidence does not establish safe interpolation; memory mapping affects loading, not arithmetic. Classify each claim by exact workload.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Decide whether to shrink vocabulary, drop layers, factor weights, or train a native small model for a multilingual mobile specialist. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Prefer a native small model when heavy surgery plus recovery exceeds the cost and risk of training/adapting an already supported architecture.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Evaluate domain terminology loss and update/rollback implications of each technique on AWS-to-edge delivery.

**Model guidance:** Vocabulary or layer reduction can erase rare hazard terminology. Re-run domain and escalation tests, preserve rollback, and reject any critical regression.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A feasibility screen with equations, conversion risk, retraining needs, runtime support, and stop conditions for four techniques.

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

**Deliverable:** A ranked backlog for experiments beyond core quantization.

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
