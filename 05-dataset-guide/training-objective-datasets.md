# Training-objective datasets

## Why one corpus cannot serve every objective

An objective defines the signal consumed by an optimizer or evaluator. CPT learns continuation from document streams;
SFT learns response behavior from prompts and reviewed targets; preference learning consumes relative judgments;
calibration estimates numerical behavior; evaluation measures a frozen system against independent truth. Build one
canonical evidence layer, then render separate views with their own eligibility, schemas, manifests, and checks.

## Status and core rule

**AUTHORED SPECIFICATION / NOT EXECUTED.** This guide condenses
[CH-25](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md).
Build one governed canonical lineage graph, then materialize separate immutable views. A source corpus, training view,
calibration set, and benchmark are not interchangeable.

## Objective matrix

<!-- markdownlint-disable MD013 -->
| Objective                           | Unit and learning/measurement signal              | Required content                                          | Forbidden leakage/failure                                       |
| ----------------------------------- | ------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------- |
| CPT/DAPT                            | document stream; next-token targets               | representative approved prose, boundaries, revisions      | chat wrappers, final benchmarks, superseded/current blending    |
| SFT                                 | prompt/messages + approved assistant target       | source-supported response, uncertainty, safe behavior     | raw prose as answer; hidden evaluator state                     |
| multi-turn                          | ordered trajectory + state-before-turn            | clarification, correction, delayed evidence               | future observations visible early                               |
| diagnostic reasoning                | evidence state + safe next action                 | hypotheses, allowed tests, prerequisites, stop conditions | symptom→cause shortcut; hidden diagnosis                        |
| classification/extraction           | input + ontology/schema target                    | null/unknown semantics, evidence spans                    | inaccessible fields; invented values                            |
| preference                          | same prompt + candidates + scoped judgment        | rubric dimensions, ties/both-bad, randomized order        | chosen treated automatically as truth                           |
| response/logit/feature distillation | prompt + pinned teacher signal                    | raw/reviewed output, alignment, independent evidence      | teacher treated as authority                                    |
| quantization calibration            | deployment-representative rendered inputs         | lengths, symbols, languages, tails, exact template        | final-test items; label-driven tuning                           |
| pruning calibration                 | activations/statistics and optional masks/targets | statistic, eligible structures, mixture                   | benchmark reuse; undocumented masks                             |
| evaluation                          | independent item + scoped truth/rubric            | construct, strata, uncertainty, critical failures         | any training, prompting, calibration, judging, or selection use |
<!-- markdownlint-restore -->

## Canonical core

Every canonical record carries stable identity, typed content, source and transform lineage, rights eligibility,
quality and review state, risk, difficulty, family split, and effective dates. The complete records later in this
chapter demonstrate those fields without presenting an empty object as usable data. Production requires strict schemas
and registry resolution.

## Objective view contracts

### CPT

Fields: `id`, `text`, language, boundary/EOS policy, document/revision, root, authority, effective dates, split, risk.
Deduplicate and split before chunking. Preserve warning/table context, document boundaries, and realized token counts by
source. CPT teaches language exposure, not response policy.

### SFT

Fields: `messages` (or prompt/completion), exact chat template hash, assistant-only/completion loss mask, source claims,
risk, review, root, split. Include supported answers, clarifying questions, insufficient-information responses, safe
refusals, corrections, structured outputs, and benign controls. Inspect rendered tokens, truncation, role order, masks,
and EOS.

### Multi-turn and diagnostics

Store hidden state separately from rendered messages: what is known and unknown before the turn, hazards, allowed and
preferred actions, prohibited actions, the observation revealed only after an action, and updated hypotheses. The
populated multi-turn record later in this chapter demonstrates the representation.

Do not expose future evidence. Diagnostic targets distinguish observation, possible cause, discriminating action,
confirmed cause, and disposition.

### Structured prediction

Version the ontology/schema, field types, controlled vocabularies, units, null/unknown/not-applicable semantics,
evidence spans, label precedence, abstention, and migrations. Score syntax, field correctness, source fidelity, and
safety independently.

### Preference

Keep prompt, all candidates, generator/revision/decoding, randomized display order, rubric, per-dimension scores,
annotator qualification, confidence, ties, `both_unacceptable`, and adjudication. Factuality and safety outrank style.
Route both-bad cases to expert target authoring rather than forcing a pair.

### Distillation

Response records retain raw and accepted teacher output, generator configuration, claim evidence, and review. Logit
records retain exact input IDs, checkpoint/tokenizer/template hashes, temperature, mask, positions, precision, top-$k$
IDs/logits, and omitted-mass statistic. Feature records retain layer mapping, shapes, dtypes, positions, and storage
hashes. Tokenizer mismatch requires an explicit alignment method.

### Quantization and pruning calibration

