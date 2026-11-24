<!-- markdownlint-disable MD013 -->

# Research questions and hypotheses

## Status and common experimental contract

**Status: FIFTEEN STUDIES SPECIFIED / NOT EXECUTED.** Every entry is a `PLANNED EXPERIMENT`; no hypothesis is accepted,
rejected, or measured. This operational summary preserves the full question set from
[CH-45](../02-textbook/part-13-experimental-research-roadmap/ch-45-research-questions-and-hypotheses.md). Resolve each
entry into the framework [experiment plan](../06-experiment-framework/templates/experiment-plan-template.md) before
running it.

This chapter is a design manual, not a result report. Each question follows the
same reasoning pattern:

1. explain the engineering uncertainty and the decision it blocks;
2. state a null hypothesis that can survive or be rejected;
3. change one independent variable while fixing controls;
4. screen cheaply, then confirm only credible candidates;
5. stop for invalidity, harm, futility, success, or budget exhaustion; and
6. make a decision only when the confidence interval, hard gates, and resource
   benefit all support it.

The numerical examples are `ILLUSTRATIVE`. They use fictional values to show
the calculation and interpretation. They must never be copied into a result
registry as project measurements.

An experimental candidate is the complete tuple of model/adaptor lineage, tokenizer, chat template and decoding, data
snapshots, training/transformation recipe, export/precision, runtime, hardware/software environment, and workload. If
more than the declared independent variable changes, report a tuple comparison rather than a causal effect.

All studies use disjoint immutable manifests for CPT, SFT, general replay, distillation, pruning recovery, calibration,
validation, sealed domain/general/safety/OOD tests, and performance prompts. Group source/scenario/entity and synthetic
descendants before splitting. Sealed roots are forbidden for training, calibration, teacher generation, checkpoint
selection, threshold setting, or prompt development.

Unless narrowed below, the controlled variables include immutable model/tokenizer/template revisions, data and evaluator
hashes, accepted token/search budget, seeds, decoding, runtime, workload, and environment. Common outcomes come from the
[metric registry](../06-experiment-framework/specifications/metric-specification.md): domain quality, unsupported
claims, calibration, general retention, critical safety and escalation, artifact/install bytes, post-load/peak memory,
startup/visible TTFT, prefill/decode/end-to-end latency, reliability, sustained energy/thermal behavior, and measured
resource use.

For every question, preregister one primary estimand and metric, minimum important difference (MID) or non-inferiority
margin, analysis unit, interval/test, multiplicity family, missing/failure policy, power rationale, hard gates, resource
cap, and harm/futility/success stopping rules. Smoke one seed; use at least three independent training seeds for
decision-critical finalists where feasible. Pair identical item roots, block systems trials by device/host/day/thermal
state, and derive repetition counts from pilot variance. Safety is non-compensatory.

### Turning a hypothesis into a decision rule

Suppose the candidate and baseline have paired domain scores of 0.812 and
0.800. The observed difference is `0.012`, but its 95% confidence interval is
`[-0.006, 0.030]`. A registered superiority MID of 0.010 is not established:
the interval includes both harm and a useful gain. The correct conclusion is
`INCONCLUSIVE`, not “1.2 points better.” More repetitions may be justified only
if a pre-approved power or near-boundary trigger permits them.

For non-inferiority with candidate-minus-baseline margin `-0.020`, the lower
confidence bound must exceed `-0.020`. For a hard safety rule of zero critical
unsafe responses, one critical event stops the branch even if average quality
and latency improve.

## RQ1 — DAPT versus SFT

**Question:** Does domain-adaptive pretraining (DAPT) improve specialist knowledge more than SFT alone for a given
compute budget?

- **Falsifiable hypothesis:** Equal-budget DAPT→SFT exceeds SFT-only by the registered domain MID while remaining
  non-inferior on general retention and passing safety; missing superiority or any hard-gate failure falsifies it.
- **Baseline / variable:** Same base trained with SFT only; vary no DAPT, domain DAPT, and DAPT with general replay.
- **Dataset / procedure:** Rights-approved automotive CPT plus identical SFT, validation, and sealed
  domain/general/safety suites; match declared accelerator-hours, tokens, or wall-time (never conflate them), train
  paired seeds, select on validation, then confirm exact artifacts.
- **Hardware / repetition / statistics:** One pinned training topology; smoke then ≥3 finalist seeds; item-root/seed
  hierarchical or paired-bootstrap analysis, Holm across DAPT arms, non-inferiority for retention.
- **Limitations / decision:** Corpus quality, replay, contamination, and budget definition limit generalization. Adopt
  only with adjusted domain gain, all gates, and justified incremental lifecycle cost.

### RQ1 practical interpretation

**Underlying issue.** SFT teaches answer behavior from curated examples; DAPT
first exposes the model to raw domain language. DAPT may improve terminology
and implicit knowledge, but it also consumes compute and can damage general
capability. The question is whether the extra phase buys useful knowledge under
the same resource boundary.

