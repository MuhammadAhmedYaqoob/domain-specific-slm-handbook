# Motorcycle technicians dataset guide

## Contract and status

**AUTHORED DOMAIN PROFILE / NOT EXECUTED OR RIGHTS-APPROVED.** Support education and supervised diagnostic reasoning. Do
not authorize riding, roadworthiness, racing modifications, emissions/safety defeat, hazardous live work, or exact
service values without current model-specific authority.

## Ontology and tasks

Cover two-/four-stroke and electric powertrains; single/multi-cylinder engines; cooling/lubrication; carburetion and
electronic fuel injection; ignition, stator, regulator/rectifier, batteries and charging; clutch and sequential
transmission; chain/belt/shaft final drive; chassis geometry; forks/rear suspension/preload and sag concepts;
conventional/combined/ABS brakes; tires/wheels; controls; fault diagnosis and repair verification.

Bind every record to make/model/year/market, engine/configuration, road/off-road use, stock/modified state, ABS variant,
final drive, and source revision. Preserve observation, hypothesis, test, cause, and action as separate concepts.

High-risk cells: brake contamination or imbalance, tire condition/direction/load/pressure, chain slack/alignment/guards,
steering-head/suspension faults, unstable lifting, fuel/hot surfaces, stored spring energy, batteries/high voltage, and
unsafe test riding.

## Sources and rights

Discover through NHTSA motorcycle recalls/complaints, applicable EPA certification data, rights-cleared manufacturer
portals/manuals/bulletins/wiring, regulator research, licensed vocational/OER material, and commissioned qualified
technicians. Forums supply vocabulary and candidate scenarios only after terms/privacy review.

Service manuals, diagnostic software, torque tables, and wiring are commonly controlled; acquire explicit rights. A
rider-safety page is not a service manual. Record source/version, authority scope, rights, restrictions, market,
platform/rebadge family, update signal, and objective.

## Dataset and separation plan

- Shared mechanical roots may teach physics, measurement, and diagnostic discipline.
- Motorcycle-specific SFT teaches two-wheel context, exact applicability, safe support, and ride-stop behavior.
- Multi-turn cases reveal modifications, ABS variant, tire state, or riding conditions after an initial hypothesis.
- Structured records extract platform/configuration, symptom, code, measurement, hazard, and next action.
- Preference pairs contrast scoped diagnosis with automotive/truck substitution.
- Calibration includes compact wiring, technical identifiers, unit-heavy text, dialogue, and long procedures.
- Final evaluation holds out full platform/rebadge, source, fault, and procedure families.

Cluster duplicated automotive/motorcycle explanations and shared-platform manuals before splitting. Keep synthetic
paraphrases with their source/case/template.

## Expert and safety gates

Qualified motorcycle technicians review technical records; braking, tire/chassis, electric high-voltage, and
competition-modification material requires scoped specialists and safety review. Reject invented
torque/pressure/clearance, wrong market/model transfer, ride authorization with unresolved brake/steering/tire hazards,
unapproved modifications, unsafe support/test sequence, or code-to-part replacement.

## Evaluation plan

General: component theory, fuel/ignition/charging, final drive, suspension, braking/ABS, electrical diagrams, units, and
next-test reasoning. Safety: ride-stop decisions, brakes/tires/steering, chain/wheel alignment, lifting, fuel, stored
energy, HV, wrong-platform traps, bypass pressure, multi-turn correction, and OOD platforms.

Measure domain substitution, factual/applicability/process quality, unsupported exact values, safe escalation,
over-refusal, contrast consistency, and critical failures. `PROPOSED POLICY`: zero defined continue-riding,
brake/tire/steering, lifting, or HV critical failures; thresholds otherwise require expert pilots.

## Updates and transfer experiment

Monitor rights, OEM revisions, recalls, regulations, and platform changes. Compare target-only independent model, common
base + motorcycle adapter, shared-mechanical CPT + adapter, and joint mechanic model at matched accepted data and
declared compute. No strategy is presumed superior; motorcycle, truck, and automotive safety suites remain independent.

## Knowledge structure and terminology

A motorcycle combines familiar engine and electrical principles with two-wheel stability, compact packaging, exposed
drive components, and configuration-sensitive chassis behavior. Represent the machine as linked powertrain, charging,
final-drive, brake, tire, steering, suspension, control, and modification states. Never transfer a car procedure merely
because a component name matches.

Preserve distinctions such as stator versus alternator assembly, regulator/rectifier, side-stand interlock, combined
braking versus ABS, chain slack versus chain wear, preload versus spring rate, static versus rider sag, cold versus hot
pressure, and road versus competition configuration.

For exact values, current applicable manufacturer material outranks general handbooks. Regulators establish recalls and
rules; standards establish definitions; qualified training and experts explain principles; owner reports discover
language and cases only. Check recalls and safety notices continuously, service revisions per release, and training
material annually.

## Illustrative dataset record

```json
{
  "record_id": "moto-case-start-0019",
  "machine": {
    "manufacturer": "Example Cycle Works",
    "model": "Trail 400",
    "year": 2025,
    "market": "US",
    "use": "road-legal dual sport",
    "modification_state": "unknown",
    "abs_variant": "equipped",
    "final_drive": "chain"
  },
  "observation": "starter turns slowly after several days parked",
  "known": ["starter rotation is slow", "storage interval is several days"],
  "unknown": ["battery state", "terminal condition", "charging history", "accessory modifications"],
  "hypotheses": ["low battery state", "connection resistance", "starter load"],
  "next_decision": "Clarify modifications and collect approved low-energy preliminary evidence before attribution.",
  "hazards": ["unstable support", "battery short-circuit current"],
  "source_ids": ["fictional-motorcycle-electrical-course-v1"],
  "root_family_id": "fam-moto-start-0019",
  "split": "train",
  "review_status": "accepted_for_education"
}
```

The fictional record includes unknowns deliberately; a good target asks for them instead of hallucinating a cause.

## Worked mini-pipeline

1. Map tasks across powertrain, charging, final drive, brakes, tires, steering, suspension, and controls.
2. Register rights-cleared current manufacturer, regulator, and educational sources by platform and market.
3. Extract warnings and specification context without detaching table footnotes.
4. Normalize units and identifiers, retaining conditions such as loaded/unloaded and cold/hot.
5. Cluster rebadges, platform-sharing manuals, revisions, translations, and copied forum passages.
6. Adjudicate conflicts by platform, modification, ABS option, market, and revision.
7. Reject records that omit support stability, exact applicability, or decisive safety context.
8. Split complete platform and fault families before generating dialogue or preference rows.
9. Render instruction records that distinguish evidence, hypothesis, and ride-stop escalation.
10. Obtain motorcycle-technician review; use tire, brake, chassis, and high-voltage specialists where applicable.
11. Evaluate on independent platforms and fault roots, with a separate sealed safety suite for brakes, tires, steering,
    support, chain, fuel, and high voltage.
12. Monitor recalls, service revisions, platform changes, and rights; rebuild affected descendants.

Evaluation separation matters especially in transfer experiments: automotive-derived training roots cannot reappear as
paraphrased motorcycle test items, and shared mechanical language must not hide motorcycle-specific safety errors.

## Cross-links

- [Dataset guide](../README.md)
- [CH-39 mechanic domains](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md)
- [Automotive profile](automotive-mechanics.md)
