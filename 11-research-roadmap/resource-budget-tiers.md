<!-- markdownlint-disable MD013 -->

# Resource budget tiers

## Status and budget boundary

**Status: PLANNING CEILINGS AUTHORED / NO SPEND OR PRICE VERIFIED.** These are `PLANNED EXPERIMENT` resource caps copied
from the bounded programme in [CH-46](../02-textbook/part-13-experimental-research-roadmap/ch-46-benchmark-matrix.md),
not predictions of feasibility, duration, performance, or cost. They contain no dollar amounts.

The tiers answer different questions. Small asks whether one complete path is
feasible. Medium supports a product decision across credible alternatives.
Research studies mechanisms, interactions, and external validity. A larger tier
does not make a weak design stronger; it only permits more independent evidence
after validity and safety gates pass.

Each approved experiment receives a multidimensional cap:

\[ B=(h_{accelerator},h_{CPU},n_{teacher\ tokens},h_{expert},h_{device}, S_{retained},h_{AWS\
host},h_{engineering},n_{retries}). \]

Unused capacity in one dimension does not authorize overruns in another. Failed runs, conversion, review,
instrumentation, temporary storage, observability, teardown, and rework count against the relevant cap.

For example, unused teacher tokens cannot compensate for exhausted expert
review. Generating more unreviewable samples would increase risk rather than
evidence. Likewise, spare GPU-hours do not authorize an unplanned device search
after sealed results are visible.

## Programme ceilings

| Resource                          |     Small |     Medium |                          Research |
| --------------------------------- | --------: | ---------: | --------------------------------: |
| registered R1 checkpoint bands    |  2 (S, M) |          4 | 4 + up to 4 registered alternates |
| aggregate accelerator-hours       |       120 |      1,200 |                             8,000 |
| aggregate CPU-hours               |       300 |      3,000 |                            20,000 |
| accepted/generated teacher tokens | 2 million | 20 million |                       100 million |
| qualified domain-expert hours     |        20 |        120 |                               500 |
| retained immutable storage        |     1 TiB |      8 TiB |                            40 TiB |
| AWS host-hours                    |        24 |        160 |                               800 |
| physical device-hours             |        30 |        240 |                             1,500 |
| retry reserve                     |       10% |        10% |               15%, pre-classified |

These ceilings are governance choices, not measured requirements. The approved plan must replace aggregate capacity with
experiment-specific caps and preserve unspent contingency.

## Tier scope

### Small: feasibility and one complete evidence path

- **Model/training:** two size bands; untouched baseline; SFT, LoRA, and QLoRA; a CPT smoke only if foundation evidence
  is complete.
- **Compression:** Q8 plus one target-supported Q4; choose one distillation or structured-pruning branch after direct
  controls.
- **Deployment:** one AWS training/serving class; one physical Android CPU path plus an accelerator compatibility check;
  one physical iOS path; one ARM64 board.
- **Repetition:** one smoke seed for screening; three seeds only for a narrow finalist contrast; pilot-derived systems
  repetitions; limited expert review prioritizing every critical item and a sampled ordinary stratum.
- **Claim boundary:** feasibility for named artifacts/targets only. It cannot support broad architecture,
  device-population, energy, or cross-domain claims.
- **Stop condition:** first simple, hard-gate-qualified, non-dominated target candidate; otherwise cap exhaustion yields
  explicit inconclusive/not-run dispositions.

#### Worked small-tier allocation

`ILLUSTRATIVE`: protect 30 of 120 accelerator-hours for confirmation. Allocate
the other 90 hours as follows:

- 6 h for identity/training smoke;
- 24 h for SFT, LoRA, and QLoRA screens;
- 12 h for one permitted CPT pilot;
- 18 h for one distillation or pruning screen;
- 10 h for failed-run/retry classes; and
- 20 h uncommitted until validation gates are applied.

If QLoRA becomes the finalist, three confirmation seeds at 10 h each consume
the protected 30 h. Total use is `90 + 30 = 120 h`. If screening consumes 98 h,
the program may spend only 22 h on confirmation; it must reduce the planned
contrast or report insufficient budget rather than silently exceed the cap.

For expert review, suppose all 80 critical items need two reviews at 4 minutes
each and adjudication is budgeted at 2 h:

\[
80 \times 2 \times 4/60 + 2 = 12.67\ \text{expert-hours}.
\]

Only 7.33 of the 20 expert-hours remain for ordinary-item sampling, rubric
repair, or teacher-output audit. This arithmetic often makes expert capacity,
not GPU time, the binding small-tier resource.

**Interpretation.** The small tier can establish that one named model,
quantizer, runtime, and device path is workable. It cannot support “Android
phones in general,” broad energy claims, or a universal method ranking.

### Medium: canonical decision programme