**Variables and controls.** The independent variable is the adaptation path:
`SFT`, `DAPT→SFT`, or `DAPT+general replay→SFT`. Fix the base revision, tokenizer,
SFT records, SFT token count, validation rule, seeds, and final evaluation.
Budget matching must name one constraint—accelerator-hours, training tokens, or
wall time—because matching one does not match the others.

**Sequence and stopping.** Run data-rights and leakage gates, an untouched
baseline, then one DAPT smoke. Stop for unstable loss, safety regression,
retention below margin, or validation futility. Confirm at least three paired
seeds only if the smoke shows a plausible domain gain.

**Worked example (`ILLUSTRATIVE`).** If SFT scores `0.720` and DAPT→SFT scores
`0.755`, the gain is `0.035`. With a domain MID of `0.020` and 95% interval
`[0.014, 0.056]`, superiority is not yet established because the lower bound is
below `0.020`. If replay improves general retention from `-0.032` to `-0.009`
against a `-0.020` margin, the replay arm may be viable, but only after safety
and budget gates.

**Evidence that closes the gap.** Paired seed records, immutable corpora,
token/accounting logs, contamination analysis, domain and retention intervals,
and exact-artifact safety review. The decision is DAPT only if its lower bound
clears the domain MID, all retention/safety gates pass, and incremental compute
and maintenance are acceptable.

## RQ2 — QLoRA versus full fine-tuning

- **Falsifiable hypothesis:** QLoRA is non-inferior to full fine-tuning within the approved domain margin, passes
  retention/safety and merge parity, and improves a registered training-resource metric.
- **Baseline / variable:** Full fine-tuning; LoRA is secondary; vary update method and only preregistered exploratory
  ranks.
- **Dataset / procedure:** Identical automotive SFT/validation/sealed suites; verify masks and target modules, retain
  OOM/divergent seeds, compare adapter and merged output, and evaluate the merged high-precision artifact. Training
  quantization is not deployment quantization.
- **Hardware / repetition / statistics:** Same supported GPU where feasible; if full FT needs another topology, separate
  resource-matched product comparison; ≥3 finalist seeds; one-sided non-inferiority plus paired resource intervals and
  Holm.
- **Limitations / decision:** Optimizer capacity, architecture targets, topology, merge precision, and tuning can
  confound. Select QLoRA only if quality and hard gates pass and measured savings exceed the operational MID.

### RQ2 practical interpretation

**Underlying issue.** Full fine-tuning updates every weight and often needs much
more memory. QLoRA trains low-rank adapters over a quantized frozen base. The
lower training footprint is useful only if the merged or deployable artifact
retains required behavior.

**Variables and controls.** Change the update method. Hold data order, effective
batch, optimizer family, token budget, stopping policy, evaluation, and
high-precision deployment evaluation fixed. If full FT requires another GPU
topology, report both a method comparison on the common feasible setup and a
product/resource comparison across topologies.

**Sequence and stopping.** Verify loss masks and target modules, run one smoke
per method, test adapter-versus-merged parity, then confirm finalist seeds. Stop
on merge mismatch, repeated divergence, OOM beyond the registered retry class,
or quality/safety futility.

**Worked example (`ILLUSTRATIVE`).** Full FT uses `96 GPU-h` and scores `0.804`;
QLoRA uses `28 GPU-h` and scores `0.797`. The difference is `-0.007` with
interval `[-0.016, 0.003]`. With non-inferiority margin `-0.020`, quality
passes. Resource saving is `(96-28)/96=70.8%`. QLoRA qualifies only if the
registered resource MID is at most this saving and merged-artifact safety,
retention, and parity also pass.

**Evidence that closes the gap.** Peak-memory traces, actual GPU-hours,
hyperparameter and seed records, failed attempts, adapter and merge hashes,
paired evaluation, and operational complexity review.

## RQ3 — Android quantization

- **Falsifiable hypothesis:** At least one supported quantizer produces a hard-gate-qualified, non-dominated Android
  artifact relative to its higher-precision parent and alternatives.
- **Baseline / variable:** Runtime-native higher precision and conservative 8-bit; vary exact quantizer, bit
  width/granularity, exceptions, activation/KV precision, and separately declared backend.
- **Dataset / procedure:** Frozen quantization calibration, sealed suites, and short/medium/long performance prompts;
  smoke conversion, verify hashes/template/greedy parity and placement, reject fallback, then randomize cold/warm and
  sustained physical-phone blocks.
- **Hardware / repetition / statistics:** At least one physical low/mid-range Android and a higher-capability device
  when tier permits; pilot-powered timing plus ≥3 sustained finalist sessions; paired quality non-inferiority and
  blocked device analysis with simultaneous Pareto intervals.
- **Limitations / decision:** Nominal bit widths and runtimes are not equivalent; device generalization needs multiple
  SKUs. Retain qualified candidates on target-specific uncertain fronts; choose only with frozen product preferences.

### RQ3 practical interpretation

**Underlying issue.** A smaller checkpoint may expand during loading, fall back
to slow CPU operators, or lose rare safety behavior. “4-bit” describes too
little; format, group size, exceptions, activation precision, KV precision,
kernel, and runtime all matter.

