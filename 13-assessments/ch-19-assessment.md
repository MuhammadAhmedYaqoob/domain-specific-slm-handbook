# CH-19 assessment: Mobile Inference Optimization

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [19.1 The objective is a usable product, not a high peak number](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#191-the-objective-is-a-usable-product-not-a-high-peak-number)
- [19.3 KV-cache mechanics and memory](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#193-kv-cache-mechanics-and-memory)
- [19.28 Sources and evidence notes](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#1928-sources-and-evidence-notes)
- [19.11 Continuous batching and batch-size trade-offs](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#1911-continuous-batching-and-batch-size-trade-offs)
- [19.26 Exercises and laboratory work](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#1926-exercises-and-laboratory-work)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **prefill versus decode** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Prefill processes the prompt and builds cache; decode generates one token against that cache. They have different arithmetic and bandwidth profiles.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **prefill versus decode** with **KV-cache growth** using one condition where the distinction changes an engineering decision.

**Model answer:** KV-cache growth is linear in layers, batch, cached tokens, KV heads, head width, and bytes per element.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **thermal-aware sustained tuning** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Thermal-aware tuning optimizes sustained—not burst—throughput, energy, and user experience under device temperature limits.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Derive KV-cache bytes from layers, KV heads, head dimension, context, batch, and precision; compare MHA/GQA/MQA and a halved context.

**Worked example:** For L=24, B=1, S=4096, Hkv=8, dh=64, FP16, KV=2×24×1×4096×8×64×2=201,326,592 bytes=192 MiB. INT8 cache is 96 MiB before metadata. Halving context also halves this payload.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Critique speculative-decoding or GPU-offload speed claims for workload, acceptance rate, draft cost, transfer overhead, and sustained temperature. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** A speculative-decoding speedup without acceptance rate, draft cost, transfer overhead, prompt/output lengths, and sustained temperature cannot be generalized to a phone.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Tune threads, context, mmap, cache precision, and offload for a single mobile user rather than importing server batching advice. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Tune threads, context, mmap, cache precision, and offload one factor at a time for a single-user workload; server batching assumptions do not transfer.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Define separate short diagnostic and long manual-summary workloads on Android/iOS, with battery and safety-output checks.

**Model guidance:** Never optimize by truncating warnings or refusals. Hazardous diagnostic requests must preserve complete escalation language.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A controlled optimization sweep reporting TTFT, prefill/decode, peak RAM, energy/token, temperature, throttling, and failures.

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

**Deliverable:** A sustained mobile configuration with a documented Pareto choice.

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
