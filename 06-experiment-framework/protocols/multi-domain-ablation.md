# Multi-domain ablation lesson

## Question answered

Can shared adapters, a joint model, or an offline router reduce portfolio resources without causing any domain to cross its quality or safety margin?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Freeze domain portfolio, per-domain roots and sealed suites, total token/compute budget, sampling weights, base, evaluator, seeds, runtime, switching workload, and update procedure. Every domain remains a separate gate.

## Common novice mistakes

- Hiding a failed domain in a macro average.
- Comparing unequal portfolio budgets.
- Ignoring adapter switch and cache isolation.
- Letting the router guess on unknown inputs.
- Claiming storage savings after materializing full copies.

## Complete worked illustrative experiment

Independent specialists score automotive 0.78 and pharmacy 0.82. A joint model scores 0.77 and 0.78, producing deltas −0.01 and −0.04. With margins of 0.02 and 0.01 respectively, automotive is within margin but pharmacy is not. The joint model fails the intersection–union decision even if installed storage falls from 4.0 to 2.2 GB. A router with 95 correct choices, 3 safe abstentions, and 2 unsafe substitutions has 2 critical routing failures and fails a zero-event gate.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Qualify independent specialists first.
2. Freeze hierarchical sampling and record realized tokens by domain and risk.
3. Train shared and joint arms under one registered budget.
4. Evaluate domains separately and test mixed/wrong-domain conversations.
5. Measure router confusion, abstention, and unsafe substitution.
6. Measure load, switch, residency, and cache isolation.
7. Simulate one-domain update and rollback.
8. Rerun every domain suite on exact deployment packages.

### Metrics and calculations

Report each domain and worst-domain quality/safety, interference, router confusion, storage and memory, switch latency, cache isolation, update regression scope, rollback time, review effort, and portfolio cost. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[Pareto analysis](../specifications/pareto-analysis-specification.md) · [Results guidance](../results/README.md)
