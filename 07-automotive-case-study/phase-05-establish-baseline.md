# Phase 5: Establish the untouched baseline

[Previous phase](phase-04-process-data.md) · [Next phase](phase-06-domain-adaptation.md)

## Learning objectives

- Record model behavior before training.
- Separate deterministic comparison from sampled robustness.
- Combine machine scoring with blinded expert review.

## Theory and purpose

A baseline gives every later delta a denominator. Without preserved raw outputs, improvements can be artifacts of
prompt changes, decoding, tokenizer differences, or evaluator drift. Base and instruction checkpoints need their
valid prompt formats. Primary paired comparison should hold prompts, token budgets, and deterministic decoding
constant; sampling belongs in a separate robustness study.

## Prerequisites

- Pinned candidate artifacts from Phase 2.
- A sealed, independently authored evaluation suite from Phase 4.
- Blinded reviewers and a rubric covering facts, diagnostic ordering, uncertainty, refusal, and hazards.

## Step-by-step method

1. Hash the model, tokenizer, template, suite, environment, and hardware description.
2. Render prompts and audit token IDs before generation.
3. Run greedy generation with fixed maximum input and output lengths.
4. Preserve every output, timeout, invalid record, and token count.
5. Blind candidate identity and obtain expert scores with disagreement adjudication.
6. Publish item-level results and aggregate confidence intervals as the immutable comparator.

## Worked automotive example

**`ILLUSTRATIVE` baseline.** On 100 fictional cases, `auto-instruct-r1` earns domain accuracy 0.61, diagnostic
next-step score 0.54, unsupported-claim rate 0.12, and two critical safety failures. `auto-base-r1` earns 0.43,
0.31, 0.18, and three failures under its neutral completion format. Neither is release-eligible; the values
identify weaknesses for training. For the P0300 case, the instruction model correctly defines the code but
prematurely recommends plugs, losing diagnostic-ordering points.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.baseline-result/v1",
  "run_id": "run-baseline-illustrative-001",
  "model_id": "auto-instruct-r1",
  "cases": 100,
  "domain_accuracy": 0.61,
  "diagnostic_next_step": 0.54,
  "unsupported_claim_rate": 0.12,
  "critical_safety_failures": 2,
  "decoding": {"do_sample": false, "max_new_tokens": 256},
  "evidence": "ILLUSTRATIVE"
}
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** This validation reads a populated summary and checks denominator and metric
bounds. With the illustrative artifact it prints `false`, as it should. The script does not generate or score
responses; those components need separate tested implementations and expert review.

```python
import json
from pathlib import Path

result = json.loads(Path("results/runs/baseline/summary.json").read_text("utf-8"))
assert result["cases"] == 100
assert 0 <= result["domain_accuracy"] <= 1
assert 0 <= result["unsupported_claim_rate"] <= 1
eligible = result["critical_safety_failures"] == 0
print({"release_eligible": eligible})
```

## Expected behavior

The run preserves one output or explicit failure per sealed case and reproduces deterministic generations under the same environment. Scores remain linked to raw outputs and blinded reviews.

## Common failures and recovery

- **Failure:** Changing the prompt template between baseline and trained model invalidates paired attribution.
- **Failure:** Discarding timeouts makes a weak model appear more accurate.
- **Failure:** An automated semantic score can reward fluent but unsupported repairs.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Any critical failure makes the artifact ineligible regardless of its average score; preserve the exact prompt and response for review.

## Interpret the result

Use baseline weaknesses to design training data, never to modify sealed test cases or rubrics.

## Transition

[Phase 6](phase-06-domain-adaptation.md) asks whether CPT improves domain representation enough to justify extra compute and forgetting risk.
