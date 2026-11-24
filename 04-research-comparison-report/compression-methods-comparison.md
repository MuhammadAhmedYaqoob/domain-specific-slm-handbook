# Compression methods comparison

**Research freeze:** 2026-11-23  
**Status:** authored synthesis; no compressed artifact, benchmark, or paper result was reproduced.

## What “compression” must improve

Compression can target stored weights, dense dimensions, executed operations, Key-Value (KV) cache, or behavior
transferred into a smaller architecture. These are not equivalent. A smaller archive can load into the same dense
tensors; zeros can execute through the same dense kernels; low-bit weights can coexist with high-precision activations
and cache.

A method has deployment value only when the final model/runtime/backend/hardware tuple improves a measured objective
while retaining domain, general, and safety gates.

## Lesson: compress the bottleneck that actually matters

The historical motivation for model compression is simple: trained models exceeded available storage, memory bandwidth,
compute, or energy budgets. The engineering difficulty is that these budgets are different. Entropy coding can shrink a
download without reducing matrix multiplication. Unstructured pruning can create many zeros while dense kernels execute
the original shapes. Distillation can create a smaller architecture but requires another training process.

As an illustrative decision, suppose a 1.5B domain model fits storage but exceeds runtime memory at an 8K-token context.
First separate weight memory from Key-Value (KV) cache, activations, and buffers. If weights dominate, runtime-native
quantization is the simplest candidate. If cache dominates, reduce context or test supported cache quantization. If
latency remains too high after weight quantization, compare a native smaller model, a distilled student, and
structured-pruned dense model. Arbitrary sparsity is useful only when a matching sparse kernel exists.

Use compression when it improves a declared target metric without crossing quality or safety gates. Avoid stacking
methods because they are individually fashionable. Every additional transformation adds calibration, recovery,
conversion, provenance, and rollback work.

## Family comparison

<!-- markdownlint-disable MD013 -->
| Family                                     | Changes                                              | Storage                                     | Resident memory                                | Latency/throughput path                            | Recovery/calibration                              | Initial priority                        |
| ------------------------------------------ | ---------------------------------------------------- | ------------------------------------------- | ---------------------------------------------- | -------------------------------------------------- | ------------------------------------------------- | --------------------------------------- |
| Weight quantization                        | Numerical representation of weights                  | Usually lower                               | Lower weight component; other memory unchanged | Requires kernels consuming exact packed type       | PTQ may need calibration; QAT needs target recipe | Core practical                          |
| Weight+activation quantization             | Weights and activation execution types               | Lower weights                               | Can lower activations/workspaces               | Strong server/NPU potential with supported GEMM    | Calibration/QAT usually important                 | Advanced target-specific                |
| KV-cache quantization                      | Cache values                                         | Weight file unchanged                       | Lower context/concurrency cache                | Can alter bandwidth and add conversion             | Runtime-specific quality/context tests            | Advanced practical                      |
| Unstructured pruning                       | Arbitrary weights set to zero                        | Lower only with sparse encoding/compression | Lower only with sparse resident format         | Needs sparse kernels and break-even sparsity       | Calibration; recovery often needed                | Research baseline                       |
| N:M/semi-structured pruning                | Fixed sparse pattern                                 | Potentially lower                           | Backend-specific                               | Hardware/library/dtype/shape-specific sparse path  | Calibration/recovery                              | Target-specific                         |
| Structured pruning                         | Layers, heads, channels, neurons, dimensions removed | Lower dense tensors                         | Usually lower relevant tensors                 | Ordinary dense kernels can benefit if shapes align | Architecture surgery and recovery                 | Advanced practical                      |
| Distillation                               | Student behavior/representations                     | Smaller if student is smaller               | Smaller dense architecture                     | Portable when student runtime is supported         | Teacher data/logits/features and training         | Core edge candidate                     |
| Low-rank/tensor decomposition              | Factorized existing matrices                         | Potentially lower                           | Depends on factor runtime                      | Extra sequential kernels can erase gains           | Fine-tuning/reconstruction                        | Experimental/conditional                |
| Vocabulary/embedding compression           | Vocabulary/embedding rows or dimensions              | Can reduce large embedding component        | Lower embedding/output state                   | Output projection/tokenizer changes                | Retokenization/retraining and parity              | Advanced, high risk                     |
| Layer dropping/early exit/adaptive compute | Fewer layers per request/token                       | Lower only if layers removed                | Layer/cache reduction if static                | Dynamic paths need runtime control flow            | Recovery and exit policy                          | Static: advanced; dynamic: experimental |
| Weight sharing/clustering/entropy coding   | Repeated values or encoded storage                   | Can reduce file/download                    | Often decoded at load                          | Usually no direct compute reduction                | Rebuild/codec support                             | Packaging/research                      |
| Native small-model training                | Smaller architecture from the start                  | Smaller                                     | Smaller                                        | Coherent kernels/shape                             | Full pretraining/adaptation                       | Mandatory comparator                    |
<!-- markdownlint-restore -->

