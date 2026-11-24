<!-- markdownlint-disable MD013 -->

# Open risks and evidence gaps

## Status and use

**Status: OPEN-GAP REGISTER AUTHORED / CLOSURES NOT ESTABLISHED.** Every row below is unresolved unless its closure
evidence is attached to an approved experiment or release record. Documentation, paper, or vendor support does not prove
this project’s artifact works. Do not change a row to closed from narrative assurance alone.

This document is both a register and a testing guide. A risk is not merely
something that “might go wrong.” It is an unresolved assumption that can make a
result unlawful, unsafe, uninterpretable, irreproducible, or irrelevant to the
deployment decision. Each explanation below states the mechanism of failure,
why it matters, the practical test, and the evidence needed to close the gap.
All tests are planned; none is claimed as executed.

Risk states are `OPEN`, `MITIGATING`, `BLOCKED`, `ACCEPTED_FOR_BOUNDED_USE`, and `CLOSED_WITH_EVIDENCE`. Critical
rights, privacy, sealed-test integrity, identity, and safety gaps are non-acceptance release blockers.

## Evidence and data gaps

<!-- markdownlint-disable MD013 -->
| ID  | Open gap / consequence                                                                                                                                 | Required control and closure evidence                                                                                                     | Owner                           | Current state |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | ------------- |
| E1  | Training/evaluation source rights and redistribution terms are not approved; artifacts may be unlawful to train or ship                                | source-level rights records, approved use/redistribution decision, exclusions propagated to manifests, rights/privacy sign-off            | data-rights custodian           | OPEN          |
| E2  | Corpus quality, provenance, revision, contradictions, PII, and synthetic lineage are not established; model may learn unsupported or sensitive content | immutable source manifests; provenance/quality review; PII and contradiction dispositions; synthetic parent lineage and acceptance ledger | data custodian                  | OPEN          |
| E3  | Train/calibration/validation/sealed leakage and pretraining contamination are unknown; scores may be inflated                                          | root/entity/template/near-duplicate split audit, access logs, teacher/judge isolation, contamination analysis, fresh confirmation roots   | evaluation/data owners          | OPEN          |
| E4  | Domain benchmarks and reference answers lack demonstrated coverage and authority                                                                       | coverage matrix by task/risk/rare strata, authority/version records, qualified review, adjudication, rubric fixtures and agreement        | domain/evaluation owner         | OPEN          |
| E5  | MIDs, non-inferiority margins, release thresholds, and product preferences are unset                                                                   | documented stakeholder/risk rationale approved before unblinding; sensitivity plan and immutable target profile                           | product + safety + statistician | OPEN          |
| E6  | Statistical power and variance are unknown; negative results may be inconclusive                                                                       | pilot variance by root/seed/device block, power/simulation record, frozen repetition/analysis and missing-data policy                     | statistical owner               | OPEN          |
| E7  | Public benchmark contamination and evaluator/model-judge bias may invalidate generalization                                                            | private/fresh roots, human audit, judge calibration/blinding, disagreement and sensitivity reports                                        | evaluation owner                | OPEN          |
| E8  | No experiment has raw evidence; all method benefits remain hypotheses                                                                                  | approved plan, complete run/trial/failure bundles, immutable raw outputs, registered analysis, independent reviews                        | research lead                   | OPEN          |
<!-- markdownlint-restore -->
<!-- markdownlint-disable MD013 -->

### How evidence and data gaps fail experiments

**E1 — Rights and redistribution.** A technically good corpus can still be
unusable if its licence forbids training, derivative distribution, or the
intended commercial use. Test each source, not only the assembled dataset.
Closure requires a source-level decision, propagated exclusions, and sign-off;
a generic “publicly available” label does not close the gap.

**E2 — Corpus integrity.** Unknown provenance, stale revisions,
contradictions, PII, or synthetic ancestry can teach unsupported or private
content. Sample and profile every source class, trace synthetic records to
parents, run privacy/contradiction checks, and record dispositions. Immutable
manifests plus review evidence close the question for that snapshot.

**E3 — Leakage and contamination.** If one manual, scenario, or synthetic
descendant appears in both training and testing, a score can measure recall
rather than generalization. Group by source root, entity, template, and
near-duplicate cluster before splitting; isolate teachers and judges from
sealed roots. Access logs and a fresh-root confirmation show that selection did
not use the answer key.

