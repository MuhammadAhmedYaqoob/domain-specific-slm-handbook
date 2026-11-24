<a id="part-13"></a>

# Part XIII — Experimental Research Roadmap

This part converts unresolved choices into falsifiable questions and a staged benchmark program. It protects the project
from treating plausible architecture prose as empirical evidence.

## Conceptual map

An uncertainty becomes a falsifiable hypothesis; the hypothesis defines variables, controls, metrics, and gates; cheap
screens remove infeasible candidates; controlled training comparisons test learning choices; compression and
exact-target trials test deployment choices; statistical and practical interpretation update the next stage. Staging
avoids an unmanageable Cartesian product without hiding why candidates were eliminated.

## Learning outcomes and dependencies

Using Parts II–XII, readers should be able to formulate hypotheses, define independent/dependent/control variables,
budget a staged campaign, handle interactions, preserve raw evidence, and update decisions after null or negative
results. Read the [experiment checklist](../back-matter/appendix-experiment-reporting-checklist.md) concurrently.

## Chapters

1. [CH-45 — Research Questions and Hypotheses](ch-45-research-questions-and-hypotheses.md) defines fifteen questions
   spanning domain adaptation, PEFT, compression, multi-domain interference, edge behavior, safety, and cost.
2. [CH-46 — Benchmark Matrix](ch-46-benchmark-matrix.md) maps model/data/method/runtime/device factors to staged
   experiments, metrics, gates, and evidence artifacts.

## Execution order

Run cheap static and smoke gates first, then training-method comparisons, compression ablations, exact-device
benchmarks, domain/safety review, and lifecycle/cost validation. Promote only candidates surviving the previous stage.

## Decision checkpoint

Pre-register hypotheses, baselines, exclusions, seeds, repetitions, statistical treatment, stop conditions, and
acceptance gates. Until valid traces exist, every matrix cell remains `PLANNED EXPERIMENT`.

### Practice before continuing

Take “INT4 is best for mobile” and rewrite it as a target-specific hypothesis with model, quantizer, runtime, device,
quality tolerance, memory metric, latency metric, repetitions, and rejection rule. The original sentence is an
unsupported generalization; the rewritten form is testable.

## Cross-links

[Part VIII](../part-08-evaluation-benchmarking-and-research-experiments/README.md) ·
[Limitations](../back-matter/errata-and-limitations.md) ·
[Part XIV](../part-14-final-engineering-architecture/README.md)
