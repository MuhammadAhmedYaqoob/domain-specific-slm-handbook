<a id="errata-limitations"></a>

# Errata, Unresolved Research, and Limitations

**Register date:** 2026-11-23  
**Scope:** textbook research draft  
**Authority:** the [open-issues ledger](../../15-audit-ledgers/open-issues-and-omissions.md), readiness ledger, and
claim/source registers control closure.

Read this page before treating a chapter design as a demonstrated system. A limitation tells you what inference is not
licensed by the current evidence; the research priorities tell you how to close that gap.

## Current limitations

### Execution and reproduction

- The repository contains extensive designs, listings, protocols, and research synthesis, but a listing is not proof of
  execution.
- No blanket claim is made that training, conversion, AWS deployment, mobile integration, physical-device benchmarks,
  energy tests, or end-to-end release drills have run.
- Paper- and vendor-reported results are not independently reproduced unless a claim explicitly carries `REPRODUCED`
  with the required trace.
- The CH-45/46 research program remains largely `PLANNED EXPERIMENT`; it must not be summarized as findings.

### Figures and tables

- FIG-13–FIG-15 are protocols for project-generated quality trade-off charts, not measured findings.
- TBL-21–TBL-23 define how to report performance, energy, and quality retention without supplying invented values.
- CH-47’s seven architecture diagrams use stable identities `FIG-27`–`FIG-33`.
- See the honest [figure index](figure-index.md) and [table index](table-index.md).

### Compatibility and hardware

- Runtime, format, operator, precision, and accelerator support changes quickly. Documentation evidence does not prove
  that a project artifact converts, loads, avoids fallback, or meets performance targets.
- CPU/GPU/NPU behavior varies by device SKU, SoC, OS, firmware, driver, SDK, runtime, backend, model graph, workload,
  thermal state, and application integration.
- Peak TOPS/FLOPS and bandwidth specifications do not predict application speed.
- Target-device RAM, TTFT, decode, energy, thermal, stability, and quality-parity gates remain open until exact-device
  execution.

### Models, data, and licenses

- Model cards, licenses, access terms, revisions, and artifact availability may change after the cutoff.
- “Public” sources are not automatically licensed for training. Source-specific rights and provenance review remains
  mandatory.
- Deduplication and contamination checks reduce but cannot prove the absence of evaluation leakage.
- Synthetic data can amplify teacher errors, narrow coverage, and obscure provenance; qualified validation remains
  necessary.
- Parameter training does not provide exact, current, or complete knowledge storage. Rare facts, conflicting sources,
  dates, and numeric details remain failure-prone.

### Quality and safety

- Aggregate benchmark scores do not establish safe use in automotive, pharmacy, animal-care, industrial, electrical, or
  other high-consequence workflows.
- Domain suites cannot cover all real-world contexts, combinations, adversarial prompts, languages, or vulnerable
  groups.
- Refusal and escalation behavior may regress after adaptation, compression, conversion, runtime changes, or
  prompt-template changes.
- R4 autonomous prescribing/operation remains prohibited. R3 behavior requires qualified professional verification and
  non-compensatory gates.
- Offline inference limits fresh external fact access; safe abstention is required where currency is essential.

### Cloud, cost, and operations

- AWS services, quotas, regions, instance availability, APIs, security guidance, and pricing change. Revalidate at
  deployment time.
- Cost equations and sample budgets are estimates unless tied to billed/allocated actuals. Engineering, data rights,
  expert review, support, and device lifecycle costs are easy to omit.
- Monitoring on privacy-sensitive edge deployments is inherently partial. Absence of telemetry is not evidence of
  absence of failure.
- Signed delivery and rollback designs need end-to-end drills, including interruption, low storage, incompatibility,
  revocation, and offline recovery.

### Architectural limitation

The project intentionally excludes runtime RAG and remote-teacher fallback from primary inference. This simplifies
offline independence and narrows the attack/operations surface, but can reduce factual freshness, citation traceability,
and long-tail recall. The design must not conceal those trade-offs or reintroduce retrieval under another name.

## Unresolved research priorities

1. Which CPT/SFT/preference sequence produces the best domain gain without unacceptable general or safety regression?
2. When do full tuning, LoRA/QLoRA, and newer variants differ materially after controlled compute/data matching?
3. Which compression order preserves critical domain behavior on each exact runtime/device?
4. How do weight and KV-cache precision jointly affect long-context quality, peak memory, energy, and thermal
   throttling?
5. Which pruning structures yield real target acceleration rather than sparse artifacts only?
6. Which distillation signals transfer rare domain and abstention behavior into smaller students?
7. When do shared bases/adapters reduce lifecycle cost without unsafe cross-domain interference or routing errors?
8. How stable are human and model-based evaluators across domain, language, and generated-answer style?
9. What minimum physical-device protocol predicts sustained real application behavior?
10. How should freshness, abstention, and update cadence be balanced under the no-runtime-RAG constraint?
11. What privacy-preserving telemetry is sufficient for edge drift and incident detection?
12. How sensitive is total cost to expert review, failed experiments, device fragmentation, and support duration?
13. Can calibrated uncertainty survive adaptation, quantization, conversion, and runtime-specific numerics?
14. Which safety tests are predictive of real near misses rather than benchmark-specific compliance?
15. Which target-specific Pareto candidates remain non-dominated after uncertainty and minimum-important-difference
    rules?

The detailed hypotheses and matrix are
[CH-45](../part-13-experimental-research-roadmap/ch-45-research-questions-and-hypotheses.md) and
[CH-46](../part-13-experimental-research-roadmap/ch-46-benchmark-matrix.md).

## Errata log

Corrections are recorded in the controlling issue ledger with affected edition, evidence, severity, owner, and release.
The absence of a listed correction is not evidence that the text has no errors.

For a new erratum, record date, reporter, affected file/anchor and edition, original text, correction, severity,
technical/safety impact, evidence, owner, status, and release included. Never silently overwrite a safety- or
result-affecting correction.

## Release warning

This edition is a research draft. Publication or production claims require the
[definition of done](../../00-governance/definition-of-done.md), resolved links/IDs, completed evidence traces,
qualified reviews, and explicit release approval.
