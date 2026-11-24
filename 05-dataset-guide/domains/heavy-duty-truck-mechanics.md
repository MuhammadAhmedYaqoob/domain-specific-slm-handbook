# Heavy-duty truck mechanics dataset guide

## Contract and status

**AUTHORED DOMAIN PROFILE / DATASET AND RIGHTS UNAPPROVED.** Intended use is education and supervised diagnostic
sequencing. Prohibited: compliance or roadworthiness determination, movement authorization, brake/emissions/safety
bypass, hazardous pressure release, or exact configuration-specific procedure without current authority.

## Ontology and coverage

Model the vehicle as a graph: chassis/VIN class, engine family/calibration, transmission, axles, wheel ends, brake
controller/foundation brakes, aftertreatment, steering/suspension, body/upfit/hydraulics, tractor–trailer coupling,
electrical architecture, telematics, duty cycle, load, and jurisdiction.

Cover diesel/alternative/electric powertrains; high-pressure fuel/turbocharging; cooling/lubrication; EGR/DPF/SCR/DEF;
regeneration and derate concepts; transmission/driveline; SAE J1939 concepts; SPN/FMI/occurrence/lamp/conditions;
pneumatic brakes; hydraulics; preventive maintenance and inspection records. Codes and derates are evidence, not
confirmed causes.

High-risk cells: air/spring brakes and stored pressure, wheel/tire energy, wheel ends, coupling/trailer, steering,
unsupported/elevated bodies, hydraulics, roadside work, aftertreatment heat, high-pressure fuel, multi-battery current,
alternative fuel, and high voltage.

## Source portfolio

Candidate entry points include FMCSA open data and current Part 396 materials, eCFR Title 49, NHTSA/FARS, EPA heavy-duty
regulation/research, licensed SAE J1939 and related standards, and rights-cleared OEM/component/body-builder
documentation. Operational/crash/violation data prioritize hazards; they are not repair instructions. Paid standards and
service portals require explicit rights.

Register component/chassis compatibility, engine calibration, body/upfit/trailer, jurisdiction, revision, authority
scope, rights, restrictions, update cadence, and split family. Fleet records may contain personnel, customer, location,
and confidential configuration data.

## Objective and leakage plan

- CPT: approved heavy-duty technical language with configuration/version tags.
- SFT: safe evidence ordering, J1939 scope, pneumatic/hydraulic concepts, uncertainty, escalation.
- Multi-turn: reveal trailer, load, upfit, lamp state, occurrence, or operating condition later.
- Diagnostic: next safe discriminating action, not component guessing.
- Structured: vehicle graph, SPN/FMI, measurements, conditions, hazards, records.
- Preference: reject emissions/brake bypass, movement pressure, wrong light-vehicle assumptions.
- Calibration: technical identifiers, tables, units, long histories, dialogues.
- Evaluation: hold out fleets, component/OEM families, model years, fault roots, and later effective rules.

Keep OEM/component combinations, revisions, fleets, events, and synthetic descendants together. Separate truck data from
superficially similar automotive hydraulic/OBD examples.

## Expert and safety gates

Use qualified heavy-duty technicians; air-brake, wheel-end, hydraulics, alternative-fuel, high-voltage, aftertreatment,
and jurisdictional records need scoped specialists. Compliance reviewers interpret applicable requirements; the model
does not.

Reject unsafe pressure/stored-energy exposure, brake or emissions defeat, invented thresholds, wrong configuration,
unsafe movement authorization, unsupported roadside action, or absent qualified review.

## Evaluation and release

General suite: diesel/aftertreatment/J1939, driveline, pneumatics/hydraulics, configuration graph, records, and
diagnostic reasoning. Safety suite: air/spring brakes, wheel ends, coupling, movement, hydraulics, elevated loads,
fuel/HV, aftertreatment heat, adversarial bypass, OOD fleet/component, and multi-turn configuration changes.

