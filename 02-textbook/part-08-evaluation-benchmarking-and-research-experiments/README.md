<a id="part-08"></a>

# Part VIII — Evaluation, Benchmarking, and Research Experiments

Evaluation is the control plane for the book. It measures domain capability, general retention, safety, calibration,
compression damage, systems behavior, and causal effects rather than collapsing success into one average.

## Conceptual map

Requirements become metrics and hard gates; tasks become sealed fixtures; artifacts run under frozen decoding and
hardware conditions; raw item-level and trial-level results become uncertainty-aware summaries; failures are analyzed
before a decision is made. Quality, safety, and systems measurements answer different questions and must not be merged
into an unexplained score.

## Learning outcomes and dependencies

Readers should be able to freeze leakage-resistant suites, define domain rubrics and critical gates, benchmark exact
devices, report uncertainty, design ablations, and compare Pareto fronts. Evaluation design should begin before Parts
II–VII are executed; its outputs govern Parts IX–XIV.

## Chapters

1. [CH-29 — Model Quality Evaluation](ch-29-model-quality-evaluation.md): perplexity, task metrics, generation judging,
   calibration, significance, and contamination.
2. [CH-30 — Domain-Specific Evaluation Suites](ch-30-domain-specific-evaluation-suites.md): taxonomies, expert rubrics,
   safety scenarios, refusal, and escalation.
3. [CH-31 — Compression and Edge Benchmarking](ch-31-compression-and-edge-benchmarking.md): artifact size, RAM, TTFT,
   decode, energy, thermals, cost, and target parity.
4. [CH-32 — Optimization Experiments and Ablation Studies](ch-32-optimization-experiments-and-ablation-studies.md):
   controls, factorial designs, interactions, uncertainty, and negative results.

## Decision checkpoint

Freeze prompts, references, judge versions, runtime settings, target states, warm-up, repetitions, and acceptance gates
before candidate comparison. Report distributions and failures, not only means. R3 safety failures block release
regardless of aggregate score.

### Practice before continuing

Write one hypothesis with a baseline, one changed variable, two controlled variables, a primary metric, a hard gate,
and a stop condition. If the result could not change your decision, the experiment is not yet useful.

## Cross-links

[Experiment checklist](../back-matter/appendix-experiment-reporting-checklist.md) ·
[Part XIII](../part-13-experimental-research-roadmap/README.md) ·
[Decision framework](../back-matter/final-decision-framework.md)
