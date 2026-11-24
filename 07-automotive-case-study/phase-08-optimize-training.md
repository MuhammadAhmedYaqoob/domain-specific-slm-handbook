# Phase 8: Compare full fine-tuning, LoRA, and QLoRA

[Previous phase](phase-07-supervised-fine-tuning.md) · [Next phase](phase-09-distillation.md)

## Learning objectives

- Explain what each method trains and stores.
- Design a fair resource-quality comparison.
- Verify adapter merge and distinguish training precision from release precision.

## Theory and purpose

Full fine-tuning updates every parameter. LoRA freezes the parent and learns low-rank matrices whose product adds
a weight update. QLoRA stores the frozen parent in a low-bit training representation while learning LoRA
parameters. QLoRA reduces training residency; it does not automatically create a smaller final inference
artifact. Fair comparison fixes parent, data, supervised tokens, seeds, evaluation schedule, and stopping rules.

## Prerequisites

- One parent and one SFT snapshot/template from Phase 7.
- Hardware envelopes capable of at least one method plus a smaller smoke configuration for all methods.
- A merge/reload parity test and target export probe.

## Step-by-step method

1. Declare target modules, rank, alpha, dropout, precision, optimizer, and exact trainable parameter count.
2. Use equal supervised-token budgets and identical validation schedules.
3. Record peak allocated and reserved memory, wall time, throughput, failures, and checkpoint bytes.
4. Evaluate domain, safety, retention, and uncertainty by seed.
5. Merge LoRA adapters into the exact parent, reload, and compare fixed-input logits.
6. Advance non-dominated methods; retain an adapter deployment only if the runtime supports and verifies it.

## Worked automotive example

**`ILLUSTRATIVE` comparison.** Full tuning reaches domain 0.82 using 22.4 GiB peak VRAM in 9.1 hours. LoRA reaches
0.81 using 11.2 GiB in 5.0 hours. QLoRA reaches 0.80 using 7.4 GiB in 5.8 hours. All have zero fictional critical
failures and retention above 0.75. LoRA advances because its 0.01 quality difference does not exceed the
predeclared practical margin and it halves peak VRAM. QLoRA remains the low-memory training alternative.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.training-method-decision/v1",
  "evidence": "ILLUSTRATIVE",
  "arms": {
    "full": {"domain": 0.82, "peak_vram_gib": 22.4, "hours": 9.1},
    "lora": {"domain": 0.81, "peak_vram_gib": 11.2, "hours": 5.0},
    "qlora": {"domain": 0.80, "peak_vram_gib": 7.4, "hours": 5.8}
  },
  "practical_quality_margin": 0.015,
  "primary_parent": "auto-sft-lora-r1",
  "low_memory_training_option": "auto-sft-qlora-r1"
}
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` calculation.** For equally shaped matrices, each adapter contributes `hidden × rank` for A and
B. Real architectures have unequal projections, so compute the count from named trainable tensors and compare it
with the trainer report. The example teaches scale, not a specific checkpoint fact.

```python
hidden, rank, layers, adapted_matrices = 1536, 16, 28, 7
lora_parameters = layers * adapted_matrices * 2 * hidden * rank
base_parameters = 1_500_000_000
fraction = lora_parameters / base_parameters
print({"lora_parameters": lora_parameters, "trainable_fraction": round(fraction, 4)})
```

## Expected behavior

All three methods consume equal supervised tokens and report trainable parameters, memory, time, quality, safety, and retention. Merged adapters reload with fixed-logit parity before export.

## Common failures and recovery

- **Failure:** Different token budgets turn a method comparison into a compute comparison.
- **Failure:** Reporting allocated memory while another arm reports reserved memory is unfair.
- **Failure:** A merged adapter can load yet differ because of wrong parent hash or merge precision.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Every arm receives the same hazard evaluation; resource efficiency cannot compensate for a safety regression.

## Interpret the result

Choose per resource envelope. “Equivalent” requires a declared practical margin and uncertainty, not similar point estimates.

## Transition

[Phase 9](phase-09-distillation.md) tests whether a smaller native student can improve mobile trade-offs beyond direct same-size training.
