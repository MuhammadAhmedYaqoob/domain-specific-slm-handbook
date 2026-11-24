# Phase 11: Test pruning and recovery

[Previous phase](phase-10-quantization.md) · [Next phase](phase-12-combined-optimization.md)

## Learning objectives

- Distinguish zero weights from executable sparsity.
- Compare unstructured, N:M, and structured pruning.
- Include recovery cost in the deployment decision.

## Theory and purpose

Unstructured pruning zeros individual weights; it saves compute only when storage and kernels exploit the pattern.
N:M sparsity uses hardware-specific regular patterns. Structured pruning removes layers, heads, channels, or
neurons and can shrink a dense graph, but architecture surgery is difficult and may break export. Recovery
fine-tuning can restore quality while adding compute and lineage.

## Prerequisites

- Selected dense checkpoint and target runtimes.
- Training-only calibration and recovery data.
- Architecture invariants, sparse-kernel evidence, and export tests.

## Step-by-step method

1. Run sensitivity analysis on a copy and protect tied or fused weights.
2. Create dense-zero controls at several ratios and verify exact sparsity.
3. Test N:M only on hardware and kernels that claim the exact pattern.
4. Materialize structured candidates as valid resized graphs rather than masks.
5. Apply fixed-budget recovery and reevaluate quality, safety, memory, latency, and export.
6. Compare total benefit with recovery compute and maintenance burden.

## Worked automotive example

**`ILLUSTRATIVE` pruning study.** Thirty-percent unstructured pruning produces 30% zeros but leaves the file at
3.10 GB and Android decode at 3.2 tok/s because the dense kernel still runs. A 2:4 candidate reaches 4.7 tok/s on
a fictional compatible cloud accelerator but has no mobile kernel. Removing two of 28 layers shrinks the dense
artifact to 2.88 GB; after recovery it scores 0.77, below the 0.78 floor. No pruning arm advances.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.pruning-decision/v1",
  "evidence": "ILLUSTRATIVE",
  "arms": {
    "unstructured_30": {"logical_sparsity": 0.30, "artifact_gb": 3.10, "mobile_tok_s": 3.2},
    "nm_2_of_4": {"cloud_tok_s": 4.7, "mobile_kernel": false},
    "drop_2_layers_recovered": {"artifact_gb": 2.88, "domain_score": 0.77}
  },
  "quality_floor": 0.78,
  "decision": "reject_all_pruning_arms"
}
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` calculation.** Half the toy weights are zero, yet dense storage still allocates every element.
This is the central deployment lesson: count zeros and measure the executing representation separately.

```python
import torch

weight = torch.tensor([[0.0, 2.0, 0.0], [1.0, 0.0, 3.0]])
logical_sparsity = (weight == 0).sum().item() / weight.numel()
dense_bytes = weight.numel() * weight.element_size()
assert logical_sparsity == 0.5
print({"logical_sparsity": logical_sparsity, "dense_bytes": dense_bytes})
```

## Expected behavior

Reported sparsity matches tensor inspection, and any claimed speed or memory gain appears on the executing target kernel. Recovery artifacts retain parent lineage and must regain required quality and safety.

## Common failures and recovery

- **Failure:** Gzip-compressed checkpoint savings are mistaken for runtime savings.
- **Failure:** A sparse cloud kernel is assumed to exist on mobile.
- **Failure:** Recovery is tuned on sealed evaluation results.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Pruning can damage rare hazard behavior before average quality moves; stratify safety tests and reject any critical regression.

## Interpret the result

A negative result prevents complexity. If structured surgery fails, a native smaller student may be the better path.

## Transition

[Phase 12](phase-12-combined-optimization.md) combines only methods that showed independent value and compatible ordering.
