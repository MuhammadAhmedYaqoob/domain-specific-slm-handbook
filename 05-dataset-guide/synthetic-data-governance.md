# Synthetic data governance

## What synthetic data can and cannot do

Synthetic generation changes representation or creates controlled combinations. It can turn a verified claim into a
question, vary wording, render a latent scenario as dialogue, or construct a negative example from an explicit rule.
It cannot make a missing fact true. Fluency, teacher confidence, agreement among models, and repeated generation are
not evidence.

Use synthesis for three engineering problems:

1. **format coverage:** express the same supported knowledge as prose, dialogue, extraction, or structured output;
2. **controlled variation:** vary terminology, order, missing context, and difficulty while preserving invariants;
3. **decision-boundary coverage:** create explicit contrasts where one justified variable changes.

Keep source-authored and expert-authored cores. If a domain cell has no trustworthy anchor, create an expert research
task, not a generated answer.

## Generation begins from latent state

For scenario and dialogue generation, first define a typed state that the model never sees in full: verified facts,
unknowns, hazards, allowed actions, prohibited actions, and how observations become available. Rendering then exposes
only the appropriate subset at each turn. This prevents a common error in which the answer relies on information that
the user has not supplied.

For paraphrases, write explicit invariants: entity, quantity, unit, polarity, modality, jurisdiction, effective date,
and answerability. For counterfactuals, name the one changed variable and the rule that justifies the changed outcome.
Reject generations that alter any invariant.

## Status and governing principle

**AUTHORED SPECIFICATION / NO GENERATION EXECUTED.** This guide operationalizes
[CH-26](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-26-synthetic-data-and-knowledge-generation.md).

> Synthetic output is a candidate transformation, not evidence and not ground truth.

Every factual claim must resolve to an approved authoritative source, deterministic oracle/simulator, or qualified
independent expert decision. Teacher–judge agreement is not independent evidence.

## Permitted families

<!-- markdownlint-disable MD013 -->
| Family                     | Anchor                            | Permitted use                        | Main gate                               |
| -------------------------- | --------------------------------- | ------------------------------------ | --------------------------------------- |
| paraphrase/format/tone     | approved record                   | linguistic robustness                | invariant preservation                  |
| grounded Q&A/explanation   | cited source claims               | SFT candidates                       | claim-level support                     |
| multi-turn dialogue        | validated latent state            | clarification/state tracking         | no future-state leakage                 |
| scenario rendering         | typed constraints/oracle          | situational coverage                 | feasibility and state round-trip        |
| diagnostic trajectory      | fault model/simulator/expert case | safe next-action learning            | process and hazard review               |
| counterfactual/contrast    | explicit causal/domain rule       | decision boundaries                  | exactly controlled intervention         |
| negative/corruption        | explicit violated rule            | classification/preference/correction | never unlabeled as truth                |
| preference pair            | rubric + adjudication             | behavior optimization                | ties/both-bad and bias controls         |
| open “knowledge expansion” | none initially                    | research queue only                  | quarantine until independently verified |
<!-- markdownlint-restore -->

Synthetic data may transform evidence; it may not fill an evidence gap with fluent assertions.

## Pipeline

```text
approved evidence + rights → ontology/coverage/risk target
→ split root before generation → prompt contract
→ immutable raw generation log → deterministic gates
→ claim grounding → model triage (optional)
→ risk-scoped expert review → duplicate/leakage audit
→ diversity/coverage budget → immutable accepted snapshot
```

Generation never receives sealed evaluation text, answers, rubrics, or source families.

## Prompt contract

**Listing DG-SYN-01 — grounded generation contract (`UNVERIFIED IMPLEMENTATION`).**

```text
You draft candidate records; you do not establish truth.
Use only the supplied evidence IDs. Preserve entities, variants, jurisdiction,
effective date, quantities, units, negation, modality, and uncertainty.
Every material claim must cite evidence IDs. If support is incomplete, emit
answerability="insufficient_evidence" and list missing evidence.
Do not create diagnosis, prescribing, repair authorization, animal treatment,
legal status, exact specifications, or bypass instructions.
Return only the declared schema.
```

Pin generator checkpoint/API revision, prompt hash, decoding, seed where supported, raw output, timestamps, provider
request IDs, and all evidence hashes. Treat source text as untrusted data, not executable prompt instructions.

## Record contract

