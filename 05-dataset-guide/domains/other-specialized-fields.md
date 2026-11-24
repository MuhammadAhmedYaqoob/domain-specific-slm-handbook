# Other specialized fields: reusable onboarding logic

## Status and admission rule

**AUTHORED REUSABLE SPECIFICATION / NOT EXECUTED.** A new field is configuration plus approved evidence, not a copy of
an existing pipeline. This profile implements
[CH-41](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-41-new-domain-onboarding.md).

Admit a domain only when all applicable gates pass:

```text
bounded use ∧ attainable rights ∧ qualified expertise ∧ governed data
∧ independent evaluation ∧ target qualification ∧ owned lifecycle
```

If authoritative evidence or scoped reviewers do not exist, narrow to low-risk education or reject the domain.

## Stage-and-gate process

1. **Intake:** users, workflow, languages/jurisdictions, intended/prohibited authority, cloud/edge targets, owner,
   alternatives, no-runtime-RAG feasibility.
2. **Risk/rights:** hazard pathways, consequence tier, required/prohibited behavior, escalation, regulation,
   privacy/security, exact planned rights.
3. **Sources/data:** capability taxonomy, authority map, source register, acquisition, canonical schema, lineage,
   coverage, quality, family splits.
4. **Evaluation:** separate general/safety blueprints, qualified truth, OOD/adversarial/multi-turn cases, sealed roots,
   critical failures, proposed thresholds.
5. **Training:** baseline, objective views, hypotheses, fixed budgets, seeds, stop rules, retained general capability.
6. **Compression/targets:** exact derivative lineage and full semantic/safety rerun on every package.
7. **Release/lifecycle:** signed bundle, monitoring, update triggers, rollback, expiry, incident response, rights
   withdrawal, retirement.

Unknown rights, missing owner, absent safety reviewer, final-test exposure, critical failure, or unevaluated target
blocks progression regardless of average quality.

## New-domain declaration

**Listing DG-ND-01 — domain dataset profile.** The populated optical-instrument profile later in this chapter shows all
fields in context. Production rejects missing owners, mutable revisions, unknown roles, expired approvals, and
unresolved references.

## Taxonomy and task design

Start from decisions users need and harms wrong outputs can cause. Define:

- entities, variants, identifiers, units, states, procedures, evidence, hazards;
- relations between observation, hypothesis, test, cause, action, and verification;
- applicability dimensions: asset/population, jurisdiction, time, environment, user qualification;
- tasks: explain, extract, classify, compare, clarify, diagnose, sequence, abstain, escalate;
- explicit unknown/not-applicable/conflicting states;
- rare/high-consequence coverage and benign controls.

An ontology supports offline labeling, coverage, splitting, and evaluation. It is not a runtime knowledge graph.

## Source and data requirements

For each capability/hazard cell, find independent official, manufacturer/professional, scholarly/OER,
operational/expert, and community tracks as applicable. Store exact rights, provenance, authority scope, restrictions,
freshness, and objective. Document failed discovery and commission experts rather than inventing facts.

Use the common canonical and objective schemas. Split connected families before chunking/synthesis. Hold out source,
entity, time, institution/author, template, and incident families based on claimed generalization.

## Expert and evaluation design

Map every topic/hazard to an exact qualification scope; generic “domain expert” is insufficient. Critical training and
evaluation records require independent scoped review and adjudication.

The benchmark blueprint crosses task × ontology area × risk × variant × jurisdiction/time × format ×
OOD/adversarial/multi-turn shift. Report macro and worst-group results, unsupported claims, uncertainty, escalation,
over-refusal, and critical failures. Proposed thresholds are calibrated on development data before final-suite opening.

## Model-organization experiment

Compare, as justified:

- independent specialist model;
- common base + independent adapter;
- shared-domain CPT + adapter;
- joint multi-domain model;
- routed already-trained specialists;
- separate or joint distilled students.

Control accepted data, base/tokenizer, search budget, evaluation, seeds, and target precision. Measure domain transfer
and substitution, safety, retention, installed/peak resources, update coupling, and withdrawal complexity. Do not infer
a winner from semantic overlap.

## Update, withdrawal, and retirement

Triggers include source/revision/law/taxonomy changes, incidents, reviewer disagreement, rights revocation,
base/tokenizer/template/compression/runtime changes, and user/task drift. Trace affected descendants; re-review,
rebuild, narrow, suspend, roll back, or retire. Preserve immutable decisions and acknowledge that offline devices may
not update promptly.

## Acceptance checklist

- bounded users/use/exclusions and accountable owners;
- complete ontology, hazards, reviewer scopes, and coverage ledger;
- approved source/rights register with zero unknown admitted rights;
- immutable lineage and successful source-withdrawal rehearsal;
- independent sealed evaluation and no optimization exposure;
- risk-scoped expert approvals and calibrated policy;
- exact release artifact passes domain/safety/target tests;
- published cutoff, scope, limitations, update, rollback, incident, and retirement plan;
- no-runtime-RAG attestation passes.

## Teaching example: onboarding a fictional optical-instrument domain

The point of onboarding is to discover a domain's own knowledge structure, not to rename fields from automotive or
pharmacy. Suppose the proposed model teaches maintenance concepts for low-power educational optical microscopes. Its
knowledge graph might contain instrument, optical path, illumination, objective, stage, specimen context, observed
image artifact, adjustment state, cleaning constraint, hazard, and verification. The hierarchy of sources would be
current manufacturer material for exact models, safety guidance for illumination/electrical hazards, optics texts for
theory, and qualified technicians for reviewed educational cases.

