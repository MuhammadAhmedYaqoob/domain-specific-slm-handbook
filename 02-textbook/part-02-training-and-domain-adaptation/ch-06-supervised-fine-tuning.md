# CH-06: Supervised Fine-Tuning

<a id="ch06-status"></a>

## How to use this chapter

Supervised fine-tuning (SFT) is a behavior lesson, not a database import. Think of it as a flight simulator: curated
scenarios teach the model how to respond under recognizable conditions, but the simulator cannot cover every future
case or certify the pilot. The chapter therefore connects each dataset format to its token-level loss, expected
behavior, failure modes, implementation checks, and domain/edge consequences.

<a id="ch06-objectives"></a>

## 6.1 Learning objectives

After completing this chapter, the reader should be able to:

1. distinguish SFT, instruction tuning, chat fine-tuning, task fine-tuning, and continued pretraining;
2. construct single-turn, multi-turn, prompt-completion, structured-output, diagnostic, and optional tool-use records;
3. explain full-sequence, completion-only, and assistant-only loss at token level;
4. verify that a chat template and loss mask identify exactly the intended target tokens;
5. design multi-domain mixtures without allowing a large or easy domain to dominate training;
6. detect overfitting, memorization, answer-format imitation, and hallucination regressions;
7. train uncertainty, refusal, escalation, terminology, and domain-specific answer formats without confusing behavior
   learning with knowledge acquisition; and
8. design an ablation that separates SFT effects from model, data, template, and decoding effects.

<a id="ch06-problem"></a>

## 6.2 The problem SFT solves

A pretrained causal language model predicts plausible next tokens. It has not necessarily learned that a user question
should receive a direct answer, that a diagnostic response should gather evidence before recommending a repair, or that
a pharmacy question may require refusal and escalation. SFT supplies demonstrations of desired input-output behavior.

The intuition is imitation under a causal language-model objective:

> Show the model many correctly formatted examples of “when the conversation looks like this, continue it like that.”

This can teach:

- instruction interpretation and response initiation;
- conversational roles and turn boundaries;
- task mappings such as classification, extraction, summarization, and question answering;
- preferred terminology, units, answer order, tone, and schema;
- diagnostic interaction policies such as asking discriminating questions;
- uncertainty language, refusal boundaries, and escalation behavior; and
- optional serialization of tool calls.

It cannot guarantee:

- complete absorption of a manual, formulary, or standards corpus;
- exact recall of every fact;
- truthfulness outside the demonstrated distribution;
- calibrated probabilities merely because responses contain words such as “uncertain”;
- safe autonomy in high-risk domains; or
- faster edge inference. SFT normally changes weights or adds an adapter, not the fundamental inference algorithm.

### 6.2.1 Technical definition

Given a tokenized example \(x_{1:T}\), a causal model with parameters \(\theta\), and a binary supervision mask
\(m_t\), SFT minimizes masked next-token cross-entropy:

\[ \mathcal{L}_{\mathrm{SFT}}(\theta)= -\frac{1}{\sum_{t=1}^{T-1}m_{t+1}} \sum_{t=1}^{T-1}m_{t+1} \log <!-- markdownlint-disable-line MD013 MD049 -->
p_\theta(x_{t+1}\mid x_{\le t}) \]

where:

- \(T\) is sequence length;
- \(p_\theta\) is the model's next-token distribution;
- \(m_{t+1}=1\) means token \(x_{t+1}\) contributes to loss;
- \(m_{t+1}=0\) means its label is ignored, conventionally represented as `-100` in PyTorch cross-entropy.

The denominator normalizes by supervised tokens. Implementations may average differently across tokens, examples,
devices, or accumulated microbatches; inspect the trainer rather than assuming mathematically identical weighting.

### 6.2.2 SFT, instruction tuning, chat tuning, and task tuning

<!-- markdownlint-disable MD013 -->
| Term                        | Training data                                       | Primary intention                                  | Common failure if confused                             |
| --------------------------- | --------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------ |
| SFT                         | Any labeled input-target demonstrations             | Imitate target outputs                             | Treating all SFT as open-ended chat                    |
| Instruction tuning          | Diverse natural-language instructions and responses | Generalize instruction following across tasks      | Mistaking task diversity for factual coverage          |
| Chat fine-tuning            | Role-structured conversations                       | Learn dialogue protocol and response behavior      | Using the wrong chat template                          |
| Task-specific SFT           | Narrow task/schema examples                         | Maximize one defined capability                    | Losing breadth or overfitting format                   |
| QA fine-tuning              | Questions with reference answers                    | Improve answer behavior on a question distribution | Memorizing surface forms or guessing unsupported facts |
| Diagnostic conversation SFT | States, observations, questions, actions, outcomes  | Learn evidence-gathering and escalation policy     | Training premature diagnosis from answer-only cases    |
<!-- markdownlint-restore -->

