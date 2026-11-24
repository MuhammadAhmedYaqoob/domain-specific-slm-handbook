# Higher-throughput AWS production deployment: a worked decision lesson

This profile teaches a target-specific comparison for a fictional multi-replica service at 40 offered requests/s in one Region. Every model name and number below is hypothetical and
illustrative; no row is a benchmark result or real recommendation.

## Frame the decision

Freeze the exact device or service cohort, OS/runtime, workload, context, output length, intended use, risk tier,
network and power mode, evidence date, and owners. The illustrative hard constraints are: p99 first token ≤ 1.2 s; successful outputs ≥ 38 requests/s; success rate ≥ 99.5%; quality ≥ 84; zero critical safety failures. Commercial
rights, complete lineage, no runtime RAG, signed updates, and rollback readiness are also mandatory.

## Compare complete configurations

| Illustrative configuration | Quality | Cost units/hour | p95 first-token s | Sustained token/s | Utilization ratio | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| A: 3B Q4, four replicas | 86 | 32.00 | 0.94 | 41.0 | 0.58 | approved | pass |
| B: 5B FP8, four replicas | 91 | 48.00 | 1.08 | 39.0 | 0.77 | approved | pass |
| C: 4B Q4, three replicas | 89 | 30.00 | 0.73 | 44.0 | 0.52 | approved | reject: success rate 98.8% |

Quality is a fictional 0–100 domain score. Resource and secondary columns use the units named in their headings; a real
record must also preserve measurement conditions and dated price evidence.

## Apply gates before preferences

C: 4B Q4, three replicas receives the gate result “reject: success rate 98.8%.” Its quality, speed, or cost advantages cannot compensate for that outcome.

The rejected or incomplete option does not enter a Pareto front or weighted score. Missing required placement,
licensing, safety, or reliability data is not estimated. If the missing field were optional, the analysis would be run
once without it and again with a conservative adverse case.

## Find the Pareto trade-off

The first two configurations are non-dominated: one offers a different quality/resource trade-off from the other.
Neither is called universally best. Use policy bounds quality 82–94, allocated cost 20–50 units/hour, p99 latency 0.6–1.2 s. The first and second rows have normalized
quality, resource, and latency utilities of 0.333, 0.600, 0.433 and 0.750, 0.067, 0.200, respectively.

With weights quality 0.50, resource 0.30, and latency 0.20:

\[
U(A)=0.50(0.333)+0.30(0.600)+0.20(0.433)=0.433,
\]

\[
U(B)=0.50(0.750)+0.30(0.067)+0.20(0.200)=0.435.
\]

B leads a quality-forward calculation; A offers lower cost and better latency. C's attractive throughput and cost cannot compensate for reliability.

## Evidence, licensing, and maintenance lesson

Target execution with raw results and adequate uncertainty is required for release. External documentation can only
prioritize experiments. Review the exact artifact's use, modification, redistribution or hosting, notices, patents,
acceptable-use restrictions, and downstream terms. Separately estimate maintenance from supported releases, custom
kernels, conversion steps, security response, device or instance coverage, observability, and rollback drills.

For example, if A requires 2 engineer-days per quarterly update and B requires 5, annual maintenance is illustratively
\(2\times4=8\) versus \(5\times4=20\) engineer-days. This may influence preference only after both options pass rights
and ownership gates.

## Interpretation and next decision

B leads a quality-forward calculation; A offers lower cost and better latency. C's attractive throughput and cost cannot compensate for reliability. A real team would now test weight and threshold sensitivity, confidence intervals, optional energy or cost
data, and exact maintenance assumptions. If the preferred option changes under plausible values, record the decision
as preference-sensitive. If neither passes all gates, choose no deployment rather than relaxing constraints after
seeing results.

## Related material

- [Criteria and measurement](../criteria-and-measurement.md)
- [Pareto selection](../pareto-selection-process.md)
- [Filled decision record](../decision-record-template.md)
- [Compression and edge benchmarking](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
