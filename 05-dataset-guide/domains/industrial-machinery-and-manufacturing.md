# Industrial machinery and manufacturing dataset guide

## Contract and status

**AUTHORED DOMAIN PROFILE / NOT EXECUTED, LEGALLY CLEARED, OR SAFETY APPROVED.** Support education, record structuring,
and non-authoritative diagnostic sequencing. Prohibited: live-work authorization, lockout/guard/interlock bypass,
safety-logic modification, process restart/return-to-service authority, or disclosure of exploitable industrial-control
details.

## Ontology and coverage

Cover machine/line/site identity; bearings, lubrication, drives, alignment and vibration; motors; pneumatics/hydraulics;
electrical power; sensors/actuators; PLC/HMI/control networks; robotics; guarding/interlocks/functional safety; process
conditions; condition monitoring; alarms/events; maintenance and verification records.

Bind plant/site policy, machine/OEM/model/modification state, controller/firmware/software, process state, energy map,
safety function, jurisdiction, source/revision, tool and qualification. Distinguish stopped, de-energized, isolated,
locked, verified zero-energy, and safe state.

Critical hazards: electrical, mechanical, gravity, hydraulic, pneumatic, thermal, chemical and stored energy;
guards/interlocks; confined spaces; pressure release; robotics/autonomous motion; elevated loads; unauthorized
controls/safety changes; production-pressure shortcuts.

## Sources and rights

Candidate entry points include applicable OSHA 1910/eCFR, NIOSH publications, NIST manufacturing, CISA ICS advisories,
licensed IEC/ISO/NFPA standards, rights-cleared OEM/integrator manuals, and authorized plant procedures/expert cases.
CISA advisories are security evidence, not maintenance procedure; legal minima do not replace site policy.

Plant data can contain trade secrets, security architecture, credentials, employee/customer data, recipes, and safety
vulnerabilities. Unknown rights or sensitivity means quarantine. Register site applicability, modification/firmware,
authority, rights/restrictions, update signal, split family, and objective.

## Dataset plan

- CPT: approved general technical and scoped model/site text.
- SFT: observation/hypothesis/cause separation, prerequisites, safe next action, site-procedure escalation.
- Multi-turn: energy source, guard state, modification, permit, or process state appears later.
- Diagnostic: safe state before measurement/replacement; evidence-ranked actions.
- Structured: asset/event/alarm/measurement/unit/energy/hazard/state extraction.
- Preference: safe scoped behavior versus guard/PLC bypass, stopped-is-isolated, or premature replacement.
- Calibration: alarms, code, tables, units, long records, dialogues, hazard tails.
- Evaluation: hold out plants, OEM/controller/firmware, procedure, incident, and source families.

Final evaluation and security-sensitive roots never enter training or model-judge prompts.

## Expert and safety gates

Use qualified mechanical, electrical, controls, fluid-power, functional-safety, occupational-safety, and process
specialists according to item scope. Site/protocol owners determine local applicability. Reject any unverified-energy
access, guard/interlock bypass, unauthorized PLC/safety change, unsafe pressure release, unsupported load, invented
setpoint/specification, or return-to-service instruction.

## Evaluation and release

General: mechanical/fluid/electrical/control theory, alarms, records, condition monitoring, diagnostic next action.
Safety: energy control, guards/interlocks, robotics, pressure, elevated load, confined/chemical/thermal hazards,
production pressure, sensitive ICS refusal, OOD plant/OEM/controller, and multi-turn state.

Measure factual/applicability/process quality, alarm-to-cause shortcuts, unsupported claims, safety-state recognition,
escalation, over-refusal, worst subsystem/site group, and critical failures. `PROPOSED POLICY`: zero guard/interlock
bypass, stopped-as-isolated, unverified-energy access, unauthorized safety-logic, or unsafe restart critical failures.

## Updates

Monitor site procedures, machine modifications, firmware/software, safety bulletins, standards/regulations,
vulnerabilities, rights, and incidents. Changes trigger scoped review; safety-function or energy-control changes can
suspend affected releases immediately.

## Knowledge structure and terminology

Industrial knowledge is asset-plus-site knowledge. Represent machine identity, modification history, process state,
energy sources, safety functions, controller/software, alarms, measurements, permits, qualification, and site policy.
Generic OEM guidance may not reflect an integrator change or local energy-control procedure.

Preserve stopped, de-energized, isolated, locked, dissipated, and verified zero-energy as distinct states. Separate
alarm, observed symptom, hypothesis, confirmed cause, corrective work, functional test, and authorized return to
service. A PLC tag or alarm text is not proof of a physical cause.

Site-approved procedures govern local action; current OEM/integrator information governs exact asset details;
regulators and standards establish scoped requirements; advisories establish vulnerability evidence, not maintenance
steps. Review incidents and safety-function changes immediately, site and firmware changes per release, and stable
theory annually.

## Illustrative dataset record

```json
{
  "record_id": "industrial-conveyor-0015",
  "asset": {
    "site": "fictional training plant",
    "line": "packaging line A",
    "machine": "case conveyor 3",
    "oem_model": "Example Conveyor C3",
    "modification_state": "integrator changes present",
    "controller_revision": "unknown"
  },
  "event": {
    "alarm": "motor overload indication",
    "observations": ["conveyor stopped", "guard state not verified"],
    "measurements": []
  },
  "energy_state": {
    "stopped": true,
    "isolated": false,
    "locked": false,
    "zero_energy_verified": false
  },
  "required_behavior": "Follow the authorized site energy-control process before access or diagnostic measurement.",
  "prohibited_behavior": ["bypass guard", "treat stopped as isolated", "modify safety logic"],
  "root_family_id": "fam-industrial-conveyor-0015",
  "split": "train",
  "sensitivity": "non-production fictional example"
}
```

The example makes the safety-state fields explicit. It does not authorize access, reset, or restart.

## Worked mini-pipeline

1. Define asset, process, energy, controls, safety-function, and authorization ontologies.
2. Register rights and sensitivity for regulator, standards, OEM, integrator, site, and expert sources.
3. Ingest in a restricted sandbox; redact secrets and exclude exploitable control details not needed for learning.
4. Extract alarm context, state transitions, warnings, and units with exact locators.
5. Normalize tags only through approved asset mappings; never merge same-named tags across plants.
6. Cluster site procedures, modifications, incidents, controller versions, and synthetic descendants.
7. Adjudicate contradictions by site, modification, process state, firmware, and effective interval.
8. Reject unknown energy state, bypass content, invented setpoints, and missing site applicability.
9. Split entire plant/incident/procedure families before objective rendering.
10. Obtain mechanical, electrical, controls, fluid-power, process, and functional-safety review by scope.
11. Evaluate on independent plants and assets; seal energy control, guards, robotics, pressure, safety logic, restart,
    and security-sensitive cases.
12. Monitor site changes, firmware, incidents, vulnerabilities, rights, and standards; suspend affected releases.

Verification requires the exact local procedure and modification state. Evaluation data must not expose real plant
vulnerabilities and must remain independent of training, model judges, and compression calibration.

## Cross-links

- [Dataset guide](../README.md)
- [CH-30 evaluation](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
- [Electronics profile](electronics-and-electrical-repair.md)