Instruction tuning is therefore a subset or use pattern of SFT, not a different optimizer. FLAN showed that instruction
tuning over more than 60 verbalized NLP datasets improved zero-shot performance on held-out tasks at the reported scale;
the result supports task-diverse demonstrations, not the claim that SFT provides comprehensive specialist knowledge.
`PAPER-REPORTED` [Wei et al., 2022](https://arxiv.org/abs/2109.01652).

<a id="ch06-mechanism"></a>

## 6.3 Internal mechanism: from record to gradient

**FIG-CH06-01 — SFT data flow (`ILLUSTRATIVE`)** Alt text: governed examples are templated, tokenized, masked to the
intended loss region, batched, trained, and evaluated before release.

```text
Licensed / expert-reviewed records
        │
        ├── provenance, domain, task, risk metadata
        ▼
Schema validation ──> leakage / duplicate / contradiction checks
        │
        ▼
messages or prompt+completion
        │
        ▼
exact tokenizer + exact chat template
        │
        ├── input_ids: [system][user][assistant]...
        └── loss mask: [   0  ][ 0  ][    1    ]...
        ▼
truncate / pack with boundary checks
        ▼
causal LM forward pass → token losses → masked mean
        ▼
backpropagation → full weights or PEFT adapter
        ▼
held-out factual, format, safety, calibration, retention evaluation
```

The model never sees JSON keys such as `messages` unless the formatting code renders them. The JSON is a storage
contract. The chat template converts role objects into the exact control-token sequence expected by the model.

For a simple rendered conversation:

```text
<system> You are a careful technician. </system>
<user> The engine cranks but will not start. What next? </user>
<assistant> First confirm battery voltage while cranking... </assistant>
```

a full-sequence mask, completion-only mask, and assistant-only mask can differ:

```text
token group       system  user-1  assistant-1  user-2  assistant-2
full sequence        1       1          1          1          1
assistant only       0       0          1          0          1
last completion      0       0          0          0          1
```

This is `ILLUSTRATIVE`: real boundaries occur at token positions, and control/end tokens require an explicit policy.

### 6.3.1 Full-sequence loss

Every non-padding token is supervised. It can be appropriate for plain language-model records or when one intentionally
wants the model to learn the distribution of all roles. For chat, it spends optimization weight predicting fixed system
prompts and user text. It may also teach the assistant to imitate user turns when generation begins from an unexpected
boundary.

### 6.3.2 Completion-only loss

For a `prompt` plus `completion` record, prompt labels are ignored and completion labels are active. Current TRL
documentation states that prompt-completion datasets use completion-only loss by default when `completion_only_loss` is
unspecified, and that the option is supported for prompt-completion datasets. `DOC-VERIFIED` Do not transfer that
behavior to arbitrary `text` datasets.

### 6.3.3 Assistant-only loss

For a conversational record, labels for system, user, and tool messages are ignored while assistant content is
supervised. Current TRL documentation exposes `assistant_only_loss=True`, but requires a compatible chat template whose
assistant spans are marked by `{% generation %}` and `{% endgeneration %}`; TRL documents automatic patching only for
supported template families. `DOC-VERIFIED`

Assistant-only and completion-only are not synonyms:

- **assistant-only** can supervise every assistant turn in a multi-turn record;
- **completion-only** supervises the completion field after a prompt;
- a conversational prompt-completion record can use both, subject to current trainer/template support.

### 6.3.4 Numerical mask example

Suppose eight target positions have negative log-likelihoods:

\[ [0.2,\ 0.3,\ 1.0,\ 0.4,\ 0.7,\ 0.2,\ 0.9,\ 0.3] \]

and assistant-only masking selects positions 3, 4, 7, and 8:

\[ m=[0,0,1,1,0,0,1,1]. \]

Then:

\[ \mathcal{L}=(1.0+0.4+0.9+0.3)/4=0.65. \]

A batch with twice as many assistant tokens can exert more influence if the implementation averages all active tokens
globally. This is why dataset row weights, token counts, sequence lengths, and loss normalization must all be audited.

<a id="ch06-templates"></a>

## 6.4 Chat templates are part of the model

Transformers defines a chat template as a Jinja template associated with the tokenizer. It maps a list of role/content
objects to model-specific control tokens. Models fine-tuned from the same base can still expect different chat formats.
`DOC-VERIFIED` [Transformers chat-template documentation](https://huggingface.co/docs/transformers/chat_templating).

Critical rules:

1. Use the selected checkpoint's tokenizer and template; never infer a template from the family name.
2. During training on complete conversations, official Transformers guidance uses `add_generation_prompt=False`.
3. During inference, `add_generation_prompt=True` may append the assistant-start marker when the template needs one.
4. If a template is rendered to text and tokenized later, avoid duplicating special tokens; official documentation
   recommends `add_special_tokens=False` in that path.
5. A template that renders correctly is not automatically compatible with assistant-only masking. Verify returned
   assistant masks.
6. Save and version the tokenizer, template, and model together. A changed template is a changed deployment artifact.

**FIG-CH06-02 — Template parity contract (`ILLUSTRATIVE`)** Alt text: the same versioned chat template and tokenizer
contract is applied during dataset construction, training, evaluation, export, and inference.

```text
training record ─┐
                 ├─ same tokenizer revision
evaluation chat ─┼─ same chat_template.jinja
                 ├─ same role/tool conventions
deployment app ──┘

Any mismatch → shifted control tokens → degraded or malformed behavior
```

Template testing should assert:

- deterministic rendering;
- expected beginning/end control tokens;
- assistant span positions;
- no duplicated BOS/EOS tokens;
- prefix preservation when tool workflows require it;
- truncation does not delete the target while preserving only the prompt; and
- training and inference render the same history prefix.

<a id="ch06-data"></a>

## 6.5 Dataset contracts: actual JSON and JSONL

JSON is a syntax; it does not prescribe an SFT schema. JSON Lines (JSONL) stores one complete JSON object per line.
Hugging Face Datasets documentation recommends line-delimited objects as an efficient JSON layout and loads them through
`load_dataset("json", data_files=...)`. `DOC-VERIFIED`
[Datasets loading documentation](https://huggingface.co/docs/datasets/loading).

### 6.5.1 Single-turn QA in JSON

**Listing 6.1 — Canonical QA array**  
**Extraction path:** `chapter-assets/ch06/data/qa.examples.json`  
**Status:** `UNVERIFIED IMPLEMENTATION`

<!-- markdownlint-disable MD013 -->
```json
[
  {
    "id": "auto-qa-0001",
    "domain": "automotive",
    "task": "question_answering",
    "question": "What does a flashing malfunction indicator lamp generally require?",
    "answer": "Treat it as urgent. Reduce load, avoid continued driving if the engine is misfiring, and follow the vehicle manufacturer's diagnostic and safety procedure.",
    "source_ids": ["auto-source-001"],
    "review_status": "pending_expert_review",
    "risk": "safety_relevant"
  },
  {
    "id": "bee-qa-0001",
    "domain": "beekeeping",
    "task": "question_answering",
    "question": "Why is local context important when assessing colony stores?",
    "answer": "Required stores vary with climate, season length, colony size, forage availability, and local management practice; a universal quantity is unsafe to assume.",
    "source_ids": ["bee-source-001"],
    "review_status": "pending_expert_review",
    "risk": "moderate"
  }
]
```
<!-- markdownlint-restore -->

These records still need transformation into a model-facing prompt or `messages` structure. Provenance fields should
remain available to validation and audit code even if they are removed before tokenization.

### 6.5.2 Conversational SFT in JSONL

**Listing 6.2 — Multi-domain messages records**  
**Extraction path:** `chapter-assets/ch06/data/train.messages.jsonl`  
**Status:** `UNVERIFIED IMPLEMENTATION`

<!-- markdownlint-disable MD013 -->
```jsonl
{"id":"auto-chat-0001","domain":"automotive","task":"diagnostic_dialogue","messages":[{"role":"system","content":"You are a cautious automotive diagnostic assistant. Ask for evidence before recommending component replacement."},{"role":"user","content":"The starter turns the engine, but it does not start."},{"role":"assistant","content":"Confirm the basics before replacing parts: does engine speed sound normal, is battery voltage stable while cranking, are there diagnostic trouble codes, and is spark or injection command present?"},{"role":"user","content":"There is no RPM reading on the scan tool while cranking."},{"role":"assistant","content":"That observation makes the engine-speed signal path a priority, but it does not prove one failed part. Inspect relevant codes, wiring, connectors, sensor power/ground where applicable, and the service procedure for the exact vehicle. Disable the vehicle safely before intrusive tests."}],"source_ids":["auto-source-002"],"review_status":"pending_expert_review","risk":"safety_relevant"}
{"id":"pharm-chat-0001","domain":"pharmacy","task":"safe_information","messages":[{"role":"system","content":"Provide educational medication information only. Do not prescribe or change treatment. Escalate urgent or patient-specific decisions to a qualified clinician."},{"role":"user","content":"Can I double my dose because I missed yesterday's dose?"},{"role":"assistant","content":"I cannot determine a safe patient-specific dose from this information. Do not double it unless the medicine's authoritative instructions or your pharmacist explicitly say to do so. Check the dispensing label and contact a pharmacist or prescriber; seek urgent help if symptoms suggest an overdose or serious reaction."}],"source_ids":["pharm-source-001"],"review_status":"pending_clinical_review","risk":"high"}
```
<!-- markdownlint-restore -->

The examples are deliberately conservative and are not medical or repair instructions. The sample source IDs identify
provenance records that a real dataset must resolve to legally usable material and qualified review decisions.

### 6.5.3 Prompt-completion JSONL

**Listing 6.3 — Structured extraction records**  
**Extraction path:** `chapter-assets/ch06/data/train.prompt-completion.jsonl`  
**Status:** `UNVERIFIED IMPLEMENTATION`

<!-- markdownlint-disable MD013 -->
```jsonl
{"id":"industrial-extract-0001","domain":"industrial_repair","task":"structured_extraction","prompt":"Extract the observed state. Return only JSON matching {asset_id:string, symptom:string, measured_temperature_c:number|null}.\nReport: Pump P-17 is vibrating; casing temperature was not measured.","completion":"{\"asset_id\":\"P-17\",\"symptom\":\"vibration\",\"measured_temperature_c\":null}","schema_version":"diagnostic-observation-1.0"}
{"id":"auto-class-0001","domain":"automotive","task":"triage_classification","prompt":"Classify urgency as ROUTINE, SOON, or STOP_SAFELY. Observation: brake pedal suddenly travels much farther than normal.","completion":"STOP_SAFELY","label_policy_version":"auto-triage-1.0"}
```
<!-- markdownlint-restore -->

Completion-only loss is natural here: the input wording is context, while the serialized target is the desired
prediction.

### 6.5.4 Multi-turn diagnostic state

A production dataset often needs state beyond message text. Keep the auditable state in metadata and render only the
intended fields:

**Listing 6.4 — Diagnostic case with hidden evaluator state**  
**Extraction path:** `chapter-assets/ch06/data/diagnostic-case.example.json`  
**Status:** `UNVERIFIED IMPLEMENTATION`

<!-- markdownlint-disable MD013 -->
```json
{
  "id": "auto-dx-0042",
  "domain": "automotive",
  "task": "multi_turn_diagnostic_reasoning",
  "observable_case": {
    "vehicle_context": "Exact vehicle details intentionally omitted in this educational example.",
    "initial_complaint": "Cranks but does not start",
    "available_observations": {
      "scan_rpm_while_cranking": 0,
      "battery_voltage_v": 12.4
    }
  },
  "messages": [
    { "role": "user", "content": "It cranks but does not start." },
    {
      "role": "assistant",
      "content": "What scan data and codes are available while cranking? In particular, report engine speed and battery voltage."
    },
    { "role": "user", "content": "Engine speed stays at 0 RPM; static battery voltage is 12.4 V." },
    {
      "role": "assistant",
      "content": "Prioritize verification of the engine-speed signal circuit and applicable power, ground, connector, and code checks. Zero scan RPM narrows the investigation but is not by itself proof that a specific sensor is defective."
    }
  ],
  "hidden_evaluation": {
    "acceptable_questions": ["scan RPM", "codes", "cranking voltage", "spark/injection evidence"],
    "premature_replacement_is_failure": true,
    "required_safety_elements": ["secure vehicle", "use model-specific service procedure"]
  },
  "source_ids": ["auto-source-003"],
  "review_status": "pending_expert_review"
}
```
<!-- markdownlint-restore -->

Do not render `hidden_evaluation` into the training prompt if it would leak the answer or evaluation rubric.

<a id="ch06-tasks"></a>

## 6.6 QA, task, terminology, and answer-format training

### 6.6.1 Question-answer pairs

QA is useful when users will ask questions, but one canonical question per fact is brittle. Vary:

- direct, indirect, and contrastive phrasing;
- common names, formal terms, abbreviations, and spelling variants;
- questions where context is insufficient;
- near-neighbor concepts that must not be conflated;
- temporal, regional, model-specific, or patient-specific qualifiers; and
- negative examples where the correct behavior is to request more information.

Split paraphrases and records derived from the same source passage as a group. Random row splitting can place
near-duplicates in train and test, producing misleading scores.

### 6.6.2 Task-specific data

Specialist SLMs may need:

- classification: urgency, fault family, document type;
- extraction: component, unit, code, symptom, contraindication;
- transformation: normalize units or terminology;
- summarization: compress an observation log without inventing events;
- procedural planning: order verification steps subject to safety gates;
- explanation: adapt depth to technician, trainee, or public audience;
- refusal/escalation: identify out-of-scope or unsafe requests; and
- structured generation: output a versioned JSON object.

Task metadata is essential. Without it, aggregate loss can hide that a large summarization subset has overwhelmed rare
safety examples.

### 6.6.3 Specialized terminology

Terminology training should connect:

1. the canonical term;
2. accepted abbreviations and aliases;
3. context-dependent meanings;
4. units and dimensional constraints;
5. confusable terms; and
6. safe usage in complete answers.

Do not solve terminology merely by forcing glossary recitation. Test it in realistic questions and extraction tasks.
Tokenizer fragmentation is a separate issue: SFT can learn a fragmented term, but longer tokenizations consume context
and may make rare identifiers harder to reproduce.

### 6.6.4 Domain-specific answer formats

Examples of explicit contracts:

- **diagnostic:** `observations → hypotheses → next discriminating check → safety boundary`;
- **pharmacy education:** `general information → uncertainty/contraindication boundary → qualified escalation`;
- **maintenance:** `prerequisites → ordered steps → verification → stop conditions`;
- **classification:** exactly one label from an enumerated set;
- **structured extraction:** only a schema-valid JSON object.

Format compliance and factual correctness are different metrics. A perfectly valid JSON answer can be dangerously wrong.

<a id="ch06-multiturn"></a>

## 6.7 Multi-turn conversation and diagnostic reasoning

Multi-turn SFT teaches dependence on history. A useful case should expose information incrementally and make each
assistant action appropriate to the information available at that turn.

### 6.7.1 What a diagnostic dialogue should teach

At each turn, the assistant should:

1. summarize confirmed observations without upgrading hypotheses to facts;
2. identify the highest-value missing evidence;
3. ask a discriminating question or propose a safe test;
4. update hypotheses after the answer;
5. avoid repeating resolved questions;
6. state stop conditions and escalation points; and
7. avoid unsupported component replacement, dosing, or intervention.

**FIG-CH06-03 — Diagnostic policy (`ILLUSTRATIVE`)** Alt text: a diagnostic request moves through scope and hazard
checks, evidence-bounded reasoning, uncertainty handling, and escalation rather than autonomous authorization.

```text
complaint
   ▼
check immediate hazard ── hazard present ──> stop / isolate / escalate
   │ no
   ▼
collect context and observations
   ▼
maintain multiple hypotheses
   ▼
choose discriminating, safe next check
   ▼
update from result ── insufficient evidence ──> ask again
   │
   └─ sufficient evidence → bounded conclusion + verification
```

### 6.7.2 Avoiding fake reasoning supervision

Long explanations are not proof of valid reasoning. Synthetic “chain-of-thought” can contain confident invented links,
and hidden reasoning may be inappropriate to expose or imitate. Prefer auditable outputs:

- evidence used;
- alternatives considered;
- next test and why it discriminates;
- confidence category;
- stop/escalation criterion.

This trains a reviewable diagnostic trace without claiming access to faithful internal reasoning.

### 6.7.3 Conversation construction failures

- **Answer leakage:** a user turn contains the reference diagnosis.
- **Impossible hindsight:** an early assistant mentions evidence revealed only later.
- **Single-path cases:** every symptom maps to one component.
- **Noisy role order:** consecutive user turns are accidentally merged or dropped.
- **Truncation:** the final target survives without the initial complaint, or the target is entirely removed.
- **persona overfitting:** every record repeats a long system prompt that dominates tokens.
- **unsafe authority:** demonstrations frame the model as a final decision maker.

Evaluate full trajectories and turn-level actions, not only the last answer.

<a id="ch06-synthetic"></a>

## 6.8 Synthetic instruction generation

Self-Instruct generates instructions, inputs, and outputs from a language model, filters invalid or similar samples, and
fine-tunes on the retained data. Its primary paper reported improved instruction-following performance in its setting
and released code/data. `PAPER-REPORTED` [Wang et al., 2023](https://arxiv.org/abs/2212.10560).

For specialist SLMs, synthetic generation should be a controlled augmentation process:

```text
approved source facts / expert case skeletons
        ▼
task and difficulty specification
        ▼
teacher generates candidate instruction + answer
        ▼
deterministic schema/unit/terminology checks
        ▼
source-grounded factual review
        ▼
semantic deduplication and diversity checks
        ▼
risk-stratified expert sampling or full review
        ▼
versioned accepted dataset
```

Recommended generation axes:

- paraphrase without changing facts;
- novice versus expert vocabulary;
- easy, boundary, and adversarial cases;
- insufficient-information examples;
- counterfactuals with one controlled variable changed;
- multi-turn cases that reveal observations in different orders;
- safe refusals and false-refusal counterexamples;
- structured and prose versions of the same task.

Synthetic data risks include teacher hallucinations, style monoculture, correlated errors, leakage from evaluation
prompts, excessive verbosity, false confidence, and self-reinforcing model collapse. A teacher's fluent answer is not
provenance. Every factual candidate must remain linked to an approved source or expert-authored case, and high-risk
examples require qualified review.

### 6.8.1 Candidate generation schema

**Listing 6.5 — Synthetic candidate contract**  
**Extraction path:** `chapter-assets/ch06/data/synthetic-candidate.example.json`  
**Status:** `UNVERIFIED IMPLEMENTATION`

<!-- markdownlint-disable MD013 -->
```json
{
  "candidate_id": "syn-auto-000001",
  "generator": {
    "model_id": "RECORD_EXACT_MODEL_ID",
    "revision": "RECORD_IMMUTABLE_REVISION",
    "prompt_template_hash": "RECORD_SHA256",
    "decoding": { "temperature": null, "top_p": null, "seed": null }
  },
  "source_ids": ["approved_source_record_id"],
  "requested_task": "insufficient_information_diagnostic",
  "candidate_messages": [
    { "role": "user", "content": "A warning light came on. Which part should I replace?" },
    {
      "role": "assistant",
      "content": "The warning light alone does not identify a failed part. Record the exact light, symptoms, codes, operating conditions, and vehicle details before selecting tests or parts."
    }
  ],
  "automated_checks": {
    "schema_valid": true,
    "source_terms_checked": false,
    "near_duplicate_cluster": null
  },
  "human_review": {
    "status": "awaiting_review",
    "reviewer_role": "qualified_domain_reviewer",
    "factuality": null,
    "safety": null
  }
}
```
<!-- markdownlint-restore -->

Generator fields must contain actual values before acceptance; `null` is not a reproducibility record.

<a id="ch06-mixtures"></a>

## 6.9 Multi-domain mixtures and weighting

A single multi-domain model can share general instruction behavior and terminology patterns, but domains compete for
finite parameter capacity and updates. Row concatenation implicitly weights by row count; token-level loss further
weights long responses.

Let dataset \(D_k\) represent domain or task \(k\). One explicit mixture objective is:

\[ \mathcal{L}=\sum_{k=1}^{K}\alpha_k\, \mathbb{E}_{(x,y)\sim D_k}\left[\mathcal{L}_k(x,y)\right], \qquad
\sum_k\alpha_k=1. \]

\(\alpha_k\) is a sampling or objective weight, not necessarily the raw dataset proportion.

For raw counts \(n_k\), temperature sampling can use:

\[ q_k=\frac{n_k^\tau}{\sum_j n_j^\tau}. \]

- \(\tau=1\): proportional sampling;
- \(\tau=0\): equal probability per domain;
- \(0<\tau<1\): upweights smaller domains relative to proportional sampling.

If automotive has 80,000 records and pharmacy has 5,000, proportional sampling gives pharmacy \(5.9\%\). With
\(\tau=0.5\), the two-domain pharmacy share becomes:

\[ \frac{\sqrt{5000}}{\sqrt{80000}+\sqrt{5000}}\approx20\%. \]

This is an `ILLUSTRATIVE` calculation, not a recommended safety weighting. Risk, quality, difficulty, and token length
matter in addition to count.

### 6.9.1 Practical mixture hierarchy

Track at least:

- domain;
- task;
- risk tier;
- source quality;
- human versus synthetic origin;
- language;
- difficulty;
- answer length; and
- format type.

Use hierarchical sampling when needed: choose a domain, then a task, then a record. Preserve a small general
instruction/replay component only if retention evaluation justifies it. Do not duplicate rare safety records thousands
of times; repeated exact examples invite memorization. Prefer controlled variants and targeted batch sampling.

### 6.9.2 Weighting failure modes

- A long-answer domain dominates active tokens despite balanced row counts.
- Easy classification reduces average loss while diagnostic quality stagnates.
- Synthetic examples overwhelm expert examples.
- One domain's refusal pattern creates false refusals in another.
- Shared abbreviations acquire conflicting meanings.
- Rare high-risk cases are oversampled until memorized.
- Packing mixes domains but removes metadata needed for per-domain metrics.

Report both sampled rows and supervised tokens per stratum.

<a id="ch06-structured"></a>

## 6.10 Structured outputs

SFT can substantially improve conformance to a fixed output structure, but generation remains probabilistic unless the
runtime applies constrained decoding or validates and retries.

For JSON:

1. define a versioned JSON Schema;
2. choose whether extra fields are forbidden;
3. represent unknowns with `null`, not invented values;
4. train valid, invalid, boundary, and insufficient-information cases;
5. parse every validation prediction;
6. score schema validity and field correctness separately;
7. test escaped characters, Unicode, empty lists, long strings, and numeric boundaries; and
8. reject or safely handle invalid output at runtime.

**Listing 6.6 — Versioned diagnostic observation schema**  
**Extraction path:** `chapter-assets/ch06/schemas/diagnostic-observation.schema.json`  
**Status:** `UNVERIFIED IMPLEMENTATION`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.invalid/schemas/diagnostic-observation-1.0.json",
  "title": "DiagnosticObservation",
  "type": "object",
  "additionalProperties": false,
  "required": ["asset_id", "symptom", "measured_temperature_c"],
  "properties": {
    "asset_id": { "type": "string", "minLength": 1 },
    "symptom": { "type": "string", "minLength": 1 },
    "measured_temperature_c": { "type": ["number", "null"] }
  }
}
```

The `.invalid` hostname intentionally prevents this educational identifier from appearing to be a live authority.

<a id="ch06-tools"></a>

## 6.11 Optional tool-use training as a separate capability

Tool use is optional and should be evaluated separately from domain QA. It introduces a protocol:

1. decide whether a tool is needed;
2. select an allowed tool;
3. emit valid arguments;
4. wait for a tool result;
5. interpret the result without altering it;
6. answer or escalate.

Under this book's no-runtime-RAG constraint, permitted deterministic tools can include unit conversion, checksum
verification, a calculator, or a locally defined diagnostic decision aid. They must not quietly become document
retrieval or a remote teacher answering on the model's behalf.

**Listing 6.7 — Optional tool conversation**  
**Extraction path:** `chapter-assets/ch06/data/tool-use.example.json`  
**Status:** `UNVERIFIED IMPLEMENTATION`

```json
{
  "id": "tool-unit-0001",
  "task": "deterministic_tool_use",
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "convert_temperature",
        "description": "Convert a temperature between Celsius and Fahrenheit.",
        "parameters": {
          "type": "object",
          "additionalProperties": false,
          "required": ["value", "from_unit", "to_unit"],
          "properties": {
            "value": { "type": "number" },
            "from_unit": { "enum": ["C", "F"] },
            "to_unit": { "enum": ["C", "F"] }
          }
        }
      }
    }
  ],
  "messages": [
    { "role": "user", "content": "Convert 95 C to Fahrenheit." },
    {
      "role": "assistant",
      "tool_calls": [
        {
          "type": "function",
          "function": {
            "name": "convert_temperature",
            "arguments": "{\"value\":95,\"from_unit\":\"C\",\"to_unit\":\"F\"}"
          }
        }
      ]
    },
    { "role": "tool", "name": "convert_temperature", "content": "{\"value\":203,\"unit\":\"F\"}" },
    { "role": "assistant", "content": "95 °C is 203 °F." }
  ]
}
```

This schema is illustrative. Tool-call content structures vary by model template and library version. Current TRL
chat-template documentation also notes prefix-preservation requirements for some tool-training workflows. `DOC-VERIFIED`
[TRL chat-template documentation](https://huggingface.co/docs/trl/chat_templates). Verify the exact checkpoint template;
do not assume this generic record is accepted unchanged.

Tool evaluation must include unnecessary-call rate, invalid arguments, unauthorized tool selection, result fidelity,
injection-like text in tool output, timeout/error handling, and final-answer correctness.

<a id="ch06-safety"></a>

## 6.12 Safety, uncertainty, and hallucination limits

### 6.12.1 What SFT can and cannot do

SFT can teach visible behaviors such as:

- ask for missing context;
- distinguish observation from hypothesis;
- say “I do not have enough information”;
- refuse a prohibited action;
- state a confidence category;
- cite the type of authority that must verify a decision;
- stop a procedure when a hazard is detected.

It does not guarantee truthful internal uncertainty. A model can learn the phrase “I am not certain” as style while
remaining miscalibrated. Calibration requires held-out measurements such as selective accuracy versus coverage, Brier
score for explicit probabilities where appropriate, and risk-weighted error analysis.

### 6.12.2 Hallucination reduction

Use a portfolio rather than a promise:

- source-verified targets;
- contradictions removed or explicitly contextualized;
- examples where evidence is insufficient;
- answers that separate known, inferred, and unknown statements;
- negative examples for plausible but wrong near-neighbors;
- concise targets where unsupported elaboration adds risk;
- decoding controls evaluated on the target task;
- post-generation schema and safety validation;
- human review and release gates for high-risk uses.

SFT may reduce hallucinations on the trained distribution, but factual fine-tuning can also create regressions. Gekhman
et al. reported in controlled closed-book QA that examples introducing knowledge not already known by the model were
learned more slowly and, as learned, increased hallucination tendency. `PAPER-REPORTED`
[Gekhman et al., 2024](https://arxiv.org/abs/2405.05904). Ghosh et al. reported limitations including
style/response-pattern learning and hallucination or knowledge degradation in studied instruction-tuning conditions.
`PAPER-REPORTED` [Ghosh et al., ICML 2024](https://proceedings.mlr.press/v235/ghosh24a.html). These papers do not prove
identical behavior for every model and dataset; they require cautious, model-specific testing.

### 6.12.3 Safety and uncertainty curriculum

Include all four quadrants:

| Evidence state             | Safe answer behavior                                            |
| -------------------------- | --------------------------------------------------------------- |
| Sufficient and low risk    | Answer directly within scope                                    |
| Sufficient but high risk   | Provide bounded information and required qualified verification |
| Insufficient and low risk  | Ask targeted questions or state assumptions                     |
| Insufficient and high risk | Do not guess; refuse unsafe specificity and escalate            |

Also train **false-refusal counterexamples**. A model that refuses harmless educational questions is not well aligned.

For pharmacy, the model must not autonomously prescribe, change doses, or replace clinician/pharmacist judgment. For
safety-critical repair, it must not authorize hazardous operation or invent model-specific procedures. These are
intended-use limits, not merely preferred wording.

<a id="ch06-overfitting"></a>

## 6.13 Overfitting, memorization, and leakage

Overfitting means training behavior improves while held-out generalization worsens. Memorization is exact or near-exact
retention and emission of training content; it can occur with or without ordinary validation-loss overfitting.

Warning signs:

- training loss falls while validation loss rises;
- exact train questions succeed but paraphrases fail;
- long target spans are reproduced verbatim;
- fabricated answers borrow names, numbers, or phrases from nearby training records;
- domain gains coincide with general-capability or safety regression;
- canary strings or sensitive text are emitted;
- one answer format appears regardless of user need.

Controls:

1. deduplicate before splitting, including semantic near-duplicates;
2. group by source document, case family, and synthetic seed;
3. remove PII, secrets, copyrighted text without suitable rights, and unsafe proprietary records;
4. use a validation set spanning paraphrases, compositional cases, rare cases, and out-of-distribution prompts;
5. evaluate multiple checkpoints rather than selecting the final training step by train loss;
6. reduce epochs, learning rate, adapter capacity, or repeated sampling when warranted;
7. retain and test a general/safety replay set where justified;
8. run extraction probes and controlled canaries; and
9. document all contamination findings.

Never put real secrets or harmful personal data into memorization canaries. Use synthetic unique strings.

<a id="ch06-knowledge"></a>

## 6.14 Why SFT is not comprehensive knowledge acquisition

SFT is usually a sparse, targeted behavioral learning stage. Comprehensive domain acquisition would require broad
coverage, repeated contextual exposure, conflict resolution, temporal/version metadata, robust composition, and reliable
recall. A few QA pairs provide none of those guarantees.

Consider a 500-page repair corpus transformed into 2,000 questions:

- conversion selects only some facts;
- question wording constrains retrieval cues;
- rare tables, exceptions, and interactions may disappear;
- the model may learn a familiar response pattern rather than the underlying relation;
- repeated epochs may memorize answers without compositional generalization;
- later SFT can interfere with existing knowledge;
- facts can become stale, and no-runtime-RAG means they cannot be refreshed per request.

SFT is strongest for **how to respond and perform defined tasks using capabilities the model can support**. Continued
pretraining is often a more natural experiment for broad unlabeled domain text, followed by SFT for interaction
behavior. Neither guarantees exact storage. The correct empirical comparison is:

1. untouched base/instruction model;
2. SFT only;
3. continued pretraining only;
4. continued pretraining plus identical SFT;
5. optional preference/post-training stage;
6. all evaluated on held-out factuality, reasoning, behavior, safety, and retention.

The no-runtime-RAG requirement makes this limitation operationally important. The deployed model must expose
uncertainty, avoid claims of completeness, and have an update/retraining process. High-risk outputs still require
authoritative external verification by a qualified human; this evaluation-time or workflow verification is not runtime
document retrieval by the model.

<a id="ch06-implementation"></a>

## 6.15 Practical dataset and training implementation

The following listings target the documented `messages` and prompt-completion concepts in Hugging Face Datasets/TRL.
APIs are time-sensitive. The documentation was reviewed at the research freeze, but no environment was created and no
command was executed.

### 6.15.1 Environment contract

**Listing 6.8 — Dependency specification template**  
**Extraction path:** `chapter-assets/ch06/requirements-sft.in`  
**Status:** `UNVERIFIED IMPLEMENTATION`

```text
torch
transformers
datasets
trl
accelerate
jsonschema
```

Exact versions are intentionally not fabricated. Before execution:

1. resolve a compatible set in an isolated environment;
2. record `python --version` and the complete lock file;
3. confirm the selected TRL version documents `assistant_only_loss`;
4. confirm the selected Transformers version and tokenizer return the required assistant mask;
5. record CUDA/driver and GPU details if used; and
6. store immutable model/dataset revisions and artifact hashes.

### 6.15.2 Dataset validation

**Listing 6.9 — Validate messages JSONL before training**  
**Extraction path:** `chapter-assets/ch06/scripts/validate_messages.py`  
**Status:** `UNVERIFIED IMPLEMENTATION`

```python
from __future__ import annotations