Each record contains stable identity, type, parents and preassigned root, domain and risk, rendered content, atomic
claims and evidence, pinned generation metadata, deterministic/model/expert validation, split, and content hash. The
fully populated record below is the contract's teaching example; production schemas enforce the field types.

### Fully populated grounded example

```json
{
  "record_id": "syn-auto-qa-00073",
  "record_type": "qa",
  "parent_ids": ["rec-auto-code-condition-000184"],
  "root_family_id": "fam-auto-charging-oer-example-college",
  "domain": "automotive",
  "risk_tier": "moderate",
  "content": {
    "question": "Does a charging-system code prove the alternator has failed?",
    "answer": "No. It identifies a monitored condition. Confirm exact applicability and gather circuit evidence before attributing the cause to a component.",
    "answerability": "supported"
  },
  "claims": [
    {
      "text": "A diagnostic code identifies a monitored condition rather than proving a failed component.",
      "evidence_refs": ["rec-auto-code-condition-000184"],
      "decision": "supported"
    },
    {
      "text": "Applicable circuit evidence is needed before component attribution.",
      "evidence_refs": ["rec-auto-code-condition-000184"],
      "decision": "supported"
    }
  ],
  "generation": {
    "generator_revision": "generator-checkpoint-pinned-by-digest",
    "prompt_sha256": "a1bfeb7cd5353ad4fdb63e80b2dfa716a949b80a359b146f91ba4a60a03fc647",
    "raw_sha256": "4ed95dd0a58b1bff046639da5b801abbe86364c10af109a349e498111d150e51",
    "decoding": {"temperature": 0.2, "top_p": 0.9, "max_new_tokens": 180},
    "generation_depth": 1
  },
  "validation": {
    "deterministic_gates": ["schema-pass", "invariant-pass", "split-pass", "pii-pass"],
    "model_triage": ["claim-extraction-pass"],
    "expert_approvals": ["auto-review-2026-121"],
    "decision": "accept"
  },
  "split": "train",
  "content_sha256": "ee77254cc5fa0bedce50d45cd3a44bab1431b27800d614318bba8d31f97e6324"
}
```

The synthetic row inherits the source root. It therefore cannot enter a split different from the original claim.

## Practical generation and validation procedure

1. Select an already-approved, already-split root.
2. Choose a coverage gap such as paraphrase, clarification, negative example, or structured rendering.
3. Define the output schema, invariants, prohibited content, and evidence IDs.
4. Pin generator, prompt, decoding, and input hashes.
5. Store raw output before editing.
6. Parse and validate schema; reject malformed output rather than repairing it invisibly.
7. Atomize material claims and test each against the supplied evidence.
8. Check units, negation, entity scope, dates, jurisdiction, and turn-level state.
9. Search for duplicate, template, and benchmark overlap.
10. Route by risk to the required reviewer; save edits as a new derivative.
11. Measure acceptance rate and rejection reasons by generator, template, and risk.
12. Compare a validated synthetic mixture with expert-only and unfiltered controls on untouched evaluation roots.

## Expert verification is part of the label

“Human reviewed” is too vague. Store reviewer qualification, scope, decision, confidence, reason codes, source
consulted, and whether edits were made. A pharmacist reviewing terminology is not automatically qualified to approve a
patient-specific clinical action; an automotive technician is not automatically a high-voltage specialist. Review
scope follows the claim and hazard.

For high-consequence rows, require independent reviews before adjudication. Do not show reviewer two the first decision
until the independent judgment is captured. Disagreement is diagnostic: it can reveal ambiguous evidence, a weak
rubric, or unresolved applicability. Resolve the cause instead of averaging ratings.

## Detecting synthetic collapse

Monitor more than lexical diversity. Watch for shrinking source-family diversity, repeated reasoning patterns,
uniformly polished tone, loss of rare terminology, unsupported-claim growth, and performance that improves only on
synthetic-style tests. Track generation depth: a paraphrase of a paraphrase compounds error and should normally be
capped. The stopping rule is downstream regression on independent expert tests, not exhaustion of a generation budget.

## Multi-turn and diagnostic rules

Define latent state before rendering: user goal, verified facts, unknowns, hazards, permitted actions, and turn-level
disclosures. A turn may use only information available at that turn. Synthetic “that fixed it” is not an observed
outcome.

Diagnostic records begin from a validated fault tree, causal graph, simulator, or expert case. Generate observations
from the latent state, expose only a subset, enumerate safe discriminating actions and unsafe/non-discriminating
alternatives, then review sequence and hazards. Keep possible cause, next test, and confirmed cause distinct.

