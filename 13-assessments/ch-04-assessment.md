# CH-04 assessment: Pretraining Fundamentals

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [4.1 Purpose and learning outcomes](../02-textbook/part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md#41-purpose-and-learning-outcomes)
- [4.14 PyTorch/Transformers-oriented implementation concepts](../02-textbook/part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md#414-pytorchtransformers-oriented-implementation-concepts)
- [4.22 Engineering exercises](../02-textbook/part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md#422-engineering-exercises)
- [4.9 Accumulation and activation checkpointing](../02-textbook/part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md#49-accumulation-and-activation-checkpointing)
- [4.17 Underfitting, overfitting, and forgetting](../02-textbook/part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md#417-underfitting-overfitting-and-forgetting)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **next-token objective** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** The next-token objective minimizes negative log-probability of the observed successor under a causal mask; it teaches corpus regularities, not verified truth.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **next-token objective** with **optimizer and schedule** using one condition where the distinction changes an engineering decision.

**Model answer:** The optimizer converts gradients into parameter updates; warmup and schedules control update magnitude over time. Lower loss can still coexist with overfitting or unsafe behavior.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **distributed training state** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Distributed state includes parameters, gradients, optimizer moments, scheduler, scaler, RNG, sampler, and consumed-token position. Missing state makes a resume non-equivalent.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Compute cross-entropy and perplexity for a four-token example, then estimate effective batch tokens and optimizer-state memory under stated precision and accumulation assumptions.

**Worked example:** For target probabilities [0.5,0.25,0.8,0.1], loss = -(ln .5+ln .25+ln .8+ln .1)/4 = 1.151 nats/token and PPL=e^1.151=3.16. With microbatch 2, length 2048, 8 workers, and 16 accumulation steps, one update carries 524,288 positions before masking.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Critique a claim that lower training loss proves better domain capability; inspect whether seeds, held-out data, checkpoint selection, and loss spikes are reported. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** A lower training loss is evidence that optimization fit the training stream. It does not establish held-out domain utility unless data lineage, seeds, validation, leakage, and task outcomes are controlled.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Design scratch training versus pretrained adaptation under a fixed GPU-hour budget, including FSDP/ZeRO only if their communication and recovery costs are justified. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Prefer adaptation under a fixed GPU budget unless scratch training closes a demonstrated rights, language, tokenizer, or architecture gap.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Show how packed technical documents affect loss masking, contamination, and AWS checkpoint recovery.

**Model guidance:** Training on safety text does not qualify a model for autonomous pharmaceutical or repair decisions. Release requires independent hazard tests and qualified-human escalation.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A reproducible pretraining experiment plan with frozen model/data IDs, schedule, failure capture, checkpoint cadence, and cost ceiling.

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

**Deliverable:** A validated training contract used by CPT and SFT milestones.

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
