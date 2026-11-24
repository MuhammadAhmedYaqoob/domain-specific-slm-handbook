# Edge-device benchmark lesson

## Question answered

Does an exact model–runtime package meet quality, RAM, interaction latency, sustained performance, reliability, and energy gates on a named physical device?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Physical state can dominate the model effect. Control device SKU, OS, battery and charging, ambient/start temperature, case, display/radios, background work, cache state, prompt tokens, output length, thread settings, and candidate order.

## Common novice mistakes

- Using an emulator for performance claims.
- Mixing charging and battery trials.
- Treating tokens within one run as independent.
- Omitting thermal tails.
- Using battery percentage as joules.

## Complete worked illustrative experiment

Five illustrative sustained sessions begin at 20 token/s and end at 17, 18, 16, 17, and 17 token/s. Using session means, late rate is 17 and `sustained retention = 17 / 20 = 0.85`, or 85%. A 500 J incremental energy boundary over 2,000 generated tokens gives `500 / 2000 = 0.25 J/token`. If one session OOMs, reliability is 4/5=80%; the OOM cannot be removed from the denominator.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Verify package hashes, signature, template, and semantic parity.
2. Condition the physical device to the registered state.
3. Start monotonic event, memory, placement, power, and thermal traces.
4. Retain declared warm-ups.
5. Randomize ab/ba order within device/day blocks.
6. Run cold, warm, context, lifecycle, and sustained workloads.
7. Restore the start-state band between independent sessions.
8. Capture raw traces, exits, inventory, and deviations.

### Metrics and calculations

Separate first model token from visible TTFT. Report artifact/install bytes, loaded/peak memory, startup, prefill/decode, inter-token and end-to-end latency, early/late rate, valid energy boundary, thermal traces, and crash/OOM/timeout counts. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[Environment specification](../specifications/hardware-and-environment-specification.md) · [Metrics](../specifications/metric-specification.md)