**Variables and controls.** Within one runtime, vary one supported quantizer
configuration at a time. Fix parent weights, tokenizer, prompt set, decoding,
context/output lengths, runtime build, phone state, thread count, and thermal
start band. A runtime change creates a tuple comparison.

**Sequence and stopping.** Convert and hash, test deterministic output and
operator placement, reject silent fallback, measure load/peak memory, run paired
quality/safety, then randomized cold/warm and sustained device blocks. Stop on
unsupported operators, OOM, changed workload, safety failure, or conservative
domination.

**Worked example (`ILLUSTRATIVE`).** Parent peak memory is `3,100 MiB`, Q8 is
`2,500 MiB`, and Q4 is `1,850 MiB`. Q4 saves `1,250 MiB` or `40.3%`. Its domain
difference is `-0.012` with interval `[-0.019,-0.004]`, clearing a `-0.020`
margin. Median decode rises from `8.4` to `11.1 token/s`, but one of 40 trials
fails: `1/40=2.5%`. The failure-rate upper bound, safety, and placement gates
still determine qualification.

**Evidence that closes the gap.** Exact conversion configuration, hashes,
placement traces, app-level peak memory, all attempted trials, paired quality
outputs, and named-device sustained traces. Choose from a device-specific
Pareto set, not by bit width alone.

## RQ4 — Structured pruning latency

- **Falsifiable hypothesis:** A physically materialized structured-pruned graph improves sustained target latency by the
  MID without violating quality, safety, memory, or reliability gates.
- **Baseline / variable:** Dense parent, dense tensors with matched unstructured zeros, and similarly sized native SLM;
  vary valid layer/MLP/head/KV-group removals, ratio, and fixed recovery.
- **Dataset / procedure:** Pruning calibration, recovery train, sealed suites, mobile prompts; prune one factor,
  serialize smaller dense shapes, evaluate pre/post recovery, export identically, inspect placement, benchmark
  randomized blocks.
- **Hardware / repetition / statistics:** Same Android/runtime as parent; ≥3 recovery seeds for finalists and powered
  systems/soak blocks; ratio trend plus planned parent contrast, quality non-inferiority, blocked latency, Holm.
- **Limitations / decision:** Surgery, kernel alignment, native pretraining, and recovery budget confound. Parameter or
  checkpoint reduction alone fails; require measured target benefit and justified complexity.

### RQ4 practical interpretation

**Underlying issue.** Unstructured zeros often leave the same dense tensor
shapes and therefore do not reduce mobile work. Structured pruning must create
smaller executable dimensions aligned with the runtime’s kernels.

**Variables and controls.** Change pruning structure and ratio; keep parent,
calibration, recovery token budget, export, runtime, prompts, and device fixed.
Include a dense parent, a dense tensor containing zeros, and a similarly sized
native model to separate sparsity claims from simple size effects.

**Sequence and stopping.** Validate shape surgery, measure before recovery,
apply the frozen recovery recipe, evaluate quality/safety, export, verify
physical dimensions and placement, then benchmark. Stop if shapes do not
shrink, conversion fails, or sustained latency fails the MID.

**Worked example (`ILLUSTRATIVE`).** Removing 20% of MLP channels reduces file
size by 14%, but sustained latency changes from `120 ms/token` to
`116 ms/token`. The improvement is `(120-116)/120=3.3%`; with a latency MID of
10%, the method is operationally futile even if statistically detectable.

**Evidence that closes the gap.** Tensor-shape lineage, parameter counts,
recovery cost, runtime operator trace, quality interval, and target latency.
Adopt only a materialized graph with useful target improvement.

## RQ5 — Distillation versus direct student training

- **Falsifiable hypothesis:** A distilled student exceeds direct expert-data SFT of the identical student architecture
  by the domain MID without worse safety, retention, calibration, or accepted-data/resource policy.
- **Baseline / variable:** Same initialized student with expert-only SFT; vary reviewed response, mixed expert/teacher,
  and technically aligned logit distillation.
- **Dataset / procedure:** Expert data and source-grounded teacher candidates with acceptance/rejection audit; teacher
  cannot see sealed tests; train paired arms, blind evaluation, inspect correlated teacher/student errors.
- **Hardware / repetition / statistics:** Pinned student GPU; teacher hardware/API recorded separately; ≥3 student
  seeds; root/seed hierarchical superiority, McNemar where suitable, interval for accepted synthetic yield.
- **Limitations / decision:** Teacher rights, contamination, error transfer, style imitation, review bias, cost, and
  tokenizer mismatch. Adopt only with superiority, no critical transfer, and acceptable total review/training cost.

### RQ5 practical interpretation

**Underlying issue.** A teacher can add explanations and decision patterns, but
it can also transfer confident errors and style without knowledge. The fair
test uses the identical student architecture and compares accepted supervision,
not a stronger student or a larger token budget hidden in one arm.

**Variables and controls.** Change supervision: expert-only, reviewed teacher
responses, mixed data, or aligned logits. Fix student initialization, accepted
training tokens, optimizer, evaluation, and seed list. Record teacher
generation and review as separate resources.

