# Training-method comparison lesson

## Question answered

Does SFT, CPT→SFT, full fine-tuning, LoRA, or QLoRA improve domain capability enough to justify its resources without harming safety or general capability?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Training arms are causal only when they share the same parent, data roots, accepted-token budget, checkpoint rule, evaluator, seed policy, and final export contract. Otherwise the result compares whole products, not training methods.

## Common novice mistakes

- Giving one arm more tokens or hyperparameter search.
- Selecting checkpoints on the test set.
- Calling lower training loss better domain performance.
- Discarding divergent seeds.
- Comparing an adapter with an unverified merged export.

## Complete worked illustrative experiment

Three illustrative seeds yield domain scores of 0.71, 0.73, 0.72 for SFT and 0.74, 0.75, 0.73 for CPT→SFT. Seed-paired differences are 0.03, 0.02, and 0.01; their mean is `(0.03 + 0.02 + 0.01) / 3 = 0.02`. If the superiority MID is 0.015, the point estimate is practically meaningful, but three seeds alone do not prove superiority; the registered hierarchical interval must also clear zero. If retention changes by −0.06 against a −0.03 floor, the arm fails even if domain quality rises.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Audit rights, leakage, split roots, and exact token budgets.
2. Run the untouched parent and trainer fixtures.
3. Smoke-test one registered seed per arm while preserving failures.
4. Train qualified arms with immutable configurations and checkpoints.
5. Select checkpoints using validation only.
6. Verify adapter-versus-merge parity.
7. Evaluate and export the exact final artifacts.
8. Archive logs, telemetry, failed seeds, and costs.

### Metrics and calculations

Report paired domain quality, safety and retention gates, perplexity under one tokenizer, calibration, convergence, trainable parameters, peak VRAM, accelerator-hours, wall time, checkpoint bytes, restart cost, and dated allocated cost. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[Provenance](../specifications/artifact-and-provenance-specification.md) · [Statistics](statistical-analysis.md)
