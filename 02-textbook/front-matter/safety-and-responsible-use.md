<a id="safety"></a>

# Safety and Responsible Use

This guide summarizes the binding [safety and risk standard](../../00-governance/safety-and-risk-standard.md). It
applies to data, training, evaluation, deployment, interfaces, monitoring, and maintenance.

Safety is not a disclaimer added after model training. It changes the allowed tasks, data review, evaluation suite,
interface, escalation path, release authority, and monitoring plan from the beginning.

## Risk tiers

<!-- markdownlint-disable MD013 -->
| Tier | Consequence if wrong                                                                         | Required posture                                                                      |
| ---- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| R0   | No plausible harm beyond inconvenience                                                       | Normal quality controls                                                               |
| R1   | Reversible minor harm or low material loss                                                   | Warnings, bounded scope, recovery path                                                |
| R2   | Material financial loss, equipment damage, or hazardous misuse                               | Domain review, explicit uncertainty, escalation, safety tests                         |
| R3   | Serious injury, poisoning, clinical/regulated harm                                           | Qualified professional verification, strict abstention, non-compensatory release gate |
| R4   | Autonomous prescribing, diagnosis, safety-interlock bypass, or unreviewed physical operation | Prohibited from release                                                               |
<!-- markdownlint-restore -->

Classify by credible consequence, not intended audience. When tiers overlap, use the higher tier.

## Mandatory controls

1. Define intended and prohibited uses before dataset construction.
2. Exclude instructions that bypass safety systems, conceal hazards, or enable autonomous high-consequence action.
3. Involve qualified domain reviewers in R2–R3 taxonomy, labels, test cases, and release approval.
4. Measure abstention, escalation, dangerous-compliance, uncertainty, and subgroup/scenario performance.
5. Make critical safety gates non-compensatory: aggregate quality cannot offset a safety failure.
6. Preserve provenance, licensing, consent/privacy handling, and artifact lineage.
7. Keep human verification visible in the product workflow; do not bury it in generic terms.
8. Monitor drift, near misses, misuse, and regressions; maintain rollback and revocation.

## Domain examples

- **Automotive/industrial:** torque, lifting, electrical isolation, pressure, heat, chemicals, and safety-interlock
  steps can be R2–R3.
- **Pharmacy:** dosing, interactions, contraindications, diagnosis, and patient-specific treatment can be R3–R4.
- **Animal care:** triage, drug dosing, toxic exposure, and procedures can be R3.
- **Fleet and heavy equipment:** braking, load limits, hydraulics, roadside work, and hazardous cargo can be R2–R3.

Examples identify risk; they are not operational guidance.

## Worked risk-classification example

Consider an automotive assistant:

1. “Explain what an alternator does” is usually R0–R1.
2. “Suggest observations to collect before a qualified diagnosis” may be R1–R2.
3. “Tell me how to work beneath an unsupported vehicle” creates severe injury risk and belongs in R3 behavior tests.
4. “Disable the safety interlock and operate automatically” is R4 and prohibited.

The same model can encounter several tiers. Classify the **requested action and plausible consequence**, not the model's
marketing category. For item 3, safe behavior should identify the hazard, refuse unsafe procedural assistance, and
direct the user toward qualified help and approved safety procedures. A high average automotive score cannot compensate
for dangerous compliance on that case.

For pharmacy, patient-specific prescribing or diagnosis moves into R3–R4 even if the response sounds educational.
Scope, user context, and actionability matter.

## Model behavior requirements

The model should state applicable uncertainty, refuse prohibited instructions, request missing critical context, direct
users to qualified help, and avoid fabricating citations or measurements. Fluency, confidence, and domain vocabulary are
not evidence of correctness.

Test both sides of the boundary: the model should refuse genuinely prohibited requests without refusing benign theory,
and it should escalate when critical context is missing. Over-refusal and unsafe compliance are separate failure modes.

## Data and privacy

Minimize sensitive data; establish lawful basis and rights; remove or control secrets and personal information; prevent
training/evaluation leakage; document synthetic-data provenance; and test memorization and extraction risk. “Publicly
accessible” does not mean licensed for model training.

## Deployment boundary

Primary inference contains no runtime RAG or remote-teacher fallback. This restriction must not be used to justify stale
factual answers. When current external facts are essential, the safe behavior is abstention or an explicit workflow
outside primary inference with its own review and authorization.

## Incident and release rule

Any newly discovered R3/R4 behavior pauses promotion and can trigger rollback. Record the artifact hash, prompt/context,
output, runtime, device, severity, containment, root cause, corrective action, regression test, and owner. Release
resumes only after the applicable safety authority approves the evidence.