**Sequence and stopping.** Generate away from sealed roots, trace every output
to sources, reject unsupported samples, run a small transfer audit, then train
paired student arms. Stop if teacher rights fail, acceptance yield is too low,
critical errors transfer, or the total resource cap binds.

**Worked example (`ILLUSTRATIVE`).** Of 100,000 generated samples, 62,000 pass
automatic checks and 40,000 pass expert sampling/extrapolated acceptance:
accepted yield is `40%`. A distilled student scores `0.748` versus direct SFT
`0.719`, difference `0.029`, interval `[0.012,0.046]`. With MID `0.020`, the
lower bound does not establish the full MID. More confirmation is justified
only under the frozen near-boundary rule.

**Evidence that closes the gap.** Teacher terms, prompts and model identity,
source-grounding records, acceptance/rejection ledger, reviewer agreement,
student seed outputs, correlated-error analysis, and full generation/review
cost.

## RQ6 — Pruning/distillation order

- **Falsifiable hypothesis:** Under matched final architecture, accepted data, and optimization budget, one of
  `prune→recover/distil` or `distil→prune→recover` establishes a meaningful qualified Pareto advantage.
- **Baseline / variable:** Adapted parent, native-small/direct student, and both isolated-method parents; vary operation
  order only in the primary contrast.
- **Dataset / procedure:** Shared distillation, calibration, recovery, sealed, and performance snapshots; retain every
  stage, disable quantization, materialize the same final shape where feasible.
- **Hardware / repetition / statistics:** Same training topology and exact supported deployment target; ≥3 end-to-end
  seeds/order and powered timing; factorial/path order effect with paired bootstrap and multiplicity after safety.
- **Limitations / decision:** Final shapes and optimization landscapes may differ. Prefer an order only if it beats the
  alternative and both immediate parents; otherwise choose the simpler lineage.

### RQ6 practical interpretation

**Underlying issue.** Pruning first asks the teacher signal to recover a reduced
graph; distilling first gives the pruner a stronger student but may destroy
some of that gain. Order is causal only when final architecture, data, and
optimization budget are matched.

**Variables and controls.** Change operation order. Hold final shape, accepted
distillation set, recovery tokens, total accelerator budget, quantization
status, and target runtime fixed. Retain isolated pruning and isolated
distillation parents.

**Sequence and stopping.** Qualify both isolated methods, run both orders with
the same seeds, confirm final shapes, then compare exact deployment artifacts.
Stop combinations if either parent fails or if both paths are dominated by an
isolated parent.

**Worked example (`ILLUSTRATIVE`).** `prune→distil` scores `0.742` at
`105 ms/token`; `distil→prune` scores `0.750` at `106 ms/token`; isolated
distillation scores `0.754` at `104 ms/token`. Both orders are dominated by the
simpler isolated method, so no order should be selected.

**Evidence that closes the gap.** Complete stage lineage, matched resource
ledger, same final dimensions, parent comparisons, interaction intervals, and
target traces.

## RQ7 — Distilled/quantized versus native small SLM

- **Falsifiable hypothesis:** Within a preregistered deployed-memory or artifact band, a distilled-and-quantized student
  exceeds a directly adapted native small SLM in domain quality while meeting equal safety, reliability, and latency
  gates.
- **Baseline / variable:** Native small checkpoint chosen before results and adapted on identical accepted domain data;
  vary architecture origin and final target quantization.
- **Dataset / procedure:** Accepted expert/distillation data, sealed suites, target prompts; qualify both, adapt
  baseline, distil parent, apply best compatible quantization to both, evaluate exact artifacts under fixed RAM
  admission.
- **Hardware / repetition / statistics:** Same physical target; runtime differences make a product-tuple comparison; ≥3
  training seeds and device blocks; conditional quality superiority plus blocked systems/Pareto uncertainty.
- **Limitations / decision:** Pretraining, tokenizer, runtime, and “size” definition differ. Select distilled only if it
  wins under the frozen deployed-resource definition and justifies extra complexity.

### RQ7 practical interpretation

**Underlying issue.** Compressing a larger parent may preserve specialized
behavior better than adapting a model that was small from the start. The
comparison is a product-tuple comparison because architecture and pretraining
differ; it cannot prove that distillation alone caused the result.

**Variables and controls.** Fix the target memory band, accepted domain data,
quality/safety suites, prompts, decoding, and physical target. Apply a
compatible final quantization policy to both candidates. Record runtime
differences explicitly.

**Sequence and stopping.** Establish a direct native-small control, distil the
student, quantize both, apply RAM admission, then compare quality and sustained
behavior. Stop a candidate that misses the fixed resource band or hard gate.

**Worked example (`ILLUSTRATIVE`).** Native-small uses `1.72 GiB` and scores
`0.690`; distilled Q4 uses `1.79 GiB` and scores `0.731`. Both fit a `2.0 GiB`
cap. Difference is `0.041`, interval `[0.018,0.064]`; with MID `0.020`, the
lower bound narrowly misses. Report inconclusive superiority while retaining
the candidate if its other gates pass.

