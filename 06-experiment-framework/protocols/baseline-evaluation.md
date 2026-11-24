# Baseline evaluation lesson

## Question answered

What quality, safety, and deployment envelope does an untouched checkpoint establish before specialization?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

The baseline is the reference point for every later claim. Identical prompts, decoding, item roots, scorer, runtime, and hardware prevent a prompt edit or faster backend from being mistaken for model improvement.

## Common novice mistakes

- Tuning the prompt after seeing sealed answers.
- Comparing checkpoints with different generation limits.
- Reporting only successful requests.
- Treating an untouched instruction model as a domain-trained model.

## Complete worked illustrative experiment

Compare untouched Model A and Model B on 100 automotive items. A answers 62 correctly and B answers 68. The paired records contain 12 items that only B gets right and 6 that only A gets right, so the paired gain is `(12 - 6) / 100 = 0.06`, or 6 percentage points. B also has 2 critical unsafe answers while the registered gate is zero. The illustrative conclusion is therefore **reject B**, despite its higher average accuracy. A latency block has TTFT values 410, 430, 390, 450, and 420 ms; the sorted middle value is 420 ms. These are teaching numbers, not project measurements.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Approve and hash the plan and rights review.
2. Resolve model, tokenizer, and chat template by immutable revision.
3. Run tokenizer, stop-token, finite-logit, and known-answer scorer fixtures.
4. Generate every sealed item once under the registered policy and retain raw output.
5. Score without exposing references to the model.
6. Run cold, warm, context-boundary, and fixed-output system trials.
7. Freeze records and run paired, root-aware analysis.

### Metrics and calculations

Primary outcomes are domain score and critical-safety failures. Secondary outcomes include unsupported-claim rate, retention, calibration, artifact bytes, peak memory, startup, visible TTFT, prefill and decode rates, end-to-end latency, and failure rate. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
`100 × (candidate − baseline) / baseline`. Failure rate is `failures / attempted`, never failures/successes.

### Repetition and statistics

Use one smoke attempt only to find defects. Use preregistered independent seeds for training and independent
device/host sessions for systems claims. Pair quality items by source root. Report effect sizes and confidence
intervals; use multiplicity control for a confirmatory family. An interval that crosses the decision boundary is
`INCONCLUSIVE`, not evidence of no effect.

### Failure and decision rules

Retain OOM, crash, timeout, invalid output, unsupported, and budget-stopped attempts. Use `null` with a reason for an
unavailable measurement; zero means measured zero. Qualify only an exact target tuple that passes every hard gate and
the registered practical-effect rule.

## Further reading

[Statistics](statistical-analysis.md) · [Metric specification](../specifications/metric-specification.md)
