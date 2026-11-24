# CH-09 assessment: LoRA and Its Evolution

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [Chapter contract and evidence notation](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#chapter-contract-and-evidence-notation)
- [9.11 Practical implementations](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#911-practical-implementations)
- [9.16 Review questions and exercises](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#916-review-questions-and-exercises)
- [9.6 Merging, unmerging, and inference overhead](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#96-merging-unmerging-and-inference-overhead)
- [9.12 Adapter export and release gates](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#912-adapter-export-and-release-gates)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **low-rank update** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** LoRA expresses an update as BA with rank r while freezing W. It constrains adaptation geometry; it does not prove the task update is low rank.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **low-rank update** with **rank and scaling** using one condition where the distinction changes an engineering decision.

**Model answer:** Rank controls trainable capacity; alpha and implementation scaling control update magnitude. Comparing ranks without scaling and target modules is invalid.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **LoRA variant compatibility** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Variant compatibility depends on model modules, initialization, quantization backend, merge semantics, and library implementation; a paper method is not automatically deployable.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** For W in R^(4096x4096), derive A/B shapes and trainable count at r=8 and r=64; compute the scaling for alpha/r and alpha/sqrt(r), then discuss rank collapse.

**Worked example:** For W∈R^(4096×4096), A∈R^(r×4096), B∈R^(4096×r). At r=8, trainable count is 65,536; at r=64, 524,288. Under classic scaling α/r, α=16 gives factors 2 and 0.25 respectively. rsLoRA uses a different denominator and must be labeled.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Compare original LoRA evidence with one variant such as DoRA, PiSSA, EVA, or LoftQ; identify baseline, initialization, quantization backend, and unsupported generalization. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** Compare original LoRA and a variant only with the same base, targets, rank budget, data, schedule, and seeds. A better result after changing initialization and parameter count does not isolate the variant.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Choose target modules, rank allocation, merge policy, and quantization sequence for a 1–3B model destined for GGUF mobile inference. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Target attention and MLP modules only after an inventory; test rank allocation; merge before target quantization only when the backend requires it and equivalence passes.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Analyze whether separate automotive and pharmacy adapters can share a base without cross-domain safety leakage.

**Model guidance:** Separate automotive and pharmacy adapters, route explicitly, and test cross-domain prompts. Patient-specific clinical advice always requires qualified escalation.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A runnable-design adapter config, parameter calculator, module audit, merge-equivalence test, and rank/quality ablation plan.

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

**Deliverable:** A qualified adapter recipe for the flagship case study.

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
