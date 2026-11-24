# Zookeeping, animal care, and husbandry dataset guide

## Contract and status

**AUTHORED LOW-RESOURCE PROFILE / NOT VETERINARY, LEGAL, OR RIGHTS APPROVED.** Intended use: natural-history education,
observation structuring, welfare-oriented reasoning, record summarization, and protocol/veterinary escalation.
Prohibited: diagnosis/treatment, diet or medication prescription, restraint/transport/dangerous-animal authorization, or
disclosure of sensitive containment details.

## Ontology and scope

Cover taxonomy/natural history, individual and managed-population identity, habitat/environment, nutrition concepts,
social grouping, enrichment, behavior/training, welfare indicators, quarantine/biosecurity, reproduction, transport
concepts, records, keeper safety, and veterinary escalation.

Every record binds scientific name/taxonomy version and certainty, subspecies/population, life stage, sex,
reproductive/social/medical context, individual baseline, institution/climate, enclosure/environment, observation
duration/trend, legal/protocol scope, source version, and reviewer. Wild ecology is not automatically managed-care
guidance.

High-risk cells: welfare deterioration, injury/illness, zoonosis/biosecurity, quarantine, dangerous animals, enrichment
ingestion/injury/escape risk, cross-species transfer, restraint/transport, diet/medication pressure, and
security-sensitive requests.

## Sources and rights

Candidate entry points: AZA Animal Care Manuals/wellbeing resources, EAZA resources, WOAH codes, WAZA welfare resources,
applicable USDA APHIS/eCFR Title 9, CITES and national authorities, peer-reviewed zoo/veterinary literature, and
authorized institutional protocols. Species360/studbooks and individual records can be controlled or sensitive and
require explicit agreement.

Public professional manuals may be authoritative but not training-licensed. Legal minima are not complete best practice.
Store authority type, exact taxon/population, institution/jurisdiction, edition/effective date, rights, sensitivity,
restrictions, update trigger, and objective.

## Canonical observation

Store observable facts separately from interpretation. The canonical extension contains taxon and taxonomy version,
individual context and baseline status, timed observation and trend, interpretations, and required institutional or
veterinary handoff. The populated observation later in this chapter demonstrates the structure. Unknown
identity/context must not be completed from model memory.

## Dataset and leakage plan

- CPT: rights-approved, scoped natural-history/husbandry/welfare text.
- SFT: observation quality, uncertainty, protocol-aware education, safe refusal/escalation.
- Multi-turn: taxon, life stage, baseline, social/medical context appears later.
- Structured: observations, trends, taxonomy, context, evidence, handoff.
- Preference: scoped welfare reasoning versus cross-species diagnosis/treatment or universal refusal.
- Synthetic: only bounded transformations and validated scenario rendering; no invented care facts/set points.
- Calibration: taxonomic names, records, long context, multilingual terms, dialogues.
- Evaluation: hold out taxa, institutions, manual families, and welfare scenarios.

Report taxon groups independently. Keep translations, institutional variants, individual/case roots, and synthetic
descendants together.

## Expert and sensitivity gates

Taxon specialists and experienced keepers review husbandry/natural-history; zoo/wildlife veterinarians review health,
medication, treatment, quarantine; nutritionists review nutrition; welfare scientists review welfare constructs;
institution protocol owners review dangerous-animal/restraint/transport material; regulators/legal reviewers cover law.

Exclude exact threatened-animal locations, containment weaknesses, unpublished medical records, personnel data, and
confidential transfer/trade/biosecurity details unless specifically authorized and access-controlled. De-identification
does not grant rights.

## Evaluation and release

General: taxonomy, natural history, observation/trend, habitat/social/enrichment concepts, welfare dimensions, record
handoff, legal minimum versus best practice. Safety: species uncertainty, veterinary escalation, biosecurity, autonomous
diagnosis/treatment refusal, hazardous enrichment, dangerous-animal/security pressure, OOD taxa/institutions, and
multi-turn correction.

