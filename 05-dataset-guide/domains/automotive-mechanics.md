# Automotive mechanics dataset guide

## Contract and status

**AUTHORED DOMAIN PROFILE / DATASET NOT BUILT / RIGHTS NOT APPROVED.** Intended use is technical education and
non-authoritative diagnostic sequencing for appropriately supervised users. Prohibited: declaring a vehicle safe,
authorizing hazardous work, selecting a replacement from a code alone, defeating safety/emissions systems, or inventing
model-specific specifications.

## Ontology and coverage

Cover vehicle identity (make/model/year/market/configuration), combustion/hybrid/electric powertrains,
cooling/lubrication, fuel/emissions, OBD-II and manufacturer diagnostics, starting/charging, networks, electrical
measurement, transmission/driveline, steering/suspension, hydraulic/electronic braking, tires/wheels, HVAC/refrigerant,
restraints, maintenance, recalls, and repair verification.

Relations distinguish
`observation → monitored condition/code → hypothesis → discriminating test → confirmed cause → repair → verification`.
Every specification binds units, reference condition, source revision, and exact applicability. A code is evidence,
never a confirmed failed part.

Coverage axes: subsystem × powertrain × passenger/light-commercial × model family/year/market × symptom/code/measurement
× intermittent/constant × task × risk × difficulty. Rare/high-risk cells include high voltage, brakes, steering,
restraints, lifting/support, fuel, refrigerant, rotating/hot components, torque-critical fasteners, road testing, and
incomplete applicability.

## Source discovery and rights

Candidate entry points, not blanket approvals:

- NHTSA datasets/APIs, recalls, investigations, and vPIC for U.S. identity/recall evidence;
- EPA and eCFR Title 40 for applicable emissions/OBD regulation;
- licensed SAE/ISO standards catalogs;
- rights-cleared current OEM service information, owner manuals, bulletins, wiring and training;
- accredited curricula, licensed OER, research, and contracted technician cases;
- permitted forums only for terminology and case discovery.

Regulator data do not provide a complete repair procedure; complaints are reports, not verified diagnoses. OEM portals
and paid standards usually require explicit permission. Record VIN/model applicability, revision, effective date,
rights, restrictions, attribution, update signal, source family, and objective.

## Canonical record extensions

Automotive records add vehicle scope, powertrain and component configuration, complaint, codes, observations,
measurements, hypotheses, hazards, and procedure applicability to the common canonical record. The populated case later
in this chapter demonstrates these fields. Unknown scope remains unknown. VINs linked to owners and
customer/employee records are sensitive.

## Objective views and leakage

- CPT: approved technical explanations and current scoped documents.
- SFT: evidence-versus-hypothesis, safe clarification, units, uncertainty, escalation, and verified response formats.
- Multi-turn/diagnostic: reveal measurements incrementally; reward safe information gain and retraction.
- Structured: vehicle identity, code/measurement, warning, applicability, and evidence extraction.
- Preference: safe scoped reasoning versus code-to-part, wrong-model, bypass, or universal-refusal candidates.
- Calibration: representative codes, units, wiring text, long procedures, dialogues, and hazards.
- Evaluation: independent source/model/platform families, time-forward recalls, and sealed expert cases.

Keep editions, mirrors, bulletins, translations, procedure families, shared platforms, and synthetic descendants
together. Hold out complete OEM/model/year and fault families; do not random-split chunks.

## Quality, expert, and safety gates

Qualified automotive technicians review ordinary diagnostics; high-voltage, brake, restraint, refrigerant, and other
specialized records require scoped specialists. Safety reviewers verify prerequisites, isolation/support, stop
conditions, and escalation. Rights reviewers approve each source family.

Reject records with invented exact values, wrong variant, missing decisive isolation/support, automatic replacement from
a code, unsafe continued operation, bypass instructions, unresolved current/superseded conflict, or absent lineage.

## Evaluation and release policy

General suite: system theory, terminology, code interpretation, electrical reasoning, next-test selection, structured
extraction, and repair verification. Safety suite: brakes/steering/tires, high voltage, lifting, fuel, restraints,
refrigerant, torque-critical work, unsafe road tests, adversarial bypass, OOD model family, and multi-turn correction.

