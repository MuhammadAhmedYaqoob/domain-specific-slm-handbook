# Beekeeping dataset guide

## Contract and status

**AUTHORED LOW-RESOURCE PROFILE / DATASET NOT BUILT / RIGHTS AND LOCAL LAW UNAPPROVED.** Intended use:
biology/terminology education, inspection-note structuring, seasonal concepts, observation-versus-hypothesis reasoning,
and escalation. Prohibited: definitive diagnosis from nonspecific signs, treatment/product authorization,
pesticide/veterinary advice without current local authority, or legal/movement determination.

## Ontology and context

Cover honey-bee biology/behavior, colony/queen/brood lifecycle, equipment and inspections, nutrition/forage, swarming,
seasonal/overwintering management, Varroa and other pests/diseases, sampling methods, pesticides, honey handling,
pollination, records, registration/movement, and reportable disease.

Every record binds bee species/population, operation scale/objective, hive context, region/climate, hemisphere, date,
season/phenology, forage, colony state, pest pressure, sampling method, product/jurisdiction, evidence authority, and
effective version. Season is not inferred from month alone.

High-risk cells include look-alike disease signs, reportable disease, product legality/resistance/residues, pesticide
exposure, allergy/anaphylaxis escalation, smoker/fire, lifting, food hygiene, and movement restrictions.

## Source portfolio

Candidate entry points: USDA APHIS National Honey Bee Surveys, USDA NASS Bee and Honey/Quick Stats, USDA NAL, FAO TECA,
peer-reviewed apiculture research/methods, university extension, current national/subnational agriculture and pesticide
authorities, current labels, and commissioned regional experts. Each asset requires rights and locality review;
institutional authorship is not an open license.

Survey records preserve sampling frame, denominator, method, dates, revisions, and geography. A foreign approval or
historical treatment is not local authorization. Forums and practitioner reports reveal vocabulary/cases but cannot
establish safety truth.

## Dataset plan

- CPT: approved biology and region-scoped education.
- SFT: observation/hypothesis separation, contextual questions, uncertainty, local-authority verification, escalation.
- Multi-turn: region, season, stores, sampling, or disease signs appear later.
- Structured: colony observations, date/phenology, method, taxon, evidence, and handoff.
- Preference: scoped evidence-seeking versus confident disease/treatment claims.
- Synthetic: paraphrases, structured renderings, and validated contrasts only; evidence gaps generate expert questions,
  not answers.
- Calibration: seasonal terminology, measurements, tables, multilingual/local terms, dialogues.
- Evaluation: independent extension/source families, climate regions, seasons, and disease-event roots.

Hold out source institution, climate region, disease event, and publication family. Keep all synthetic/template
descendants with their root.

## Expert and safety gates

Experienced regional beekeepers/apiary inspectors review management; apiculture scientists or veterinarians review
disease/treatment; regulators review product, reportable disease, and movement claims. High-risk records receive
complete review.

Reject unsupported disease confirmation, illegal/unapproved treatment, cross-region or wrong-season transfer, missing
method/denominator, invented ecological state, delayed escalation, or absent source/version.

## Evaluation and release

General suite: biology, colony lifecycle, observations, seasonal/phenological reasoning, survey interpretation,
uncertainty. Safety suite: look-alikes, local treatment legality, reportable disease/movement, pesticide/residue/PPE,
allergy/fire/food hygiene, OOD climate, hemisphere contrasts, and multi-turn context.

Measure qualifier preservation, supported claims, necessary-question recall, safe abstention/escalation, over-refusal,
contrast consistency, rare-cell coverage, and critical failures. `PROPOSED POLICY`: zero illegal/unsupported treatment
and reportable-disease critical failures.

## Updates

Monitor local law, product labels, reportable disease/movement rules, surveillance methods, emerging pest evidence,
resistance, source rights, and incidents. Releases expose covered regions/species, cutoff, unsupported cells, and the
offline model's inability to verify current local rules.

## Knowledge structure and terminology

Beekeeping knowledge is ecological, seasonal, and local. A useful record connects colony observations with species,
region, climate, hemisphere, phenological stage, inspection method, sampling denominator, and time. The same calendar
month does not imply the same colony state in two climates or hemispheres.

Preserve observation versus diagnosis, prevalence versus individual-colony finding, adult bee versus brood signs,
mite drop versus a method-defined infestation measure, active ingredient versus branded product, and legal
authorization versus evidence of efficacy. Local names should map to scientific terms without erasing the original.

Use current local agriculture, pesticide, and reportable-disease authorities for legal status; current product labels
for label facts; validated survey methods and peer-reviewed research for measurement and biology; extension material
for region-scoped education; experienced inspectors and scientists for derivation. Check urgent disease and product
notices continuously, seasonal guidance before each season, and stable biology sources annually.

## Illustrative dataset record

```json
{
  "record_id": "bee-inspection-0068",
  "context": {
    "species": "Apis mellifera",
    "region": "fictional temperate region",
    "hemisphere": "northern",
    "date": "2026-05-14",
    "phenology": "spring buildup",
    "operation_scale": "small teaching apiary"
  },
  "observation": {
    "method": "visual frame inspection",
    "duration_minutes": 12,
    "facts": ["brood pattern appears irregular", "several uncapped cells observed"],
    "sample_denominator": "six brood frames inspected"
  },
  "interpretations": ["multiple causes remain possible"],
  "unknowns": ["larval appearance details", "odor", "sampling test results", "local reportability rules"],
  "required_behavior": "Document observations and seek current local inspector or veterinary guidance before disease or treatment claims.",
  "root_family_id": "fam-bee-inspection-0068",
  "split": "train",
  "review_status": "illustrative_only"
}
```

The fictional case intentionally stops at observation and escalation. It does not diagnose disease or recommend a
treatment.

## Worked mini-pipeline

1. Build coverage across biology, inspection, season, pests/disease, forage, records, food handling, and local rules.
2. Register independent local authority, survey, research, extension, and expert sources with exact rights.
3. Extract method, denominator, geography, date, product, and jurisdiction along with every statement.
4. Normalize taxon and units but retain local terms and raw measurements.
5. Group survey editions, extension derivatives, disease events, climate regions, and synthetic families.
6. Resolve conflicts by method, population, climate, season, jurisdiction, and effective date.
7. Score provenance and locality; reject unsupported diagnosis and records missing sampling method or denominator.
8. Assign complete event and region families before creating questions or dialogues.
9. Render SFT targets that ask for missing context and distinguish observation, hypothesis, confirmation, and local
   authority.
10. Obtain regional beekeeper/inspector review and scoped scientific, veterinary, or regulator review for hazards.
11. Evaluate on independent climates, seasons, institutions, and disease roots; isolate reportable-disease, product,
    pesticide, allergy, and fire cases in a sealed safety suite.
12. Recheck rules and labels before each release and expose the model cutoff and unsupported regions.

Evaluation must not reuse training survey events or paraphrased extension examples. Verification is local and
time-bound; the dataset cannot convert a model into an authority for diagnosis, treatment, movement, or pesticide use.

## Cross-links

- [Dataset guide](../README.md)
- [CH-40](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md)
- [Synthetic governance](../synthetic-data-governance.md)
