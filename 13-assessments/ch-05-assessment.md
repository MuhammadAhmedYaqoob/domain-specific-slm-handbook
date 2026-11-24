# CH-05 assessment: Continued Pretraining and Domain Adaptation

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [5.1 Purpose and learning outcomes](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md#51-purpose-and-learning-outcomes)
- [5.13 Checkpoint evaluation and selection](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md#513-checkpoint-evaluation-and-selection)
- [5.17 Implementation patterns](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md#517-implementation-patterns)
- [5.23 Engineering exercises](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md#523-engineering-exercises)
- [5.9 Preserving capabilities and mitigating forgetting](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md#59-preserving-capabilities-and-mitigating-forgetting)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **DAPT and TAPT boundaries** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** DAPT continues the language-model objective on broad domain text; TAPT uses unlabeled target-task inputs. Neither is supervised answer training.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **DAPT and TAPT boundaries** with **general-data replay** using one condition where the distinction changes an engineering decision.

**Model answer:** General-data replay mixes retained capabilities into adaptation updates. It may reduce forgetting, but spends token budget and requires lawful representative data.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **catastrophic forgetting** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Catastrophic forgetting is a material loss of prior behavior after adaptation. It can affect general language, chat control, or safety even while domain loss improves.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Given 60M domain tokens and a 30/70 replay mixture, calculate tokens drawn from each source over 1.5 effective epochs and propose domain/general perplexity gates.

**Worked example:** At 30% replay over 1.5 epochs of 60M domain tokens, domain draws are 90M tokens and represent 70% of the mixture. Total update tokens are 90M/0.70=128.571M, so replay draws are 38.571M. State whether “1.5 epochs” refers to domain exposure, not total mixture passes.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Evaluate whether a cited CPT result used a base or instruction checkpoint, compatible domain, tokenizer, and compute budget; reject causal claims unsupported by ablation. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** A CPT result on an encoder, another domain, or an instruction checkpoint is not direct evidence for a decoder-only base. Correct transfer requires matching objective, checkpoint state, data scale, and evaluation.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Choose CPT+SFT or SFT-only for sparse beekeeping data and large automotive manuals under the same compute ceiling. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Use SFT-only for a small repetitive beekeeping corpus unless a baseline gap justifies CPT; test CPT plus replay for large lawful automotive manuals against the same SFT.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Define separate automotive, pharmacy, and general-retention probes, including escalation behavior for unsafe forgotten knowledge.

**Model guidance:** Unsafe diagnostic, interaction, or treatment requests must trigger bounded guidance and expert escalation; a lower domain perplexity never overrides a critical regression.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A CPT decision memo plus mixture manifest, leakage audit, learning-rate sweep, forgetting dashboard, and stop rules.

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

**Deliverable:** A go/no-go CPT gate before CH-06 instruction tuning.

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
