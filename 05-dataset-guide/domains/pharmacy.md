# Pharmacy dataset guide

## Contract and status

**AUTHORED EDUCATIONAL PROFILE / NOT CLINICALLY OR LEGALLY APPROVED / NO DATASET BUILT.** The model may explain
terminology, label structure, evidence types, and bounded pharmaceutical facts. It is not a prescriber, dispenser,
diagnosis system, patient-specific interaction/contraindication checker, dose calculator, substitution authority,
medication-order validator, poison service, or emergency decision-maker.

## Ontology and scope

Preserve ingredient/active moiety, product, brand, strength expression, dose form, route, release characteristic,
package, product identifier, class, indication, contraindication, warning/precaution, adverse reaction, interaction,
storage, regulatory status, jurisdiction, vocabulary/version, label Set ID/version/section, effective interval, and
evidence type.

Never collapse ingredient with product, strength with dose, dose form with route, lexical synonym with equivalence,
terminology relation with recommendation, or similarly named products across jurisdictions.

Claim scope is:

```text
entity × formulation/route × population/context × jurisdiction × effective time
```

Missing decisive scope requires clarification, bounded education, or professional/current-source escalation.

## Sources and rights

Candidate authoritative entry points include openFDA drug-label APIs and terms, DailyMed/SPL services, RxNorm/RxNav
terms, Drugs@FDA, Orange/Purple Book data, FDA safety communications/recalls/shortages, PubMed, and corresponding
national regulators. `DOC-VERIFIED` entry points do not clear every payload. openFDA notes CC0 unless otherwise marked
and third-party exceptions; asset/field review remains mandatory.

Subscription compendia, interaction databases, formularies, terminology distributions, journal full text, clinical
records, and commercial decision support require explicit contracts/permissions. Patient narratives, prescriptions, and
linked records are excluded absent an approved clinical-data process. De-identification does not grant rights.

Preserve evidence type: approved label, safety communication, approval record, terminology, guideline, review, primary
study, spontaneous report, or expert derivative. A report does not prove causality/incidence; terminology does not prove
a recommendation.

## Canonical extensions

Pharmacy records add ingredient and product identifiers, raw brand and strength strings, dose form, route, package,
jurisdiction, label set/version, effective time, evidence type, and the explicit prohibition on clinical action. The
populated fictional label record later in this chapter demonstrates every field. Ambiguous mappings remain unresolved
and reviewed. Never auto-correct names, decimals, strengths, units, or release modifiers.

## Objective views and leakage

- CPT: current rights-approved labeling/education, with versions and jurisdictions; historical text explicitly marked.
- SFT: terminology, label comprehension, evidence literacy, structured extraction, uncertainty, refusal, and escalation.
- Multi-turn: exact product/formulation/context appears later; model must revise without prescribing.
- Structured: source-linked fields and explicit unknowns.
- Preference: scoped education versus unsafe compliance, fabricated certainty, or universal refusal.
- Distillation: teacher claims require independent evidence and pharmacist review.
- Calibration: names, codes, decimals, units, long labels, dialogue, safety prompts.
- Evaluation: independent product/formulation/time/jurisdiction families and time-forward changes.

Final roots are forbidden from all training, teacher, prompt, calibration, judge, and selection paths.

## Review and safety

Licensed pharmacists review consequential records; physicians, clinical pharmacologists, toxicologists/poison
specialists, regulatory professionals, privacy/security, and rights counsel review their scopes. Critical items require
two qualified independent reviews plus adjudication.

Reject any unsupported dose, interaction, contraindication, allergy, pregnancy/lactation, pediatric/geriatric,
renal/hepatic, poisoning, emergency, route, formulation, or strength claim. Patient-specific diagnosis,
start/stop/change, clearance, substitution, or delayed escalation is release-blocking.

## Evaluation

Suites cover terminology; product/formulation/route/strength distinctions; label and evidence comprehension; risk
recognition; missing context; unsafe-compliance and over-refusal matched pairs; locally approved escalation;
stale/withdrawn facts; OOD products/languages; and multi-turn correction.

Report entity exactness, factual support, applicability, risk-recognition precision/recall, unsafe compliance, correct
refusal, over-refusal, necessary-question recall, escalation, time-forward cutoff recognition, worst groups, and
critical failures. `PROPOSED POLICY`: zero defined prescribing/dose/clearance/substitution/emergency-delay critical
failures; other thresholds require clinical risk calibration.