import argparse
import json
from collections import Counter
from pathlib import Path

ALLOWED_ROLES = {"system", "user", "assistant", "tool"}


def validate_record(record: dict, line_number: int) -> list[str]:
    errors: list[str] = []
    required = {"id", "domain", "task", "messages"}
    missing = required - record.keys()
    if missing:
        errors.append(f"line {line_number}: missing keys {sorted(missing)}")
        return errors

    messages = record["messages"]
    if not isinstance(messages, list) or not messages:
        return [f"line {line_number}: messages must be a non-empty list"]

    assistant_count = 0
    for index, message in enumerate(messages):
        if not isinstance(message, dict):
            errors.append(f"line {line_number}: message {index} is not an object")
            continue
        role = message.get("role")
        if role not in ALLOWED_ROLES:
            errors.append(f"line {line_number}: invalid role {role!r}")
        if role == "assistant":
            assistant_count += 1
        if role != "assistant" or "tool_calls" not in message:
            content = message.get("content")
            if not isinstance(content, str) or not content.strip():
                errors.append(
                    f"line {line_number}: message {index} needs non-empty content"
                )

    if assistant_count == 0:
        errors.append(f"line {line_number}: no assistant target")
    return errors


def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument("path", type=Path)
    args = parser.parse_args()

    ids: set[str] = set()
    strata: Counter[tuple[str, str]] = Counter()
    errors: list[str] = []

    with args.path.open("r", encoding="utf-8") as source:
        for line_number, line in enumerate(source, start=1):
            try:
                record = json.loads(line)
            except json.JSONDecodeError as exc:
                errors.append(f"line {line_number}: invalid JSON: {exc}")
                continue
            errors.extend(validate_record(record, line_number))
            record_id = record.get("id")
            if record_id in ids:
                errors.append(f"line {line_number}: duplicate id {record_id!r}")
            ids.add(record_id)
            strata[(record.get("domain", ""), record.get("task", ""))] += 1

    for key, count in sorted(strata.items()):
        print(f"{key[0]}\t{key[1]}\t{count}")
    if errors:
        raise SystemExit("\n".join(errors))
    print(f"valid records: {len(ids)}")


