# Pareto selection as a decision lesson

All candidates and values here are hypothetical. The lesson shows why a frontier should precede a weighted score.

## Step 1: freeze the comparison

Define the target, intended use, exact workload, gates, objectives, directions, confidence level, meaningful-difference
bands, missing-data policy, and preference rule before viewing final results. Keep different hardware, runtime,
context, concurrency, power mode, and cost dates in separate comparison strata.

## Step 2: gate the options

Assume all four options pass rights, safety, compatibility, memory, and reliability gates. Their illustrative point
estimates are:

| Option | Quality, higher is better | Peak GiB, lower is better | p95 latency s, lower is better | J/token, lower is better |
| --- | ---: | ---: | ---: | ---: |
| A | 82 | 2.2 | 1.8 | 0.09 |
| B | 85 | 2.8 | 2.0 | 0.12 |
| C | 80 | 2.4 | 2.1 | 0.11 |
| D | 84 | 2.6 | 1.7 | unknown |

A dominates C: it has higher quality and lower memory, latency, and energy. C leaves the frontier. Neither A nor B
dominates the other because B buys quality with more resources. D cannot enter a four-objective release frontier until
required energy is measured. If energy is optional, show a three-objective front and an adverse-case sensitivity
analysis rather than imputing a value.

## Step 3: account for uncertainty and meaningful differences

Point estimates can overstate certainty. If A quality is 82 with a 95% interval of 80–84 and D is 84 with an interval
of 82–86, D is not conclusively better. With a predeclared two-point minimum important difference, the intervals do not
establish an improvement beyond that difference. Classify the relation as unresolved, not equal.

A release frontier uses conservative dominance: confidence bounds must establish no meaningful loss on every objective
and a meaningful gain on at least one. Publish the point-estimate front as a diagnostic when it differs.

## Step 4: apply preferences only to the frontier

For A and B, normalize with frozen bounds: quality 75–90, memory 2–3 GiB, latency 1–2.5 s, energy 0.05–0.15 J/token.

| Option | Quality utility | Memory utility | Latency utility | Energy utility |
| --- | ---: | ---: | ---: | ---: |
| A | 0.467 | 0.800 | 0.467 | 0.600 |
| B | 0.667 | 0.200 | 0.333 | 0.300 |

With weights quality 0.50, memory 0.20, latency 0.20, energy 0.10:

$$

U(A)=0.50(0.467)+0.20(0.800)+0.20(0.467)+0.10(0.600)=0.547,

$$

$$

U(B)=0.50(0.667)+0.20(0.200)+0.20(0.333)+0.10(0.300)=0.470.

$$

A is preferred under those values. If quality weight rises to 0.80 and the other three weights become 0.08, 0.08,
and 0.04, $U(A)=0.494$ and $U(B)=0.584$; B is preferred. The recommendation is therefore
**preference-sensitive**. This is useful information: stakeholders must choose whether seven-tenths of a GiB and
additional latency/energy are worth three quality points.

## Step 5: name a role and preserve alternatives

Do not announce a universal winner. A may be the balanced deployment, B the quality-forward deployment, and D a awaiting_review
experiment. Record rejected and dominated options, decisive assumptions, license obligations, maintenance ownership,
monitoring, expiry, and rollback. If no candidate clears every gate, “no qualified candidate” is the valid outcome.

## Reproducible procedure

1. Validate complete configuration identities and preserve every failure.
2. Apply hard gates and retain reason codes.
3. Build fronts in original units within comparable strata.
4. Recompute using confidence bounds and meaningful-difference bands.
5. Test thresholds, optional metrics, prices, maintenance judgments, and weights.
6. Select a declared role from the robust frontier or request more evidence.
7. Complete the [decision record](decision-record-template.md).

## Common mistakes

Scoring before gating, normalizing to observed extremes, treating missing values as zero, deleting failed trials,
pooling unlike conditions, and selecting weights after seeing results all make the conclusion look more precise than
the evidence permits.