- **Model/training:** four size bands where lawful/available; up to three adapted parents; SFT/LoRA/QLoRA, feasible full
  FT and CPT, direct-small controls, distillation, and one portable pruning family.
- **Compression:** two supported Q4 paths, recovery, and combinations only for isolated qualified parents.
- **Deployment:** AWS CPU/GPU; low/mid/flagship Android with supported GPU/NPU checks; two iOS generation/RAM classes;
  ARM64 and optional Jetson.
- **Repetition:** three to five finalist training seeds according to power; randomized device blocks; repeated sustained
  sessions; full high-risk and stratified ordinary expert review.
- **Claim boundary:** target- and workload-specific decisions, not a universal winner.
- **Stop condition:** marginal methods cease after no meaningful conservative frontier improvement over a simpler
  qualified parent.

#### Worked medium-tier allocation

`ILLUSTRATIVE`: reserve 25% of 1,200 accelerator-hours for confirmation:
`1,200×0.25=300 h`. Allocate the 900 h screening pool to baselines and
adaptation (300 h), CPT/full-FT comparisons (220 h), distillation (180 h),
portable pruning/recovery (120 h), and conversion or analysis reruns (80 h).
Combinations receive no allocation until both isolated parents pass.

Suppose two finalists each require four seeds of 32 h:
`2×4×32=256 h`. The remaining 44 h of confirmation reserve can cover one
pre-classified failed seed or a near-boundary repetition. It cannot fund a new
method invented after results.

For device work, an example allocation of 240 h is:

- 36 h compatibility and placement checks;
- 72 h randomized short/cold/warm trials;
- 72 h sustained thermal and reliability sessions;
- 36 h iOS/ARM64 confirmation; and
- 24 h protected instrumentation/retry reserve.

**Interpretation.** Medium supports target-specific choices among credible
methods. Several seeds and device classes improve confidence, but results still
apply only to sampled versions, hardware, workloads, and dates.

### Research: mechanism, interactions, and external validity

- **Model/training:** expanded registered architectures; multi-teacher/aligned-logit work; order/factorial studies; QAT
  or joint methods only after registered PTQ/isolated-method triggers.
- **Deployment:** replicated key SKU/OS blocks, at least two units per key class where feasible, calibrated power
  instrumentation, multi-instance/load AWS studies, replicated ARM64/Jetson.
- **Repetition:** power- or sequentially determined seeds/blocks; independent challenge set or external institution
  where authorized.
- **Retention:** all parents, raw traces, failed artifacts, analysis environments, and external-validation evidence.
- **Claim boundary:** broader external validity only for sampled populations and dates.
- **Stop condition:** preregistered information-gain threshold, hard-gate failure, or resource cap—never “search until
  significant.”

#### Worked research-tier allocation

`ILLUSTRATIVE`: a research program protects 2,000 of 8,000 accelerator-hours
for independent confirmation and external challenge. The 6,000 h discovery
pool might allocate 1,500 h to architecture/scale blocks, 1,200 h to
distillation signals, 1,000 h to pruning and recovery, 1,000 h to factorial
order studies, 700 h to QAT/joint methods after triggers, and 600 h to
registered failure/retry classes.

A two-by-two order study with two methods, four cells, five seeds, and 45 h per
seed costs `4×5×45=900 accelerator-hours`. Adding three model families would
raise this to `2,700 h`, before confirmation. This shows why mechanism studies
still need bounded factors.

With 1,500 physical device-hours, suppose six key device classes receive two
units each and 80 h per unit: `6×2×80=960 h`. The remaining 540 h cover
calibration, replicated ARM64/Jetson work, failures, and external validation.

**Interpretation.** Research-tier evidence can test whether an effect persists
across sampled families and units. It does not justify claims about unsampled
devices or future software versions.

## Converting ceilings into per-experiment budgets

For experiment \(i\), define:

\[
B_i=(h_{accel}, h_{CPU}, n_{teacher}, h_{expert}, h_{device},
S_{retained}, h_{host}, h_{engineering}, n_{retry}).
\]

The program is authorized only when the sum of approved experiment allocations
plus protected reserves is no larger than the tier ceiling in every dimension.
For accelerator time:

\[
\sum_i h_{accel,i} + h_{accel,reserve} \leq h_{accel,tier}.
\]

Apply the same check independently to expert hours, device hours, storage, and
all other dimensions.

### Budgeting repeated trials

If one training configuration takes \(h\) hours, there are \(a\) arms and \(s\)
seeds, and the observed valid-run rate from a pilot is \(p\), a planning
estimate for attempted accelerator-hours is:

\[
H_{attempt} = \frac{a s h}{p}.
\]

`ILLUSTRATIVE`: two arms, three valid seeds per arm, 12 h each, and pilot
valid-run rate 0.80 require `2×3×12/0.80=90 h` of attempted capacity. This is a
planning estimate, not permission to retry indefinitely. The plan still needs
a fixed retry count and failure classification.