**E4 — Benchmark authority and coverage.** A benchmark can be internally
consistent yet omit rare, dangerous, or region-specific cases. Build a coverage
matrix by task and risk, version every reference, blind qualified reviewers,
and test rubric fixtures. Closure means the intended-use claims map to reviewed
strata; it does not mean the benchmark covers all real-world knowledge.

**E5 — Thresholds and preferences.** Without a predeclared MID, margin, and
hard gate, any observed difference can be described as success after the fact.
Stakeholders, safety owners, and the statistician must approve thresholds and
target preferences before unblinding. Sensitivity analysis then shows whether
the decision is robust to reasonable preference changes.

**E6 — Power and variance.** Too few independent roots, seeds, devices, or
sessions can make a real effect indistinguishable from noise. Use pilots to
estimate clustered variance and simulate confidence-interval width or power.
Closure is a frozen repetition plan and missing-data policy justified by that
variance, not an arbitrary “three runs.”

**E7 — Public-test and evaluator bias.** Public benchmark items may have been
seen during pretraining, and a model judge may favor its own style. Use private
or fresh roots, calibrated blinded human review, judge-disagreement analysis,
and sensitivity to evaluator choice. Agreement across these checks narrows,
but does not eliminate, evaluator uncertainty.

**E8 — Absence of project evidence.** Papers and vendor claims motivate a
method but do not establish behavior for this tuple. The gap closes only for a
specific claim when approved plans, all attempts, raw outputs, registered
analysis, and independent reviews are retained. Until then, method benefits
remain hypotheses.

## Compatibility and deployment gaps

<!-- markdownlint-disable MD013 -->
| ID  | Open gap / consequence                                                                                              | Required control and closure evidence                                                                                                | Owner                     | Current state |
| --- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------- | ------------- |
| C1  | Exact checkpoints, tokenizer/template, licences, and repository revisions may change or disappear                   | late-bound immutable commits/manifests, local hash verification, licence review, archived permitted artifacts                        | model custodian           | OPEN          |
| C2  | Trainer/PEFT/quantizer/pruner/exporter/runtime combinations may reject architecture or silently alter outputs       | pinned environment; deterministic smoke; adapter/merge/export parity; operator and output-difference records                         | training/platform owner   | OPEN          |
| C3  | Claimed Android/iOS GPU/NPU execution may fall back partly or wholly to CPU                                         | compiled-graph/operator placement and processor-time trace; frozen fallback allowance; unsupported cells retained                    | mobile platform owner     | OPEN          |
| C4  | Nominal Q4/Q8 formats and sparse layouts may lack target kernels; smaller files may expand in RAM or run slower     | exact format/kernel/build record; load/post-load/peak bytes; latency and fallback measurements versus parent                         | compression/runtime owner | OPEN          |
| C5  | Structured pruning may not serialize valid smaller dense shapes or align with kernels                               | shape/count lineage, successful conversion, runtime operator evidence, parent-controlled target measurements                         | compression owner         | OPEN          |
| C6  | Distillation logits/features may be infeasible across tokenizers/architectures or teacher terms may prohibit use    | alignment validation, teacher rights, accepted-signal schema, infeasible status rather than proxy substitution                       | distillation owner        | OPEN          |
| C7  | Physical target devices, OS versions, battery condition, power instruments, or vendor toolchains may be unavailable | checked inventory, calibration/qualification records, replacement rule frozen before outcomes, affected claims narrowed              | lab owner                 | OPEN          |
| C8  | Device variability, OS pressure, thermal policy, and app integration may make one-phone results non-generalizable   | replicated named cohorts, device/day blocks, full condition hashes, app-level peak/failure tests, scoped external-validity statement | device-performance owner  | OPEN          |
| C9  | AWS instance availability, host variation, serving runtime, quotas, and Region differ at execution                  | approved Region/account, immutable infrastructure, quota check, multi-block traces, interruptions and teardown evidence              | cloud owner               | OPEN          |
| C10 | Artifact signing, packaging, download, staged update, rollback, and model/app compatibility are untested            | signed manifest, checksum and compatibility test, interrupted update, storage-peak and rollback drill                                | release/security owner    | OPEN          |
<!-- markdownlint-restore -->
<!-- markdownlint-disable MD013 -->

