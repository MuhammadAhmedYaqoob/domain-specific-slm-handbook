# Criteria, measurement, and scoring lessons

All values in this chapter are illustrative. They teach arithmetic and decision logic; they are not benchmark claims.

## Hard constraints and objectives

Hard constraints are binary release conditions. A candidate must have approved commercial rights, no prohibited
critical safety failures, complete lineage, target compatibility, and measurements inside resource and reliability
limits. Objectives are quantities to optimize only among candidates that pass every gate.

| Measure | Direction | Typical decision role |
| --- | ---: | --- |
| Domain accuracy and reasoning | maximize | quality floor and objective |
| Unsupported-claim rate | minimize | safety/quality cap |
| Critical safety failures | minimize to zero when required | non-compensatory gate |
| Peak runtime memory | minimize | cap and objective |
| p95 time to first token | minimize | interaction cap and objective |
| Sustained decode rate | maximize | floor and objective |
| Energy per generated token | minimize | edge objective or gate |
| Cost per accepted output | minimize | cloud objective |
| Commercial rights | pass | non-compensatory gate |
| Maintenance burden | minimize | ownership gate and objective |

Each observation must bind the complete configuration, target, workload, trial, instrument, unit, date, and raw result.
Cross-tokenizer performance uses equal rendered text because token counts differ.

## Worked gate example

Assume a low-memory target requires domain accuracy at least 78, zero critical failures, peak memory no more than
2.50 GiB, p95 first-token latency no more than 2.00 s, and commercial redistribution rights.

| Illustrative option | Accuracy | Critical failures | Peak GiB | p95 s | Rights | Gate result |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| Cedar-1 Q4 | 80 | 0 | 2.30 | 1.80 | approved | pass |
| Cedar-2 Q4 | 86 | 1 | 2.40 | 1.70 | approved | reject: safety |
| Cedar-1 Q8 | 83 | 0 | 3.10 | 1.60 | approved | reject: memory |

Cedar-2 Q4 cannot compensate for its critical failure with six extra accuracy points. Cedar-1 Q8 cannot compensate for
excess memory with lower latency. Only Cedar-1 Q4 proceeds to preference analysis.

## Normalization without result-dependent bounds

For a maximize objective with policy bounds \(L\) and \(U\),

\[
u_{\max}(x)=\operatorname{clip}\left(\frac{x-L}{U-L},0,1\right).
\]

For a minimize objective,

\[
u_{\min}(x)=\operatorname{clip}\left(\frac{U-x}{U-L},0,1\right).
\]

The utility is \(U(x)=\sum_j w_j u_j(x)\), with non-negative weights summing to one. Bounds are frozen from target
policy, not selected from observed minima and maxima.

For illustrative Cedar-1 Q4, use accuracy bounds 75–90, memory bounds 2.0–3.0 GiB, and latency bounds 1.0–2.5 s:

- accuracy utility: \((80-75)/(90-75)=5/15=0.333\);
- memory utility: \((3.0-2.3)/(3.0-2.0)=0.700\);
- latency utility: \((2.5-1.8)/(2.5-1.0)=0.467\).

With weights 0.50, 0.30, and 0.20, the score is
\(0.50(0.333)+0.30(0.700)+0.20(0.467)=0.470\). The result is a preference score, not a probability, safety rating, or
universal quality measure.

## Evidence confidence

Use five confidence levels: target-confirmed measurements; close-target measurements; current external evidence;
calculated estimates; and unknown or conflicting evidence. Confidence is limited by the weakest material element:
identity, target match, protocol, sample adequacy, recency, or reviewer independence.

An example explains the difference. A runtime manual stating that an operator is supported is current external
evidence. A successful load test is target-confirmed compatibility evidence. Neither proves sustained thermal
performance; that requires its own target test.

## Missing and uncertain data

Required data is never imputed. If energy is required but unmeasured, the option is incomplete. If energy is optional,
record it as unknown and compute two analyses: one without energy and one assigning the option a conservative adverse
case. A sensor below 0.01 J resolution supports “less than 0.01 J,” not zero.

Suppose options A and B score 0.72 and 0.70 without optional energy. A lacks energy data; B measures 0.08 J/token. If
A’s plausible range is 0.05–0.14 J/token, it wins under the favorable assumption and loses under the adverse one.
The correct conclusion is preference-sensitive, followed by an energy experiment—not a fabricated 0.00 J/token.

## Licensing and maintenance are different decisions

Licensing is normally a gate: review the exact artifact and terms for use, modification, redistribution, hosting,
notices, patents, acceptable-use limits, and downstream obligations. Maintenance measures future effort: supported
dependencies, security response, custom patches, reproducible builds, conversion complexity, device coverage,
observability, rollback drills, and support horizon.

Illustratively, option L costs 4 engineer-days per quarterly update and has approved redistribution; option M costs
1 day but its redistribution terms are unresolved. M is rejected, not rewarded for lower maintenance. If both licenses
pass, annual effort could be compared as \(4\times4=16\) versus \(1\times4=4\) engineer-days, with uncertainty stated.

## Measurement discipline

Freeze workloads and thresholds before final testing; retain crashes and timeouts; separate cold, warm, and sustained
results; randomize trial order; report intervals supported by the sample size; and keep unlike hardware, context,
power, Region, concurrency, and price dates in separate strata.

## Related material

- [Pareto selection lesson](pareto-selection-process.md)
- [Decision record example](decision-record-template.md)
- [Metric specification](../06-experiment-framework/specifications/metric-specification.md)
