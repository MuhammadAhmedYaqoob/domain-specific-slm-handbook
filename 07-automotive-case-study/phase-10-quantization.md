# Phase 10: Quantize for exact deployment targets

[Previous phase](phase-09-distillation.md) · [Next phase](phase-11-pruning.md)

## Learning objectives

- Describe quantization with enough metadata to reproduce it.
- Separate file size, loaded memory, speed, and quality effects.
- Test formats in the runtime and hardware that will execute them.

## Theory and purpose

Quantization represents weights or activations with fewer bits plus scales and, for asymmetric schemes, zero
points. Error depends on granularity, group size, outliers, calibration, kernels, and architecture. “4-bit” is
not a complete method. A training representation such as QLoRA NF4 is also not automatically a release format.
Conversion must be followed by exact-runtime regression evaluation.

## Prerequisites

- Merged high-precision parents from Phases 8 and 9.
- Training-only calibration data with representative lengths and tasks.
- Pinned converter, runtime, backend, and target hardware.

## Step-by-step method

1. Export a high-precision target-runtime reference and verify tokenizer/template parity.
2. Create 8-bit and at least two documented 4-bit candidates with full method metadata.
3. Record converter logs, temporary storage, duration, hashes, and failures.
4. Run fixed-prompt parity, domain, safety, and long-context regressions.
5. Measure artifact bytes, post-load and peak memory, startup, TTFT, prefill, decode, and reliability.
6. Select per target; do not rename one runtime's artifact for another runtime.

## Worked automotive example

**`ILLUSTRATIVE` Android result.** The F16 reference is 3.10 GB, peaks at 4.80 GiB, decodes at 3.2 tok/s, and
scores 0.81 domain. An 8-bit artifact is 1.62 GB, 3.10 GiB, 4.4 tok/s, and 0.81. `Q4-A` is 0.92 GB, 2.35 GiB,
6.1 tok/s, and 0.79. `Q4-B` is 0.86 GB, 2.28 GiB, 5.8 tok/s, and 0.76. With a 0.78 quality floor, Q4-A advances;
Q4-B fails despite its smaller file.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.quantized/v1",
  "evidence": "ILLUSTRATIVE",
  "id": "auto-q4-a-r1",
  "parent": "auto-sft-lora-merged-r1",
  "method": "grouped_weight_only",
  "weight_bits": 4,
  "group_size": 128,
  "activation_precision": "fp16",
  "kv_cache_precision": "fp16",
  "artifact_bytes": 920000000,
  "peak_memory_gib": 2.35,
  "decode_tokens_per_second": 6.1,
  "domain_score": 0.79
}
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` calculation.** Filtering by both quality and memory rejects the smallest candidate. Real target
selection must also include safety, latency, energy, sustained behavior, and reliability.

```python
candidates = [
    {"id": "f16", "quality": 0.81, "memory": 4.80},
    {"id": "int8", "quality": 0.81, "memory": 3.10},
    {"id": "q4-a", "quality": 0.79, "memory": 2.35},
    {"id": "q4-b", "quality": 0.76, "memory": 2.28},
]
eligible = [x for x in candidates if x["quality"] >= 0.78 and x["memory"] <= 3.0]
selected = min(eligible, key=lambda x: x["memory"])
assert selected["id"] == "q4-a"
print(selected)
```

## Expected behavior

Each export loads in its declared runtime, matches tokenizer/template behavior, and reports format metadata, memory, latency, quality, and safety. A smaller file that violates a floor is rejected.

## Common failures and recovery

- **Failure:** A smaller file can use more temporary or KV-cache memory.
- **Failure:** A format supported by a converter may lack a fast target kernel.
- **Failure:** Calibration leakage lets test cases influence the artifact.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Run the full critical-safety suite after conversion; low-frequency logits can change refusal or hazard wording.

## Interpret the result

Quantization quality is artifact-, runtime-, workload-, and target-specific. Preserve the high-precision parent for rollback.

## Transition

[Phase 11](phase-11-pruning.md) tests whether removing weights or structure creates benefits beyond quantization.