### How compatibility gaps are tested

**C1 — Mutable model identity.** Model aliases can point to new weights,
templates, or licences. Resolve immutable commits, hash every permitted
artifact, archive the governing licence, and fail closed on mismatch. This
ensures a later rerun uses the evaluated object.

**C2 — Toolchain interaction.** A trainer, adapter library, converter, and
runtime may each support a model separately but fail in combination. Run a
deterministic miniature pipeline and compare base, adapter, merged, exported,
and runtime outputs within registered tolerances. A pinned environment plus
parity evidence closes only that exact toolchain.

**C3 — Accelerator fallback.** A mobile runtime may advertise GPU or NPU
support while unsupported operators execute on CPU. Inspect compiled graphs,
processor placement, and processor-time traces under the exact model. A launch
message or enabled option is not placement evidence.

**C4 — Format versus execution.** Q4, Q8, and sparse names do not guarantee
compatible kernels. The runtime may unpack weights or add conversion overhead.
Measure file, post-load and peak bytes, startup, latency, and placement against
the parent. A smaller file closes no runtime claim by itself.

**C5 — Structured pruning portability.** Setting channels to zero is not the
same as removing dimensions. Inspect serialized tensor shapes and operator
dimensions, export the smaller graph, and benchmark it on target. Closure needs
both physical materialization and a useful measured benefit.

**C6 — Distillation alignment and terms.** Logits or features from different
tokenizers and architectures may not align, and teacher terms may restrict
derivative training. Test token/signal alignment on fixtures and obtain a
rights decision. If alignment is infeasible, record `UNSUPPORTED` rather than
quietly replacing the method with response imitation.

**C7 — Lab availability.** A planned device, OS, battery condition, toolchain,
or power instrument may not exist when needed. Inventory and qualify resources
before approval, calibrate instruments, and freeze substitution rules.
Substitution after seeing outcomes creates selection bias.

**C8 — Device variability.** One phone can differ because of silicon binning,
battery age, OS policy, RAM pressure, or background load. Block by
device/day/session, replicate key classes, and retain condition hashes and
app-level failures. Evidence supports only the sampled cohort and date.

**C9 — AWS environment drift.** Instance capacity, quotas, host hardware,
runtime versions, and regional services vary. Pin infrastructure, verify quota
and region, collect host metadata across blocks, and record interruption and
teardown. One successful host does not establish fleet-wide behavior.

**C10 — Delivery lifecycle.** A model that benchmarks locally may fail during
download, disk-pressure installation, app upgrade, or rollback. Test signed
manifests, checksum failure, interrupted transfer, peak temporary storage,
compatibility rejection, staged update, and rollback. Closure means the exact
release package survives the drill.

## Safety, security, and no-runtime-RAG gaps

<!-- markdownlint-disable MD013 -->
| ID  | Open gap / consequence                                                                                                               | Required control and closure evidence                                                                                                     | Owner                    | Current state |
| --- | ------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ------------- |
| S1  | Intended/prohibited use and risk tier are not approved; benchmark success could be overinterpreted                                   | signed scope/hazard analysis, user communication, escalation policy, non-compensatory release gates                                       | safety owner             | OPEN          |
| S2  | Pharmacy behavior may imply prescribing, dose changes, interaction clearance, substitution, or delayed emergency care                | qualified clinical review of all critical strata, correct escalation/refusal gates, bounded educational-use release decision              | pharmacy safety owner    | OPEN          |
| S3  | Automotive/industrial answers may bypass interlocks or mishandle brakes, steering, high voltage, lifting, pressure, or stored energy | qualified hazardous-repair review, applicability/uncertainty tests, critical-failure incident process                                     | repair safety owner      | OPEN          |
| S4  | Animal-care guidance may use wrong species/region/season, unsupported treatment, or delay veterinary escalation                      | domain-specific strata, qualified welfare/veterinary review, region/species limits and escalation gates                                   | animal-care safety owner | OPEN          |
| S5  | Multi-domain artifacts may substitute the wrong domain or let one domain’s average hide another’s failure                            | per-domain intersection–union gates, wrong-domain substitution and mixed-conversation tests, independent release authority                | multi-domain owner       | OPEN          |
| S6  | Training data/model artifacts may leak PII, memorized text, secrets, or sensitive facility details                                   | privacy/security review, memorization and extraction tests, restricted evidence handling, incident/rollback plan                          | privacy/security owner   | OPEN          |
| S7  | Supply-chain artifacts, converters, dependencies, or model files may be malicious or mutable                                         | trusted source policy, digest/signature verification, isolated conversion/load, dependency scan and software bill of materials            | security owner           | OPEN          |
| S8  | Without runtime retrieval, learned knowledge can be stale, incomplete, conflicting, or unrecalled                                    | dated scope, uncertainty/refusal behavior, expert escalation, update cadence, stale/out-of-scope tests, explicit residual-risk acceptance | product/safety owner     | OPEN          |
| S9  | Quantization/compression can cause rare critical regressions hidden by aggregate quality                                             | paired high-risk evaluation of exact release artifact, zero-tolerance rules where approved, uncertainty upper bounds and incident review  | evaluation/safety owner  | OPEN          |
<!-- markdownlint-restore -->
<!-- markdownlint-disable MD013 -->

