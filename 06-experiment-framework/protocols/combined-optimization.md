# Combined optimization lesson

## Question answered

Does an ordered sequence such as distill→quantize improve the target Pareto frontier beyond each simpler parent?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Order is part of identity. Preserve every intermediate and the four factorial cells—neither, A only, B only, A+B—when claiming interaction. Freeze data, budgets, seeds, evaluator, runtime, hardware, and workload.

## Common novice mistakes

- Testing only the final child.
- Changing calibration after pruning.
- Continuing after a hard-gate failure.
- Calling a smaller but slower child better.
- Ignoring lifecycle and conversion cost.

## Complete worked illustrative experiment

Illustrative quality scores are `Y0=0.74`, `YA=0.76` after distillation, `YB=0.73` after quantization, and `YAB=0.755`. The interaction is `IAB=(0.755-0.76)-(0.73-0.74)=0.005`: the combination performs 0.5 percentage points better than additive expectation. It is still not preferred if peak RAM and latency are equivalent to A within their MIDs, because A is simpler.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Approve the lineage graph and direct controls.
2. Reproduce each isolated qualified parent.
3. Apply exactly one transformation at a time.
4. Validate identity, generation, quality, and safety at every node.
5. Stop failed branches but retain evidence.
6. Measure incremental compute, storage, review, and dependency burden.
7. Export finalists to exact targets.
8. Compare final children with every direct parent and native-small control.

### Metrics and calculations

Use the full quality, safety, storage, memory, latency, energy, reliability, cost, and operational-complexity portfolio. Analyze parent-paired effects, factorial interactions, seeds, and blocked system trials. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[Statistics](statistical-analysis.md) · [Pareto analysis](../specifications/pareto-analysis-specification.md)