## Transform invariants

Paraphrases must preserve entity, negation, quantity, unit, order, modality, applicability, and answerability.
Counterfactuals change exactly one causally justified variable and record changed/invariant fields. Difficulty varies
through reasoning depth, ambiguity, distractors, missing information, rare terminology, or structure—not mere length.

Negative examples carry a violated-rule ID and corrected behavior. They belong in explicit classification, preference,
or correction structures; never place plausible falsehoods into ordinary completion data.

Preference records preserve prompt, candidate provenance, randomized order, rubric dimensions, reason codes, reviewer
scope, ties, and `both_unacceptable`. Safety/factuality cannot be traded for style.

## Validation gates

### Deterministic

- schema, encoding, roles, lengths, IDs, and hashes;
- controlled vocabularies, ontology, identifiers, dates, units/dimensions/ranges;
- state-to-render consistency;
- prohibited PII, confidential/security content, or excessive source copying;
- required warnings, stop conditions, and escalation fields;
- root/parent/template/duplicate split isolation;
- benchmark/test fingerprints and known fragments.

### Grounding

Atomize material claims into `supported`, `unsupported`, `conflicting`, or `unclear`. Unsupported/unclear
safety-critical claims reject the record; an average score cannot compensate. Citations must resolve to the correct
version and entail the claim.

### Model triage

Models may extract claims, flag contradictions, rank, or route. Pin and log them; calibrate against an expert-labeled
audit set; test position, length, style, and self-preference bias. They cannot certify truth.

### Expert review

- low risk: grounded record plus calibrated sampled audit;
- moderate: trained domain reviewer with escalation;
- high: qualified domain expert;
- critical: two scoped qualified reviews and adjudication.

Capture original ratings, edits as new derivatives, reason codes, confidence, time, conflicts, and agreement. Low
agreement triggers evidence/rubric revision, not majority-vote release.

## Diversity, novelty, and coverage

Report accepted roots by domain, task, risk, language, source, generator, template, difficulty, and rare stratum.
Distinct-\(n\), Self-BLEU, embedding dispersion, cluster entropy, or MAUVE are descriptive proxies; none verifies truth.
High novelty can mean unsupported text. Measure lexical/semantic duplication and human coverage together.

Track:

- claim support and unsupported-claim rate;
- source/generator/template concentration;
- acceptance funnel and rejection reasons by risk;
- expert time and agreement;
- required-cell coverage by independent root;
- synthetic token fraction and synthetic-only ancestry depth;
- downstream quality, safety, calibration, and tail recall.

No universal safe synthetic ratio exists. Choose mixture through preregistered expert-only, synthetic-only,
validated-mixed, and unfiltered-control ablations.

## Contamination and recursive-risk controls

Freeze final tests and root fingerprints before generation. Do not expose them to teachers, judges, prompt authors, or
selection code. Group prompts, evidence, templates, scenarios, counterfactuals, and paraphrases with their ancestor.

To reduce feedback loops:

- retain a durable human/source-authored core;
- cap recursive depth and synthetic-only ancestry;
- preserve rare strata and rejection logs;
- use independent evidence families, not merely different generators;
- compare each round against untouched expert tests;
- stop at preregistered regressions in factual error, safety, calibration, rare recall, or diversity.

Generator diversity is not evidence diversity.

## Domain hard limits

- **Pharmacy:** no patient-specific dose, diagnosis, prescribing, interaction/contraindication clearance, substitution,
  or emergency delay; authoritative versioned sources and qualified clinical review.
- **Repair/industrial/electrical:** exact model/variant and procedure applicability; isolation, support, stored energy,
  PPE, competence, and stop conditions; no bypass or unsafe live-work authorization.
- **Beekeeping/zookeeping:** preserve taxon, life stage, region, season, jurisdiction, individual context, welfare, and
  veterinary escalation; no generated treatment, diet, restraint, or legal authorization.

## Release criteria

Accepted synthetic snapshots require complete rights and ancestry, zero final-test lineage, all material claims
supported, mandatory expert approvals, duplicate/template concentration within approved limits, and a documented
comparison against non-synthetic baselines. Publish limitations and rejection rates. Synthetic data do not directly
reduce model size or prove better edge performance.

## Cross-links

- [Dataset guide](README.md)
- [Source discovery](source-discovery-and-rights.md)
- [Objective datasets](training-objective-datasets.md)
- [Safety standard](../00-governance/safety-and-risk-standard.md)
