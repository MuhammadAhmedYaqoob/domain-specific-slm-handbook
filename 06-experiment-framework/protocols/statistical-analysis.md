# Statistical analysis lesson

## Question answered

How large and uncertain is the candidate-versus-baseline effect, and does it satisfy a preregistered superiority,
non-inferiority, or equivalence rule?

## Why controls matter

Statistics cannot repair a biased experiment. Pairing the same item roots, blocking the same devices, freezing
exclusions, and choosing the hypothesis before seeing outcomes prevent ordinary variation and analyst choice from
becoming a false discovery.

## Common novice mistakes

- Treating repeated tokens, paraphrases, or turns as independent samples.
- Reading a p-value as the probability that the null hypothesis is true.
- Calling a non-significant difference “equivalent.”
- Deleting slow thermal runs or failed attempts as outliers.
- Choosing a favorable test after inspecting the results.

## Complete worked illustrative experiment

Suppose 100 paired questions give 12 candidate-only successes and 5 baseline-only successes. The paired accuracy
difference is `(12 - 5) / 100 = 0.07`. McNemar's continuity-corrected statistic is
`χ² = (|12 - 5| - 1)² / (12 + 5) = 36/17 = 2.118`; this does not by itself establish a prespecified 5% superiority
claim. Report the 7-point effect and its paired interval, not only a p-value.

For non-inferiority, let candidate quality be 0.742, baseline quality 0.750, and margin `δ=0.020`.
`Δ = 0.742 - 0.750 = -0.008`. If the one-sided 95% lower confidence bound is −0.017, then
`−0.017 > −0.020`, so non-inferiority passes. If the lower bound were −0.026, the result would be inconclusive or fail
according to the plan. These are illustrative values, not project results.

A mean and sample standard deviation for values 10, 12, and 14 are `x̄=12` and
`s=sqrt(((10-12)²+(12-12)²+(14-12)²)/(3-1))=2`. The standard error is `s/sqrt(3)=1.155`; with only three independent
runs, uncertainty is large.

## Reusable protocol

1. Copy the estimand, null, alternative, metric, MID, analysis unit, pairing key, strata, exclusions, and confidence
   level from the approved plan.
2. Verify hashes, schema, row uniqueness, units, pairing coverage, and blind state.
3. Run known-answer fixtures for metrics, intervals, multiplicity, and Pareto dominance.
4. Lock the input bundle and execute immutable analysis code.
5. For binary paired items use an exact paired or McNemar method; for clustered scores use a root-clustered bootstrap
   or justified hierarchical model; for device trials use blocks or mixed effects.
6. Report raw counts, effect, interval, assumptions, denominator, and practical margin.
7. Apply Holm adjustment to a small confirmatory family. Label exploratory analyses and require fresh confirmation.
8. Preserve failures and missingness; show conditional-on-success latency only beside failure rate.

For a sample proportion `p̂=x/n`, a plain-language 95% Wilson interval is preferred to `p̂ ± 1.96 SE` near zero or one.
Zero observed failures is not zero risk: with zero failures in 30 trials, the “rule of three” gives an approximate
upper 95% risk bound of `3/30=10%`.

Decision words have precise meanings: `SUPERIOR` clears the superiority boundary; `NON_INFERIOR` clears `−δ`;
`EQUIVALENT` clears both equivalence bounds; `INCONCLUSIVE` means the evidence does not decide. Statistical success
never overrides safety or compatibility.

See [result schema](../specifications/result-schema-specification.md) and
[metric specification](../specifications/metric-specification.md).
