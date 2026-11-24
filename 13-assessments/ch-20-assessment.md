# CH-20 assessment: Android Deployment

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [20.1 Learning outcomes and evidence boundary](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#201-learning-outcomes-and-evidence-boundary)
- [20.24 Exercises](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#2024-exercises)
- [20.9 Replacing the base model with a domain-trained model](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#209-replacing-the-base-model-with-a-domain-trained-model)
- [20.18 Crash handling and diagnostics](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#2018-crash-handling-and-diagnostics)
- [20.26 Honest completion record](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#2026-honest-completion-record)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **JNI/runtime integration** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** JNI/runtime integration connects Kotlin/Java lifecycle code to native inference safely; it does not prove ABI compatibility, memory safety, or performance.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **JNI/runtime integration** with **lifecycle-safe streaming** using one condition where the distinction changes an engineering decision.

**Model answer:** Lifecycle-safe streaming binds generation to cancellable state and prevents stale callbacks after an Activity, ViewModel, or process transition.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **signed offline model delivery** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Signed offline delivery verifies publisher identity and manifest integrity before atomic activation; a checksum alone does not authenticate the publisher.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Budget APK/AAB, downloaded artifact, mapped weights, KV cache, and native heap; diagnose a sample OOM using lifecycle and context assumptions.

**Worked example:** Example active budget: 100 MiB app/native libraries + 780 MiB mapped weights + 192 MiB KV + 180 MiB native workspace + 300 MiB reserve = 1,552 MiB. Under a 1.4 GiB allowance, shortfall is about 118 MiB; reduce context/precision or reject the device, not the reserve.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Verify Gradle/native/runtime APIs and ABI support from current official sources; distinguish static review from emulator or physical-device execution. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** Official Gradle or ABI documentation proves configuration availability. Only a release build on a named physical device proves load and generation; an emulator cannot establish thermals or NPU behavior.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Choose bundled versus post-install model delivery, CPU/GPU/NPU path, and minimum SDK/device policy. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Prefer post-install delivery for large updateable artifacts, verify signatures before activation, keep the previous version, and ensure generation is cancelled across lifecycle changes.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Design an offline mechanic chat that preserves warnings, cancellation, integrity checks, and update rollback.

**Model guidance:** An offline mechanic app must stop and escalate brake, steering, high-voltage, fuel, lifting, or fire-risk cases; incomplete streamed warnings must never be presented as complete advice.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A minimal Kotlin/JNI application design with model manifest, checksum/signature verification, streaming state machine, cancellation, and device test matrix.

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

**Deliverable:** An Android release candidate package awaiting physical-device evidence.

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
