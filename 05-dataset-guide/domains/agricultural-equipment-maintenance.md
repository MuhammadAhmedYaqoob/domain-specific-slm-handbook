# Agricultural equipment maintenance dataset guide

## Contract and status

**AUTHORED DOMAIN PROFILE / NOT EXECUTED OR RIGHTS-APPROVED.** Support education and supervised maintenance reasoning.
Do not authorize operation/return-to-service, work under raised equipment, PTO/guard/interlock bypass, chemical
application, confined-space entry, or exact model-specific values without current authority.

## Ontology and coverage

Represent tractor/power unit, diesel/electric powertrain, transmission/driveline, steering/brakes,
hydraulics/accumulators, electrical/electronic control, control bus/precision agriculture, PTO, hitch, implement,
guarding/ROPS, autonomous functions, storage and maintenance records.

Bind every record to manufacturer/model/year/market, power unit, implement/attachment, hitch/PTO speed class, hydraulic
class, controller/firmware, task, terrain/load, attachment state, jurisdiction, source/revision, and qualification.
Shared automotive/truck principles do not establish agricultural applicability.

Critical hazards: PTO entanglement, raised/unsupported implements, stored hydraulic/pneumatic/spring energy, rollover,
guarding, overhead electricity, towing, chemical application, grain/confined spaces, autonomous movement, lifting,
hot/rotating components, fuel/HV.

## Source portfolio

Candidate entry points include OSHA agricultural standards and eCFR Part 1928, NIOSH agricultural safety, licensed
ASABE/ISO/SAE standards, rights-cleared current manufacturer/operator/service manuals and bulletins, extension/OER
programs, and contracted technicians/safety specialists. Regulator guidance establishes scope, not model-specific
procedure. Standards and OEM content require asset-level rights.

Register exact machine/implement compatibility, revision, market, authority, rights/restrictions, attribution, update
triggers, split family, and objective. Farm records may contain personal, location, operational, chemical, and
proprietary data.

## Dataset plan

- CPT: approved general/mechanical and model-scoped material.
- SFT: context requests, observation/hypothesis separation, safe sequencing, escalation.
- Multi-turn: implement, load, raised state, accumulator, slope, or autonomous state appears later.
- Structured: machine graph, attachment state, measurement, hazard, prerequisite, evidence.
- Preference: scoped safe response versus PTO/guard/interlock bypass or unsupported procedure.
- Calibration: diagrams-as-text, codes, units, long manuals, dialogue, rare hazards.
- Evaluation: hold out platform, implement, controller, source, and incident families.

Keep power-unit/implement combinations, editions, translated manuals, incident roots, and synthetic descendants
together.

## Expert and safety gates

Qualified agricultural-equipment technicians review technical records; farm-safety, hydraulic, electrical/HV,
chemical-application, confined-space, and autonomy specialists review relevant hazards. Reject any rotating-PTO
exposure, unsupported raised-load work, unreleased stored energy, disabled guard/interlock, unsafe
rollover/overhead-line advice, wrong attachment compatibility, invented specification, or return-to-service authority.

## Evaluation and release

General suite: powertrain, hydraulics, PTO/hitch/implements, electrical/control, maintenance, and diagnostic next
action. Safety suite: PTO, raised load, stored energy, rollover, guards, overhead lines, chemicals, confined/grain
spaces, autonomous movement, OOD implement/platform, adversarial shortcuts, and multi-turn state.

Report applicability, process, unsupported claims, hazard recognition, escalation/refusal, over-refusal, contrast
consistency, and critical failures. `PROPOSED POLICY`: zero defined PTO, unsupported-load, stored-energy, rollover,
guard/interlock, and overhead-line critical failures.

## Updates

Monitor OEM revisions/recalls, safety rules, standards, controller/software, implement compatibility, product/chemical
rules, rights, and incidents. Material hazard changes trigger immediate review and lineage-based rebuild/withdrawal.

## Knowledge structure and terminology

Agricultural equipment knowledge joins a power unit to changing implements, environments, and tasks. Model the tractor
or carrier, hitch, PTO class, implement, hydraulic connections, guards, attachment state, terrain, load, controller,
firmware, and operator qualification as one configuration. A tractor record without the attached implement can be
unsafe or meaningless.

Preserve PTO speed class versus measured speed, raised versus mechanically supported, pressure relieved versus
zero-energy verified, implement lowered versus secured, guard versus interlock, ROPS versus cab, and autonomous-ready
versus autonomous-active state.

Current manufacturer/operator information governs exact compatibility and procedures; safety authorities describe
hazards and legal scope; standards define interfaces; extension and qualified experts teach principles. Check recalls
and hazard notices continuously, implement/controller compatibility per release, seasonal procedures before use, and
general theory annually.

## Illustrative dataset record

```json
{
  "record_id": "agri-hydraulic-0026",
  "configuration": {
    "power_unit": "Example Tractor 90",
    "implement": "Example rear-mounted mower",
    "hitch_class": "three-point category II",
    "pto_class": "540 rpm nominal class",
    "controller_firmware": "unknown",
    "terrain": "level workshop area",
    "attachment_state": "raised state not verified"
  },
  "observation": "implement lowers slowly after shutdown",
  "known": ["uncommanded lowering was observed"],
  "unknown": ["mechanical support state", "stored pressure", "leak location", "compatibility revision"],
  "required_behavior": "Keep personnel clear and follow current manufacturer energy-control and support procedures.",
  "prohibited_behavior": ["work beneath the raised implement", "assume shutdown equals zero energy"],
  "root_family_id": "fam-agri-hydraulic-0026",
  "split": "train",
  "review_status": "illustrative_only"
}
```

The fictional case teaches hazard recognition and missing-context handling, not a repair sequence.

## Worked mini-pipeline

1. Build an ontology for power units, implements, interfaces, attachment state, energy, terrain, and tasks.
2. Register exact manufacturer, safety-authority, standards, extension, and expert assets.
3. Extract compatibility tables, warnings, prerequisites, and diagrams with their footnotes.
4. Normalize interface classes and units while retaining raw model and condition strings.
5. Cluster power-unit/implement combinations, revisions, incidents, translations, and synthetic variants.
6. Adjudicate conflicting values by exact machine, implement, firmware, market, and effective date.
7. Apply hard gates for PTO exposure, unsupported loads, stored energy, guards, rollover, and overhead lines.
8. Split complete platform/implement/incident families before generation.
9. Render explanation, structured state, clarification, and safe-sequence views without return-to-service authority.
10. Use qualified equipment technicians and hazard-specific hydraulic, electrical, chemical, or autonomy reviewers.
11. Evaluate on held-out combinations and incidents; seal PTO, raised-load, rollover, chemical, confined-space, and
    autonomous-motion cases.
12. Monitor compatibility and safety changes and rebuild all affected descendants.

Evaluation separation must include attachment combinations: holding out only text passages while sharing the same
incident or implement family is leakage. Verification is exact to configuration and current source revision.

## Cross-links

- [Dataset guide](../README.md)
- [CH-30 domain evaluation](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
- [Other fields onboarding](other-specialized-fields.md)
