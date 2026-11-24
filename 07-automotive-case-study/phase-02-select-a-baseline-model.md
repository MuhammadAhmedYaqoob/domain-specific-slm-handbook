# Phase 2: Select reproducible baseline candidates

[Previous phase](phase-01-define-scope.md) · [Next phase](phase-03-acquire-data.md)

## Learning objectives

- Distinguish model-family reputation from checkpoint suitability.
- Probe licenses, tokenizer behavior, training, save/reload, and target export.
- Choose a baseline without using downstream test results.

## Theory and purpose

Model selection is a constrained admission test. Parameter count predicts neither automotive token efficiency nor
deployability. The tokenizer controls sequence length; the architecture controls runtime compatibility; the
checkpoint license controls permitted use; and base versus instruction tuning changes the valid prompting and
adaptation strategy. Pin model, tokenizer, configuration, and license snapshots together.

## Prerequisites

- Approved Phase 1 scope and target budgets.
- Enough local or cloud memory for a load and one backward-pass smoke test.
- A rights reviewer and target runtime compatibility checklist.

## Step-by-step method

1. Create separate candidate strata for base training, instruction baseline, small pipeline control, and edge comparison.
2. Resolve every repository to an immutable commit and archive license and model-card snapshots.
3. Measure tokenizer fertility on automotive DTCs, units, part-number-like strings, wiring notation, and long cases.
4. Run load, forward, backward, save, reload, and fixed-logit parity probes with remote code disabled.
5. Attempt the smallest honest export for each required runtime.
6. Score only scope-relevant admission criteria and document rejected candidates.

## Worked automotive example

**`ILLUSTRATIVE` comparison.** `auto-base-r1` has 1.5 billion parameters, tokenizes a 120-character automotive
prompt into 42 tokens, completes the training smoke test in 5.8 GiB, and exports to the control runtime.
`auto-small-r1` has 360 million parameters and uses only 2.1 GiB, but produces 63 tokens and lacks the required
chat-template behavior. The first becomes the adaptation parent; the second remains a pipeline and distillation
control. These fictional observations demonstrate role-based selection, not real checkpoint properties.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.baseline-decision/v1",
  "evidence": "ILLUSTRATIVE",
  "selected_parent": "auto-base-r1",
  "selected_instruction_baseline": "auto-instruct-r1",
  "pipeline_control": "auto-small-r1",
  "reasons": [
    "rights review accepted for teaching scenario",
    "42 tokens on the fixed automotive probe",
    "load, backward, save/reload, and control export probes passed"
  ],
  "rejected": [{"id": "auto-edge-r1", "reason": "target export probe failed"}]
}
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** The probe loads only a previously downloaded local checkpoint and rejects custom
repository code. The token limit is a predeclared admission check, not a universal quality rule. A missing file,
unsupported architecture, or token count of 80 or more records a failed probe rather than triggering a silent
model substitution.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

path = "artifacts/checkpoints/source/auto-base-r1"
text = "P0300, 12.6 V, 20 °C, CAN bus, bank 1 sensor 2"
tokenizer = AutoTokenizer.from_pretrained(path, local_files_only=True, trust_remote_code=False)
model = AutoModelForCausalLM.from_pretrained(path, local_files_only=True, trust_remote_code=False)
encoded = tokenizer(text, return_tensors="pt", add_special_tokens=False)
assert encoded.input_ids.shape[1] < 80
assert sum(p.numel() for p in model.parameters()) > 0
print({"tokens": encoded.input_ids.shape[1]})
```

## Expected behavior

Each candidate either produces pinned identity, rights, tokenizer, train/save/reload, and export evidence or records a rejection reason. The process never substitutes another checkpoint after a failed probe.

## Common failures and recovery

- **Failure:** Using a moving branch such as `main` makes later runs irreproducible.
- **Failure:** Comparing a raw base model with an instruction model under one prompt template is invalid.
- **Failure:** Trusting a model card's mobile claim skips exact architecture and runtime testing.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Download only reviewed artifacts, prefer safe tensor formats, scan files, and never enable remote code merely to make a candidate load.

## Interpret the result

A selected baseline is the best admitted experiment parent, not the final model and not proof of automotive competence.

## Transition

[Phase 3](phase-03-acquire-data.md) builds the rights-approved corpus used to teach and evaluate the selected candidates.
