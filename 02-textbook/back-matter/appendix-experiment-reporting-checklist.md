<a id="experiment-checklist"></a>

# Appendix C — Experiment Reporting Checklist

Use this checklist for every training, compression, quality, safety, systems, cost, or reproduction claim. A checked
item needs a retained artifact or explicit not-applicable reason.

Start before running the experiment. The checklist is most valuable when it prevents an ambiguous result, not when it
is reconstructed afterward.

## Miniature example

`ILLUSTRATIVE`: “INT4 reduces peak memory by at least 25% without more than a two-point domain-score drop versus FP16 on
device D.” The independent variable is precision; dependent variables are peak memory and domain score; model, prompts,
runtime, device state, and decoding are controlled. The 25% and two-point limits are decision thresholds, not results.
Pre-register how failed runs and uncertainty will be handled, then retain raw profiler output and per-item scores.

## Question and protocol

- [ ] Stable experiment ID, owner, date, hypothesis, and decision it informs.
- [ ] Baseline, independent/dependent variables, controls, confounders, and exclusions.
- [ ] Primary/secondary metrics with units and direction.
- [ ] Hard gates, minimum important differences, stop/failure criteria, and analysis plan fixed before results.
- [ ] Evidence label and promotion criteria declared.

## Model and software

- [ ] Base, tokenizer, chat template, adapters, checkpoints, converted artifacts, and hashes.
- [ ] Licenses and usage constraints reviewed.
- [ ] Source revision, dirty-state/diff, configuration, command, seeds, and environment variables without secrets.
- [ ] OS, container digest, Python/compiler, libraries, runtime/backend, drivers/firmware, and lockfiles.
- [ ] Determinism settings and known nondeterministic operations.

## Data

- [ ] Source and dataset snapshot IDs/hashes, rights/provenance, transformation lineage, and schema.
- [ ] Train/validation/test counts in records and tokens; sampling/mixing/packing policy.
- [ ] Split unit and rationale; exact/near-duplicate and contamination analysis.
- [ ] PII/secrets, safety, quality, language, domain, and subgroup checks.
- [ ] Synthetic generator/prompt/revision, sampling, filtering, verification, and synthetic fraction.
- [ ] Sealed evaluation access controls and reviewer qualifications.

## Training and transformation

- [ ] Objective, masks, sequence length, batch/accumulation, optimizer, schedule, precision, clipping, and
      regularization.
- [ ] Trainable/frozen modules, PEFT rank/scaling/targets, quantization/pruning/distillation configuration.
- [ ] Checkpoint/resume behavior, early stopping, failures/restarts, and total tokens/steps.
- [ ] Hardware topology, parallelism/sharding/offload/checkpointing, runtime, wall time, utilization, and resource
      usage.
- [ ] Every candidate preserves parent artifact lineage.

## Quality and safety

- [ ] Prompt/reference/scorer fixtures and judge model/version are frozen.
- [ ] Decoding parameters and repetition count are reported.
- [ ] Domain, general retention, factuality, calibration, robustness, refusal, escalation, dangerous compliance, and
      subgroup/scenario metrics.
- [ ] Human rubric, blinding, adjudication, inter-rater treatment, and conflicts of interest.
- [ ] R2–R3 critical failures listed individually; R4 release prohibition enforced.

## Edge and serving systems

- [ ] Exact device/SKU/SoC, OS/firmware, runtime/backend, conversion tool, model format, and operator/fallback report.
- [ ] Cold/warm state, prompt/output lengths, batch/concurrency, context, sampling, warm-up, repetitions, and run order.
- [ ] Artifact size, loaded and peak memory, TTFT, inter-token/decode throughput, end-to-end latency, error rate, and
      output parity.
- [ ] Ambient/cooling, battery/power source, initial temperature, screen/radio/background state, duration, thermals,
      throttling, and energy boundary.
- [ ] Cloud instance/service/region, autoscaling/concurrency, network boundary, availability, and billed/estimated
      duration.

## Statistics

- [ ] Sample size and unit of analysis.
- [ ] Raw distributions, central tendency, dispersion, interval method, and paired/unpaired design.
- [ ] Multiple-comparison handling where applicable.
- [ ] Missing, failed, timed-out, and excluded trials retained with reasons.
- [ ] Practical significance reported separately from statistical significance.

## Cost and environmental accounting

- [ ] Price source, region/currency, date, discounts/commitments/taxes assumptions, and rate hashes/snapshots.
- [ ] Training, serving, storage, transfer, logging, data, review, engineering, device, and lifecycle boundaries.
- [ ] Estimates visibly separated from invoices/allocated actuals; sensitivity included.
- [ ] Energy method, boundary, sample interval, uncertainty, and idle baseline recorded.

## Artifacts and review

- [ ] Raw immutable logs/results, parsed dataset, analysis code, plots, and report are linked and hashed.
- [ ] No manually copied number is the sole source of a result.
- [ ] Figure/table captions state metric, unit, target, workload, uncertainty, and evidence.
- [ ] Reproduction instructions start from a clean environment and identify unavailable prerequisites.
- [ ] Independent technical, evidence, safety, and domain reviews recorded as applicable.
- [ ] Negative/null results and unresolved anomalies are preserved.
- [ ] Decision record states accept/revise/reject and why.

## Minimum report header

Record: experiment ID; status; evidence label; model/data/config hashes; code/container revision; hardware/runtime;
start/end time; hypothesis; metrics/gates; result artifact URI/hash; failures; reviewers; and decision.

Cross-links: [CH-29](../part-08-evaluation-benchmarking-and-research-experiments/ch-29-model-quality-evaluation.md),
[CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md),
[CH-32](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md),
[reproducibility standard](../../00-governance/reproducibility-standard.md).
