# Domain dataset development guide

## What dataset creation actually is

Dataset creation is not “collect text, clean it, and split it.” It is the engineering of a traceable learning signal.
The model sees tokens, but the engineer must preserve the facts that give those tokens meaning: who published them,
which version applies, what may be done with them, which asset or population they describe, and which uncertainty or
hazard changes the answer.

Think of the guide as four connected layers:

1. **Evidence layer:** original bytes, source identity, rights, versions, and acquisition receipts.
2. **Knowledge layer:** normalized claims, entities, units, relationships, contradictions, and applicability.
3. **Learning layer:** objective-specific records for CPT, SFT, preference learning, distillation, or calibration.
4. **Measurement layer:** independent evaluation records that never influence training or model selection.

The same paragraph cannot be dropped unchanged into every layer. A manual paragraph may be useful CPT prose; an SFT
record needs an explicit user task and a reviewed target; an evaluation item needs independent ground truth and a
scoring rubric.

## The end-to-end learning path

Each stage answers a different question:

1. **Rights and provenance — may we use it, and can we prove where it came from?** Publicly readable is not the same as
   permitted for copying, transformation, training, redistribution, or model release.
2. **Source discovery — what evidence would cover the intended capability?** Search by ontology cell and source class,
   not by downloading whatever is easiest.
3. **Extraction — how do bytes become reviewable structure?** Preserve pages, headings, tables, warnings, and OCR
   confidence so reviewers can return to the original.
4. **Normalization and units — how do equivalent forms become comparable without erasing evidence?** Keep raw text,
   create separate display and matching forms, and represent measurements as typed values.
5. **Deduplication — which records teach the same underlying material?** Detect exact copies, near copies, editions,
   translations, templates, and synthetic descendants.
6. **Contradiction handling — when do different claims disagree, and when do they merely have different scope?** Compare
   subject, predicate, value, unit, jurisdiction, variant, and effective time before adjudicating.
7. **Quality scoring — what deserves review first?** Score provenance, extraction, relevance, freshness, and
   verifiability separately. Hard rights, privacy, and safety failures are never averaged away.
8. **Lineage — what produced this record and what depends on it?** Every transformation is a directed edge with pinned
   code, configuration, and input/output hashes.
9. **Splits and leakage — what information must remain independent?** Assign connected source and case families to a
   split before chunking or synthesis.
10. **Objective schemas — what exact learning signal is required?** Materialize distinct CPT, SFT, dialogue,
    diagnostic, extraction, preference, distillation, calibration, and evaluation views.
11. **Synthetic data — what bounded variation is useful?** Generate transformations of verified evidence, not new
    authority.
12. **Expert validation — who is qualified to accept the record?** Match reviewer scope and review depth to the
    consequence of an error.

## A small running example

Suppose a rights-approved training handout says, “A diagnostic code identifies a monitored condition; test the circuit
before replacing a component.” The raw artifact is hashed and stored. Extraction records the page and warning box. A
canonical claim links `diagnostic_code` to `monitored_condition`, not to `failed_component`. From that root:

- CPT receives the reviewed paragraph with its document boundary;
- SFT receives a question and an answer that asks for vehicle context and recommends evidence gathering;
- a preference set contrasts that answer with an unsupported “replace the sensor” answer;
- calibration receives a non-evaluation example with realistic code formatting;
- final evaluation receives a separately authored case from an independent fault family.

All descendants retain the root family. If the handout is withdrawn, lineage identifies every affected row and model
run. This is the standard the rest of the guide teaches.

## Status and purpose

**AUTHORED GUIDE / IMPLEMENTATIONS UNVERIFIED / DATASETS NOT BUILT / RIGHTS NOT APPROVED (2026-11-23).** This guide
converts CH-23–26, CH-30, and CH-38–41 into reusable dataset specifications. Selected official entry points were
`DOC-VERIFIED` in those chapters; that does not grant training rights. Markdown listings are examples and remain
`UNVERIFIED IMPLEMENTATION` unless an execution record says otherwise. No corpus, expert panel, benchmark, or release is
claimed.