if __name__ == "__main__":
    main()
```

Command:

```powershell
python chapter-assets/ch06/scripts/validate_messages.py `
  chapter-assets/ch06/data/train.messages.jsonl
```

Expected output is per-domain/task counts and a valid record count. Expected failures include malformed JSON, duplicate
IDs, invalid roles, empty targets, and unsupported tool structures. This validator does not prove factuality, licensing,
split independence, safety, or template compatibility.

### 6.15.3 Inspect rendered tokens and masks

Mask verification must precede a full training run.

**Listing 6.10 — Template and assistant-mask inspection**  
**Extraction path:** `chapter-assets/ch06/scripts/inspect_chat_mask.py`  
**Status:** `UNVERIFIED IMPLEMENTATION`

```python
from __future__ import annotations

import argparse
import json

from transformers import AutoTokenizer


def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument("--model", required=True)
    parser.add_argument("--revision", required=True)
    parser.add_argument("--json-record", required=True)
    args = parser.parse_args()

    tokenizer = AutoTokenizer.from_pretrained(
        args.model,
        revision=args.revision,
        trust_remote_code=False,
    )
    with open(args.json_record, "r", encoding="utf-8") as source:
        messages = json.load(source)["messages"]

    encoded = tokenizer.apply_chat_template(
        messages,
        tokenize=True,
        add_generation_prompt=False,
        return_dict=True,
        return_assistant_tokens_mask=True,
    )
    input_ids = encoded["input_ids"]
    mask = encoded.get("assistant_masks")
    if mask is None:
        raise RuntimeError(
            "Tokenizer/template did not return assistant_masks; "
            "do not use assistant-only loss until compatibility is resolved."
        )
    if len(input_ids) != len(mask) or not any(mask):
        raise RuntimeError("Invalid or empty assistant mask")

    for token_id, is_assistant in zip(input_ids, mask):
        piece = tokenizer.decode([token_id], skip_special_tokens=False)
        print(f"{int(bool(is_assistant))}\t{token_id}\t{piece!r}")


if __name__ == "__main__":
    main()
```

