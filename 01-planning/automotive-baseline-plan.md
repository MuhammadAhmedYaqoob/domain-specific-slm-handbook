# Automotive capstone learning plan

## Why automotive is the baseline

Automotive mechanics is a useful first domain because it combines abundant technical language, structured diagnostic
reasoning, measurable factual questions, multi-turn symptom investigation, and genuine safety constraints. It forces
the learner to distinguish broad mechanical principles from vehicle-specific specifications that a parameter-only
model cannot reliably keep exact or current. The domain also transfers to motorcycle, truck, agricultural, and
industrial maintenance without pretending that those domains are interchangeable.

The capstone tests the complete learning objective: acquire rights-approved knowledge, teach useful diagnostic
behavior, preserve uncertainty and escalation, compress the model, and verify the same artifact on cloud and edge. The
canonical execution path is the [17-phase case study](../07-automotive-case-study/README.md).

## Prerequisite exit test

Before starting, explain causal attention, distinguish CPT from SFT, train and merge a LoRA adapter, calculate weight
and KV-cache memory, design leakage-safe splits, interpret confidence intervals, and explain why file size does not
determine mobile latency. Complete CH-01–11, CH-16–19, and CH-23–31 first.

## Intended and prohibited use

The product is an offline-capable educational and diagnostic-support assistant. It may explain component theory, generic
OBD-II concepts, diagnostic reasoning, observation gathering, safe sequencing, uncertainty and escalation.

It is not a repair authority and may not invent or recall unverified vehicle-specific torque, fluid, wiring,
calibration, part, service-interval or procedure specifications. It may not autonomously direct work involving high
voltage, restraint systems, braking, steering, lifting, pressure, fuel, hot/rotating machinery, toxic exhaust or stored
energy. Exact vehicle-specific procedures require authoritative information for that year/make/model/configuration and a
qualified person. Runtime retrieval is not used to hide this limitation.

## Domain coverage contract

The source and benchmark taxonomies must distinguish:

- engine air, fuel, ignition, lubrication, cooling, mechanical condition and controls;
- generic diagnostics, symptoms, observations, test selection, hypothesis ranking and confirmation;
- OBD-II architecture, generic code semantics, freeze-frame and live-data reasoning without code-only part replacement;
- 12 V electrical foundations, starting/charging, sensors/actuators, voltage drop and schematic reasoning;
- transmission/driveline fundamentals with platform-specific escalation;
- brakes, steering, suspension, tires, restraint, high-voltage/hybrid/EV and emissions as elevated-risk strata;
- workshop safety, tools, measurements, units, uncertainty, stop conditions and qualified escalation;
- adversarial, underspecified, out-of-scope and stale-knowledge cases.

Coverage is measured by approved item roots, difficulty, hazard and task type—not token volume alone.

## Hardware-independent baseline selection

The Model Research Lead creates an immutable candidate record for each exact checkpoint in feasible 0.5B, 1–2B and 3–4B
bands, plus an optional larger quality reference. No family name is a candidate.

Each record must contain publisher, exact repository/revision/hash, base or instruct state,
parameter/config/tokenizer/template facts, context configuration, license/use restrictions, model-card limitations,
weight formats, dependency requirements, candidate training method, and documented runtime/export paths.

Selection procedure:

1. Reject missing/ambiguous weights, tokenizer, license or commercial-use terms.
2. Reject architecture/runtime combinations that cannot be probed on any declared target.
3. Smoke load and generate in a locked source environment; preserve failures.
4. Probe target conversion with a tiny or untrained artifact before expensive training.
5. Run the identical untouched [baseline protocol](../06-experiment-framework/protocols/baseline-evaluation.md).
6. Block candidates with critical unsafe behavior, invalid templates, unacceptable retention, resource overflow or
   unsupported required export.
7. Advance a diverse survivor set by size and architecture; do not pick a winner from model-card benchmarks.

The 0.5B band tests minimum viable memory and exposes capacity limits; 1–2B is the primary mobile-learning band; 3–4B
tests whether added capacity justifies memory and latency; the optional larger model is a quality reference, not a
mobile promise. Include both base and instruct checkpoints only when the experiment asks whether domain knowledge
acquisition differs from response-format adaptation.

## Data and benchmark freeze

The Data Lead must approve sources before acquisition at scale. Every source requires rights basis, canonical
URL/identifier, version/effective date, receipt hash, permitted transformations/distribution, attribution, quality
authority, geographic/vehicle scope, update cadence and takedown path.

Root-group splitting precedes objective transforms. Near duplicates and derivatives of one source root stay in one
split. Sealed evaluation material is prohibited from CPT, SFT, preference, teacher generation, quantization/pruning
calibration, prompting, threshold selection and recovery.

The Evaluation Lead freezes:

- domain factual/conceptual items;
- diagnostic-process cases scored for observation requests, hypothesis quality, safe sequencing, unsupported assertions
  and escalation;
- critical safety/refusal and over-refusal cases;
- uncertainty/OOD/underspecified cases;
- general capability and catastrophic-forgetting controls;
- multi-turn consistency and structured-output cases;
- fixed performance prompts with short/medium/long contexts and outputs.

Qualified automotive review and adjudication are mandatory for high-risk strata. Passing an aggregate score cannot
compensate for a critical-safety failure.

## Baseline metrics and gates

Primary outcomes are the preregistered domain rubric and critical-safety-failure count. Secondary outcomes include exact
match/F1 where valid, unsupported-claim rate, diagnostic rubric dimensions, escalation correctness,
refusal/over-refusal, calibration, robustness, OOD, multi-turn consistency, retention, valid-output rate, artifact
bytes, loaded/peak memory, startup, visible TTFT, prefill/decode, sustained rate, failures and allocated AWS cost where
measured.

Numeric floors and non-inferiority margins are set by the Safety Reviewer and Evaluation Lead **before** sealed
evaluation. Mandatory hard gates are identity/rights, no sealed leakage, no critical unsafe response, valid
tokenizer/template, source/export semantic parity, target memory fit, reliability, and complete lineage.

**Illustrative gate example:** require zero critical unsafe responses in 120 independently reviewed high-risk cases, at
least 0.78 mean diagnostic-rubric score, no more than a 0.03 absolute decline on the general-retention score, peak RAM
below 3.5 GiB, p95 visible TTFT below 3.0 seconds, and sustained decode above 8 token/s on the named primary device.
These values are teaching examples, not project results or universal safety thresholds. Before execution, replace each
value with a reviewed requirement derived from the intended use, benchmark reliability, available device, and user
study; preserve the rationale and approval date.

## Seventeen-phase dependency chain

1. Scope → approved intended/prohibited use and hazard taxonomy.
2. Baseline selection → exact eligible checkpoint records.
3. Acquisition → rights-approved receipts and bytes.
4. Processing → immutable objective datasets and split/leakage evidence.
5. Untouched baseline → raw quality/safety/system profile.
6. CPT decision → controlled accept/reject record.
7. SFT → governed specialist checkpoints.
8. Full/LoRA/QLoRA → resource-matched method decision.
9. Distillation → reviewed teacher/student decision.
10. Quantization → target-native exports and parity.
11. Pruning → executable benefit or explicit rejection.
12. Combined optimization → incremental survivor set.
13. Final evaluation → sealed quality/safety evidence.
14. AWS deployment → digest-pinned, authenticated release and rollback.
15. Mobile deployment → signed Android/iOS packages and offline operation.
16. Benchmark → controlled raw cloud/physical-device trials.
17. Selection → separate target-specific Pareto decisions.

Each phase consumes the accepted manifest from its predecessor and writes the immutable output named in the
[case-study phase map](../07-automotive-case-study/README.md). A failure returns to the last accepted artifact; it never
weakens the gate.

## How to study the capstone

Treat phases 1–5 as experimental-design work, not preparation to rush through. In phases 6–9, change one learning
mechanism at a time. In phases 10–12, compare deployment artifacts rather than checkpoint sizes. In phases 13–16, score
the frozen artifacts without tuning on sealed data. In phase 17, write separate decisions for quality, mobile, balance,
memory, and AWS cost because one universal winner would hide target trade-offs.

After every phase, write a short memo answering: what changed, what remained controlled, which evidence supports the
decision, what failed, and what the next phase is allowed to assume.

## Target and budget procedure

The Cloud Lead selects AWS tuples through the [AWS target plan](aws-target-plan.md). The Edge Lead selects named
physical devices through the [edge target plan](edge-target-plan.md). Baseline model selection remains provisional until
at least one source runtime and one intended deployment path pass compatibility probes.

The approved experiment plan records maximum accelerator-hours, storage, generated teacher tokens, AWS currency,
endpoint hours and physical-device time. Small/medium/research scope follows the
[experiment sequence](hands-on-experiment-sequence.md). Missing funds or hardware produce `NOT_RUN_BUDGET` or `BLOCKED`,
not favorable estimates.

## Final decisions

Only reviewed, gate-passing exact artifacts may receive these labels:

- highest-qualified domain quality;
- best qualified mobile tuple;
- best balanced target-specific tuple;
- lowest-memory qualified tuple;
- lowest allocated-cost qualified AWS tuple.

“No qualified candidate” is valid. A selection includes exact model/export/runtime/hardware/workload hashes and cannot
be generalized to another target.

## Cross-links

- [Automotive domain guide](../05-dataset-guide/domains/automotive-mechanics.md)
- [Experiment framework](../06-experiment-framework/README.md)
- [Decision criteria](../14-decision-framework/criteria-and-measurement.md)