The family table compares what physically changes. Quantization changes representation; pruning may change values or
shapes; distillation transfers behavior into a chosen student; and native small-model training starts with a coherent
smaller design. Methods that reduce dense dimensions are generally more portable because ordinary kernels can consume
the result. Methods that retain shapes need special encodings and kernels before theoretical savings become executable
savings.

### Additional-method disposition

- **Low-rank weight factorization, matrix decomposition, and tensor decomposition** replace dense tensors with
  factors/cores. They save work only when the runtime executes the factorized operators efficiently; otherwise extra
  launches and intermediate tensors can negate parameter savings. Advanced/experimental.
- **Layer dropping** statically removes blocks and can reduce dense compute and per-layer KV cache; it requires recovery
  and configuration/export surgery. **Early-exit** and other adaptive-computation methods choose depth dynamically,
  adding confidence policy and runtime control-flow requirements. Static: advanced practical; dynamic: experimental.
- **Weight sharing and parameter tying** reduce unique stored parameters when designed into or retrained with the
  architecture. Post hoc sharing can harm capacity and does not necessarily reduce executed operations. Conditional
  research.
- **Model merging and adapter merging** combine trained states; they are model-combination/release techniques, not
  guaranteed compression. Merged LoRA usually restores a full dense artifact. Core operational step, not a size claim.
- **Architecture shrinking** explicitly reduces width, depth, heads, or MLP dimensions and overlaps structured
  pruning/distillation. It is advanced practical when resulting shapes remain kernel-friendly.
- **Vocabulary optimization and embedding compression** can matter disproportionately for SLMs, but tokenizer changes
  alter sequence lengths, embeddings/output heads, templates, and compatibility. Advanced and high risk.
- **Weight clustering** shares centroids and **entropy coding** compresses serialized symbols. Both can reduce
  storage/download; runtime memory or speed improves only if the deployed kernel consumes the encoding rather than
  expanding it.
- **Memory-mapped loading** reduces startup copies/page residency patterns but is a loading strategy, not a reduction in
  mathematical model size.
- **Hardware-aware neural architecture search and joint architecture/compression search** can optimize a target tuple
  but demand a large search/retraining budget and risk benchmark overfitting. Experimental.

Technical treatment and project applicability are expanded in
[CH-14: Additional model compression techniques](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md).

## Quantization methods

<!-- markdownlint-disable MD013 -->
| Method/ecosystem        | Mechanism                                                                | Calibration                                                                         | Hardware/runtime fit                                             | Maturity judgment                  |
| ----------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------- |
| LLM.int8()/bitsandbytes | INT8 with higher-precision outlier path; 4-bit NF4/FP4 training modules  | Load path not centered on representative calibration; thresholds/model still matter | GPU training/cloud; not a mobile interchange format              | Established for PEFT/cloud         |
| GPTQ                    | Layer-wise approximate second-order weight reconstruction                | Required                                                                            | Mature CUDA/cloud ecosystem; multiple noninterchangeable layouts | Core cloud experiment              |
| AWQ                     | Activation-aware saliency and scaling before low-bit weight quantization | Required                                                                            | Mature cloud/edge-GPU implementations where exact kernels exist  | Core practical                     |
| SmoothQuant             | Move activation difficulty into weights for W8A8                         | Required                                                                            | Server INT8 and supported NPU export                             | Established target-specific        |
| GGUF/GGML quant types   | Runtime/container-specific block and mixed tensor encodings              | Optional importance matrix for some paths                                           | llama.cpp CPU/Metal/Vulkan ecosystem                             | Core edge baseline                 |
| HQQ                     | Half-quadratic optimization of quantization parameters                   | Not required by method                                                              | CUDA/Triton/ATen; layout controls kernel availability            | Advanced                           |
| AQLM                    | Additive codebooks at extreme low bits                                   | Required; expensive optimization                                                    | Narrower CPU/GPU kernels                                         | Advanced/experimental deployment   |
| SpQR                    | Low-bit dense weights plus sparse high-precision outliers                | Required                                                                            | Specialized representation/kernels                               | Advanced                           |
| QuIP/QuIP#              | Incoherence transforms and extreme low-bit codebooks                     | Required                                                                            | Specialized CUDA; QuIP# repository maintenance/license caveats   | Experimental                       |
| BitNet/native low-bit   | Train binary/ternary architecture rather than PTQ a dense model          | Training recipe, not PTQ calibration                                                | bitnet.cpp and supported checkpoint path                         | Separate experimental model family |
<!-- markdownlint-restore -->

