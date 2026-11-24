# Hands-on experiment learning sequence

## Purpose and entry requirements

This sequence teaches experimental reasoning while progressively eliminating weak or incompatible SLM candidates.
Complete the foundations, data, evaluation, adaptation, compression, and deployment-target readings before running the
corresponding stage. Commands remain `UNVERIFIED IMPLEMENTATION` and trials remain `PLANNED EXPERIMENT` until their
declared environments produce retained evidence.

Each stage starts from an approved plan, names exact resource ceilings, writes one record per attempt, preserves
failures, and ends in `QUALIFY`, `REJECT`, or `INCONCLUSIVE`. A later stage cannot rescue a failed rights or
critical-safety gate. Sealed tests confirm frozen finalists; they do not select them.

## Why this order works

Environment identity comes first so later failures are interpretable. Data and evaluation freeze precedes training so
the target cannot move after results are seen. An untouched baseline establishes whether training helps. SFT/PEFT and
CPT isolate knowledge and behavior changes before compression introduces approximation. Quantization is screened before
more costly pruning and distillation because it often offers the simplest portable memory reduction. Combined pipelines
are attempted only when each component has independent value. Conversion parity precedes cloud and physical-device
benchmarks; final Pareto analysis includes only candidates that pass hard gates.

## Staged sequence

1. **Environment and provenance smoke test**
   - Owner: Reproducibility Engineer. Dependencies: governance G0 and exact candidate identities.
   - Inputs: code commit, lock, container, model/tokenizer revisions, tiny public/approved fixture.
   - Outputs: environment manifest, hashes, install/load/generate/save/resume logs and failure records.
   - Gate: clean rebuild and deterministic fixture pass. Cap: one smallest candidate and one accelerator/CPU smoke; no
     training campaign.

2. **Dataset pipeline and evaluation freeze**
   - Owner: Data Lead; Safety and Rights Reviewers approve.
   - Inputs: acquisition receipts and schemas.
   - Outputs: immutable objective-specific snapshots, root groups, leakage report, coverage strata,
     validation/calibration/sealed manifests.
   - Gate: rights, provenance, deduplication, split isolation, critical-case coverage and reviewer approval. This
     precedes adaptation and baseline prompt inspection beyond approved evaluation operations.

3. **Untouched checkpoint baselines**
   - Owner: Evaluation Lead.
   - Protocol: [baseline evaluation](../06-experiment-framework/protocols/baseline-evaluation.md).
   - Outputs: raw responses, per-item scores, quality/safety/general/OOD profile, source-runtime memory/latency envelope
     and failures.
   - Gate: identity, template, scorer fixtures, no critical safety failure, valid outputs, resource cap. Eliminate
     candidates that cannot plausibly fit any target or meet the validation floor.

4. **SFT baseline**
   - Owner: Training Lead.
   - Inputs: best eligible base candidates and frozen SFT train/validation sets.
   - Outputs: adapter/full child as registered, learning curves, selected checkpoint, lineage and identical evaluation.
   - Gate: domain improvement target, retention/safety margins, stable training and exportable artifact. Cap: one smoke
     seed per candidate, then confirmation seeds only for survivors.

5. **Full FT, LoRA and QLoRA comparison**
   - Owner: Training Lead.
   - Protocol: [training comparison](../06-experiment-framework/protocols/training-comparison.md).
   - Controls: base/data/token order/sequence/effective batch/checkpoint rule/evaluator; declare compute- or
     token-matched budget.
   - Gate: non-inferiority margin plus measured resource advantage and merge/export parity. Infeasible full FT is
     `BUDGET_BLOCKED`, not approximated.

6. **CPT ablation**
   - Owner: Training Lead with Data Lead.
   - Arms: untouched, SFT, CPT diagnostic, CPT→identical SFT; include general replay only as preregistered.
   - Gate: knowledge-heavy improvement under fixed budget without unacceptable forgetting or safety regression. Reject
     CPT if SFT is equivalent within practical indifference.

7. **Quantization screen**
   - Owner: Compression Lead.
   - Protocol: [quantization benchmark](../06-experiment-framework/protocols/quantization-benchmark.md).
   - Sequence: high-precision source → target-supported 8-bit/4-bit methods → lower-bit only for survivors and supported
     kernels.
   - Gate: conversion/load, semantic/safety parity, peak memory, measured target latency and artifact integrity. Cap
     methods by target support; no exhaustive format Cartesian product.

8. **Pruning and recovery screen**
   - Owner: Compression Lead.
   - Protocol: [pruning benchmark](../06-experiment-framework/protocols/pruning-benchmark.md).
   - Arms: dense control, structured/dimension-reducing candidates, sparse candidate only with executable kernel,
     recovery where registered.
   - Gate: quality retention and real target memory/latency benefit after including recovery cost. Nominal zero count is
     not advancement evidence.

9. **Distillation screen**
   - Owner: Distillation Lead; Safety Reviewer approves generated data.
   - Protocol: [distillation benchmark](../06-experiment-framework/protocols/distillation-benchmark.md).
   - Arms: expert-only student, teacher-generated, mixed, soft-target where feasible, distilled→quantized.
   - Gate: smaller student beats its direct-training control on registered utility while preserving safety, rights and
     target fit.