The guide governs lawful discovery, immutable collection, objective-specific views, synthetic derivatives, domain
ontologies, expert review, leakage control, evaluation, updates, withdrawal, and onboarding. It supports offline
training only: the primary model does not retrieve documents or call a teacher at runtime.

## Required workflow

1. Freeze intended/prohibited uses, users, jurisdictions, languages, hazards, and knowledge cutoff.
2. Create a capability × variant × jurisdiction × time × hazard × objective coverage matrix.
3. Register candidate sources before acquisition; unresolved rights remain quarantined.
4. Preserve raw bytes, receipts, hashes, policy evidence, metadata, and transformation lineage.
5. Group source, edition, case, entity, template, translation, and synthetic families before splitting.
6. Materialize separate CPT, SFT, dialogue, diagnostic, structured, preference, distillation, calibration, and
   evaluation views.
7. Apply deterministic, rights, privacy, quality, safety, and risk-scoped expert gates.
8. Seal final evaluation roots from all training, prompting, calibration, judging, and selection.
9. Publish immutable manifests and a dataset card; rehearse source withdrawal.
10. Re-review on source, law, product, taxonomy, incident, model, tokenizer, or policy change.

## Guide navigation

### Shared controls

- [Source discovery and rights](source-discovery-and-rights.md) — discovery tracks, authority, source/rights registers,
  dispositions, and acquisition approval.
- [Collection, processing, and lineage](collection-processing-and-lineage.md) — raw evidence, extraction, normalization,
  privacy, deduplication, splits, manifests, and withdrawal.
- [Training-objective datasets](training-objective-datasets.md) — non-interchangeable schemas for every
  model-development objective.
- [Synthetic-data governance](synthetic-data-governance.md) — evidence-bounded generation, provenance, validation,
  contamination, and recursive-risk controls.

### Domain profiles

- [Automotive mechanics](domains/automotive-mechanics.md)
- [Motorcycle technicians](domains/motorcycle-technicians.md)
- [Heavy-duty truck mechanics](domains/heavy-duty-truck-mechanics.md)
- [Pharmacy](domains/pharmacy.md)
- [Beekeeping](domains/beekeeping.md)
- [Zookeeping, animal care, and husbandry](domains/zookeeping-animal-care-and-husbandry.md)
- [Agricultural equipment maintenance](domains/agricultural-equipment-maintenance.md)
- [Industrial machinery and manufacturing](domains/industrial-machinery-and-manufacturing.md)
- [Electronics and electrical repair](domains/electronics-and-electrical-repair.md)
- [Other specialized fields](domains/other-specialized-fields.md)

## Common release gates

All gates are conjunctive:

- exact rights basis, restrictions, attribution, and descendant-withdrawal path approved;
- complete source-to-record-to-snapshot lineage and immutable hashes;
- no prohibited personal, confidential, or security-sensitive content;
- no train/evaluation family overlap or test-informed optimization;
- high/critical records approved by scoped qualified reviewers;
- current/superseded facts and jurisdiction/variant applicability separated;
- coverage includes rare hazards, uncertainty, escalation, and benign controls;
- exact dataset and evaluation manifests pass schema, duplicate, leakage, and safety checks.

Unknown rights, missing lineage, exposed final tests, or an unresolved critical safety conflict block publication.
Aggregate quality cannot override them.

## Status vocabulary

`DOC-VERIFIED` means documentation was checked, not that an asset is licensed or an API works. `STATICALLY REVIEWED`
means content was inspected without execution. Listings remain `UNVERIFIED IMPLEMENTATION`; protocols are
`PLANNED EXPERIMENT`; examples are `ILLUSTRATIVE`. Use `EXECUTED` or `REPRODUCED` only with commands, environment,
inputs, outputs, and hashes.

## Dependencies

- [Bookmap](../BOOKMAP.md)
- [Governance](../00-governance/README.md)
- [CH-23–26 data engineering](../02-textbook/part-06-data-acquisition-and-dataset-engineering/README.md)
- [CH-30 evaluation suites](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
- [CH-38–41 domain extension](../02-textbook/part-11-extending-the-platform-to-other-domains/README.md)
