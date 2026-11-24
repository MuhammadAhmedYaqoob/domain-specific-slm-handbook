# Pareto analysis lesson

## Question answered

Which qualified candidates are not meaningfully worse on every target objective and meaningfully better on at least
one?

## Why controls matter

Pareto comparison is valid only within one hardware, runtime, workload, power state, intended use, and evaluation
snapshot. Hard gates must be applied before optimization so safety cannot be traded for speed.

## Common novice mistakes

- Combining AWS and edge points on one frontier.
- Normalizing by observed minima and maxima.
- Ranking missing energy as zero.
- Selecting from a two-dimensional plot while ignoring other objectives.

## Complete worked illustrative experiment

For quality (maximize), peak RAM (minimize), and TTFT (minimize):

| Candidate | Quality | RAM GB | TTFT ms |
| --- | ---: | ---: | ---: |
| A | 0.76 | 2.4 | 500 |
| B | 0.75 | 1.6 | 430 |
| C | 0.73 | 2.0 | 550 |

With MIDs 0.01 quality, 0.2 GB RAM, and 30 ms TTFT, B dominates C: quality is better by 0.02, RAM by 0.4 GB, and TTFT
by 120 ms. Neither A nor B dominates the other because A has higher quality while B uses less RAM and is faster.
The illustrative frontier is `{A, B}`. Confidence intervals could make a relation unresolved.

## Fully populated illustrative YAML

```yaml
schema: slm-pareto/v1
status: ILLUSTRATIVE
target_profile: android-midrange-example-v1
input_bundle_sha256: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
hard_gates: [safety, quality_noninferiority, peak_ram, reliability]
objectives:
  - {metric: quality.domain, direction: maximize, mid: 0.01}
  - {metric: memory.peak_runtime_gb, direction: minimize, mid: 0.2}
  - {metric: time.ttft_p95_ms, direction: minimize, mid: 30}
uncertainty: {method: root-and-device-block-bootstrap, confidence: 0.95}
candidates: [A, B, C]
relations: [B-dominates-C, A-unresolved-versus-B]
frontier: [A, B]
decision: preference-review-required
```

## Reusable specification

Freeze objectives, units, directions, MIDs, intervals, gates, optional-objective policy, and input hash. Compute
pairwise `DOMINATES`, `DOMINATED_BY`, `EQUIVALENT_WITHIN_MID`, or `UNRESOLVED`. Use confidence bounds for decision
dominance, publish sensitivity to MIDs and constraints, and apply stakeholder preferences only after extracting the
frontier. Prefer a simpler parent when equivalent within all MIDs.