Use non-evaluation roots representative of exact deployment rendering, domain/task mixture, length, symbols, units,
languages, safety prompts, multi-turn histories, and tail cases. Quantization calibration estimates
ranges/reconstruction; pruning calibration estimates importance/masks. Keep separate manifests and prove
interchangeability experimentally before sharing.

### Evaluation

Each item has construct, prompt, hidden case state, scoped reference, accepted alternatives, required/forbidden
concepts, rubric, critical failures, risk/difficulty/variant strata, source/version evidence, independent root,
reviewer/adjudication, expiry, and seal class. Development, regression, final, and time-forward suites are distinct.

## Split and usage policy

Assign connected source/case/entity/author/template/translation/synthetic families before any transformation. Final test
and its source family are forbidden from:

- CPT/SFT/preference/distillation;
- quantization or pruning calibration;
- teacher prompts, few-shot examples, model-judge training;
- prompt/rubric/threshold selection;
- checkpoint, mixture, compression, export, or runtime selection.

When an evaluation item informs a change, retire it to development history and commission a fresh independent root.

## Balancing and coverage

Sample hierarchically: domain → task → risk/difficulty → authority/source → record. Report configured and realized rows,
input tokens, target tokens, unique roots, and repetitions. Sampling frequency and loss weight are separate controls.
Cap source, organization, template, and generator concentration.

Difficulty and consequence are independent:

- D1 direct;
- D2 one transformation/qualifier;
- D3 multiple evidence items or missing context;
- D4 compositional, long-tail, counterfactual, or multi-turn;
- D5 adversarial ambiguity requiring exact abstention/escalation.

Maintain a risk-weighted coverage ledger. Rare critical cases require diverse independent roots, controlled contrasts,
qualified review, and non-compensatory evaluation; repeating a few cases is not coverage.

## Required validation

Common:

- strict schema and referential integrity;
- complete acyclic lineage and rights resolution;
- root-level split and supersession consistency;
- exact/near/semantic/lineage leakage report;
- PII, confidential, security, and unsafe-content review;
- distributions by domain/task/risk/difficulty/authority/source;
- immutable files, hashes, code/config/environment pins.

Objective-specific:

- CPT: boundaries, packing, boilerplate, token share;
- SFT: template, roles, masks, truncation, target support;
- multi-turn: state consistency and no future leakage;
- diagnostic: feasibility, prerequisites, safe sequence;
- structured: ontology, null semantics, evidence spans;
- preference: same prompt, order randomization, ties/both-bad;
- distillation: teacher pin, alignment, grounding, omitted mass;
- calibration: exact rendered IDs and deployment coverage;
- evaluation: construct validity, independent truth, seal/access logs.

## Safety and release

All high/critical targets require risk-scoped expert acceptance. A model judge may triage but cannot approve
consequential truth. Releases state knowledge cutoff, jurisdictions, intended/prohibited use, unsupported strata,
synthetic composition, known leakage uncertainty, update cadence, and withdrawal procedure. Training cannot guarantee
exact recall, freshness, or safety.

## Complete illustrative record set

These records share a teaching theme but are deliberately assigned according to objective. Production records need
real approved source IDs and cryptographic hashes.

### CPT record

```json
{
  "id": "cpt-auto-000184",
  "text": "Diagnostic codes identify monitored conditions. Confirm circuit evidence before attributing a fault to a component.",
  "language": "en-US",
  "boundary_policy": "document_eos",
  "document_revision": "src-auto-oer-charging-v2",
  "root_family_id": "fam-auto-charging-oer-example-college",
  "authority_tier": "A3",
  "effective_from": "2026-02-12",
  "split": "train",
  "risk_tier": "moderate"
}
```

This is prose exposure. It has no user role and should not be treated as an instruction target.

### SFT record

```json
{
  "id": "sft-auto-000184",
  "messages": [
    {
      "role": "user",
      "content": "The scan tool shows a charging-system code. Should I replace the alternator?"
    },
    {
      "role": "assistant",
      "content": "The code does not confirm a failed component. Identify the vehicle and exact code, preserve operating-condition data, and follow current model-specific tests. Use qualified service information if a hazardous voltage system may be involved."
    }
  ],
  "chat_template_revision": "template-v4",
  "loss": "assistant_only",
  "claim_record_ids": ["rec-auto-code-condition-000184"],
  "review_status": "accepted",
  "root_family_id": "fam-auto-charging-oer-example-college",
  "split": "train"
}
```

The target teaches bounded behavior: distinguish evidence from cause, request applicability, and avoid hazardous
procedural detail.

### Multi-turn diagnostic record

```json
{
  "id": "dialogue-auto-0021",
  "messages_visible_before_turn": [
    {"role": "user", "content": "The charging warning lamp comes on intermittently."}
  ],
  "known_before_turn": ["warning lamp is intermittent"],
  "unknown_before_turn": ["vehicle identity", "exact code", "operating conditions", "measurements"],
  "hazards": ["configuration may include high voltage"],
  "allowed_actions": ["ask for vehicle configuration", "request exact code and conditions"],
  "preferred_next_actions": ["identify powertrain and exact code"],
  "prohibited_actions": ["declare a component failed", "recommend hazardous probing"],
  "new_observation_after_action": "The vehicle is a conventional 12 V system and code P0622 is stored.",
  "hypotheses_after_turn": ["control-circuit issue", "connection issue", "charging-unit issue"],
  "root_family_id": "fam-auto-case-0021",
  "split": "validation"
}
```

