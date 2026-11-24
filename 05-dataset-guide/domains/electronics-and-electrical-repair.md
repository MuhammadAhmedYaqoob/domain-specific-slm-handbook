# Electronics and electrical repair dataset guide

## Contract and status

**AUTHORED DOMAIN PROFILE / NOT EXECUTED OR RIGHTS/SAFETY APPROVED.** Support circuit education, observation/measurement
interpretation, and supervised fault isolation. Prohibited: unqualified energized work, safety-barrier defeat, repair
authorization for hazardous devices, or exact model-specific values without current service authority.

## Ontology and coverage

Cover circuit laws/topology; passive/active components; analog/digital/power electronics; power supplies;
sensors/actuators; schematics; instruments/probes; soldering/rework and ESD; batteries/charging; embedded/firmware
context; fault isolation; repair verification and repairability.

Every record binds device/model/revision, voltage/energy class, AC/DC, earth/reference topology, isolation class,
measurement category, instrument/probe rating, battery chemistry/state, firmware, source revision, and qualification.
Separate observation, measurement, hypothesis, verified cause, repair, and verification.

Critical hazards: mains/high voltage, arc flash, stored charge, incorrect grounding/reference, isolation barriers,
damaged lithium cells/thermal runaway, lasers, RF/microwave, CRT/high voltage, fire, and unsafe rework.

## Sources and rights

Candidate entry points include NIST publications, OSHA electrical materials, CPSC recalls, rights-cleared manufacturer
datasheets/application notes/service documents, licensed IEC/IEEE/NFPA standards, licensed OER, and contracted qualified
experts. Community repair guides require page/media-level license, attribution, privacy, and third-party review. Public
editability is not training permission.

Datasheets may permit reading but restrict redistribution; service manuals and standards commonly require explicit
rights. Register exact revision, safety notices, authority scope, rights/restrictions, update trigger, split family, and
objective.

## Dataset plan

- CPT: approved theory and scoped technical text.
- SFT: measurement reasoning, safe prerequisites, uncertainty, escalation, and verification.
- Multi-turn: energy/isolation/reference/instrument rating or device revision appears later.
- Diagnostic: choose safe discriminating measurements after establishing safe state.
- Structured: schematic entities, measurements/units, ratings, observations, hazards, applicability.
- Preference: safe scoped response versus unrated probing, barrier defeat, wrong reference, or part guessing.
- Calibration: symbols, equations, net names, units, code, tables, long schematics-as-text, dialogues.
- Evaluation: hold out device/schematic/OEM/revision, fault, and source families.

Keep datasheet/manual revisions, copied schematics, repair-guide mirrors, and synthetic variants together.

## Expert and safety gates

Qualified electronics technicians/engineers and electrical-safety specialists review technical/safety content; battery,
RF, laser, microwave, high-voltage, and arc-flash items require scoped specialists. Reject energized hazardous probing
without prerequisites, unrated instruments, unsafe discharge/shorting, damaged-cell charging, isolation-barrier defeat,
invented component values/revision, or missing escalation.

## Evaluation and release

General: theory, components, schematic interpretation, measurements, analog/digital/power diagnosis, solder/rework,
structured extraction. Safety: energy state, instrument category/rating, earth/reference, stored charge, lithium damage,
laser/RF/microwave/CRT/fire, wrong-revision traps, adversarial live-work/bypass, and multi-turn correction.

Measure field/unit exactness, applicability, safe sequence, unsupported claims, diagnostic utility, escalation/refusal,
over-refusal, OOD schematic transfer, and critical failures. `PROPOSED POLICY`: zero defined energized-work,
instrument-rating, stored-charge, isolation, or damaged-cell critical failures.

## Updates

Monitor recalls, manufacturer revisions, component notices, standards/regulations, firmware, battery/safety guidance,
rights, and incidents. Safety-relevant changes trigger expedited review and affected-descendant identification.

## Knowledge structure and terminology

Electronics knowledge links schematic topology, component behavior, energy state, reference node, instrument capability,
observations, measurements, and device revision. A voltage without two reference points and conditions is incomplete.
A safe low-voltage technique must not be transferred to mains, high-energy batteries, RF, or isolated circuits.

Preserve ground, earth, chassis, signal reference, and isolated return; nominal, measured, tolerance, and limit;
powered-off, de-energized, discharged, and verified safe; CAT rating, voltage rating, probe rating, and bandwidth.
Separate suspected component from measured evidence and confirmed cause.

Current manufacturer service information governs exact device procedures; datasheets govern their exact part revision;
safety authorities and standards define scoped hazards and ratings; OER and experts teach theory. Monitor recalls and
battery notices continuously, service/firmware revisions per release, component notices on change, and theory annually.

## Illustrative dataset record

```json
{
  "record_id": "electronics-power-0048",
  "device": {
    "type": "fictional bench controller",
    "model": "Example BC-24",
    "revision": "B",
    "supply": "24 V DC nominal",
    "isolation_class": "unknown",
    "firmware": "2.1"
  },
  "observation": "device resets when a motor output is enabled",
  "measurement": {
    "quantity": "supply voltage",
    "value_raw": "not measured",
    "reference_nodes": null,
    "instrument_rating": null
  },
  "unknowns": ["supply transient", "wiring resistance", "load current", "reference topology"],
  "preferred_next_action": "Resolve topology and approved measurement conditions before proposing a test.",
  "hazards": ["stored energy status unknown"],
  "root_family_id": "fam-electronics-power-0048",
  "split": "train",
  "review_status": "illustrative_only"
}
```

The example is fictional and intentionally withholds a numeric diagnosis. It teaches the model to ask for reference,
rating, and conditions.

## Worked mini-pipeline

1. Map circuit domains, topology, energy class, measurement tasks, instruments, and repair verification.
2. Register exact manufacturer, component, standards, safety, OER, and expert sources.
3. Extract schematic entities, warning blocks, tables, and revision notes without flattening net relationships.
4. Normalize symbols, prefixes, and units while retaining raw notation, tolerance, and reference nodes.
5. Cluster schematic copies, device revisions, datasheet families, repair-guide mirrors, and synthetic variants.
6. Adjudicate conflicts by component grade, device revision, conditions, and source date.
7. Reject missing energy/reference context, unsafe probing, unrated instruments, or invented values.
8. Split complete device/schematic/fault families before generating questions and preference pairs.
9. Render theory, safe measurement reasoning, structured extraction, uncertainty, and escalation records.
10. Obtain qualified electronics and electrical-safety review plus battery, RF, laser, microwave, or high-voltage
    specialists where needed.
11. Evaluate on held-out devices and schematic families; seal mains, stored charge, lithium damage, isolation, and
    instrument-rating cases.
12. Monitor recalls, component changes, firmware, safety guidance, rights, and incidents.

Verification means matching device and component revisions and confirming measurement context. The model must not
authorize hazardous energized work or certify a repaired device.

## Cross-links

- [Dataset guide](../README.md)
- [CH-30 evaluation](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
- [Industrial profile](industrial-machinery-and-manufacturing.md)
