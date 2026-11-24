# Safety and domain review ledger

## What is checked and why

This ledger checks whether each specialist domain has clear intended-use limits, hazard and escalation behavior,
authoritative ground truth, risk-weighted tests, and review by people qualified for that domain. It matters because a
model can score well on average while giving one dangerous instruction about brakes, medication, high voltage, animal
welfare, or stored mechanical energy.

`Authored controls` means the book teaches safeguards. `Qualified review passed` would mean named experts approved
frozen content, data, tests, and the exact model artifact. **Qualified review is not yet performed for any domain.**

## Domain review records

<!-- markdownlint-disable MD013 -->
| Domain and principal risks | Controls learners can find | Current review record | Complete validation method |
| --- | --- | --- | --- |
| Automotive, motorcycle, and heavy-truck repair: brakes, steering, lifting, fuel/pressure, restraints, high voltage, hot/rotating parts | Hazard-first limits in [CH-37](../02-textbook/part-10-complete-hands-on-project/ch-37-build-an-automotive-mechanic-slm-end-to-end.md) and [CH-39](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md) | Not yet performed: no qualified repair, OEM, EV/high-voltage, or commercial-vehicle approval; no artifact tested | Named experts approve authoritative data, intended use, rubrics, critical cases, escalation wording, and exact outputs; predefined critical failures must be zero |
| Pharmacy education: contraindications, interactions, dosing misuse, stale regulation, clinical overreach | Non-prescribing boundary, uncertainty, interaction recognition, and escalation in [CH-38](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-38-pharmaceutical-knowledge-slm.md) | Not yet performed: no pharmacist/clinician, regulatory, or rights approval; update process not exercised | Licensed professionals and regulatory/rights reviewers examine frozen sources, content, benchmark, model, high-risk outputs, intended use, and update cadence |
| Beekeeping: disease, treatment, chemical use, and regional rules | Regional limits, uncertainty, and escalation in [CH-40](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md) | Not yet performed: no apiculture or veterinary review | Region-qualified experts approve source set and disease/treatment/escalation suite; test exact artifact across seasonal and regional cases |
| Zookeeping, animal care, and husbandry: species welfare, feeding, habitat, and veterinary delay | Species-specific uncertainty and veterinary escalation in CH-40 | Not yet performed: no keeper, veterinary, or welfare review | Scope species and jurisdictions; experts approve ground truth, welfare-critical cases, refusal/escalation, and exact outputs |
| Agricultural equipment: crushing, rollover, hydraulic pressure, PTO, electrical and chemical exposure | Domain data path plus repair hazard principles | Not yet performed: no equipment or machinery-safety review | Qualified technicians and safety specialists define equipment scope, isolation rules, critical suite, and escalation; evaluate exact model |
| Industrial machinery and manufacturing: lockout/tagout, stored energy, rotating and electrical hazards | Production and safety chapters discuss isolation and escalation | Not yet performed: no industrial or functional-safety review | Qualified reviewers set jurisdiction and machine boundaries, verify stop/escalation behavior, and approve critical tests and artifact |
| Electronics and electrical repair: shock, arc, fire, batteries, and stored charge | Scope and hazard limits are taught | Not yet performed: no qualified electrical-safety review | Define voltage/equipment classes and isolation rules; qualified experts approve data, procedures, tests, and exact model outputs |
| New or other domains | Risk-tier and onboarding gates in [CH-41](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-41-new-domain-onboarding.md) | Not yet performed; configuration alone cannot qualify a domain | Before data collection or training, name intended use, risk tier, jurisdiction, experts, authoritative truth, incident process, thresholds, and rollback |
<!-- markdownlint-restore -->

## Passing and failing evidence

**Pass:** a licensed pharmacist reviews a hashed pharmaceutical source set, sealed benchmark, exact model artifact, and
all high-risk outputs; findings are corrected; retests pass; the signed record states intended use, residual risk,
expiry, and artifact hash.

**Fail:** the chapter includes a disclaimer and the model achieves a high average score, but no qualified expert
reviewed contraindication errors or escalation behavior. A disclaimer cannot compensate for unsafe functionality.

## Cross-domain gaps

No real project data set has complete rights, PII, poisoning, contradiction, contamination, and expert-content
approval. No sealed safety suite, threshold, subgroup analysis, or expert-agreement record exists. No trained or
compressed artifact has been tested, even though conversion can change refusal behavior. No cloud/mobile incident,
revocation, rollback, or disconnected-update drill has run.

The no-runtime-RAG requirement remains truthful and important: learned parameters may be stale or hallucinate, so
versioned releases, conservative limits, and authoritative verification outside generation are essential.

## Release stop and responsibility

Unknown rights, prohibited high-risk functionality, one predefined catastrophic instruction, missing mandatory
escalation, or absent expert approval blocks release regardless of aggregate score. Responsibility is unassigned until
named reviewers accept it. Every review must record qualifications, immutable content/data/artifact hashes, date,
findings, remediation, retest, residual risk, expiry, and signed disposition.