Passing finite tests never proves factual completeness or zero risk. A model cannot be described as autonomous clinical
or hazardous-work authority.

### Why safety evidence is non-compensatory

**S1 — Undefined scope.** Without approved intended and prohibited use, a high
benchmark score can be promoted into an unsafe role. Write the hazard analysis,
escalation behavior, user communication, and release gates first. Closure is an
authorized bounded scope, not a claim of general safety.

**S2 — Pharmacy overreach.** A fluent answer can be mistaken for prescribing,
dose adjustment, interaction clearance, or emergency triage. Test adversarial
and ordinary critical strata with qualified clinical review, including correct
refusal and escalation. Any release remains educational and bounded unless a
different regulated evidence program authorizes more.

**S3 — Hazardous repair.** Incorrect sequences around high voltage, brakes,
steering, lifting, pressure, or stored energy can injure people even when most
facts are correct. Test applicability questions, uncertainty, prerequisites,
interlocks, and escalation with qualified repair reviewers. Critical unsafe
compliance triggers incident review rather than averaging into quality.

**S4 — Animal-care specificity.** Species, region, season, and veterinary
urgency change appropriate guidance. Build separate strata and verify that the
model asks for missing context, limits unsupported treatment, and escalates.
Closure is scoped to reviewed species/regions and does not generalize silently.

**S5 — Cross-domain substitution.** A joint model can answer a pharmacy
question with a mechanical pattern or let a large low-risk domain hide a small
high-risk failure. Use per-domain gates, wrong-domain prompts, and mixed
conversations. Every domain must pass independently.

**S6 — Sensitive information leakage.** Training can preserve PII, secrets,
copyrighted passages, or facility details. Run privacy filtering,
memorization/extraction challenges, access-control review, and incident
rollback drills. Finite tests reduce but cannot prove zero memorization.

**S7 — Supply chain.** Model files and converters can execute or load malicious
content, while mutable dependencies can change behavior. Use trusted sources,
digests/signatures, isolated conversion and loading, dependency scans, and a
software bill of materials. Re-export or dependency change reopens the gap.

**S8 — Learned-parameter limitations.** Without runtime retrieval, knowledge
can be stale, incomplete, conflicting, or simply not recalled. Test dated,
out-of-scope, conflicting, and stale scenarios; require uncertainty and expert
escalation; set an update cadence. Evidence supports bounded behavior, never
complete factual storage.

**S9 — Rare compression regressions.** Average score can remain stable while
quantization changes a small number of critical answers. Compare parent and
exact release child item by item on high-risk strata and calculate an upper
bound for critical failure. Approved zero-tolerance events stop release.

## Execution, resource, and decision gaps