**Evidence that closes the gap.** Frozen size definition, both training
lineages, equivalent data exposure, exact runtime artifacts, paired evaluation,
and complexity/cost ledger.

## RQ8 — General-capability loss

- **Falsifiable hypothesis:** Every release-critical general capability remains within its preregistered non-inferiority
  margin after specialization, with no critical safety regression.
- **Baseline / variable:** Exact pre-specialization parent; vary SFT, DAPT→SFT, full FT, LoRA, QLoRA, intensity, and
  separately replay.
- **Dataset / procedure:** Sealed language, instruction, reasoning, structured-output, harmless/OOD/refusal suites plus
  domain/safety; compare paired errors, then confirm finalists on fresh roots.
- **Hardware / repetition / statistics:** One reproducible inference environment and selected deployment artifact;
  training-seed hierarchy; root-clustered intersection–union non-inferiority with Holm across capabilities.
- **Limitations / decision:** Finite/contaminated suites and template effects cannot represent all knowledge. Release
  only when every required capability passes or an explicit non-safety scope trade-off is approved.

### RQ8 practical interpretation

**Underlying issue.** Domain specialization can overwrite instruction
following, general language, reasoning, structured output, or refusal behavior.
A large domain gain is not acceptable if a required general capability falls
outside its approved margin.

**Variables and controls.** Compare each specialized artifact with its exact
parent. Fix tokenizer/template, decoding, test roots, and evaluator. Stratify by
capability and risk rather than averaging everything into one score.

**Sequence and stopping.** Evaluate the untouched parent first, then each
training stage, and confirm finalists on fresh roots. Stop immediately for a
critical safety regression; stop an adaptation intensity when multiple required
capabilities cross futility bounds.

**Worked example (`ILLUSTRATIVE`).** General instruction score changes from
`0.810` to `0.798`, difference `-0.012`, interval `[-0.019,-0.005]`, which
passes a `-0.020` margin. Structured-output accuracy changes by `-0.027` with
the same margin and fails. The release fails because intersection–union
non-inferiority requires every registered capability to pass.

**Evidence that closes the gap.** Parent/child paired outputs, root-clustered
intervals, per-capability multiplicity control, fresh confirmation roots, and
template-parity checks.

## RQ9 — Shared adapters versus independent specialists

- **Falsifiable hypothesis:** For a frozen domain portfolio/workload, shared base plus adapters reduces measured
  installed/serving/update resources while every domain remains within independent-specialist quality/safety margins.
- **Baseline / variable:** Independent specialist artifacts; vary independent, merged adapters, and dynamic adapters
  separately.
- **Dataset / procedure:** Governed per-domain train/sealed suites, cross-domain substitution/routing, update and
  rollback drills; measure exact base-adapter pairs, sequential/concurrent use, resident/staged storage.
- **Hardware / repetition / statistics:** One edge and one AWS class after runtime qualification; ≥3 selected-domain
  seeds, repeated switch/load and clean-reset update drills; domain-stratified intersection–union non-inferiority and
  paired resources.
- **Limitations / decision:** Capacity, base choice, packaging rights, and runtime copies matter. Adapter bytes alone
  are insufficient; all domains and lifecycle-resource MID must pass.

### RQ9 practical interpretation

**Underlying issue.** One shared base plus adapters appears storage-efficient,
but runtime copies, adapter switching, incompatible licences, and coupled base
updates may erase the saving. Every domain must retain its quality and safety.

**Variables and controls.** Compare independent specialists, one shared base
with separate adapters, and only then merged/dynamic adapter strategies. Fix
domain datasets, workload mix, release gates, target hardware, and update
scenario.

**Sequence and stopping.** First qualify at least two domains independently.
Measure installed and resident bytes, switch time, concurrent use, and a
base/adapter rollback drill. Stop if any high-risk domain fails, the wrong
adapter is selected, or lifecycle resources do not improve.

**Worked example (`ILLUSTRATIVE`).** Three independent 2.0 GiB models require
`6.0 GiB`. One 1.9 GiB base plus three 120 MiB adapters requires
`1.9 + 3×0.117 = 2.251 GiB`, a `62.5%` installed-size reduction. If the runtime
duplicates the base for concurrent domains, resident memory may still approach
`5.7 GiB`; installed bytes alone cannot answer the question.

**Evidence that closes the gap.** Exact package and resident-memory traces,
base-adapter compatibility hashes, per-domain non-inferiority, switching
failures, update/rollback results, and rights review.

## RQ10 — One multi-domain model versus specialists

- **Falsifiable hypothesis:** A joint SLM is non-inferior to specialists on every required domain/safety stratum under a
  frozen total parameter/storage budget.
- **Baseline / variable:** Independent specialists plus separate adapters; vary joint weights, joint adapter, separate
  adapters, and independent artifacts.
- **Dataset / procedure:** Hierarchically sampled governed domains; sealed per-domain, cross-domain confusion, OOD,
  safety, and mixed-conversation suites; audit realized exposure and simulate one-domain update.
- **Hardware / repetition / statistics:** Budget-matched training topology and same target; ≥3 finalist seeds;
  per-domain paired analysis and intersection–union test with adjusted contrasts.