Report per tractor, tractor–trailer, vocational/upfit, and powertrain group. Measure substitution, applicability, safe
sequence, escalation, unsupported claims, and critical failures. `PROPOSED POLICY`: all pneumatic-brake and
movement-permission critical cases are non-compensatory with zero defined critical failures.

## Updates

Monitor regulations/guidance, recalls, OEM/component revisions, calibration/firmware, standards, terms, and incident
evidence. Any material brake, movement, or aftertreatment change triggers expedited review and descendant analysis.

## Knowledge structure and terminology

Heavy-duty data must describe a vehicle configuration graph. Chassis, engine, transmission, brake controller,
aftertreatment, axle, trailer, body upfit, calibration, duty cycle, and jurisdiction can come from different
manufacturers and revisions. A flat “truck model” field is insufficient.

Preserve SPN, FMI, occurrence count, lamp state, operating condition, and controller source as separate fields. Do not
translate a J1939 diagnostic message into a failed part. Distinguish service brake, parking brake, spring brake,
foundation brake, supply pressure, application pressure, commanded state, measured state, derate, and shutdown.

Current component/OEM documentation establishes configuration-specific procedures; regulators establish applicable
rules; licensed standards establish protocol semantics; incident data identify hazards but do not prove repair causes.
Check brake and movement-related changes immediately, calibration and service revisions per snapshot, and background
educational material annually.

## Illustrative dataset record

```json
{
  "record_id": "truck-case-j1939-0034",
  "configuration": {
    "chassis": "Example Class 8 tractor",
    "engine_family": "Example Diesel X13",
    "transmission": "automated manual",
    "trailer": "not connected",
    "aftertreatment": "DPF-SCR",
    "market": "US",
    "calibration": "unknown"
  },
  "diagnostic_message": {
    "spn": 4364,
    "fmi": 18,
    "occurrence_count": 2,
    "lamp_state": "amber",
    "operating_condition": "reported under load"
  },
  "known": ["controller reported a below-normal condition"],
  "unknown": ["exact calibration", "freeze-frame values", "related messages", "current derate state"],
  "prohibited_inference": "failed component identified",
  "preferred_next_action": "Resolve exact engine/calibration applicability and current OEM diagnostic sequence.",
  "hazards": ["hot aftertreatment", "roadside exposure"],
  "root_family_id": "fam-truck-j1939-0034",
  "split": "train",
  "review_status": "illustrative_only"
}
```

The identifiers are used to demonstrate schema structure. The record does not supply a repair or movement decision.

## Worked mini-pipeline

1. Build a configuration ontology that links chassis, engine, transmission, brakes, axles, trailer, and upfit.
2. Register current regulator, standards, OEM, component, and training sources with exact rights and authority.
3. Extract message definitions, conditions, warnings, and procedure dependencies with structural locators.
4. Normalize pressure, temperature, torque, and identifier fields without losing raw units or conditions.
5. Cluster component manuals reused across chassis, fleet events, revisions, and synthetic derivatives.
6. Model apparent conflicts by calibration, duty cycle, jurisdiction, effective date, and component combination.
7. Score applicability completeness; quarantine cases missing a decisive controller or configuration field.
8. Assign whole fleet/event/configuration families before objective rendering.
9. Create SFT and diagnostic rows that prioritize safe information gathering, never brake bypass or movement authority.
10. Route records to qualified heavy-duty, air-brake, wheel-end, hydraulic, high-voltage, or compliance reviewers.
11. Evaluate on independent fleets and component families; seal air-brake, coupling, movement, wheel-end, and stored
    energy cases from all training and calibration.
12. Track rule, calibration, recall, and bulletin changes and trace each update through descendants.

Verification is configuration-specific. A plausible answer based on a different engine, controller, or trailer is a
wrong answer even if its general mechanical explanation is sound.

## Cross-links

- [Dataset guide](../README.md)
- [CH-39](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md)
- [Motorcycle profile](motorcycle-technicians.md)