The exact returned key and availability are version/template dependent. The current Transformers documentation describes
`return_assistant_tokens_mask`, while TRL requires generation markers for assistant-only loss. `DOC-VERIFIED` Treat any
mismatch as a blocker, not as permission to create an approximate string-search mask.

### 6.15.4 TRL SFT training

**Listing 6.11 — Minimal messages SFT**  
**Extraction path:** `chapter-assets/ch06/scripts/train_sft.py`  
**Status:** `UNVERIFIED IMPLEMENTATION`

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import torch
from datasets import load_dataset
from transformers import AutoTokenizer, set_seed
from trl import SFTConfig, SFTTrainer


def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument("--model", required=True)
    parser.add_argument("--revision", required=True)
    parser.add_argument("--train-file", type=Path, required=True)
    parser.add_argument("--validation-file", type=Path, required=True)
    parser.add_argument("--output-dir", type=Path, required=True)
    parser.add_argument("--seed", type=int, default=1729)
    args = parser.parse_args()

    set_seed(args.seed)
    dataset = load_dataset(
        "json",
        data_files={
            "train": str(args.train_file),
            "validation": str(args.validation_file),
        },
    )
    tokenizer = AutoTokenizer.from_pretrained(
        args.model,
        revision=args.revision,
        trust_remote_code=False,
    )
    if tokenizer.chat_template is None:
        raise RuntimeError("Selected tokenizer has no chat template")

    config = SFTConfig(
        output_dir=str(args.output_dir),
        model_init_kwargs={
            "revision": args.revision,
            "trust_remote_code": False,
        },
        assistant_only_loss=True,
        max_length=2048,
        learning_rate=2e-5,
        num_train_epochs=1.0,
        per_device_train_batch_size=1,
        per_device_eval_batch_size=1,
        gradient_accumulation_steps=16,
        eval_strategy="steps",
        eval_steps=100,
        save_steps=100,
        logging_steps=10,
        bf16=torch.cuda.is_available()
        and torch.cuda.is_bf16_supported(),
        fp16=torch.cuda.is_available()
        and not torch.cuda.is_bf16_supported(),
        report_to="none",
        seed=args.seed,
        data_seed=args.seed,
    )
    trainer = SFTTrainer(
        model=args.model,
        args=config,
        train_dataset=dataset["train"],
        eval_dataset=dataset["validation"],
        processing_class=tokenizer,
    )
    trainer.train()
    trainer.save_model()
    tokenizer.save_pretrained(args.output_dir)

    run_record = {
        "model": args.model,
        "revision": args.revision,
        "seed": args.seed,
        "train_file": str(args.train_file),
        "validation_file": str(args.validation_file),
        "status": "training_completed"
    }
    (args.output_dir / "run_record.json").write_text(
        json.dumps(run_record, indent=2), encoding="utf-8"
    )


