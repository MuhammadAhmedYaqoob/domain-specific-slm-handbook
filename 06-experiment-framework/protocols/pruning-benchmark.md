# Pruning benchmark lesson

## Question answered

Does pruning create a physically executable child that improves a target metric after accounting for recovery cost?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Keep parent, calibration roots, ratio, recovery budget, evaluator, runtime, and workload fixed. Retain dense parent, unrecovered child, recovered child, and a similarly sized native model.

## Common novice mistakes

- Counting zero weights as a smaller dense artifact.
- Claiming speedup without sparse-kernel evidence.
- Showing only the recovered child.
- Hiding failed recovery seeds.
- Confusing parameter reduction with lower peak memory.

## Complete worked illustrative experiment

A structured child removes 20% of MLP width. Domain score falls from 0.75 to 0.66, then recovery raises it to 0.73. Recovery regained `(0.73 - 0.66) / (0.75 - 0.66) = 0.7778`, or 77.8% of lost score. Decode rises from 18 to 21 token/s, a `(21 - 18) / 18 = 16.7%` gain. If the quality margin is 0.03, the point difference of −0.02 is acceptable, but qualification requires interval and safety gates. An unstructured 30% mask with unchanged 18 token/s is not an acceleration result.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Validate architecture invariants and removable units.
2. Score the dense parent and dense-zero control.
3. Compute decisions from pruning calibration only.
4. Materialize and serialize the graph and verify dimensions.
5. Evaluate the unrecovered child.
6. Recover under frozen tokens, compute, and seeds.
7. Export through the same target path and prove operator placement.
8. Benchmark randomized blocks and analyze ratio-by-recovery interaction.

### Metrics and calculations

Report domain non-inferiority, safety, original and final shapes, logical and stored parameters, mask/index bytes, artifact and memory changes, prefill/decode, energy/thermal behavior, failures, recovery resources, and export burden. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[Combined optimization](combined-optimization.md) · [Metric specification](../specifications/metric-specification.md)