The future observation is hidden until the selected action occurs. Exposing it in the initial prompt would teach
look-ahead leakage.

### Structured extraction record

```json
{
  "id": "extract-auto-0042",
  "schema_version": "diagnostic-observation/v3",
  "input": "At idle, the warning lamp flickers; no voltage measurement was recorded.",
  "target": {
    "symptom": "charging warning lamp flickers",
    "operating_condition": "idle",
    "measurement": null,
    "measurement_status": "not_measured",
    "confirmed_cause": null
  },
  "evidence_spans": {"symptom": [13, 34], "operating_condition": [3, 7]},
  "root_family_id": "fam-auto-case-0042",
  "split": "train"
}
```

`null` means no value is available; `not_measured` explains why. It must not be silently converted to zero.

### Preference record

```json
{
  "id": "pref-auto-0091",
  "prompt": "A charging code is present. What part should be replaced?",
  "candidates": [
    {"candidate_id": "A", "text": "Replace the alternator because the code names the failed part."},
    {"candidate_id": "B", "text": "Do not select a part from the code alone. Confirm configuration and circuit evidence."}
  ],
  "display_order": ["B", "A"],
  "rubric_scores": {
    "A": {"factuality": 0, "safety": 1, "applicability": 0, "clarity": 2},
    "B": {"factuality": 3, "safety": 3, "applicability": 3, "clarity": 3}
  },
  "chosen": "B",
  "tie": false,
  "both_unacceptable": false,
  "annotator_scope": "qualified_automotive_technician",
  "confidence": 0.98,
  "root_family_id": "fam-auto-charging-oer-example-college",
  "split": "train"
}
```

The label expresses rubric-scoped preference; it does not make candidate B true without independent evidence.

### Response-distillation record

```json
{
  "id": "distill-auto-0030",
  "input": "Explain why a fault code is not a replacement instruction.",
  "teacher_revision": "teacher-checkpoint-pinned-by-digest",
  "temperature": 0,
  "raw_output": "A code reports that monitoring logic detected a condition...",
  "accepted_output": "A code reports a monitored condition. Use applicable tests to distinguish possible causes before repair.",
  "claim_evidence_ids": ["rec-auto-code-condition-000184"],
  "review_status": "accepted",
  "root_family_id": "fam-auto-charging-oer-example-college",
  "split": "train"
}
```

Store raw and accepted output so edits and teacher error rates remain measurable.

### Calibration record

```json
{
  "id": "cal-auto-0017",
  "rendered_text": "[USER] Interpret code P0622 without assuming a failed part. [ASSISTANT]",
  "tokenizer_revision": "tokenizer-pinned-by-digest",
  "chat_template_revision": "template-v4",
  "length_tokens": 31,
  "strata": ["automotive", "identifier-heavy", "moderate-risk", "short"],
  "eligible_use": ["quantization_calibration"],
  "root_family_id": "fam-auto-calibration-independent-0017",
  "split": "calibration"
}
```

Calibration records represent deployment inputs but supply no final-test labels.

### Evaluation record

```json
{
  "id": "eval-auto-final-0107",
  "construct": "evidence_before_component_attribution",
  "prompt": "A stored code names a generator field-control circuit. State what is known and the safest next decision.",
  "reference": "The code establishes a detected circuit condition, not a failed generator. Identify applicability and gather approved circuit evidence.",
  "required_concepts": ["code is evidence", "applicability", "test before replacement"],
  "forbidden_concepts": ["automatic generator replacement", "unsafe probing"],
  "critical_failures": ["component replacement from code alone"],
  "risk_tier": "moderate",
  "difficulty": "D3",
  "root_family_id": "fam-final-auto-0107",
  "review": "two-reviewer-adjudicated",
  "expires_at": "2027-11-01",
  "seal_class": "final_restricted"
}
```

This visible item teaches the schema only. Real final items belong in a restricted store inaccessible to data
generation, prompt design, calibration, judges, and model selection.

## Choosing and validating a view

For each view, write down the learning signal, rendering step, eligible roots, forbidden roots, loss or metric, and
failure modes before creating rows. Inspect rendered token IDs rather than trusting source JSON. Sample by independent
roots, not row count; ten thousand paraphrases of ten cases still provide only ten evidence roots. Freeze the manifest
before training and attach it to every run.

## Cross-links

- [Dataset guide](README.md)
- [Collection and lineage](collection-processing-and-lineage.md)
- [Synthetic governance](synthetic-data-governance.md)
- [CH-30 evaluation](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
