# CH-21 assessment: iOS Deployment

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [21.1 Scope and evidence boundary](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md#211-scope-and-evidence-boundary)
- [21.11 Benchmark, safety, and exercises](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md#2111-benchmark-safety-and-exercises)
- [21.5 Tokenizer, initialization, streaming, and lifecycle](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md#215-tokenizer-initialization-streaming-and-lifecycle)
- [21.9 Minimal offline SwiftUI app](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md#219-minimal-offline-swiftui-app)
- [21.12 Official sources and honest completion](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md#2112-official-sources-and-honest-completion)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **Swift concurrency** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Swift concurrency uses tasks, actors, cancellation, and isolation to keep UI and mutable inference state safe; it does not make native kernels thread-safe automatically.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **Swift concurrency** with **Core ML/Metal/MLX execution** using one condition where the distinction changes an engineering decision.

**Model answer:** Core ML, Metal, MLX Swift, and ExecuTorch expose different graph, quantization, and device paths. Runtime choice is checkpoint- and target-specific.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **app lifecycle and storage** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** App lifecycle and storage govern initialization, backgrounding, memory pressure, signed downloads, activation, rollback, and offline availability.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Build a unified-memory budget including model, KV cache, Metal/Core ML intermediates, and app headroom; explain jetsam risk and context controls.

**Worked example:** With 800 MiB weights, 160 MiB KV, 220 MiB intermediates, 100 MiB runtime, and 300 MiB app headroom, unified-memory demand is 1,580 MiB. A 1.5 GiB budget is short by about 44 MiB; use measured peak allocations because unified memory is shared.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Verify one conversion/runtime path for exact iOS/device versions and separate simulator, Mac, and physical-iPhone evidence. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** A simulator or Apple-silicon Mac run does not prove iPhone execution. Verify conversion, code signing, exact device/OS, peak memory, sustained thermals, and offline behavior.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Choose Core ML, MLX Swift, ExecuTorch, or another supported runtime under App Store, quantization, streaming, and deployment-target constraints. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Choose the runtime whose exact operators and quantization are supported, then isolate mutable generation state in an actor and propagate cancellation.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Design offline pharmacy education UX that cannot imply prescribing and reliably surfaces escalation.

**Model guidance:** Pharmacy UX must not imply diagnosis or prescribing. Interaction, overdose, pregnancy, pediatric, or uncertain cases require pharmacist/clinician or emergency escalation.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A SwiftUI inference architecture with actor/cancellation model, signed artifact installation, streaming, memory-pressure handling, and test plan.

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

**Deliverable:** An iOS qualification package aligned with Android metrics.

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
