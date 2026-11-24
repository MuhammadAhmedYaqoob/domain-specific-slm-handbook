# CH-27 assessment: Single-Domain Versus Multi-Domain Strategies

## How to use this learning assessment

Use it twice: first closed-book for retrieval practice, then open-book to correct the answer in a different color or
commit. A correct answer names the mechanism, its boundary, and one engineering consequence. Revisit missed items after
2 days, 1 week, and 3 weeks; do not merely reread the model answer.

## Exact chapter reading map

- [27.0 Chapter contract, status, and evidence notation](../02-textbook/part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#270-chapter-contract-status-and-evidence-notation)
- [27.21 Review questions and engineering exercises](../02-textbook/part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#2721-review-questions-and-engineering-exercises)
- [27.8 Strategy 7 — multi-task instruction tuning](../02-textbook/part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#278-strategy-7-multi-task-instruction-tuning)
- [27.16 Licensing, access control, removal, and provenance](../02-textbook/part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#2716-licensing-access-control-removal-and-provenance)
- [27.23 Honest completion status and cross-links](../02-textbook/part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#2723-honest-completion-status-and-cross-links)

## 1. Retrieval practice — 20 points

### A1 — 5 points

**Question:** Define **independent models** in the chapter's terms. What problem does it solve, and what does it *not* establish?

**Model answer:** Independent models maximize domain isolation but duplicate weights, deployment, and maintenance.

**Why common distractors fail:** A definition based only on a product name, historical date, or claimed advantage omits
the mechanism and its boundary.

### A2 — 5 points

**Question:** Contrast **independent models** with **shared-base adapters** using one condition where the distinction changes an engineering decision.

**Model answer:** Shared-base adapters reuse one immutable base and load domain-specific updates. They reduce storage only if adapter routing and base compatibility are maintained.

**Why common distractors fail:** The two ideas may interact, but treating them as synonyms hides which design variable
changed and prevents a controlled comparison.

### A3 — 5 points

**Question:** Explain how **multi-domain interference** operates and identify one failure mode that a superficial implementation would miss.

**Model answer:** Multi-domain interference occurs when shared parameters or routing cause one domain’s data to degrade or contaminate another domain’s behavior.

**Why common distractors fail:** A mechanism-level answer cannot infer quality, speed, safety, or compatibility from
its label alone; those outcomes require target-specific evidence.

### A4 — 5 points

**Question:** Connect all three concepts in one causal chain. State one tempting but invalid inference at each boundary.

**Solution guidance:** Link the three mechanisms in order: input or design choice → internal effect → measurable product
outcome. At each arrow, name the tempting invalid inference. In particular, do not infer factual correctness from
fluency, runtime support from format availability, or deployment readiness from one aggregate metric.

## 2. Worked mechanism or calculation — 15 points

**Problem:** Calculate installed storage and peak switch memory for independent models versus one base plus adapters; include base duplication, merge artifacts, and cache reset.

**Worked example:** Three independent 1.0 GiB models use 3.0 GiB installed. One 1.0 GiB base plus three 60 MiB adapters uses 1.176 GiB, saving 1.824 GiB. During a switch, loading old and new 60 MiB adapters simultaneously adds 120 MiB unless unloading is atomic.

**Check your work (5 points):**

1. Write the formula before substituting values.
2. Label units and distinguish decimal GB from GiB where relevant.
3. State exclusions such as metadata, padding, allocator fragmentation, or review labor.
4. Test one boundary case and one failure case.
5. Label invented inputs `ILLUSTRATIVE` and forecasts `ESTIMATE`; reserve `EXECUTED` for retained run evidence.

## 3. Evidence-evaluation example — 15 points

**Task:** Evaluate claims that adapter sharing improves cost without measuring interference, switching latency, or safety isolation. Submit a claim-evidence table containing the atomic claim, exact scope, evidence label, source ID and locator, verification date, limitation, and a corrected claim if the original overreaches. A source's presence is not enough: explain entailment.

**Worked critique:** A storage-saving claim is incomplete without interference tests, switch latency, resident memory, license compatibility, and per-domain safety regressions.

For full credit, rewrite the overbroad claim into one that the cited source actually entails. Record the exact artifact
or population, source locator, evidence status, verification date, and the experiment still needed. A citation is not
evidence of entailment when its model, workload, hardware, jurisdiction, or outcome differs.

## 4. Engineering decision — 15 points

**Scenario:** Choose architecture for automotive, pharmacy, beekeeping, and zookeeping across mobile and AWS with different update cadences. Name hard constraints, alternatives rejected, quality/safety gates, operational burden, cost boundary, unknowns, and the experiment that would reverse the decision. Do not use “best,” “supported,” or “production-ready” without a comparison set and qualifying evidence.

**Model decision guidance:** Use independent pharmacy isolation unless shared-base evidence passes strict confusion and safety gates; adapters are more plausible for closely related mechanic domains.

Show hard constraints first, then viable alternatives, rejected alternatives, uncertainties, acceptance gates, and the
measurement that would reverse the decision. “Best,” “supported,” and “production-ready” require a declared comparison
set and qualifying evidence.

## 5. Domain, edge, AWS, and safety transfer — 10 points

**Application:** Design routing and isolation so pharmacy requests cannot invoke an unsafe mechanic adapter or mixed weights.

**Model guidance:** Routing failures must fail closed. Pharmacy prompts may never invoke mechanic weights, and all clinical content requires qualified escalation.

Identify what belongs to training/evaluation, the signed update plane, and local or cloud inference. The primary
specialist path must not retrieve domain prose at runtime. A safe answer states limits, asks for missing context when
appropriate, refuses operationally dangerous detail, and escalates consequential decisions to a qualified person.

## 6. Engineering exercise walkthrough — 20 points

**Assignment:** A weighted strategy matrix plus interference experiment, switch-latency protocol, licensing analysis, and rollback topology.

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

**Deliverable:** A multi-domain architecture ADR feeding CH-28.

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