- **Limitations / decision:** Active capacity, imbalance, tokenizer fit, rights, and coupled updates differ. One failed
  high-risk domain rejects the joint release.

### RQ10 practical interpretation

**Underlying issue.** A joint model can share useful concepts, but domains
compete for fixed capacity and updates become coupled. Average performance can
hide failure in a small or high-risk domain.

**Variables and controls.** Change packaging/training strategy: independent
models, separate adapters, joint adapter, or joint weights. Freeze the total
parameter or installed-storage budget, realized token exposure per domain, and
per-domain tests.

**Sequence and stopping.** Balance and audit training exposure, evaluate each
domain and wrong-domain substitution, then simulate an update to one domain.
Stop on any critical stratum failure, unacceptable interference, or coupled
update risk.

**Worked example (`ILLUSTRATIVE`).** Automotive and beekeeping differences are
`+0.010` and `-0.008`, but pharmacy is `-0.031` against a `-0.015` margin. The
mean difference may look acceptable, yet the joint model fails because every
required domain must pass.

**Evidence that closes the gap.** Exposure ledger, per-domain seed intervals,
confusion and mixed-conversation tests, fixed-budget accounting, and update
isolation drill.

## RQ11 — Architecture and KV-cache effects

- **Falsifiable hypothesis:** At matched quality and deployed-weight bands, architecture/KV factors produce practically
  meaningful mobile memory-slope or decode differences.
- **Baseline / variable:** Registered dense decoder with full-precision KV; vary shape, KV heads/layout/precision,
  context cap, and native sliding-window policy.
- **Dataset / procedure:** Short/long/context-boundary prompts and sealed regression; predict KV bytes, measure
  allocation and reset/reuse, sweep context, retain placement evidence.
- **Hardware / repetition / statistics:** Physical mid-range Android plus iPhone/ARM where comparable; randomized
  repeated sweeps and independent soaks; blocked/multivariable characterization and planned near-match intervals.
- **Limitations / decision:** Pretraining, kernels, tokenizer, and architecture confound observational comparisons. Make
  tuple-scoped claims unless an isolated ablation exists; promote only qualified target-envelope improvements.

### RQ11 practical interpretation

**Underlying issue.** Mobile memory grows with context because keys and values
are retained for each layer. Fewer KV heads, lower KV precision, or windowed
attention can reduce growth, but architecture families also differ in training
and kernels.

**Variables and controls.** For an isolated test, change KV precision or
context policy in one compatible model/runtime. For architecture comparisons,
match deployed-weight and quality bands and label conclusions tuple-scoped.
Fix prompt token counts, output length, device state, and allocation boundary.

**Sequence and stopping.** Predict the memory slope, sweep context lengths,
measure allocation and reset behavior, verify output quality near boundaries,
then run sustained decode. Stop if allocation is hidden/unreliable or the
candidate changes effective context semantics.

**Worked example (`ILLUSTRATIVE`).** If measured peak memory is `2,000 MiB` at
1,024 tokens and `2,384 MiB` at 2,048 tokens, observed slope is
`384/1024=0.375 MiB/token`. A Q8 KV variant at `0.205 MiB/token` reduces slope
by `45.3%`. It qualifies only if long-context quality and latency remain within
their gates.

**Evidence that closes the gap.** Formula inputs, allocation traces at multiple
contexts, context-boundary quality, placement evidence, and named-device
replication.

## RQ12 — Best tuple under fixed mobile RAM

- **Falsifiable hypothesis:** At least one compatible model–artifact–runtime tuple fits the exact peak-RAM cap and is
  non-dominated on quality, interaction latency, sustained behavior, and reliability.
- **Baseline / variable:** Current production tuple or smallest qualified native tuple; vary exact model, export,
  runtime/backend, context cap, and thread/acceleration configuration.
- **Dataset / procedure:** Sealed suites and standardized/soak prompts; funnel compatibility→quality→load/prefill/decode
  peaks→sustained tests, retaining OOM, truncation, and fallback.
- **Hardware / repetition / statistics:** Exact target phone SKU, separate cohort panels; pilot-powered blocks and ≥3
  finalist soaks; hard RAM gate, paired quality, blocked systems, conservative Pareto dominance.
- **Limitations / decision:** OS pressure, app memory, formats, and one-SKU external validity. “Best” is a
  frozen-preference selection from qualified non-dominated tuples; missing energy stays unresolved.

### RQ12 practical interpretation

**Underlying issue.** The model must coexist with the app, OS, tokenizer,
temporary buffers, and KV cache. File size is not the admission metric; peak
app-level memory under the intended workload is.

**Variables and controls.** Vary complete model/export/runtime/configuration
tuples. Fix the exact phone SKU, OS/build, RAM cap, prompt/context distribution,
output cap, app version, and background-state protocol.

**Sequence and stopping.** Check compatibility, then quality/safety, then load,
prefill, decode, and soak peaks. Record OOM and truncation. Stop any tuple whose
upper memory bound exceeds the cap or whose reliability fails.

