# Figure and Table Standard

## Why visuals need evidence labels

A polished chart can make a hypothetical trend look measured. A comparison table can hide that one row came from a
paper, another from a vendor, and a third from an estimate. Every figure and table MUST make its evidence class obvious
without requiring the reader to inspect surrounding prose.

Required visuals use `FIG-01` through `FIG-26`; required comparisons use `TBL-01` through `TBL-25`, as assigned in the
[figure/table ledger](../15-audit-ledgers/figure-table-ledger.md) and
[diagram plan](../01-planning/diagram-plan.md). Do not renumber IDs to close gaps.

Each artifact needs an explicit anchor, ID, title, caption, evidence label, canonical location, owner, status, and ledger
entry. Define units, population, conditions, legend, and evidence in or directly with the artifact.

## Four kinds of visual evidence

### Measured by this project

Use `EXECUTED` or `REPRODUCED`. Link the protocol, result, raw data, environment, analysis code/configuration, and
artifact hash. State trials, aggregation, uncertainty, units, workload, target, date, exclusions, and failures.

### Reported elsewhere

Use `PAPER-REPORTED`, `VENDOR-REPORTED`, or `DOC-VERIFIED`. Cite the exact source location, preserve its units and
uncertainty, and warn when conditions differ. Digitizing a published plot does not reproduce its experiment.

### Estimated

Use `ESTIMATE`. Give the formula, assumptions, dated inputs, sensitivity or uncertainty, and validation state. Make an
estimated series visually distinct from a measured series.

### Illustrative

Use `ILLUSTRATIVE` for teaching diagrams, fictional values, and conceptual trends. Say explicitly that the visual is not
measured. Do not place precise-looking data points on unlabeled benchmark axes. An illustrative Pareto frontier may
explain Pareto optimality; it may not select this project's winning model.

Never merge different evidence classes into an unlabeled cell or series. If one artifact contains several classes,
label every row or series.

## Worked caption examples

Good illustrative caption:

> **FIG-07 — Quantization calibration flow. [`ILLUSTRATIVE`]** Calibration samples pass through the float model to
> estimate scaling parameters before conversion. Not measured; arrows show conceptual data flow only. Source:
> [quantization chapter](../02-textbook/part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md).
> Limitations: backend-specific calibration steps are omitted.

Good measured caption pattern with populated teaching values:

> **FIG-21 — Decode throughput on the lab phone. [`ILLUSTRATIVE`]** Fictional teaching data for 30 hypothetical runs on
> a Pixel 8 at 128 generated tokens; tokens/s shown as median and interquartile range. Source: worked example only.
> Limitations: these values were not observed and must not support a device decision.

The second example deliberately uses `ILLUSTRATIVE` even though it demonstrates how a measured caption would describe
conditions. It must never be relabeled `EXECUTED` without actual run artifacts.

Anti-example:

> **Mobile speed.** The 4-bit model is much faster.

This omits model, device, workload, units, uncertainty, date, and evidence.

## Design figures for accurate reading

Figures MUST:

- use meaningful alt text that explains the information;
- label axes, units, scales, legends, dimensions, and improvement direction;
- call out logarithmic, truncated, normalized, dual, or broken axes;
- show uncertainty and individual points or distributions when practical;
- remain distinguishable without color and readable in print and narrow displays;
- avoid decorative 3-D, distorted areas, hidden baselines, cherry-picked ranges, and undisclosed smoothing; and
- distinguish observations from fitted or extrapolated lines.

Architecture diagrams also identify trust boundaries, network boundaries, stores, data direction, optional paths, and
whether a component belongs to training, evaluation, updates, or inference.

## Design tables for fair comparison

Tables MUST:

- place units in headings and define symbols and abbreviations in notes;
- include evidence and an “as of” date for support, prices, availability, and benchmarks;
- reserve zero for measured zero and explain whether `—` means not reported or not applicable;
- preserve denominators, trial counts, failures, uncertainty, and test conditions;
- separate observations from recommendations and state the decision rule; and
- avoid ranking configurations tested under incomparable conditions.

Use the [benchmark table form](../06-experiment-framework/templates/benchmark-table-template.md) for experiments.

## Author and reviewer workflow

1. Assign the ID before creating the artifact.
2. Decide its evidence class.
3. Add labels, units, scope, provenance, limitations, and accessible encoding.
4. Verify every plotted mark or table cell against its source.
5. Preserve source data and generation instructions under the
   [reproducibility standard](reproducibility-standard.md).
6. Ask a second reviewer to spot-check measurements or external transcriptions.
7. Require the [safety expert gate](safety-and-risk-standard.md) for safety-critical visuals.

Broken provenance, invented precision, a missing evidence class, or an illustration that looks measured blocks release.
This policy is authoritative; a planned visual remains unmeasured until qualifying evidence exists.
