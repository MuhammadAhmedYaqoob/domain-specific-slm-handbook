# Phase 7: Teach safe diagnostic behavior with SFT

[Previous phase](phase-06-domain-adaptation.md) · [Next phase](phase-08-optimize-training.md)

## Learning objectives

- Design a task mixture that teaches behavior rather than slogans.
- Audit chat templates, truncation, and assistant-only loss.
- Use development data without contaminating sealed evaluation.

## Theory and purpose

Supervised fine-tuning maps prompts to desired response patterns. It is effective for answer structure,
terminology, uncertainty, and safe diagnostic ordering, but it does not guarantee broad factual knowledge.
Assistant-only loss prevents user text from becoming the target. A chat-template or mask defect can train on the
wrong tokens while still producing a decreasing loss.

## Prerequisites

- Accepted base or CPT parent from Phase 6.
- Immutable train and validation snapshots with task, risk, source, and difficulty labels.
- An exact tokenizer and chat template plus sufficient compute for a two-batch overfit test.

## Step-by-step method

1. Balance direct QA, terminology, structured extraction, multi-turn diagnosis, insufficient-information, counterfactual, escalation, and false-refusal examples.
2. Render representative conversations and inspect token IDs and assistant masks.
3. Run a two-batch overfit test to verify gradients, labels, save/reload, and deterministic logits.
4. Train with a fixed supervised-token budget and validation-only checkpoint selection.
5. Track metrics by task and risk, not only average loss.
6. Test memorization, refusal overreach, general retention, and safety before advancing.

## Worked automotive example

**`ILLUSTRATIVE` target response.** For the P0300 case, the training answer first checks ventilation and unsafe
operating conditions, then lists confirmed observations. It proposes ignition, fuel, air-leak, compression, and
sensor/input hypotheses; asks for related codes and cylinder counters; and explains how a cylinder-specific
pattern changes the next check. It does not name a replacement part. In a fictional 8,000-record mixture, 25%
covers diagnosis, 20% theory, 15% units, 15% insufficient information, 15% safety, and 10% general retention.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```yaml
schema: automotive-slm.sft/v1
evidence: ILLUSTRATIVE
parent: auto-cpt-replay-r1
seed: 1729
max_length: 2048
assistant_only_loss: true
epochs: 1.0
learning_rate: 0.00002
effective_batch_size: 16
supervised_tokens: 6200000
selection: validation_only
task_mix:
  diagnosis: 0.25
  theory: 0.20
  units: 0.15
  insufficient_information: 0.15
  safety: 0.15
  general_retention: 0.10
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` mask audit.** The example proves that prompt tokens are ignored and assistant tokens are
supervised in a toy rendered record. In the real trainer, inspect masks after the exact template and truncation
logic; a plausible aggregate fraction alone does not prove role boundaries.

```python
def assistant_fraction(labels: list[int]) -> float:
    supervised = sum(token != -100 for token in labels)
    return supervised / len(labels)

sample_labels = [-100] * 12 + [101, 202, 303, 404, 505, 606, 707, 808]
fraction = assistant_fraction(sample_labels)
assert 0 < fraction < 1
assert all(x == -100 for x in sample_labels[:12])
print({"assistant_token_fraction": fraction})
```

## Expected behavior

Rendered prompts use the exact template, prompt tokens carry ignore labels, assistant targets carry loss, and save/reload parity holds. Training stops on invalid masks, leakage, or unstable validation.

## Common failures and recovery

- **Failure:** A template without assistant-mask support silently trains on prompt tokens.
- **Failure:** Truncation can remove the answer while retaining a valid-looking prompt.
- **Failure:** Repeated synthetic phrasing teaches style and unsupported certainty.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Include safe helpful responses as well as refusals; otherwise the model may refuse benign theory yet answer hazardous procedural prompts.

## Interpret the result

Validation loss is a training-health signal. Domain correctness and release safety remain Phase 13 questions.

## Transition

[Phase 8](phase-08-optimize-training.md) holds this task constant while comparing full fine-tuning, LoRA, and QLoRA.