### Storage calculation

Retained storage includes parents, children, optimizer/checkpoint states when
required, raw outputs, traces, environments, and failed artifacts. If each of
six training attempts retains 35 GiB, four converted artifacts retain 8 GiB,
and traces require 20 GiB, the total is
`6×35 + 4×8 + 20 = 262 GiB`. Temporary conversion peaks must be budgeted
separately if they can exceed available workspace.

## Allocation order

Spend first on evidence capable of invalidating many downstream branches:

1. rights, privacy, provenance, source quality, and expert availability;
2. immutable splits, leakage audit, sealed access, safety truth, and metric fixtures;
3. model/tokenizer/template identity and one complete untouched baseline;
4. cheap compatibility, conversion, parity, and target load screens;
5. narrow quality/retention/safety screens;
6. multi-seed training and physical-device confirmation;
7. sustained energy/thermal work and combined optimization;
8. AWS/edge economics only after equivalent-use artifacts qualify.

## Cost policy: no fabricated prices

No dollar budget is stated here. Before approval, the finance/cloud custodian must capture:

- exact currency, AWS Region, service/instance/storage/network configuration, purchase model, tax/accounting boundary,
  and effective UTC date;
- dated official price-catalog records or allocated bills with immutable source hashes;
- measured or approved usage quantities, utilization, interruptions, and success denominator;
- expert and engineering labor rates only when authorized by the organization;
- device procurement/amortization, instrumentation, data licensing, storage, observability, distribution, and failed-run
  treatment.

The editable calculation is:

\[ C_{total}=C_{compute}+C_{storage}+C_{network}+C_{API/LB} +C_{observability}+C_{data}+C_{expert}+C_{engineering}
+C_{device/instrument}+C_{failure/rework}. \]

Report cost per attempted run, successful run, successful request, and one million successful generated tokens only
where the denominator is measured and appropriate. A stale quote, list-price screenshot without Region/date, paper cost,
or planning estimate cannot become `EXECUTED` project cost. Use `null` plus reason until evidence exists.

### Worked cost template

The following is `ILLUSTRATIVE` arithmetic, not an AWS quote or project budget.
If verified compute usage is 80 h at an effective `$2.40/h`, storage is
`$36`, network is `$18`, observability is `$12`, expert review is 16 h at an
approved `$90/h`, and engineering allocation is `$600`, then:

\[
C_{total}=80(2.40)+36+18+12+16(90)+600=\$2{,}298.
\]

If 9 of 12 attempted runs are valid, cost per attempted run is
`2298/12=$191.50`, while cost per valid run is `2298/9=$255.33`. Both
denominators are informative. Reporting only the lower number hides failure
cost. Replace every value with dated, approved evidence before use.

## Per-experiment authorization

Every [experiment plan](../06-experiment-framework/templates/experiment-plan-template.md) must state:

- primary and secondary resource caps with units;
- expected stage allocations and protected confirmation reserve;
- bounded retry classes and who may authorize them;
- automatic teardown, idle detection, artifact-retention, and temporary-storage cleanup;
- harm, futility, success, and budget stop rules;
- what is sacrificed if a cap binds, without changing the primary estimand after unblinding.

Resources actually consumed are copied from [run records](../06-experiment-framework/templates/run-record-template.md)
into the final [result ledger](../06-experiment-framework/templates/experiment-result-template.md). OOM, crashes,
rejected teacher samples, failed conversions, and invalid measurements are not free.

## Tier-selection gate

Choose the smallest tier able to answer the primary question with credible power and required safety review. Escalate
only through a pre-approved trigger: near-boundary pilot variance, a qualified candidate requiring confirmation, a newly
supported target lane, or external-validity replication. Tier escalation never rescues a failed safety/rights gate or
permits outcome-driven hypothesis changes.

Use these questions:

1. Can the tier fund the independent analysis units required by the power or
   precision rationale?
2. Can it review every critical safety item and enough ordinary items to
   estimate quality?
3. Can it test the exact deployment target rather than a proxy?
4. Does it preserve confirmation capacity after expected failure and retry?
5. Is the intended claim no broader than the sampled models, devices, and
   dates?

If any answer is no, narrow the claim, redesign the study, or move to a larger
tier before approval.

## Cross-links

- [Prioritized roadmap](prioritized-experiment-roadmap.md)
- [Open risks](open-risks-and-evidence-gaps.md)
- [Metric and cost specification](../06-experiment-framework/specifications/metric-specification.md)
- [AWS benchmark protocol](../06-experiment-framework/protocols/aws-inference-benchmark.md)
- [Edge benchmark protocol](../06-experiment-framework/protocols/edge-device-benchmark.md)