Report factual/applicability/process scores, unsupported claims, premature replacement, necessary-question recall, safe
escalation, over-refusal, contrast consistency, worst subgroup, and critical failures. `PROPOSED POLICY`: zero defined
critical failures; no general average overrides safety or wrong-variant exact-value failure. Thresholds require expert
calibration.

## Updates

Monitor source rights/terms, OEM revisions/bulletins, recalls, regulations, identifiers, and incident findings. Urgent
safety changes trigger review and can block releases. Every dataset/model exposes cutoff, markets, covered
configurations, and limitations.

## How to think about automotive knowledge

Automotive knowledge is a configuration-dependent causal graph, not a list of codes and parts. The same symptom can
arise from power supply, wiring, communication, control logic, a sensor, an actuator, or a mechanical condition. A
useful record therefore separates complaint, observation, code, hypothesis, test, result, confirmed cause, repair, and
post-repair verification.

Terminology must preserve `DTC` versus symptom, `provisional` versus confirmed code, `commanded` versus measured value,
open circuit versus high resistance, ground reference versus chassis metal, and generic OBD meaning versus
manufacturer-specific procedure. Normalize units into typed fields but retain the raw display.

Use a source hierarchy by claim: current applicable OEM information for model-specific procedures and values;
regulators for recalls and rules; standards for protocol definitions; qualified curricula and experts for general
theory; field reports only as candidate scenarios. Review recalls and safety notices continuously, OEM revisions at
each snapshot, and general theory on a scheduled annual cycle.

## Illustrative dataset record

```json
{
  "record_id": "auto-case-12v-0042",
  "vehicle_scope": {
    "make": "Example Motors",
    "model": "Harbor",
    "model_year": 2024,
    "market": "US",
    "powertrain": "conventional gasoline",
    "engine_motor_transmission": ["2.0 L engine", "6-speed automatic"]
  },
  "diagnostic_state": {
    "complaint": "charging warning lamp flickers at idle",
    "codes": ["P0622 stored"],
    "observations": ["lamp is intermittent", "no recent repair reported"],
    "measurements": [],
    "hypotheses": ["control-circuit connection", "wiring fault", "charging-unit fault"]
  },
  "preferred_next_action": "Verify exact OEM applicability and inspect approved preliminary evidence before component attribution.",
  "hazards": ["rotating components", "battery short-circuit current"],
  "procedure_applicability": "illustrative_only",
  "source_ids": ["fictional-oer-auto-charging-v2"],
  "root_family_id": "fam-auto-case-12v-0042",
  "split": "train",
  "review": {"role": "qualified_automotive_technician", "decision": "accepted_for_education"}
}
```

The make and model are fictional. The record teaches state representation, not a voltage limit or repair procedure.

## Worked mini-pipeline

1. Define coverage cells for charging theory, identifiers, observations, safe tests, and verification across
   conventional, hybrid, and electric variants.
2. Register an approved general-theory source and separate current OEM sources; record their different authority.
3. Extract warnings, code definitions, tables, and procedure steps with page/section locators.
4. Normalize code typography and units while preserving raw strings and reference conditions.
5. Deduplicate mirrored manuals and cluster shared platform, edition, translation, and bulletin families.
6. Convert statements to scoped claims; route apparently conflicting values by model, market, and revision.
7. Score extraction and applicability; reject invented limits and incomplete configurations.
8. Assign complete platform/case families to train or validation before producing dialogues.
9. Render CPT theory, SFT clarification, structured observations, and preference contrasts separately.
10. Have a qualified technician review ordinary records and a scoped specialist review high-voltage material.
11. Build final evaluation from independent platform and fault families, including code-to-part traps and safe
    escalation; never use those roots for generation or calibration.
12. Publish a manifest with cutoff and covered configurations; monitor recalls, bulletins, and rights changes.

Verification means checking every model-specific claim against the exact current source and having the appropriately
qualified reviewer approve its scope. The dataset and model do not determine roadworthiness.

## Cross-links

- [Dataset guide](../README.md)
- [CH-23 source plan](../../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-23-finding-domain-specific-training-data.md)
- [CH-30 evaluation](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
