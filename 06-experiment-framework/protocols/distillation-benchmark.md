# Distillation benchmark lesson

## Question answered

Does reviewed teacher supervision improve a fixed student over direct expert-data training, and does the advantage survive deployment optimization?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Fix teacher and student identities, evidence roots, prompt generation policy, acceptance rubric, accepted-token budget, student initialization, training schedule, and sealed tests. Teacher output is candidate training data, never ground truth.

## Common novice mistakes

- Allowing sealed questions into teacher prompts.
- Counting generated rather than accepted examples.
- Comparing unequal student architectures.
- Ignoring teacher-error transfer.
- Using soft targets across incompatible tokenizations.

## Complete worked illustrative experiment

A teacher generates 1,000 candidates; reviewers accept 720, edit 80, and reject 200. Raw acceptance yield is `720 / 1000 = 72%`; if edited examples count as usable after re-review, usable yield is `(720 + 80) / 1000 = 80%`. Direct-SFT student scores across three seeds are 0.65, 0.66, 0.64; distilled scores are 0.69, 0.68, 0.67. Paired gains are 0.04, 0.02, and 0.03, with mean 0.03. A new correlated unsafe error found in both teacher and student blocks adoption regardless of that gain.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Freeze independent evidence roots and hide sealed suites.
2. Generate teacher candidates from approved offline evidence.
3. Validate and blind-review every candidate.
4. Build equal-budget expert, teacher, and mixed manifests.
5. Train from the same student initialization with registered seeds.
6. Verify token alignment before any soft-target arm.
7. Evaluate exact students and inspect transferred errors.
8. Quantize only a qualified distilled parent and rerun all gates.

### Metrics and calculations

Measure domain quality versus direct SFT, factual and correlated errors, safety, calibration, retention, accepted yield, coverage, teacher/training cost, expert hours, deployment memory/latency/energy, and quantized-child delta. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
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

[No-runtime-RAG standard](../../00-governance/no-runtime-rag-standard.md) · [Quantization](quantization-benchmark.md)
