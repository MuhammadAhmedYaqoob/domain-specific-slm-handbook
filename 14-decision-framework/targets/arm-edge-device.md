# ARM edge device: a worked decision lesson

This profile teaches a target-specific comparison for unattended offline inference on a named 8 GiB ARM64 Linux board with active cooling. Every model name and number below is hypothetical and
illustrative; no row is a benchmark result or real recommendation.

## Frame the decision

Freeze the exact device or service cohort, OS/runtime, workload, context, output length, intended use, risk tier,
network and power mode, evidence date, and owners. The illustrative hard constraints are: peak memory ≤ 5.5 GiB; boot-to-ready ≤ 20 s; sustained decode ≥ 10 token/s; watchdog recovery ≤ 60 s; zero critical safety failures. Commercial
rights, complete lineage, no runtime RAG, signed updates, and rollback readiness are also mandatory.

## Compare complete configurations

| Illustrative configuration | Quality | Peak GiB | p95 first-token s | Sustained token/s | J/token | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| A: 1.5B Q4 CPU | 82 | 3.00 | 1.70 | 11.0 | 0.20 | approved | pass |
| B: 2.5B Q4 GPU | 88 | 4.90 | 1.10 | 19.0 | 0.27 | approved | pass |
| C: 2.0B INT8 NPU | 86 | 4.10 | 0.90 | 23.0 | 0.16 | approved | reject: watchdog recovery 94 s |

Quality is a fictional 0–100 domain score. Resource and secondary columns use the units named in their headings; a real
record must also preserve measurement conditions and dated price evidence.

## Apply gates before preferences

C: 2.0B INT8 NPU receives the gate result “reject: watchdog recovery 94 s.” Its quality, speed, or cost advantages cannot compensate for that outcome.

The rejected or incomplete option does not enter a Pareto front or weighted score. Missing required placement,
licensing, safety, or reliability data is not estimated. If the missing field were optional, the analysis would be run
once without it and again with a conservative adverse case.

## Find the Pareto trade-off

The first two configurations are non-dominated: one offers a different quality/resource trade-off from the other.
Neither is called universally best. Use policy bounds quality 78–90, memory 2.5–5.5 GiB, latency 0.8–2.0 s. The first and second rows have normalized
quality, resource, and latency utilities of 0.333, 0.833, 0.250 and 0.833, 0.200, 0.750, respectively.

With weights quality 0.45, resource 0.30, and latency 0.25:

$$

U(A)=0.45(0.333)+0.30(0.833)+0.25(0.250)=0.462,

$$

$$

U(B)=0.45(0.833)+0.30(0.200)+0.25(0.750)=0.622.

$$

B leads the illustrative utility but A remains a lower-memory frontier option. C's unattended recovery failure is non-compensatory.

## Evidence, licensing, and maintenance lesson

Target execution with raw results and adequate uncertainty is required for release. External documentation can only
prioritize experiments. Review the exact artifact's use, modification, redistribution or hosting, notices, patents,
acceptable-use restrictions, and downstream terms. Separately estimate maintenance from supported releases, custom
kernels, conversion steps, security response, device or instance coverage, observability, and rollback drills.

For example, if A requires 2 engineer-days per quarterly update and B requires 5, annual maintenance is illustratively
$2\times4=8$ versus $5\times4=20$ engineer-days. This may influence preference only after both options pass rights
and ownership gates.

## Interpretation and next decision

B leads the illustrative utility but A remains a lower-memory frontier option. C's unattended recovery failure is non-compensatory. A real team would now test weight and threshold sensitivity, confidence intervals, optional energy or cost
data, and exact maintenance assumptions. If the preferred option changes under plausible values, record the decision
as preference-sensitive. If neither passes all gates, choose no deployment rather than relaxing constraints after
seeing results.

## Related material

- [Criteria and measurement](../criteria-and-measurement.md)
- [Pareto selection](../pareto-selection-process.md)
- [Filled decision record](../decision-record-template.md)
- [Compression and edge benchmarking](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