**Worked example (`ILLUSTRATIVE`).** A device policy allows `2.50 GiB` peak.
Tuple A peaks at `2.32 GiB` with upper confidence bound `2.47 GiB`; tuple B
peaks at `2.41 GiB` with upper bound `2.61 GiB`. Only A clears admission,
although B’s point estimate fits. Among admitted tuples, choose by the frozen
quality/latency/energy preferences.

**Evidence that closes the gap.** App-level memory traces by phase, OS pressure
metadata, failure denominator, exact workload, and uncertainty-aware
target-specific Pareto analysis.

## RQ13 — AWS versus edge economics

- **Falsifiable hypothesis:** For a declared demand/SLO/fleet/horizon, one qualified deployment has lower total relevant
  cost or materially greater required capability; a sensitivity-bounded break-even region can be established.
- **Baseline / variable:** One AWS and one edge artifact with equivalent intended use and quality floor; vary volume,
  concurrency, output distribution, topology/purchase model, fleet, update rate, and horizon.
- **Dataset / procedure:** Measured cloud/device traces, approved demand distributions, observed effort, and dated
  official prices/bills; build scenarios and uncertainty propagation—never invent prices.
- **Hardware / repetition / statistics:** Exact AWS Region/service and physical cohorts; multiple load days/blocks;
  bootstrap technical inputs and use Monte Carlo/sensitivity for break-even, not significance tests on tariff
  arithmetic.
- **Limitations / decision:** Privacy/offline capability, staff cost, hardware allocation, and feature differences
  resist monetization. Choose AWS only when quality/SLO/security pass and sensitivity supports it; otherwise state a
  conditional boundary.

### RQ13 practical interpretation

**Underlying issue.** Cloud has metered serving and network costs; edge has
device, integration, distribution, and update costs. The comparison is invalid
if cloud and edge provide different quality, privacy, availability, or features.

**Variables and controls.** Vary deployment location and scenario inputs:
requests, concurrency, generated-token distribution, fleet size, update rate,
purchase model, and horizon. Fix intended use, minimum quality, security, and
SLO. Use dated official prices or bills only.

**Sequence and stopping.** Qualify equivalent artifacts first, collect load and
device traces, define cost boundary, then run sensitivity and break-even
analysis. Stop if equivalent use cannot be established or required cost inputs
remain unverified.

**Worked example (`ILLUSTRATIVE`, not current prices).** Let cloud fixed cost be
`$120/month` and variable cost `$0.004/request`; edge allocated monthly cost is
`$520`. Break-even request count is
`(520-120)/0.004 = 100,000 requests/month`. If plausible variable cost ranges
from `$0.003` to `$0.006`, break-even ranges from about `133,333` to `66,667`.
The decision should report this region, not a false single-point certainty.

**Evidence that closes the gap.** Dated regional prices/bills, measured
successful workload, feature-equivalence record, labor/accounting policy,
failure costs, and sensitivity distributions.

## RQ14 — Thermal throttling and battery drain

- **Falsifiable hypothesis:** Late-window decode or incremental energy/token degrades beyond MID versus the early window
  and is associated with recorded thermal/power transitions after accounting for context growth.
- **Baseline / variable:** Same-artifact short burst and matched idle trace; vary runtime/model, duration/duty cycle,
  charging as separate condition, ambient/start band, and power mode.
- **Dataset / procedure:** Fixed sustained schedule; condition device, collect idle, randomize unplugged sessions,
  record token timestamps, context, clocks, thermal states, power, and failures; restore start state.
- **Hardware / repetition / statistics:** Physical target with validated external/platform instrumentation; battery
  percentage cannot establish joules/token; ≥3 independent sessions/condition, more after pilot; run-level early/late
  mixed effects.
- **Limitations / decision:** Context, OS policy, background load, and ambient drift mimic heat. Claim causation only
  with state/clock evidence; qualify only if late-window throughput, energy, thermal, and failure limits pass.

### RQ14 practical interpretation

**Underlying issue.** A short benchmark can finish before a phone heats up.
Sustained use may lower clocks, reduce decode speed, increase energy per token,
or trigger failures. Context growth itself also slows inference, so thermal
claims need synchronized state evidence.

**Variables and controls.** Change artifact/runtime, duration, or duty cycle.
Fix prompt schedule, generated tokens, ambient band, starting battery/thermal
state, charging policy, background activity, and power mode. Treat charging as
a separate condition.

**Sequence and stopping.** Validate power instrumentation, collect matched idle,
condition the device, randomize sessions, and record token timestamps, context,
clocks, thermal states, power, and failures. Stop for safety temperature limits,
invalid instrumentation, or registered reliability failure.

**Worked example (`ILLUSTRATIVE`).** Early decode is `12.0 token/s`; late decode
is `8.7 token/s`. Retention is `8.7/12.0=72.5%`, a `27.5%` degradation. Gross
session energy is `900 J`, matched idle is `180 J`, and 2,400 tokens are
generated, so incremental energy is `(900-180)/2400=0.30 J/token`. Battery
percentage alone cannot support this calculation.

**Evidence that closes the gap.** Calibrated synchronized power traces,
idle-matched boundary, device state/clock logs, independent sessions, context
covariate, and early-versus-late interval.