10. **Selected combined pipelines**
    - Owner: Experiment Lead.
    - Protocol: [combined optimization](../06-experiment-framework/protocols/combined-optimization.md).
    - Branches: merge→quantize; prune→recover→quantize; distill→quantize; structured prune→distill/recover only if prior
      stages justify it.
    - Gate: incremental ablation proves each added operation has target value. Stop on dominated, incompatible or
      budget-exceeding branches.

11. **Runtime conversion and semantic parity**
    - Owner: Edge Runtime Lead or Serving Lead by target.
    - Inputs: reviewed survivor artifacts.
    - Outputs: exact format/runtime packages, operator/fallback report, tokenizer/template parity, hashes and rollback
      parent.
    - Gate: load, deterministic known-answer parity, full safety regression and target memory fit. Unsupported
      conversion remains visible.

12. **AWS deployment benchmark**
    - Owner: Cloud Lead; FinOps Reviewer validates allocation.
    - Protocol: [AWS inference benchmark](../06-experiment-framework/protocols/aws-inference-benchmark.md).
    - Gate: exact Region/image/instance tuple passes output parity, offered-load SLO, tails, error, scaling/rollback,
      security and dated allocated-cost gates; teardown evidence required.

13. **Physical edge functional and performance benchmark**
    - Owner: Edge Lead with Android/iOS/platform owner.
    - Protocol: [edge-device benchmark](../06-experiment-framework/protocols/edge-device-benchmark.md).
    - Gate: signed release package on named physical device passes cold/warm/context/lifecycle, peak RAM,
      quality/safety, reliability and offline-update tests. Simulator evidence is functional only.

14. **Sustained energy and thermal confirmation**
    - Owner: Performance Engineer.
    - Inputs: only edge tuples that passed stage 13.
    - Outputs: conditioned repeated 10–30 minute traces, early/late rate, thermal states, energy/token where
      instrumentation is valid, battery and failure data.
    - Gate: sustained—not burst—interaction target and thermal/reliability limits. Missing counters are `null`, never
      zero.

15. **Multi-domain strategy test**
    - Owner: Multi-Domain Lead.
    - Protocol: [multi-domain ablation](../06-experiment-framework/protocols/multi-domain-ablation.md).
    - Compare only justified strategies: independent specialists, shared base/adapters, unified mixture or router.
    - Gate: per-domain non-inferiority, isolation/safety, storage/RAM/switch latency, update and rollback complexity
      under a fixed budget.

16. **Pareto analysis and target selection**
    - Owner: Decision Board; Experiment Lead prepares evidence.
    - Specification: [Pareto analysis](../06-experiment-framework/specifications/pareto-analysis-specification.md).
    - Inputs: reviewed candidates that pass every target hard gate.
    - Outputs: separate fronts per exact AWS/edge target and workload; uncertainty-aware dominance; selections for
      highest quality, mobile, balanced, low memory and cost-efficient AWS, or “no qualified candidate.”
    - Gate: objective registry, units, practical indifference, missing-data policy, sensitivity analysis and approval.
      Never publish a universal winner.

## Resource policy

- **Small budget:** one base size band, SFT/LoRA, one target-native quantizer, one AWS smoke and one available physical
  device; later methods stay not run.
- **Medium budget:** two or three size bands, SFT/LoRA/QLoRA, CPT ablation, limited distillation/pruning, AWS load test
  and Android+iOS confirmation.
- **Research budget:** confirmation seeds, additional quantizers/runtimes/devices, justified combined branches and
  multi-domain ablation.

Exact accelerator-hours, cloud currency and device time are entered in each approved plan; this document fabricates
none. Budget exhaustion yields `INCONCLUSIVE` or `NOT_RUN_BUDGET`.

## Worked illustrative study run

For a constrained first pass, use one exact 1–2B candidate, one rights-approved automotive subset, one SFT LoRA arm,
one target-native 4-bit conversion, one AWS smoke tuple, and one mid-range Android device. Allocate two sessions to
environment and data gates, one to the untouched baseline, two to SFT, one to conversion, one to AWS, two to physical
edge tests, and one to analysis. These session counts are illustrative, not evidence of expected duration.

After each session, replace the remaining duration and compute assumptions with observed setup, training, conversion,
queue, and conditioning times. If the candidate fails rights, critical safety, export, or memory gates, stop and retain
the rejection; do not spend the unused sessions on downstream measurements. Medium and research budgets should add
arms only to answer a named hypothesis, not to fill a Cartesian matrix.

## Stage learning record

For every stage, write: the question, prerequisite evidence, independent variable, controls, resource ceiling, raw
artifact paths, decision rule, result with uncertainty, failure analysis, and permitted next step. The learner passes
the sequence when a reviewer can reconstruct why each branch advanced or stopped and can reproduce at least one
complete path from source model to target-specific decision.

## Cross-links

- [Experiment framework](../06-experiment-framework/README.md)
- [Automotive plan](automotive-baseline-plan.md)
- [Learning dependencies](learning-dependency-map.md)
