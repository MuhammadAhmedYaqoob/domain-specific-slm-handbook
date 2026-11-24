# CH-08 assessment: Full Fine-Tuning Versus PEFT

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [8.1 Status, purpose, and learning outcomes](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md#81-status-purpose-and-learning-outcomes)
- [8.3 Common mathematics and accounting](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md#83-common-mathematics-and-accounting)
- [8.12 Implementation patterns](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md#812-implementation-patterns)
- [8.22 Review questions and exercises](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md#822-review-questions-and-exercises)
- [8.9 P-Tuning and P-Tuning v2](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md#89-p-tuning-and-p-tuning-v2)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **trainable parameter scope** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Trainable parameter scope identifies which weights receive gradients. Freezing most weights reduces optimizer state but not necessarily activation or base-weight memory.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **trainable parameter scope** with **adapter methods** using one condition where the distinction changes an engineering decision.

**Model answer:** Adapters, prompt/prefix methods, IA3, and BitFit alter different parameter locations and capacities; “PEFT” is not one mechanism.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **memory accounting** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Memory accounting separates weights, gradients, optimizer state, activations, temporary buffers, and quantization overhead. Trainable-parameter percentage alone is not VRAM.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** For a declared linear-layer inventory, compute trainable parameters for full FT, BitFit, and LoRA; separate weights, gradients, optimizer states, and activations.

**Worked example:** For one 4096×4096 linear layer, full FT trains 16,777,216 weights. BitFit trains 4,096 bias terms if present. LoRA rank 8 trains r(in+out)=65,536 parameters, 0.391% of the matrix. Add every targeted layer before comparing totals.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Critique a VRAM comparison that changes batch size, sequence length, precision, or checkpoint between methods. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** A VRAM chart is non-entailing if full FT and LoRA use different sequence lengths, microbatches, precision, or checkpointing. Normalize the workload and report peak telemetry.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Select full FT, LoRA, prefix tuning, or IA3 for four domains sharing one base on AWS with separately downloadable mobile artifacts. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Use separate adapters where domain isolation and updates matter; merge only after parity, license, rollback, and cross-domain safety tests.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Explain adapter isolation and merging risks for pharmacy versus low-risk terminology adapters.

**Model guidance:** A pharmacy adapter must not inherit unsafe behavior from another domain. Keep explicit routing, per-adapter hazard tests, and qualified escalation for clinical content.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A matched experiment plan comparing full FT and two PEFT methods, including memory telemetry, quality/forgetting gates, and artifact lineage.

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

**Deliverable:** A training-method ADR feeding CH-09 and CH-27.

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