## RQ15 — Genuine deployment improvement

- **Falsifiable hypothesis:** An optimization improves at least one registered deployment objective beyond MID without
  failing any hard gate or shifting cost into runtime expansion, load, conversion, custom kernels, or maintenance.
- **Baseline / variable:** Every candidate’s immediate deployable parent plus native-small control where relevant; vary
  quantization, pruning, distillation, architecture shrink, merge, conversion, and qualified combinations/order.
- **Dataset / procedure:** All frozen suites/calibration/workloads; retain every parent, transform one stage at a time,
  measure offline/load/per-inference overhead, evaluate the exact release artifact, and build target fronts.
- **Hardware / repetition / statistics:** Named AWS, Android, iOS, and ARM targets only where supported; stochastic
  transformation seeds, powered systems trials, independent edge soaks; parent-paired intervals, selected factorial
  interactions, Holm, uncertain Pareto analysis.
- **Limitations / decision:** Benefits are hardware-specific and lifecycle complexity partly qualitative. Label only
  `genuine for target X`, `checkpoint-only`, `unsupported`, `dominated`, or `inconclusive`; genuine requires all hard
  gates and non-dominated improvement over each immediate parent.

### RQ15 practical interpretation

**Underlying issue.** An optimization can shrink a file while increasing load
time, runtime expansion, conversion failures, custom-kernel maintenance, or
energy. A genuine improvement must help the deployed system, not merely one
offline artifact metric.

**Variables and controls.** Change one optimization stage at a time and compare
with its immediate deployable parent. Fix the target, workload, quality/safety
suite, app/server boundary, and lifecycle accounting. Include a native-small
control where compression is being justified.

**Sequence and stopping.** Measure parent, transform, verify parity and
placement, measure exact child, then attempt combinations only when isolated
children qualify. Stop a child that fails any hard gate, is dominated by its
parent, or moves cost outside the declared boundary.

**Worked example (`ILLUSTRATIVE`).** A compressed child reduces file size from
`2.8` to `1.6 GiB` but peak memory changes from `2.9` to `3.0 GiB`, startup from
`1.8` to `3.1 s`, and decode from `10.2` to `9.9 token/s`. It is
`checkpoint-only`, not a deployment improvement. Another child with unchanged
quality, peak memory `2.1 GiB`, and decode `12.0 token/s` may be genuine if
safety, reliability, energy, and maintenance gates also pass.

**Evidence that closes the gap.** Parent-child lineage, conversion/load
overheads, placement, quality and safety intervals, physical-target systems
traces, failures, and maintenance assessment. Scope the conclusion to the exact
target and date.

## Shared stopping and reporting

Stop a branch for unresolved rights/privacy/provenance, sealed leakage, critical safety incident,
identity/template/merge parity failure, unsupported conversion or silent fallback, workload-changing OOM, registered
futility, budget exhaustion, or domination by a simpler qualified parent without a registered rescue condition. Preserve
the disposition through the [failure template](../06-experiment-framework/templates/failure-report-template.md) and
final [result template](../06-experiment-framework/templates/experiment-result-template.md).

Success is also a stopping rule. Once a simple candidate passes all hard gates
and additional methods cannot add the registered frontier improvement within
the rescue budget, stop the target program. Continuing until a favorable
*p*-value appears is not research; it is outcome-driven search.

### Resource-tier interpretation

- **Small tier:** establish one complete evidence path. Screen broadly with one
  seed, confirm one narrow contrast, and make claims only for named artifacts
  and devices.
- **Medium tier:** answer the main product decisions with several seeds,
  multiple device classes, direct controls, and sustained tests.
- **Research tier:** study mechanisms, order interactions, external validity,
  calibrated energy, and broader cohorts. It still uses gates and stopping
  rules; it is not an unrestricted search.

The detailed allocations and arithmetic are in
[resource budget tiers](resource-budget-tiers.md).

## Cross-links

- [Roadmap index](README.md)
- [Priorities and dependencies](prioritized-experiment-roadmap.md)
- [Budget tiers](resource-budget-tiers.md)
- [Open risks](open-risks-and-evidence-gaps.md)
- [Baseline protocol](../06-experiment-framework/protocols/baseline-evaluation.md)
- [Training comparison protocol](../06-experiment-framework/protocols/training-comparison.md)
- [Quantization protocol](../06-experiment-framework/protocols/quantization-benchmark.md)
- [Pruning protocol](../06-experiment-framework/protocols/pruning-benchmark.md)
- [Distillation protocol](../06-experiment-framework/protocols/distillation-benchmark.md)
- [Combined-optimization protocol](../06-experiment-framework/protocols/combined-optimization.md)
- [Multi-domain protocol](../06-experiment-framework/protocols/multi-domain-ablation.md)
- [Statistical protocol](../06-experiment-framework/protocols/statistical-analysis.md)
- [Benchmark table template](../06-experiment-framework/templates/benchmark-table-template.md)
- [CH-46 benchmark matrix](../02-textbook/part-13-experimental-research-roadmap/ch-46-benchmark-matrix.md)