The quantization table narrows method choice by target. A calibration-free method is not automatically easier if its
layout lacks a production kernel. A widely supported 8-bit or 4-bit runtime format can outperform a more compressed
research representation in engineering value because it loads reliably, streams tokens, and can be maintained.

Formats are not algorithms: “GGUF,” “INT4,” “FP4,” and “ONNX” are insufficient without exact encoding,
group/granularity, scales/zero points, exclusions, cache type, converter, kernel, and device. See
[quantization methods](quantization-methods.md).

## Pruning methods

<!-- markdownlint-disable MD013 -->
| Method                                    | Structure produced                                  | Principal evidence               | Deployment caveat                                                          | Priority            |
| ----------------------------------------- | --------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------- | ------------------- |
| Magnitude / movement / importance pruning | Arbitrary zeros or selected structures              | Established baseline literature  | Dense runtime may do unchanged work                                        | Control             |
| SparseGPT                                 | Unstructured or N:M reconstructed sparsity          | ICML 2023 paper/official code    | Paper quality result is not target speed                                   | Sparse baseline     |
| Wanda                                     | Activation-weighted magnitude sparsity              | ICLR 2024 paper/official code    | Calibration distribution affects scores                                    | Sparse baseline     |
| LLM-Pruner                                | Coupled structural removal + LoRA recovery          | NeurIPS 2023 paper/official code | Architecture-specific surgery; degeneration risk                           | Advanced practical  |
| SliceGPT                                  | Orthogonal transforms then dense row/column removal | ICLR 2024 paper/official code    | Residual/tied/export assumptions and aligned widths                        | Advanced practical  |
| ShortGPT                                  | Static layer removal; dynamic generation variant    | ACL Findings 2025                | Static removal can harm generation; dynamic skipping needs runtime support | Experimental/recent |
<!-- markdownlint-restore -->

The pruning table shows why method quality and deployment quality are separate. SparseGPT and Wanda are valuable
research controls for weight removal. LLM-Pruner, SliceGPT, and static layer removal can create smaller dense work, but
must preserve architecture invariants and aligned dimensions. ShortGPT-style dynamic skipping additionally requires
runtime control flow.

For mobile portability, structured layer/width removal is more promising than arbitrary zeros because ordinary dense
kernels can consume the result. Even then, irregular dimensions may lose vector/tensor-core efficiency. See
[pruning methods](pruning-methods.md).

## Distillation methods

<!-- markdownlint-disable MD013 -->
| Method                         | Teacher access                  | Student signal                                | Best use                          | Caveat                                            |
| ------------------------------ | ------------------------------- | --------------------------------------------- | --------------------------------- | ------------------------------------------------- |
| Response/sequence distillation | Text outputs                    | Student next-token loss on accepted responses | Practical black-box route         | One decoded sequence hides teacher uncertainty    |
| Logit distillation             | Full/top-k logits               | KL/soft-target objective                      | Rich open-teacher signal          | Vocabulary/position alignment and storage         |
| Feature/attention distillation | Internal tensors                | Hidden/relationship losses                    | Related architectures             | Layer, width, head, mask, tokenizer mismatch      |
| Online/on-policy KD            | Teacher during student sampling | Teacher feedback on student states            | Exposure-mismatch research        | High compute and coupling                         |
| Self/multi-teacher KD          | Same/multiple teachers          | Responses/logits/features                     | Diversity or conflict experiments | Reinforced errors, calibration, license conflicts |
| Data-free/synthetic KD         | Generated transfer data         | Usually responses                             | Low-data bootstrap                | Not evidence-free; facts require validation       |
<!-- markdownlint-restore -->

The distillation table progresses from observable teacher text to increasingly privileged internal signals. Response
distillation is compatible with black-box teachers and different tokenizers but loses uncertainty. Logit, feature, and
attention methods expose richer information while imposing alignment, storage, and access costs. The required control
is always a natively pretrained model with the student's deployment budget.