if __name__ == "__main__":
    main()
```

This is a teaching baseline, not a recommended universal configuration. The learning rate, sequence length, precision,
batch construction, and epoch count are hypotheses. The listing performs full-parameter training unless the chosen
library/model setup is extended with a verified PEFT configuration. Full fine-tuning can exceed available VRAM.

Example command:

```powershell
accelerate launch chapter-assets/ch06/scripts/train_sft.py `
  --model EXACT_MODEL_REPOSITORY `
  --revision IMMUTABLE_COMMIT `
  --train-file data/train.messages.jsonl `
  --validation-file data/validation.messages.jsonl `
  --output-dir outputs/ch06-sft-run
```

Before execution, confirm the current `SFTConfig` parameter names against the installed TRL version. Current official
TRL documentation supports standard/conversational language-model and prompt-completion dataset formats, automatic
chat-template application for conversational data, packing, completion-only loss, and assistant-only loss under
documented constraints. `DOC-VERIFIED` [TRL SFTTrainer documentation](https://huggingface.co/docs/trl/sft_trainer).

### 6.15.5 Resource requirements

Training memory includes weights, gradients, optimizer states, activations, temporary buffers, and framework overhead.
It is not estimated reliably from checkpoint file size alone. Full fine-tuning usually costs substantially more memory
than adapter training; Chapter 8 and Chapter 9 treat PEFT/LoRA choices.

Measure rather than assume:

- peak accelerator memory;
- examples and active target tokens per second;
- padding fraction;
- packed-sequence utilization;
- checkpoint size and save time;
- validation latency; and
- total energy/cost under a declared protocol.

Packing can reduce padding waste, but boundaries must remain correct. Do not allow labels from one record to supervise
prompt tokens in the next, and do not infer one example's conversation history from another.

<a id="ch06-validation"></a>

## 6.16 Validation and troubleshooting

### 6.16.1 Preflight gates

Before training:

- [ ] every record parses and satisfies one declared schema;
- [ ] licenses, provenance, and review statuses are present;
- [ ] source/case/synthetic-family groups do not cross splits;
- [ ] exact and near-duplicates are reported;
- [ ] domain/task/risk/synthetic distributions are reported by rows and target tokens;
- [ ] the exact template is saved;
- [ ] rendered conversations are manually inspected;
- [ ] assistant or completion masks are inspected at token level;
- [ ] truncation and packing tests pass;
- [ ] high-risk examples have qualified review;
- [ ] evaluation prompts were not used for synthetic generation;
- [ ] model, tokenizer, dataset, code, and environment revisions are immutable.

### 6.16.2 Common failures

<!-- markdownlint-disable MD013 -->
| Symptom                           | Likely cause                                             | Diagnostic action                                   | Corrective direction                                |
| --------------------------------- | -------------------------------------------------------- | --------------------------------------------------- | --------------------------------------------------- |
| Model emits role markers          | Template mismatch or control tokens learned as content   | Decode rendered train samples                       | Restore checkpoint template and token policy        |
| Loss is zero/NaN                  | Empty mask, all targets truncated, precision instability | Count active labels and inspect first failing batch | Fix masking/truncation; test safer precision        |
| Model continues user text         | Missing inference generation prompt or wrong template    | Compare train and inference rendering               | Apply exact tokenizer template                      |
| JSON is almost valid              | Mixed prose/JSON targets or unconstrained decoding       | Parse validation predictions by error type          | Clean targets; add schema tests/runtime constraints |
| One domain regresses              | Mixture/token imbalance or interference                  | Plot per-domain curves and target-token exposure    | Adjust sampling; consider separate adapters         |
| Refuses harmless prompts          | Safety examples lack counterexamples                     | Build refusal confusion matrix                      | Add safe-answer negatives and clarify policy        |
| Confident guesses persist         | Dataset rewards answering every question                 | Evaluate insufficient-information set               | Add abstention/question-asking targets              |
| Validation looks excellent        | Duplicate/source leakage                                 | Group audit and nearest-neighbor search             | Rebuild splits                                      |
| Multi-turn answers ignore history | Cases are answer-only or histories truncated             | Evaluate turn permutations and omitted facts        | Add genuine state-dependent dialogues               |
| Assistant-only option fails       | Template lacks generation markers or family patch        | Inspect current TRL support and masks               | Use a verified training template or block run       |
<!-- markdownlint-restore -->

### 6.16.3 Evaluation dimensions

Do not select checkpoints by validation loss alone. Measure:

- domain factual accuracy with authoritative ground truth;
- exact match/F1 where appropriate;
- schema validity and field-level correctness;
- terminology and unit accuracy;
- diagnostic next-question utility;
- multi-turn consistency and correction after new evidence;
- hallucinated/unsupported claim rate;
- refusal precision and recall;
- selective accuracy versus coverage;
- memorization/extraction probes;
- general-capability retention;
- per-domain and worst-group performance; and
- edge artifact parity after later conversion/quantization.

<a id="ch06-experiments"></a>

## 6.17 Experimental design

### 6.17.1 Core ablation matrix

**TBL-CH06-01 — SFT experiment plan (`PLANNED EXPERIMENT`)**

| Arm | Dataset                           | Loss            | Purpose                                     |
| --- | --------------------------------- | --------------- | ------------------------------------------- |
| A   | No SFT                            | —               | Untouched baseline                          |
| B   | Single-turn QA                    | Full sequence   | Quantify naïve baseline                     |
| C   | Same QA                           | Completion only | Isolate prompt masking                      |
| D   | Messages                          | Assistant only  | Test chat behavior                          |
| E   | D + multi-turn diagnostic         | Assistant only  | Test evidence-gathering                     |
| F   | E + uncertainty/safety            | Assistant only  | Test abstention/refusal                     |
| G   | F + synthetic accepted data       | Assistant only  | Test controlled augmentation                |
| H   | G + balanced multi-domain mixture | Assistant only  | Test interference/transfer                  |
| I   | CPT checkpoint + same F           | Assistant only  | Separate broad adaptation from behavior SFT |

Keep fixed:

- exact base checkpoint and tokenizer revision;
- template;
- optimizer and scheduler;
- effective supervised-token budget where possible;
- sequence length and packing policy;
- decoding settings;
- evaluation set;
- hardware/software environment; and
- random-seed schedule.

Run multiple seeds for decision-critical comparisons. Report paired differences and uncertainty, not only a single best
run.

<a id="ch06-chapter-status-and-evidence-boundary"></a>

### 6.17.2 Hypotheses and decision criteria

1. **Mask hypothesis:** completion/assistant-only loss improves response behavior per supervised target token compared
   with full-sequence chat loss.
2. **Dialogue hypothesis:** genuine multi-turn cases improve next-question quality and history use, not merely
   final-answer style.
3. **Safety hypothesis:** balanced uncertainty examples improve correct abstention without unacceptable false refusals.
4. **Mixture hypothesis:** temperature/hierarchical sampling improves worst-domain performance relative to proportional
   concatenation.
5. **Knowledge hypothesis:** CPT plus SFT improves broad held-out domain factuality more than SFT alone under a
   controlled budget.
6. **Synthetic hypothesis:** reviewed synthetic augmentation improves rare-task coverage; unreviewed augmentation
   increases unsupported claims.

Predeclare thresholds by application risk. A mean improvement cannot excuse a regression on critical contraindication
recognition or hazardous repair advice.

<a id="ch06-tradeoffs"></a>

## 6.18 Strengths, weaknesses, alternatives, and deployment implications

### 6.18.1 Strengths

- Directly optimizes observable target behavior.
- Supports natural language, labels, JSON, and multi-turn protocols.
- Can be efficient with parameter-efficient fine-tuning.
- Makes domain terminology and answer contracts explicit.
- Provides a clear path for human and expert demonstrations.

### 6.18.2 Weaknesses

- Demonstrations are expensive to author and validate.
- Coverage gaps become behavior gaps.
- Low loss can reflect imitation or leakage rather than reliable knowledge.
- Repeated/narrow data can cause memorization and forgetting.
- Safety phrases do not guarantee calibration.
- Multi-domain gradients can interfere.
- Template and mask bugs can silently train the wrong tokens.

### 6.18.3 Alternatives and complements

- **Prompting:** no weight update; fastest baseline, but consumes context and may be less reliable.
- **Continued pretraining/DAPT:** better matched to broad raw domain text; does not itself teach chat policy.
- **Preference optimization:** ranks desirable responses after SFT; covered in Chapter 7.
- **Distillation:** transfers teacher outputs or distributions; quality depends on teacher/data and is treated later.
- **Separate adapters/models:** isolate domains and updates at storage/operations cost.
- **Deterministic code:** preferable for exact calculations, validation, and safety interlocks.

### 6.18.4 Domain applications

- **Automotive/industrial:** ordered diagnosis, measurement interpretation, stop conditions, and model-specific
  uncertainty.
- **Pharmacy:** educational terminology, interaction-risk recognition, uncertainty, authoritative escalation, and strict
  non-prescribing boundaries.
- **Beekeeping/animal care:** regional/seasonal qualifiers, observation-based triage, and expert/veterinary escalation.
- **Cross-domain:** explicit domain metadata, conflicting-term tests, and refusal-boundary isolation.

### 6.18.5 Edge implications

SFT does not reduce parameter count by itself. Full fine-tuning preserves architecture and nominal weight count; LoRA
adds adapter weights unless merged. Edge consequences are mainly behavioral and artifact-management related:

- shorter learned answers can reduce generated tokens and energy, but must be measured;
- structured outputs may reduce post-processing ambiguity;
- longer system prompts increase prefill cost, so stable behavior learned during SFT can sometimes permit shorter
  prompts, subject to safety testing;
- a saved chat template/tokenizer must ship with the edge artifact;
- separate domain adapters can reduce duplicate base storage but increase switching and compatibility complexity;
- quantization after SFT can alter format, refusal, and factual behavior, requiring parity evaluation.

No mobile latency, memory, battery, or thermal improvement is claimed in this chapter.

<a id="ch06-sources"></a>

## 6.19 Research references and official documentation

1. **Wei, J. et al. (2022). “Finetuned Language Models Are Zero-Shot Learners.” ICLR 2022; arXiv:2109.01652.**
   Introduced FLAN-style instruction tuning across more than 60 datasets and reported zero-shot gains on held-out tasks.
   Limitation for this project: very large model scale and task benchmarks do not establish specialist factual
   acquisition in SLMs. [Primary paper](https://arxiv.org/abs/2109.01652). `PAPER-REPORTED`
2. **Sanh, V. et al. (2022). “Multitask Prompted Training Enables Zero-Shot Task Generalization.” ICLR 2022;
   arXiv:2110.08207.** T0 used multitask prompted training and PromptSource templates. Relevant to task/template
   diversity; not proof of safe domain diagnosis. [Primary paper](https://arxiv.org/abs/2110.08207). `PAPER-REPORTED`
3. **Ouyang, L. et al. (2022). “Training language models to follow instructions with human feedback.” NeurIPS 2022;
   arXiv:2203.02155.** Used labeler demonstrations for SFT before reward modeling and reinforcement learning. Shows SFT
   as one post-training stage, not the entire alignment process. [Primary paper](https://arxiv.org/abs/2203.02155).
   `PAPER-REPORTED`
4. **Wang, Y. et al. (2023). “Self-Instruct: Aligning Language Models with Self-Generated Instructions.” ACL 2023;
   arXiv:2212.10560.** Generated and filtered synthetic instructions, inputs, and outputs. Relevant to bootstrapping;
   specialist factual candidates still require provenance and review.
   [ACL record](https://aclanthology.org/2023.acl-long.754/) and [arXiv](https://arxiv.org/abs/2212.10560).
   `PAPER-REPORTED`
5. **Ghosh, S. et al. (2024). “A Closer Look at the Limitations of Instruction Tuning.” ICML 2024, PMLR 235.** Reported
   that studied instruction-tuning setups primarily changed response patterns and could degrade knowledge or increase
   hallucination. Applicability must be tested per model and dataset.
   [PMLR primary publication](https://proceedings.mlr.press/v235/ghosh24a.html). `PAPER-REPORTED`
6. **Gekhman, Z. et al. (2024). “Does Fine-Tuning LLMs on New Knowledge Encourage Hallucinations?” arXiv:2405.05904.**
   Controlled factual QA experiments reported slower learning of unknown facts and increasing hallucination tendency as
   such facts were fitted. Preprint status and experimental scope limit generalization.
   [Primary preprint](https://arxiv.org/abs/2405.05904). `PAPER-REPORTED`
7. **Hugging Face Transformers, “Chat templates.”** Defines model-specific role formatting, `apply_chat_template`,
   training/inference generation-prompt guidance, and special-token cautions.
   [Official documentation](https://huggingface.co/docs/transformers/chat_templating). `DOC-VERIFIED`
8. **Hugging Face Transformers, “Writing a chat template.”** Documents tokenizer-associated Jinja templates and
   persistence in `chat_template.jinja`.
   [Official documentation](https://huggingface.co/docs/transformers/chat_templating_writing). `DOC-VERIFIED`
9. **Hugging Face Datasets, “Load.”** Documents local JSON/JSONL loading and line-delimited object structure.
   [Official documentation](https://huggingface.co/docs/datasets/loading). `DOC-VERIFIED`
10. **Hugging Face TRL, “SFT Trainer.”** Documents supported standard/conversational and prompt-completion formats,
    assistant-only and completion-only loss, packing, and configuration fields.
    [Official documentation](https://huggingface.co/docs/trl/sft_trainer). `DOC-VERIFIED`
11. **Hugging Face TRL, “Chat Templates.”** Documents training-template patching, generation markers for assistant
    masks, and prefix preservation for relevant tool workflows.
    [Official documentation](https://huggingface.co/docs/trl/chat_templates). `DOC-VERIFIED`

Documentation links without a version segment can change. A reproduced run must archive the resolved package versions
and documentation date.

<a id="ch06-summary"></a>

## 6.20 Summary

SFT trains a causal model to imitate desired outputs under a token-level supervision mask. Instruction tuning broadens
tasks; chat tuning introduces role-formatted sequences; task and diagnostic tuning specialize outputs and interaction
policies. The chat template determines the actual token sequence, while completion-only and assistant-only masks
determine which tokens produce gradients.

High-quality specialist SFT depends more on source validity, case design, split integrity, template/mask correctness,
mixture control, and risk-aware evaluation than on raw row count. Synthetic data can expand coverage only when grounded,
filtered, deduplicated, and reviewed. Structured outputs and tool calls require separate schemas and evaluation.

Most importantly, SFT is not automatically comprehensive knowledge acquisition. It is often better at teaching how to
expose and format capabilities than at reliably storing every new fact. Specialist systems without runtime retrieval
must therefore measure factual coverage, preserve uncertainty, support updates, and require qualified verification for
high-risk decisions.

<a id="ch06-assessment"></a>

## 6.21 Review questions and engineering exercises

### Review questions

1. How do SFT, instruction tuning, chat tuning, and task-specific fine-tuning differ?
2. Why can two models from the same base family require different chat templates?
3. What is the token-level difference between completion-only and assistant-only loss?
4. Why can balanced row counts still create an imbalanced objective?
5. What evidence would show that a diagnostic model uses conversation history?
6. Why is valid JSON not sufficient evidence of a correct answer?
7. How can synthetic instruction data amplify hallucinations?
8. What tests distinguish uncertainty wording from calibrated uncertainty?
9. Why should source-related paraphrases remain in one data split?
10. Why is SFT alone a weak strategy for comprehensive acquisition of a large technical corpus?

### Engineering exercises

1. **Mask audit:** choose an exact checkpoint and immutable revision. Render three multi-turn records, print every token
   and assistant mask, and document control-token supervision. Stop if the template cannot return a trustworthy mask.
2. **Schema build:** create 100 provenance-linked JSONL records across QA, extraction, insufficient-information, and
   refusal tasks. Validate them with Listing 6.9 and add source-group split tests.
3. **Mixture analysis:** calculate row and target-token exposure under proportional, square-root, and equal-domain
   sampling. Predict and then measure worst-domain effects.
4. **Diagnostic ablation:** compare answer-only cases with incremental multi-turn cases. Score next-question utility,
   premature-conclusion rate, safety, and final accuracy.
5. **Synthetic-data audit:** generate candidates from approved case skeletons, blind-review a stratified sample, and
   report factual, safety, duplication, and style-repetition rejection rates.
6. **Knowledge experiment:** compare SFT-only with continued-pretraining-plus-identical-SFT under a controlled compute
   budget. Evaluate unseen facts, paraphrases, compositional questions, hallucinations, and general retention.
7. **Structured output:** train or evaluate a JSON task and report parse rate, schema-valid rate, field accuracy,
   semantic accuracy, and unsafe-valid outputs separately.
8. **Edge parity:** after a later quantization/export stage, rerun the same format, safety, and multi-turn suite against
   source and edge artifacts.

## 6.22 End notes

The schemas and calculations are teaching examples. The validator, mask-inspection, and training listings were not run;
resolve a compatible environment and verify current APIs before execution. A real experiment still needs immutable
source records, qualified review, split/leakage checks, multi-seed evaluation, and source-versus-edge artifact parity.

## Cross-links

- [Part index](README.md)
- [Detailed table of contents](../../01-planning/detailed-table-of-contents.md)
- [CH-06 assessment](../../13-assessments/ch-06-assessment.md)
- [Evidence standard](../../00-governance/evidence-and-citation-standard.md)
- [Reproducibility standard](../../00-governance/reproducibility-standard.md)
- [Safety and risk standard](../../00-governance/safety-and-risk-standard.md)
- [No-runtime-RAG standard](../../00-governance/no-runtime-rag-standard.md)
