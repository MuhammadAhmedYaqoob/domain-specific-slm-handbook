# Part IV — Model Compression and Optimization

<a id="part-04"></a>

Compression is a constrained optimization problem, not a file-conversion step. This part connects weight storage, loaded
and peak memory, bandwidth, compute, KV cache, runtime kernels, output quality, and safety.

## Conceptual map

First establish where bytes and time go. Quantization changes numeric representation; pruning removes or zeros weights
or structures; distillation trains a smaller student; architecture changes redesign the computation. Every method
creates a new artifact that must pass quality and target-runtime gates. Combining methods is an ordered experiment,
because the output and damage of one step become the input to the next.

## Learning outcomes and dependencies

Using Parts I–III, readers learn to build a memory ledger; select PTQ or QAT and granularity; distinguish sparse files
from accelerated sparsity; design sequence/logit/feature distillation; and order interacting optimizations. Read
[Part V](../part-05-edge-ai-and-mobile-inference/README.md) concurrently because a compression format has value only
when a target runtime executes it well.

## Chapters

1. [CH-10 — Fundamentals](ch-10-the-fundamentals-of-model-compression.md): objectives, constraints, and resource
   accounting.
2. [CH-11 — Quantization](ch-11-quantization-complete-research-and-implementation.md): affine/symmetric schemes,
   granularity, weight-only/W8A8, QAT, and KV-cache precision.
3. [CH-12 — Pruning](ch-12-pruning-complete-research-and-implementation.md): unstructured, N:M, and structured methods
   plus recovery.
4. [CH-13 — Knowledge Distillation](ch-13-knowledge-distillation-complete-research-and-implementation.md): sequence,
   logits, hidden states, attention, and teacher/student design.
5. [CH-14 — Additional Techniques](ch-14-additional-model-compression-techniques.md): factorization, sharing,
   vocabulary/layer reduction, and architecture changes.
6. [CH-15 — Combining Techniques](ch-15-combining-optimization-techniques.md): ordering, interaction effects, Pareto
   analysis, and rollback.

## Decision checkpoint

Freeze an uncompressed reference and change one factor at a time before testing combinations. Reject candidates that
violate critical quality/safety or exact-target memory, latency, energy, thermal, and operator-support gates.

### Practice before continuing

For a one-billion-parameter model, calculate ideal FP16, INT8, and INT4 payloads. Then write five reasons the smallest
number may not yield the fastest or lowest-peak-memory runtime.

## Cross-links

[Hardware/memory reference](../back-matter/appendix-hardware-and-memory-reference.md) ·
[Part V](../part-05-edge-ai-and-mobile-inference/README.md) ·
[CH-31 benchmarking](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
