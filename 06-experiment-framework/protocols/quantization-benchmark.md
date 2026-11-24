# Quantization benchmark lesson

## Question answered

Which target-supported quantized child preserves quality while measurably reducing storage, memory, latency, energy, or cost?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Use one immutable high-precision parent, one calibration manifest, identical prompts and decoding, and the same target runtime where a quantizer-only claim is intended. A format-plus-runtime change is a deployable-tuple comparison.

## Common novice mistakes

- Equating nominal bit width with actual package size.
- Using sealed tests for calibration.
- Assuming a successful conversion uses accelerated kernels.
- Ignoring metadata and fallback.
- Reporting file shrinkage as latency improvement.

## Complete worked illustrative experiment

An illustrative parent is 2.40 GB and a 4-bit child is 0.78 GB. Storage reduction is `(2.40 - 0.78) / 2.40 = 0.675`, or 67.5%. Parent quality is 0.760 and child quality is 0.748, so `Δ = 0.748 - 0.760 = -0.012`. With a non-inferiority margin of 0.020, the point estimate is inside the margin. Peak RAM falls from 3.2 to 1.6 GB, but median TTFT rises from 480 to 520 ms; quantization improved storage and memory, not latency. Qualification still requires the confidence lower bound above −0.020 and all safety gates.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Review operators and kernels before approving the child matrix.
2. Prove calibration roots are disjoint from sealed evaluation.
3. Convert every arm from the same parent and retain commands and logs.
4. Verify tensors, dtypes, scales, exclusions, and hashes.
5. Run known-token, greedy-parity, nan, and fallback fixtures.
6. Evaluate exact children before performance ranking.
7. Benchmark cold, warm, context, fixed-output, and sustained workloads.
8. Publish parent-paired deltas and target-specific pareto membership.

### Metrics and calculations

Record quality and safety, raw and packaged bytes, effective bits per weight, conversion time and resources, loaded and peak memory, TTFT, prefill/decode rates, energy, thermal behavior, placement, and every crash/OOM/timeout. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[Edge protocol](edge-device-benchmark.md) · [Pareto analysis](../specifications/pareto-analysis-specification.md)
