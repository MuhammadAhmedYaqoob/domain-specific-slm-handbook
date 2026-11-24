# Consumer laptop: a worked decision lesson

This profile teaches a target-specific comparison for local inference on a named 16 GiB Windows ultrabook in balanced power mode. Every model name and number below is hypothetical and
illustrative; no row is a benchmark result or real recommendation.

## Frame the decision

Freeze the exact device or service cohort, OS/runtime, workload, context, output length, intended use, risk tier,
network and power mode, evidence date, and owners. The illustrative hard constraints are: peak system memory ≤ 10 GiB; p95 first token ≤ 1.8 s; sustained decode ≥ 12 token/s; battery energy ≤ 0.30 J/token; zero critical safety failures. Commercial
rights, complete lineage, no runtime RAG, signed updates, and rollback readiness are also mandatory.

## Compare complete configurations

| Illustrative configuration | Quality | Peak GiB | p95 first-token s | Sustained token/s | J/token | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| A: 2B Q4 CPU | 85 | 4.80 | 1.55 | 13.0 | 0.24 | approved | pass |
| B: 4B Q4 iGPU | 90 | 7.60 | 1.22 | 21.0 | 0.29 | approved | pass |
| C: 7B Q4 dGPU | 94 | 11.20 | 0.82 | 31.0 | 0.41 | approved | reject: memory and energy |

Quality is a fictional 0–100 domain score. Resource and secondary columns use the units named in their headings; a real
record must also preserve measurement conditions and dated price evidence.

## Apply gates before preferences

C: 7B Q4 dGPU receives the gate result “reject: memory and energy.” Its quality, speed, or cost advantages cannot compensate for that outcome.

The rejected or incomplete option does not enter a Pareto front or weighted score. Missing required placement,
licensing, safety, or reliability data is not estimated. If the missing field were optional, the analysis would be run
once without it and again with a conservative adverse case.

## Find the Pareto trade-off

The first two configurations are non-dominated: one offers a different quality/resource trade-off from the other.
Neither is called universally best. Use policy bounds quality 82–94, memory 4–10 GiB, latency 0.8–1.8 s. The first and second rows have normalized
quality, resource, and latency utilities of 0.250, 0.867, 0.250 and 0.667, 0.400, 0.580, respectively.

With weights quality 0.50, resource 0.25, and latency 0.25:

$$

U(A)=0.50(0.250)+0.25(0.867)+0.25(0.250)=0.404,

$$

$$

U(B)=0.50(0.667)+0.25(0.400)+0.25(0.580)=0.579.

$$

B leads for this plugged-capable balanced profile. C demonstrates that a faster, higher-quality model can still fail portable-resource gates.

## Evidence, licensing, and maintenance lesson

Target execution with raw results and adequate uncertainty is required for release. External documentation can only
prioritize experiments. Review the exact artifact's use, modification, redistribution or hosting, notices, patents,
acceptable-use restrictions, and downstream terms. Separately estimate maintenance from supported releases, custom
kernels, conversion steps, security response, device or instance coverage, observability, and rollback drills.

For example, if A requires 2 engineer-days per quarterly update and B requires 5, annual maintenance is illustratively
$2\times4=8$ versus $5\times4=20$ engineer-days. This may influence preference only after both options pass rights
and ownership gates.

## Interpretation and next decision

B leads for this plugged-capable balanced profile. C demonstrates that a faster, higher-quality model can still fail portable-resource gates. A real team would now test weight and threshold sensitivity, confidence intervals, optional energy or cost
data, and exact maintenance assumptions. If the preferred option changes under plausible values, record the decision
as preference-sensitive. If neither passes all gates, choose no deployment rather than relaxing constraints after
seeing results.

## Related material

- [Criteria and measurement](../criteria-and-measurement.md)
- [Pareto selection](../pareto-selection-process.md)
- [Filled decision record](../decision-record-template.md)
- [Compression and edge benchmarking](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