## Freshness and release

Monitor source versions, label changes, safety communications, recalls/withdrawals, status, terminology, rights/terms,
and incidents. Changes enter human adjudication and can block new releases; they never trigger autonomous retraining.
Every artifact exposes jurisdictions, cutoff, intended/prohibited uses, expiry, update route, and offline staleness
limits.

## How to structure pharmaceutical knowledge safely

Pharmaceutical data is versioned entity-and-evidence data. A name can refer to an ingredient, active moiety, branded
product, package, dose form, or vocabulary concept. Keep those levels separate and attach route, strength expression,
release characteristic, jurisdiction, population/context, source section, and effective date to every claim.

Terminology relations such as synonymy, ingredient membership, and class membership are not clinical equivalence or a
recommendation. Likewise, a spontaneous report is a report, not proof of causality or incidence. Dataset labels must
preserve evidence type and the limits of what that type can establish.

Use the applicable regulator and current approved labeling for regulatory status and label content; versioned
terminologies for identifiers and relations; reviewed guidelines for their exact scope; primary studies and reviews for
research questions; qualified pharmacists for educational derivations. Never make a general web summary the authority
for a consequential record. Monitor safety communications, recalls, withdrawals, and label changes continuously;
refresh terminology on its published cadence and re-review educational derivatives when anchors change.

## Illustrative dataset record

```json
{
  "record_id": "pharm-label-structure-0012",
  "task": "label_field_extraction",
  "input": "Fictional label example: Active ingredient Alphaine; tablet; oral route; 10 mg strength.",
  "target": {
    "ingredient_raw": "Alphaine",
    "dose_form": "tablet",
    "route": "oral",
    "strength_raw": "10 mg",
    "patient_dose": null,
    "recommendation": null
  },
  "pharmaceutical_scope": {
    "ingredient_ids": ["EXAMPLE-INGREDIENT-001"],
    "product_ids": ["EXAMPLE-PRODUCT-010"],
    "brands_raw": [],
    "strengths_raw": ["10 mg"],
    "dose_forms": ["tablet"],
    "routes": ["oral"],
    "package_ids": ["EXAMPLE-PACKAGE-100"],
    "jurisdiction": "fictional-training-jurisdiction",
    "label_set_id": "EXAMPLE-SET-0001",
    "label_version": 3,
    "effective_at": "2026-10-01"
  },
  "evidence_type": "fictional_label_training_example",
  "clinical_action_allowed": false,
  "root_family_id": "fam-pharm-fictional-label-001",
  "split": "train",
  "review": {"scope": "pharmaceutical_terminology", "decision": "accepted_for_schema_teaching"}
}
```

Every product and identifier is explicitly fictional. The example teaches extraction and null semantics; it makes no
claim about a real medicine, dose, interaction, or use.

## Worked mini-pipeline

1. Define a product–ingredient–form–route–package–label ontology and explicit prohibited clinical actions.
2. Register exact regulator assets, terminology releases, and educational sources; review payload-level rights.
3. Acquire immutable versions and preserve label section IDs, set IDs, effective dates, and update receipts.
4. Parse structured fields without auto-correcting names, decimals, units, or release modifiers.
5. Normalize identifiers through versioned mappings while retaining raw strings and unresolved ambiguity.
6. Cluster label versions, packages, terminology releases, and derived questions into root families.
7. Represent contradictions as formulation, jurisdiction, source type, or time differences; send unresolved cases to
   qualified review.
8. Split by product/formulation/source/time family before generation; reserve later independent changes for
   time-forward evaluation.
9. Render terminology explanation, label comprehension, structured extraction, uncertainty, refusal, and escalation
   records; do not render patient-specific decisions.
10. Require pharmacist review for consequential content and additional scoped review where needed.
11. Seal independent product and label families for evaluation, including unsafe-compliance and over-refusal pairs.
12. On a source change, suspend affected records, adjudicate the new version, rebuild descendants, and publish the new
    cutoff.

Verification means resolving every statement to the exact source version and qualified review scope. An offline model
cannot confirm current status and must not be presented as a prescriber, dispenser, interaction checker, or emergency
service.

## Cross-links

- [Dataset guide](../README.md)
- [CH-38 pharmaceutical SLM](../../02-textbook/part-11-extending-the-platform-to-other-domains/ch-38-pharmaceutical-knowledge-slm.md)
- [Synthetic governance](../synthetic-data-governance.md)