<!-- markdownlint-disable MD013 -->
| ID  | Open gap / consequence                                                                                  | Required control and closure evidence                                                                                                                          | Owner                      | Current state |
| --- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | ------------- |
| X1  | Qualified experts may be unavailable or insufficiently independent                                      | named qualifications/roles, conflict review, scheduled capacity, blinded scoring and adjudication record                                                       | domain-program owner       | OPEN          |
| X2  | Compute, storage, device, teacher, and engineering capacity may not support confirmatory repetitions    | approved [tier](resource-budget-tiers.md), reservations, per-plan caps, protected confirmation reserve, explicit not-run policy                                | program owner              | OPEN          |
| X3  | Retry and failure classification could hide instability or inflate success                              | pre-classified bounded retry policy; immutable [failure records](../06-experiment-framework/templates/failure-report-template.md); all attempts in denominator | experiment owner           | OPEN          |
| X4  | Hardware/software drift can break comparability                                                         | content-addressed environment/hardware/workload records, drift checks, requalification trigger                                                                 | reproducibility owner      | OPEN          |
| X5  | Energy instrumentation and boundary may be invalid; battery percentage is insufficient for joules/token | instrument calibration, idle-matched boundary, synchronized raw traces, uncertainty and missing policy                                                         | performance-lab owner      | OPEN          |
| X6  | AWS and labor costs may drift or be fabricated from stale assumptions                                   | dated official price/bill hashes, measured usage, currency/Region/purchase model, approved labor allocation, sensitivity analysis                              | finance/cloud owner        | OPEN          |
| X7  | Cloud and edge may provide unequal quality/scope, invalidating economics                                | equivalent intended use and quality floor, feature/SLO/security boundary, conditional scenario report                                                          | product/economics owner    | OPEN          |
| X8  | Staged elimination may miss favorable interactions; broad search invites cherry-picking                 | one preregistered rescue trigger/cap, parent controls, independent confirmation; report staged survival bias                                                   | research/statistical owner | OPEN          |
| X9  | Pareto ranking may join unlike conditions or hide missing objectives                                    | target-specific profiles, hard-gate eligibility, physical units, uncertain dominance, missing as incomplete, preference sensitivity                            | analysis owner             | OPEN          |
| X10 | Deliverable ownership or approval handoffs may be ambiguous                                             | named role matrix in the approved plan, acceptance criteria, signed technical/statistical/safety/rights reviews                                                | program owner              | OPEN          |
| X11 | Monitoring, feedback, incident response, deprecation, and retraining are not proven                     | production runbooks, privacy review, canary/rollback drill, escalation and model/data version linkage                                                          | operations owner           | OPEN          |
| X12 | No method may satisfy all gates                                                                         | explicit no-qualified-candidate decision, failure coverage, residual options and scope reduction—never relaxed safety evidence after results                   | release authority          | OPEN          |
<!-- markdownlint-restore -->
<!-- markdownlint-disable MD013 -->

### How program and decision gaps distort conclusions

**X1 — Expert bottleneck.** Reviewers may be unavailable, conflicted, or too
few to adjudicate critical items. Verify qualifications, independence, blinded
roles, scheduled hours, and agreement procedures before generation or testing.
Unreviewed critical evidence cannot be replaced by model judging.

**X2 — Insufficient resources.** A study that can screen but not confirm may
produce attractive yet unstable winners. Select an approved
[resource tier](resource-budget-tiers.md), reserve confirmation capacity, and
define `NOT_RUN_BUDGET` handling. Spending more after results requires a
pre-approved trigger.

**X3 — Retry bias.** Retrying only failures and reporting the best successful
attempt hides instability. Classify retryable faults before execution, cap
retries, retain all failures, and include all eligible attempts in reliability
denominators.

**X4 — Environment drift.** Driver, runtime, firmware, OS, or workload changes
can make two runs incomparable. Hash environments and workloads, record
hardware, and define requalification triggers. If drift occurs, block or
separate the comparison rather than pooling it.

**X5 — Invalid energy boundary.** Battery percentage is coarse and includes
uncontrolled system activity. Calibrate instrumentation, synchronize power and
token traces, subtract a matched idle boundary, and propagate uncertainty.
Otherwise energy remains `null` with a reason.

**X6 — Stale or invented cost.** Prices depend on date, region, purchase model,
utilization, and accounting policy. Capture official dated records or bills,
measured usage, currency, labor policy, and sensitivity. A worked example or
paper price cannot become project cost.

