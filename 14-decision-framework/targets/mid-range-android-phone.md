# Mid-range Android phone: a worked decision lesson

This profile teaches a target-specific comparison for offline inference on a named 8 GiB Android cohort. Every model name and number below is hypothetical and
illustrative; no row is a benchmark result or real recommendation.

## Frame the decision

Freeze the exact device or service cohort, OS/runtime, workload, context, output length, intended use, risk tier,
network and power mode, evidence date, and owners. The illustrative hard constraints are: peak app memory ≤ 4.00 GiB; p95 first token ≤ 1.5 s; sustained decode ≥ 12 token/s; thermal retention ≥ 80%; zero critical safety failures. Commercial
rights, complete lineage, no runtime RAG, signed updates, and rollback readiness are also mandatory.

## Compare complete configurations

| Illustrative configuration | Quality | Peak GiB | p95 first-token s | Sustained token/s | J/token | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| A: 1.3B Q4 CPU | 83 | 2.55 | 1.31 | 14.0 | 0.10 | approved | pass |
| B: 2.2B Q4 GPU | 88 | 3.72 | 1.18 | 18.0 | 0.15 | approved | pass |
| C: 2.8B Q4 NPU | 90 | 3.88 | 1.02 | 22.0 | 0.09 | approved | reject: thermal retention 68% |

Quality is a fictional 0–100 domain score. Resource and secondary columns use the units named in their headings; a real
record must also preserve measurement conditions and dated price evidence.

## Apply gates before preferences

C: 2.8B Q4 NPU receives the gate result “reject: thermal retention 68%.” Its quality, speed, or cost advantages cannot compensate for that outcome.

The rejected or incomplete option does not enter a Pareto front or weighted score. Missing required placement,
licensing, safety, or reliability data is not estimated. If the missing field were optional, the analysis would be run
once without it and again with a conservative adverse case.

## Find the Pareto trade-off

The first two configurations are non-dominated: one offers a different quality/resource trade-off from the other.
Neither is called universally best. Use policy bounds quality 80–92, memory 2–4 GiB, latency 0.8–1.5 s. The first and second rows have normalized
quality, resource, and latency utilities of 0.250, 0.725, 0.271 and 0.667, 0.140, 0.457, respectively.

With weights quality 0.55, resource 0.25, and latency 0.20:

$$

U(A)=0.55(0.250)+0.25(0.725)+0.20(0.271)=0.373,

$$

$$

U(B)=0.55(0.667)+0.25(0.140)+0.20(0.457)=0.493.

$$

B receives the larger illustrative score and trades memory for quality. C cannot compensate for failing sustained thermal behavior.

## Evidence, licensing, and maintenance lesson

Target execution with raw results and adequate uncertainty is required for release. External documentation can only
prioritize experiments. Review the exact artifact's use, modification, redistribution or hosting, notices, patents,
acceptable-use restrictions, and downstream terms. Separately estimate maintenance from supported releases, custom
kernels, conversion steps, security response, device or instance coverage, observability, and rollback drills.

For example, if A requires 2 engineer-days per quarterly update and B requires 5, annual maintenance is illustratively
$2\times4=8$ versus $5\times4=20$ engineer-days. This may influence preference only after both options pass rights
and ownership gates.

## Interpretation and next decision

B receives the larger illustrative score and trades memory for quality. C cannot compensate for failing sustained thermal behavior. A real team would now test weight and threshold sensitivity, confidence intervals, optional energy or cost
data, and exact maintenance assumptions. If the preferred option changes under plausible values, record the decision
as preference-sensitive. If neither passes all gates, choose no deployment rather than relaxing constraints after
seeing results.

## Related material

- [Criteria and measurement](../criteria-and-measurement.md)
- [Pareto selection](../pareto-selection-process.md)
- [Filled decision record](../decision-record-template.md)
- [Compression and edge benchmarking](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