Measure supported/applicable claims, taxon/context preservation, necessary questions, welfare escalation, over-refusal,
sensitive-detail refusal, contrast consistency, worst taxon, and critical failures. `PROPOSED POLICY`: zero autonomous
veterinary treatment, cross-species prescription, dangerous-animal authorization, sensitive containment disclosure, or
missed decisive escalation.

## Updates

Monitor taxonomy, law/accreditation, professional manuals, institution protocols, health/welfare evidence, rights, and
incidents. Every release declares taxa, institutions/regions, cutoff, unsupported operations, expiry, and offline update
limitations.

## Knowledge structure and terminology

Managed-animal knowledge is taxon-, individual-, institution-, and time-specific. Structure records around taxonomy,
individual baseline, life stage, sex, social/reproductive context, environment, observed behavior, duration, trend,
keeper interpretation, protocol handoff, and source version. Wild ecology can inform natural history but does not
automatically define managed-care practice.

Preserve observation versus interpretation, enrichment goal versus enrichment item, intake offered versus consumed,
species norm versus individual baseline, quarantine versus routine separation, and legal minimum versus institution
policy or professional practice. Taxonomy mappings need a named version because scientific names change.

Use current institution-approved protocols for local operations, professional manuals for their stated taxa and scope,
regulators for applicable requirements, and peer-reviewed zoo/veterinary literature for research evidence. Sensitive
records require authorization independent of scientific quality. Review individual and protocol records when changed,
urgent health/welfare notices immediately, taxonomy on release, and general natural-history material annually.

## Illustrative dataset record

```json
{
  "record_id": "zoo-observation-0037",
  "taxon": {
    "scientific_name": "Panthera leo",
    "taxonomy_version": "fictional-taxonomy-2026",
    "certainty": "confirmed"
  },
  "individual_context": {
    "individual_id": "anonymized-lion-A",
    "life_stage": "adult",
    "sex": "female",
    "baseline_known": true
  },
  "observation": {
    "description": "remained apart from the social group during two scheduled observation periods",
    "started_at": "2026-11-20T09:00:00Z",
    "duration": "two 20-minute periods",
    "trend": "new relative to documented baseline"
  },
  "interpretations": ["cause is not established by the observation"],
  "required_handoff": {
    "institution_protocol": true,
    "veterinary_review": "follow current institutional escalation criteria"
  },
  "sensitivity": "internal_educational_example",
  "root_family_id": "fam-zoo-observation-0037",
  "split": "train",
  "review_status": "illustrative_only"
}
```

The example teaches precise observation and handoff. It does not diagnose, prescribe care, or reveal containment
details.

## Worked mini-pipeline

1. Map taxa, life stages, welfare dimensions, observation tasks, environments, and escalation hazards.
2. Register professional, regulatory, research, and authorized institutional sources; classify sensitivity and rights.
3. Extract taxon scope, evidence type, edition, conditions, and operational versus educational status.
4. Normalize taxonomy through a versioned mapping while retaining original names and certainty.
5. Group manual editions, institution variants, individual cases, translations, and synthetic descendants.
6. Treat apparent conflicts as possible taxon, individual, environment, legal, or time differences.
7. Reject records containing unauthorized locations, containment weaknesses, medical detail, or unsupported care facts.
8. Split whole taxon/manual/institution/case families before rendering objective views.
9. Create observation-structuring, uncertainty, welfare reasoning, refusal, and protocol-handoff targets.
10. Route to taxon specialists, keepers, veterinarians, nutritionists, welfare scientists, or protocol owners by scope.
11. Evaluate on independent taxa and institutions; keep veterinary treatment, quarantine, dangerous-animal, and
    sensitive-security cases in a sealed safety suite.
12. Rebuild affected descendants after taxonomy, protocol, law, rights, or welfare-evidence changes.

Independent evaluation should test transfer to held-out taxa without rewarding cross-species assumptions. Verification
requires exact taxon and institutional scope; no dataset record authorizes treatment, restraint, transport, or access.

## Cross-links

- [Dataset guide](../README.md)
- [CH-40](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md)
- [Beekeeping profile](beekeeping.md)
