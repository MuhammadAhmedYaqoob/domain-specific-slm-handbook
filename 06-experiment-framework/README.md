# Educational experiment framework

## Question answered

How can an engineer move from a research question to a reproducible, target-specific conclusion without confusing an
illustrative example with project evidence?

## Why controls matter

An experiment changes one declared factor while holding plausible alternatives fixed. If model, prompt, runtime,
hardware, and workload all change, the result can compare deployable tuples but cannot identify which change caused
the effect.

## Common novice mistakes

- Starting runs before freezing the hypothesis and decision rule.
- Selecting on sealed tests.
- Recording only successful runs.
- Treating a smaller file as proof of faster inference.
- Naming a universal winner across unlike targets.

## Worked illustrative lifecycle

An illustrative plan compares parent P with quantized child Q on 100 paired items and five device sessions. Q scores
0.74 versus P at 0.75, so `Δ=-0.01`; its peak RAM is 1.6 versus 3.0 GB, a 46.7% reduction. The quality margin is 0.02,
but one critical unsafe response violates a zero-event gate. The correct illustrative decision is rejection, not “best
mobile model.” No number in this lesson is a project measurement.

<a id="required-directory-flow"></a>

## Reusable workflow

1. Write and approve a populated [experiment plan](templates/experiment-plan-template.md).
2. Apply the relevant lesson: [baseline](protocols/baseline-evaluation.md),
   [training](protocols/training-comparison.md), [quantization](protocols/quantization-benchmark.md),
   [pruning](protocols/pruning-benchmark.md), [distillation](protocols/distillation-benchmark.md),
   [combined optimization](protocols/combined-optimization.md), [edge](protocols/edge-device-benchmark.md),
   [AWS](protocols/aws-inference-benchmark.md), or [multi-domain](protocols/multi-domain-ablation.md).
3. Create a [run record](templates/run-record-template.md) for every attempt and a
   [failure report](templates/failure-report-template.md) for every terminal failure.
4. Validate [metrics](specifications/metric-specification.md), [environment](specifications/hardware-and-environment-specification.md),
   [provenance](specifications/artifact-and-provenance-specification.md), and
   [result schema](specifications/result-schema-specification.md).
5. Analyze frozen records with the [statistics lesson](protocols/statistical-analysis.md).
6. Publish a reviewed result and apply [Pareto analysis](specifications/pareto-analysis-specification.md) only after
   every hard gate passes.

Lifecycle states are `DRAFT → APPROVED → RUNNING → EXECUTED → REVIEWED → QUALIFIED|REJECTED|INCONCLUSIVE`.
`EXECUTED` means evidence exists, not that the experiment succeeded. Runtime retrieval and remote-teacher
substitution remain prohibited; consequential domains require qualified review and explicit escalation.