DistilBERT, TinyBERT, and MiniLM are mature encoder references, not direct decoder recipes. Sequence-level KD is
established; MiniLLM is a modern generative on-policy research baseline. A distilled student must beat a natively
pretrained model of the same deployment budget. See [distillation methods](distillation-methods.md).

## Hardware fit matrix

<!-- markdownlint-disable MD013 -->
| Target                 | First methods                                              | Conditional methods                         | Avoid assuming                                  |
| ---------------------- | ---------------------------------------------------------- | ------------------------------------------- | ----------------------------------------------- |
| ARM/mobile CPU         | Runtime-native INT8/4-bit; native/distilled dense model    | Aligned structured pruning; KV quantization | Unstructured zeros or NF4 training format speed |
| Mobile GPU             | Runtime-native low-bit and supported graph                 | Structured student, mixed precision         | CUDA AWQ/GPTQ result portability                |
| Mobile NPU             | Vendor/exporter-supported W8A8/INT4 recipe                 | Exact-target QAT                            | Generic ONNX/INT4 means delegation              |
| AWS NVIDIA low-batch   | BF16/FP16 baseline, supported weight-only GPTQ/AWQ         | TensorRT-LLM formats, 2:4                   | Mobile quantizer ranking transfer               |
| AWS throughput/prefill | BF16/FP16, W8A8/FP8 where supported                        | TensorRT-LLM, structured student            | Nominal bit width predicts throughput           |
| Apple                  | llama.cpp Metal/MLX low-bit; Core ML supported compression | Core ML QAT/stateful path                   | Conversion proves ANE placement                 |
<!-- markdownlint-restore -->

## Recommended order

1. Qualify untouched source and target runtimes.
2. Adapt and preserve a high-precision parent.
3. Merge adapter where required and verify parity.
4. Compare high precision, INT8, and one or two mature 4-bit runtime-native children.
5. If constraints remain unmet, compare a native smaller model, distilled dense student, and structured-pruned/recovered
   model.
6. Quantize architecture-reduced candidates from fresh high-precision parents.
7. Use mixed precision if uniform low bit narrowly fails.
8. Use target-specific QAT only after PTQ is isolated as the blocker.
9. Use arbitrary sparsity, extreme 2-bit methods, or joint pruning/KD only with a concrete kernel/research hypothesis.

Detailed ordering and rollback rules are in [combined optimization pipelines](combined-optimization-pipelines.md).

## Experiment matrix and records

Use successive halving rather than a full Cartesian product:

- baseline FP/BF16/FP16 as appropriate;
- INT8 and two supported 4-bit recipes;
- Wanda and SparseGPT research controls;
- one structured candidate with no recovery and recovery;
- native small, validated response-distilled, and optional logit-distilled student;
- selected architecture winners at selected precisions;
- at most three physical-device finalists per target.

Every row records source and tokenizer revisions; license; training/calibration/recovery/test hashes;
quantizer/pruner/teacher details; nominal and effective bits; physical parameter/nonzero counts; artifact and resident
bytes; conversion time; peak memory by phase; TTFT, prefill, decode and sustained performance; energy/thermal
conditions; domain/general/safety metrics with uncertainty; runtime/kernel/device; failures; and raw-result locations.

## Stop rules

Reject or stop when:

- a quality or risk-stratified safety gate fails;
- the format loads through dense/fallback kernels and does not improve the target;
- peak memory does not cross the required budget;
- latency/energy improvement is below the pre-registered practical threshold;
- the candidate is Pareto-dominated by a simpler artifact;
- the target converter cannot represent modified shapes/operators;
- calibration/recovery leaks evaluation data; or
- provenance, license, or rollback cannot be reconstructed.

## Evidence limits and references

No quality retention, storage ratio, speedup, memory saving, energy saving, or target compatibility is claimed as a
project result. Paper findings remain `PAPER-REPORTED`; implementation support remains version- and tuple-specific.

- [Quantization survey](quantization-methods.md)
- [Pruning survey](pruning-methods.md)
- [Distillation survey](distillation-methods.md)
- [Combined pipeline survey](combined-optimization-pipelines.md)
- [Quantization protocol](../06-experiment-framework/protocols/quantization-benchmark.md)
- [Pruning protocol](../06-experiment-framework/protocols/pruning-benchmark.md)
- [Distillation protocol](../06-experiment-framework/protocols/distillation-benchmark.md)
- [Edge-device protocol](../06-experiment-framework/protocols/edge-device-benchmark.md)
