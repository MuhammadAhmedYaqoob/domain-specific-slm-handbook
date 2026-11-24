# Low-end Android phone: a worked decision lesson

This profile teaches a target-specific comparison for offline single-user inference on a named 4 GiB ARM64 Android cohort. Every model name and number below is hypothetical and
illustrative; no row is a benchmark result or real recommendation.

## Frame the decision

Freeze the exact device or service cohort, OS/runtime, workload, context, output length, intended use, risk tier,
network and power mode, evidence date, and owners. The illustrative hard constraints are: peak app memory ≤ 2.50 GiB; p95 first token ≤ 2.0 s; sustained decode ≥ 8 token/s; install package ≤ 1.2 GiB; zero critical safety failures. Commercial
rights, complete lineage, no runtime RAG, signed updates, and rollback readiness are also mandatory.

## Compare complete configurations

| Illustrative configuration | Quality | Peak GiB | p95 first-token s | Sustained token/s | J/token | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| A: 0.8B Q4 CPU | 79 | 1.82 | 1.72 | 10.4 | 0.08 | approved | pass |
| B: 1.3B Q4 CPU | 84 | 2.43 | 1.91 | 8.6 | 0.12 | approved | pass |
| C: 1.1B Q4 accelerator | 86 | 2.21 | 1.44 | 13.0 | 0.07 | unresolved redistribution | reject: license |

Quality is a fictional 0–100 domain score. Resource and secondary columns use the units named in their headings; a real
record must also preserve measurement conditions and dated price evidence.

## Apply gates before preferences

C: 1.1B Q4 accelerator receives the gate result “reject: license.” Its quality, speed, or cost advantages cannot compensate for that outcome.

The rejected or incomplete option does not enter a Pareto front or weighted score. Missing required placement,
licensing, safety, or reliability data is not estimated. If the missing field were optional, the analysis would be run
once without it and again with a conservative adverse case.

## Find the Pareto trade-off

The first two configurations are non-dominated: one offers a different quality/resource trade-off from the other.
Neither is called universally best. Use policy bounds quality 75–90, memory 1.5–2.5 GiB, latency 1–2 s. The first and second rows have normalized
quality, resource, and latency utilities of 0.267, 0.680, 0.280 and 0.600, 0.070, 0.090, respectively.

With weights quality 0.50, resource 0.30, and latency 0.20:

$$

U(A)=0.50(0.267)+0.30(0.680)+0.20(0.280)=0.394,

$$

$$

U(B)=0.50(0.600)+0.30(0.070)+0.20(0.090)=0.339.

$$

A is the illustrative balanced preference; B remains the quality-forward frontier option. C is rejected before scoring despite stronger numbers.

## Evidence, licensing, and maintenance lesson

Target execution with raw results and adequate uncertainty is required for release. External documentation can only
prioritize experiments. Review the exact artifact's use, modification, redistribution or hosting, notices, patents,
acceptable-use restrictions, and downstream terms. Separately estimate maintenance from supported releases, custom
kernels, conversion steps, security response, device or instance coverage, observability, and rollback drills.

For example, if A requires 2 engineer-days per quarterly update and B requires 5, annual maintenance is illustratively
$2\times4=8$ versus $5\times4=20$ engineer-days. This may influence preference only after both options pass rights
and ownership gates.

## Interpretation and next decision

A is the illustrative balanced preference; B remains the quality-forward frontier option. C is rejected before scoring despite stronger numbers. A real team would now test weight and threshold sensitivity, confidence intervals, optional energy or cost
data, and exact maintenance assumptions. If the preferred option changes under plausible values, record the decision
as preference-sensitive. If neither passes all gates, choose no deployment rather than relaxing constraints after
seeing results.

## Related material

- [Criteria and measurement](../criteria-and-measurement.md)
- [Pareto selection](../pareto-selection-process.md)
- [Filled decision record](../decision-record-template.md)
- [Compression and edge benchmarking](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