**X7 — Unequal cloud/edge products.** Cloud may use a larger model or edge may
offer offline privacy, making cost-per-request comparison misleading. Establish
equivalent intended use, minimum quality, SLO, security, and feature boundaries
first. If equivalence fails, report conditional scenarios rather than a winner.

**X8 — Staged-search bias.** Early elimination can miss a beneficial
interaction, but testing every interaction invites cherry-picking. Pre-register
one plausible rescue trigger, eligible branch, cap, and independent
confirmation. Report that survivors were selected through earlier stages.

**X9 — Invalid Pareto ranking.** Pooling devices, contexts, or missing energy
can make one tuple appear non-dominated under incomparable conditions. Build
separate target profiles, apply hard gates first, use physical units and
uncertainty, and leave incomplete objectives missing.

**X10 — Ambiguous handoffs.** Data, model, safety, and release owners may assume
someone else approved a critical artifact. Name responsible and approving roles
with acceptance criteria in the plan. Signed records close the organizational
gap.

**X11 — Unproven operations.** A good offline benchmark says nothing about
monitoring, privacy-preserving feedback, incident response, deprecation, or
retraining. Exercise canary, rollback, escalation, and model/data version
linkage with production runbooks before release.

**X12 — No qualified candidate.** The research may honestly find that every
method fails at least one hard gate. Preserve failure coverage, issue an
explicit no-candidate decision, and consider narrower scope or different
hardware in a new plan. Never relax a safety threshold after seeing results.

## Closure protocol

A risk may move to `CLOSED_WITH_EVIDENCE` only when:

1. its owner links immutable evidence IDs/hashes and the applicable
   [run](../06-experiment-framework/templates/run-record-template.md),
   [failure](../06-experiment-framework/templates/failure-report-template.md), or
   [result](../06-experiment-framework/templates/experiment-result-template.md);
2. the stated closure criterion—not a weaker substitute—is met on the exact artifact, environment, and target in scope;
3. deviations, exclusions, failures, and residual uncertainty are recorded;
4. required technical, statistical, domain/safety, and rights/privacy approvers sign;
5. closure date, expiry/recheck trigger, and affected claims/releases are named.

`ACCEPTED_FOR_BOUNDED_USE` requires an authorized residual-risk statement, mitigations, monitoring, expiry, and
rollback. It is forbidden for unresolved rights, privacy breach, sealed leakage, identity mismatch, or critical safety
failure.

### Worked closure example

`ILLUSTRATIVE`: C4 cannot be closed by observing that a Q4 file is 40% smaller.
Assume the exact Android package is hashed, operator placement shows no
forbidden fallback, peak app memory falls from 3,000 MiB to 2,100 MiB, quality
is non-inferior, and sustained latency improves beyond its MID over 20 blocked
trials. Those records can close C4 for that model, runtime build, phone/OS,
workload, and date after review. A runtime upgrade or re-export reopens it.

If placement cannot be measured, the correct state is `BLOCKED` or
`ACCEPTED_FOR_BOUNDED_USE` only where policy permits and the residual risk is
explicit. Narrative confidence is not closure evidence.

## Recheck triggers

Reopen affected rows when any model/tokenizer/template/data/evaluator revision changes; a licence or source is revised;
runtime/kernel/OS/firmware/device changes; a new domain or intended use is added; safety incident or material regression
occurs; cost evidence expires; monitoring indicates drift; or an artifact is re-exported, compressed, merged, or
repackaged.

## Current closure summary

- Evidence/data gaps closed: **0**
- Compatibility/deployment gaps closed: **0**
- Safety/security gaps closed: **0**
- Execution/decision gaps closed: **0**
- Current release implication: **experiments and production release remain unqualified**

These counts are documentary status, not measured risk.

## Cross-links

- [Roadmap index](README.md)
- [Research questions](research-questions-and-hypotheses.md)
- [Prioritized roadmap](prioritized-experiment-roadmap.md)
- [Safety standard](../00-governance/safety-and-risk-standard.md)
- [Reproducibility standard](../00-governance/reproducibility-standard.md)
- [Artifact/provenance specification](../06-experiment-framework/specifications/artifact-and-provenance-specification.md)
- [Environment specification](../06-experiment-framework/specifications/hardware-and-environment-specification.md)
