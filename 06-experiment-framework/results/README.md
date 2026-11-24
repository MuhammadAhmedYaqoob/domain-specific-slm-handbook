# How to interpret experiment results

## Question answered

How should an engineer interpret success, a null finding, a regression, or a failed run without claiming that this
project has executed experiments?

## Evidence boundary

No project experiment is represented in this registry. Every number in this lesson is illustrative. A future entry
requires immutable raw attempts, validated schemas, frozen analysis, exact artifact lineage, and completed reviews.

## Why controls matter

An outcome has meaning only relative to a preregistered baseline, margin, target, and failure policy. Controls separate
model effects from prompt, data, runtime, hardware, and thermal effects.

## Common novice mistakes

- Treating “not statistically significant” as “the methods are equal.”
- Calling an execution failure a bad model result without checking the failure class.
- Celebrating one improved metric while a hard gate regresses.
- Generalizing one device result to all devices.

## Four worked illustrative interpretations

### 1. Success

A candidate quality effect is `Δ=+0.030` with 95% CI `[+0.012,+0.048]`; the superiority rule requires the lower bound
above zero and the MID is 0.010. All hard gates pass. Interpretation: this exact candidate is superior on the
registered metric and target. It is not universally best.

### 2. Null or inconclusive finding

The effect is `Δ=+0.008`, CI `[-0.010,+0.026]`. The interval includes harm, no change, and a useful gain. Interpretation:
the experiment does not decide. It does not prove equality. More independent data or a less variable design may be
useful if the decision is worth the cost.

### 3. Regression

Quality improves by 0.020, but critical unsafe events change from `0/200` to `3/200`. The unsafe rate is 1.5%, and a
zero-event gate fails. Interpretation: reject or contain the candidate; average quality cannot compensate.

### 4. Failed run

One of 20 device sessions OOMs during long-context prefill. Success-conditioned TTFT may still be reported for 19
sessions, but reliability is `1 - 1/20 = 95%`, with uncertainty. The failure may reveal target infeasibility,
instrumentation failure, or an infrastructure fault; classify from evidence. Preserve the attempt and follow the
registered retry rule.

## Plain statistical guide

Effect for a higher-is-better metric is `Δ=candidate-baseline`. A 95% confidence interval describes values compatible
with the model and sampling procedure; it is not a 95% probability that a fixed parameter lies inside after the data
are observed. A p-value is the probability, under the null and assumptions, of data at least as incompatible with the
null as observed; it is not the probability that the null is true.

For non-inferiority margin `δ=0.02`, qualify only when the one-sided lower bound of `Δ` exceeds `-0.02`. For equivalence
between `-0.02` and `+0.02`, both bounds must be inside that interval. Failure to find superiority is not equivalence.

## Reusable ingestion and review protocol

1. Verify plan, result, analysis, environment, artifact, and raw-bundle hashes.
2. Reconcile every success, crash, OOM, timeout, invalid, unsupported, and not-run state.
3. Confirm units, denominators, interval methods, missing reasons, and pricing context.
4. Check safety, rights, compatibility, memory, reliability, and intended-use gates before Pareto analysis.
5. Bound the conclusion to the exact domain, artifact, runtime, hardware, workload, and date.
6. Add reviewed entries append-only; corrections supersede and revocations remain visible.

See the [result template](../templates/experiment-result-template.md),
[result schema](../specifications/result-schema-specification.md), and
[Pareto lesson](../specifications/pareto-analysis-specification.md).
