<a id="final-decision-framework"></a>

# Final Engineering Decision Framework

This synthesis converts the book into an accept/revise/reject process. It does not select a model in advance and does
not turn planned evidence into results.

Use it as a tutorial: write the decision first, remove candidates that fail hard gates, compare survivors on target-
specific trade-offs, and retain the reasoning so a future change can reopen the decision.

## 1. Freeze the decision

Name the intended users, allowed and prohibited tasks, domains/languages, highest risk tier, AWS and edge targets,
offline/network assumptions, context/workload, update cadence, budget, and owner. Define measurable hard gates and
optimization objectives before selecting candidates.

## 2. Enforce non-negotiable boundaries

A candidate is ineligible if it requires runtime vector search, dynamic corpus retrieval, retrieved prompt augmentation,
or remote-teacher fallback for primary inference. Domain capability must reside in the selected base/model parameters or
trained adapters.

R4 autonomous prescribing, diagnosis, safety-interlock bypass, or unreviewed physical operation is prohibited. Rights,
privacy, security, and critical R2–R3 safety requirements are hard gates.

## 3. Establish the reference

Freeze model/tokenizer/chat-template revisions, licenses, source/data manifests, uncompressed quality baseline,
domain/general/safety suites, target workloads, exact devices/runtimes, cloud configuration, cost boundary, and
experiment protocol. Without a frozen reference, later “improvement” is not attributable.

## 4. Generate candidates deliberately

Compare the smallest credible set:

- base and instruct checkpoints that meet license and architecture constraints;
- CPT, SFT, preference training, full fine-tuning, and PEFT only where the learning objective requires them;
- independent models, unified model, shared base/adapters, or routing based on isolation and lifecycle needs;
- quantization, pruning, distillation, and architecture changes in controlled stages;
- runtime/backend/device tuples rather than a model name in isolation;
- cloud serving and signed offline-edge release paths.

## 5. Execute staged gates

### Gate A — Data and legal

All sources have provenance, rights, purpose, privacy handling, takedown lineage, and leakage controls. Evaluation
remains sealed.

### Gate B — Quality and retention

Domain quality, factuality, robustness, calibration, general retention, and language/subgroup coverage meet predeclared
thresholds with uncertainty.

### Gate C — Safety

Critical scenario, dangerous-compliance, refusal, escalation, and qualified-review gates pass. This gate is
non-compensatory.

### Gate D — Artifact and compatibility

The exact model/tokenizer/adapter converts reproducibly; signatures/hashes verify; required operators execute on the
intended backend; fallback is understood.

### Gate E — Edge systems

Peak memory, storage, cold/warm TTFT, sustained decode, energy, thermal behavior, stability, and app lifecycle pass on
exact targets.

### Gate F — Cloud systems

Serving correctness, concurrency, latency, availability, authentication, observability, autoscaling, canary, rollback,
and cleanup pass under the declared AWS tuple.

### Gate G — Security and operations

Threat model, supply-chain integrity, secrets/IAM, privacy, monitoring, incidents, rollback/revocation, update,
retention, and decommissioning are owned and tested.

### Gate H — Cost and maintainability

Dated total-cost estimates and available actuals fit the approved envelope. Engineering complexity, review burden,
update isolation, runtime lock-in, and support lifecycle are acceptable.

Any mandatory gate failure removes the candidate; do not average it away.

## 6. Rank survivors by target

Construct separate Pareto sets for each deployment target and risk context. Objectives may include domain score, safety
margin, peak memory, TTFT, decode rate, energy, package size, cloud cost, update size, and operational complexity.
Preserve units, uncertainty, objective direction, and minimum important differences.

One artifact need not win every target. A cloud model, high-end phone model, low-memory model, or domain-specific
adapter set may be selected separately if governance and lifecycle remain coherent.

### Illustrative walkthrough

Assume three candidates have already passed legal and critical-safety gates. The values below are `ILLUSTRATIVE`, not
benchmarks:

- **A:** domain score 88, peak 5.2 GiB, TTFT 1.1 s.
- **B:** domain score 85, peak 2.8 GiB, TTFT 0.8 s.
- **C:** domain score 80, peak 1.6 GiB, TTFT 1.4 s.

For a phone with a 3.0 GiB model-process limit and a minimum score of 83, A fails memory and C fails quality; B is the
only eligible candidate. For AWS, if all fit and quality is the primary objective, A may remain on the Pareto frontier.
This does not make A “best overall.” It shows that target constraints change eligibility before preferences are applied.

Now suppose B fails one critical dangerous-compliance case. B is removed despite its attractive memory and latency.
The correct outcome is **revise** or **reject**, not averaging the failure into its score.

## 7. Prefer the simplest passing architecture

Among practically equivalent survivors, prefer fewer transformations, smaller operational surface, clearer isolation,
broader verified runtime support, easier rollback, and lower lifecycle cost. Novelty is not a criterion.

## 8. Issue the decision record

Record:

- decision and scope;
- selected and rejected exact artifact tuples;
- gate results and evidence links;
- Pareto analysis and policy weights, if any;
- known limitations and unsupported targets;
- safety/domain/legal/security approvers;
- release, monitoring, rollback, and reevaluation triggers;
- expiration/review date.

Possible outcomes are:

- **Accept** — all mandatory gates pass for the named scope.
- **Conditional accept** — only non-release research use is permitted while named evidence closes.
- **Revise** — a remediable gate failed; owner and retest are defined.
- **Reject** — boundary violation, unacceptable risk, or infeasibility.

### Decision record example

`ILLUSTRATIVE`: “Accept B revision `b7` for mid-range Android devices D1–D3 with runtime R 1.4, maximum 2,048-token
context, and educational automotive use only. Reject A for this target because peak memory exceeded the gate. Reject C
because domain quality fell below threshold. Reevaluate after runtime, model, tokenizer, safety suite, or OS changes.”
Real records link every statement to retained evidence and named approvers.

## 9. Reopen decisions when reality changes

Reevaluate after model/tokenizer/data/license changes; runtime/OS/firmware/device changes; new domains or risk tiers;
safety/security incidents; observed drift; material cost changes; or evidence that assumptions were wrong. Never
transfer an old approval to a changed artifact tuple without parity evidence.

## Minimum evidence package

Use the [experiment checklist](appendix-experiment-reporting-checklist.md),
[hardware reference](appendix-hardware-and-memory-reference.md), [figure/table status](figure-index.md),
[limitations](errata-and-limitations.md), CH-31/32 benchmark protocols, CH-42/43 production controls, CH-46 matrix, and
CH-47 architecture record.
