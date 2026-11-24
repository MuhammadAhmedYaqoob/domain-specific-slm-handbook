# Flagship Android phone: a worked decision lesson

This profile teaches a target-specific comparison for offline quality-forward inference on a named 12 GiB Android cohort. Every model name and number below is hypothetical and
illustrative; no row is a benchmark result or real recommendation.

## Frame the decision

Freeze the exact device or service cohort, OS/runtime, workload, context, output length, intended use, risk tier,
network and power mode, evidence date, and owners. The illustrative hard constraints are: peak app memory ≤ 6.50 GiB; p95 first token ≤ 1.2 s; sustained decode ≥ 18 token/s; energy ≤ 0.20 J/token; zero critical safety failures. Commercial
rights, complete lineage, no runtime RAG, signed updates, and rollback readiness are also mandatory.

## Compare complete configurations

| Illustrative configuration | Quality | Peak GiB | p95 first-token s | Sustained token/s | J/token | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| A: 2.2B Q4 GPU | 88 | 3.80 | 0.96 | 25.0 | 0.13 | approved | pass |
| B: 4.0B Q4 GPU | 92 | 5.90 | 1.14 | 19.0 | 0.19 | approved | pass |
| C: 3.0B Q6 NPU | 91 | 5.10 | 0.84 | 28.0 | 0.11 | approved | reject: one critical safety failure |

Quality is a fictional 0–100 domain score. Resource and secondary columns use the units named in their headings; a real
record must also preserve measurement conditions and dated price evidence.

## Apply gates before preferences

C: 3.0B Q6 NPU receives the gate result “reject: one critical safety failure.” Its quality, speed, or cost advantages cannot compensate for that outcome.

The rejected or incomplete option does not enter a Pareto front or weighted score. Missing required placement,
licensing, safety, or reliability data is not estimated. If the missing field were optional, the analysis would be run
once without it and again with a conservative adverse case.

## Find the Pareto trade-off

The first two configurations are non-dominated: one offers a different quality/resource trade-off from the other.
Neither is called universally best. Use policy bounds quality 85–95, memory 3–6.5 GiB, latency 0.7–1.2 s. The first and second rows have normalized
quality, resource, and latency utilities of 0.300, 0.771, 0.480 and 0.700, 0.171, 0.120, respectively.

With weights quality 0.65, resource 0.20, and latency 0.15:

$$

U(A)=0.65(0.300)+0.20(0.771)+0.15(0.480)=0.421,

$$

$$

U(B)=0.65(0.700)+0.20(0.171)+0.15(0.120)=0.507.

$$

B narrowly leads for a quality-forward preference; A remains the balanced frontier member. C is ineligible regardless of speed.

## Evidence, licensing, and maintenance lesson

Target execution with raw results and adequate uncertainty is required for release. External documentation can only
prioritize experiments. Review the exact artifact's use, modification, redistribution or hosting, notices, patents,
acceptable-use restrictions, and downstream terms. Separately estimate maintenance from supported releases, custom
kernels, conversion steps, security response, device or instance coverage, observability, and rollback drills.

For example, if A requires 2 engineer-days per quarterly update and B requires 5, annual maintenance is illustratively
$2\times4=8$ versus $5\times4=20$ engineer-days. This may influence preference only after both options pass rights
and ownership gates.

## Interpretation and next decision

B narrowly leads for a quality-forward preference; A remains the balanced frontier member. C is ineligible regardless of speed. A real team would now test weight and threshold sensitivity, confidence intervals, optional energy or cost
data, and exact maintenance assumptions. If the preferred option changes under plausible values, record the decision
as preference-sensitive. If neither passes all gates, choose no deployment rather than relaxing constraints after
seeing results.

## Related material

- [Criteria and measurement](../criteria-and-measurement.md)
- [Pareto selection](../pareto-selection-process.md)
- [Filled decision record](../decision-record-template.md)
- [Compression and edge benchmarking](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
