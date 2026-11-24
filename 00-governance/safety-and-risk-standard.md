# Safety and Risk Standard

## Safety starts with plausible consequences

A specialist model can sound confident while being stale, incomplete, or wrong. In pharmacy, repair, and animal care,
that error can cause more than a poor answer. This standard scales review and model behavior to the harm a plausible
mistake could cause.

Every safety claim MUST name its intended use, domain, population, jurisdiction, model and runtime version, and
evaluation. A broad claim such as “safe for mechanics” is not acceptable.

## Assign the highest applicable risk tier

| Tier | Plausible consequence | Example | Minimum gate |
| --- | --- | --- | --- |
| `R0` | no meaningful harm | historical overview | technical review |
| `R1` | reversible inconvenience or low impact | conceptual maintenance explanation | technical review and limitations |
| `R2` | equipment damage, material loss, privacy/security impact, or non-urgent welfare harm | diagnostic sequence or animal-care guidance | qualified domain review and safety evaluation |
| `R3` | serious injury, poisoning, fire, critical failure, major human/animal harm, or regulated decision | drug interaction, brake repair, or high-voltage procedure | qualified domain expert and independent safety/regulatory reviewer |
| `R4` | autonomous action could cause death or serious harm, or exercises prohibited regulated authority | autonomous prescribing or unattended control of critical machinery | prohibited in this project |

When uncertain, choose the higher tier. A disclaimer does not reduce the consequence of an error.

### Worked classification

“Explain what an anti-lock braking system does” is generally `R1`: the reader receives conceptual education.

“Provide the exact steps to bypass an anti-lock braking fault and return the vehicle to service” is at least `R3`: an
incorrect or misused procedure could lead to serious injury. The request also conflicts with the prohibited repair
behavior below.

### Anti-example

> For educational use only. Follow these patient-specific dosing instructions.

The disclaimer does not neutralize the dosing risk or make autonomous prescribing acceptable.

## Define intended and prohibited use

Each specialist model MUST publish who it is for, what it may help with, excluded uses, required user qualifications,
operating conditions, known limits, geographic and temporal scope, and escalation routes.

- Pharmacy models must not prescribe, independently diagnose, select patient-specific treatment, or replace a licensed
  clinician or pharmacist.
- Repair models must not authorize bypassing lockout/tagout, guards, current service documentation, torque
  specifications, high-voltage isolation, brake/steering checks, or qualified inspection.
- Animal-care models must not replace veterinary assessment for urgent, severe, or uncertain cases.

`R4` functionality MUST NOT be built, presented as deployable, or released. `R3` material may be educational only after
authoritative sourcing, expert review, explicit limits, safe ordering, uncertainty behavior, and escalation pass.

## Record each serious hazard

Every `R2` or `R3` area needs an immutable ID such as `HAZ-AUTO-0012` in the
[safety ledger](../15-audit-ledgers/safety-and-domain-review-ledger.md). Record:

- the hazard, initiating condition, exposed person or system, severity, likelihood rationale, and tier;
- intended use and foreseeable misuse;
- affected data, content, model, runtime, and versions;
- preventive, detection, and mitigation controls;
- test cases and hard acceptance thresholds;
- reviewer qualifications and independence;
- residual risk, owner, expiry, release decision, incident link, and rollback path.

A high average score cannot cancel one catastrophic failure. Safety thresholds for critical cases are
non-compensatory.

## Control data and ground truth

High-risk training and evaluation records MUST preserve source, license, jurisdiction, version date, validity period,
and qualified review. Current authoritative guidance outranks forum or synthetic material. Check synthetic high-risk
facts against authoritative ground truth; agreement between two models is not validation. Quarantine contradictions
until an expert resolves them.

Evaluation must include unsafe requests, missing context, contraindications, shortcuts, prompt attacks, distribution
shifts, and cases where escalation is the correct response. Minimize sensitive data, control access, and apply approved
retention and deletion rules. The [security chapter](../02-textbook/part-12-production-engineering-safety-and-maintenance/ch-42-model-security-and-reliability.md)
adds supply-chain and privacy controls but does not replace domain safety review.

## Required model behavior for `R2` and `R3`

The system MUST:

- ask for or identify missing critical context without inventing it;
- distinguish education from a safe-to-act instruction;
- communicate uncertainty;
- refuse prohibited actions and unsafe shortcuts without enabling them;
- prioritize immediate hazard containment and qualified escalation;
- never invent a citation, dose, measurement, part specification, or procedure;
- preserve warnings when output is shortened or streamed; and
- be evaluated for unsafe compliance, missed escalation, critical failures, and excessive refusal using the
  [metric specification](../06-experiment-framework/specifications/metric-specification.md).

## Review and release workflow

1. Assign the risk tier and intended-use boundary.
2. Create hazard records and hard acceptance criteria.
3. Review sources, rights, contradictions, privacy, and evaluation coverage.
4. Test safe responses, refusals, and escalation.
5. For `R2`, obtain qualified domain review, independent where practical.
6. For `R3`, obtain approval from a qualified domain expert and an independent safety or regulatory reviewer on the
   same immutable candidate.
7. Verify integrity, no-runtime-RAG compliance, rollback, monitoring, and user disclosures.

Any critical unresolved defect blocks release. Approval expires when guidance, jurisdiction, intended use, hazards, or
the model, data, or runtime version changes.

Harm reports require an incident ID, severity triage, evidence preservation, version analysis, containment,
notification, remediation, regression testing, and closure review. Critical `R3` failures require a release hold or
rollback. Keep incident history after a fix.

This policy is authoritative. It does not imply that any model or domain content has passed expert review.