Terminology must distinguish magnification from resolution, objective magnification from total displayed
magnification, image artifact from specimen feature, cleaning method from chemical compatibility, and observation from
confirmed cause. Update cadence follows the source: safety notices immediately, model manuals and firmware on release,
and stable theory on a scheduled review.

```yaml
schema: dataset-domain-profile/v1
domain:
  id: "educational-optical-instruments"
  owner: "instrument-data-product-owner"
  users: ["laboratory educators", "qualified instrument technicians"]
  languages: ["en-US"]
  jurisdictions: ["US-general-education"]
  intended_uses: ["terminology education", "observation structuring", "supervised fault isolation"]
  prohibited_uses: ["laser service", "electrical live-work authorization", "return-to-service certification"]
  knowledge_cutoff: "2026-11-01"
risk:
  highest_tier: "R3"
  hazards:
    - id: "illumination-energy"
      triggers: ["unknown lamp or laser class", "open energized housing"]
      required_behavior: ["identify source class", "use current manufacturer safety procedure", "escalate"]
      prohibited_behavior: ["authorize exposure", "bypass interlock"]
      escalation: ["qualified optical-instrument safety specialist"]
      reviewer_scope: "optical and electrical safety"
ontology:
  entities: ["instrument", "objective", "illumination", "optical_path", "observation", "measurement"]
  relations: ["observation_suggests_hypothesis", "test_discriminates_hypothesis", "repair_requires_verification"]
  applicability_keys: ["manufacturer", "model", "revision", "illumination_class", "jurisdiction"]
  controlled_vocabularies: ["optics-terms/v1", "hazard-state/v2"]
  rare_critical_cells: ["unknown laser class", "interlock request", "contaminated optical surface"]
tasks:
  general: ["explain terminology", "compare optical concepts"]
  diagnostic: ["separate image observation from cause", "select safe next information"]
  structured: ["extract model, objective, observation, and hazard"]
  uncertainty_refusal: ["request missing applicability", "refuse interlock bypass"]
sources:
  authority_classes: ["manufacturer", "safety authority", "licensed textbook", "qualified expert"]
  manifest_sha256: "74d725d4e7426e55394d18bd1d39d6cc3f1ed110343a9fe2c761836e3499d8c2"
  unknown_rights_allowed: 0
  freshness_triggers: ["safety notice", "manual revision", "illumination-source change"]
data:
  canonical_schema: "optical-instrument-record/v1"
  objectives: ["cpt", "sft", "structured", "preference", "calibration", "evaluation"]
  split_group_keys: ["source_family", "instrument_family", "case_family", "synthetic_family"]
  coverage_dimensions: ["subsystem", "model", "observation", "hazard", "difficulty"]
  synthetic_policy: "grounded-transformations/v1"
experts:
  reviewer_scopes: ["optical technician", "electrical safety", "rights and privacy"]
  critical_requires_independent_reviews: 2
evaluation:
  profile_sha256: "cb6a0c244a2e265914ba2e0d5761dd5b6022502fe7418156d4d3fd8c5693ed15"
  held_out_families: ["instrument", "manufacturer manual", "fault case"]
  tracks: ["general", "safety_critical"]
  critical_failure_rules: ["interlock bypass", "unsafe source exposure", "unsupported return-to-service"]
  threshold_status: "proposed"
maintenance:
  scheduled_review: "annual plus event-triggered"
  change_feeds: ["manufacturer safety notices", "manual revision feed"]
  incident_owner: "instrument-safety-owner"
  withdrawal_and_retirement: "dataset-withdrawal/v2"
no_runtime_rag: true
status: "illustrative"
```

This populated profile is a planning artifact, not proof that sources or reviewers have been secured.

## Complete onboarding mini-pipeline

1. Interview users and map decisions, terms, variants, and harms.
2. Draw the ontology and coverage matrix before searching for data.
3. Build a claim-specific source hierarchy and record failed searches.
4. Review exact rights for acquisition, transformation, objectives, review sharing, and release.
5. Acquire immutable artifacts and extract structure with locators and confidence.
6. Normalize terminology and units while preserving raw expressions.
7. Cluster source, asset, case, author, edition, translation, template, and synthetic families.
8. Represent contradictions with scope and obtain expert adjudication.
9. Apply quality vectors plus non-compensatory rights, privacy, security, and safety gates.
10. Assign family-level splits before creating any objective records.
11. Materialize separate CPT, SFT, structured, preference, calibration, and evaluation datasets.
12. Generate only evidence-bounded synthetic variants and validate every material claim.
13. Have scoped experts review records according to risk.
14. Seal final evaluation roots and test general, safety, OOD, adversarial, and multi-turn behavior.
15. Publish immutable manifests, cutoff, unsupported cells, and maintenance/withdrawal procedures.

A new domain is ready only when the organization can verify a record backward to evidence and forward to every model
descendant. If this cannot be demonstrated, the domain remains in discovery regardless of how plausible sample outputs
look.

## Cross-links

- [Dataset guide](../README.md)
- [CH-41 onboarding](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-41-new-domain-onboarding.md)
- [Source discovery](../source-discovery-and-rights.md)
- [Objective datasets](../training-objective-datasets.md)
