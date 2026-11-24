# Pruning methods

## Scope and evidence policy

This survey covers every pruning topic and named method in `prompt.txt`, researched through 2026-11-23. `PAPER-REPORTED`
results are not reproduced here. Sparse parameter counts and checkpoint compressibility are not accepted as latency
evidence; speed, memory, and edge compatibility remain `TEST-REQUIRED`.

Pruning removes weights, structures, or computation. Its deployment value depends on the produced shape:

- **Unstructured sparsity:** arbitrary individual zeros. It can preserve model dimensions and may need sparse storage
  plus sparse GEMM kernels.
- **Semi-structured N:M sparsity:** exactly N retained (or removed, depending on convention) within each group of M.
  Always state the convention. NVIDIA's common 2:4 pattern has accelerator-specific support.
- **Structured pruning:** removes whole heads, channels, neurons, rows/columns, blocks, or layers, potentially yielding
  smaller dense tensors that ordinary kernels can accelerate.

## Lesson: zeros are not the same as less work

### Problem, intuition, and history

Pruning asks whether all learned weights or structures are needed. Early methods removed small-magnitude weights;
later methods used gradients, activations, or approximate curvature to estimate importance. The intuition is like
removing unnecessary material from a design, but deployment depends on whether the remaining structure fits the
machine. Random holes require sparse storage and kernels. Removing an entire layer or aligned channel can reduce an
ordinary dense operation.

### Mechanism and concrete example

Unstructured pruning masks individual weights. N:M pruning imposes a regular pattern, such as two retained values in
each group of four under one common convention. Structured pruning rewrites model dimensions or depth. For an Android
automotive assistant, a 50% unstructured mask may produce a compressible file while the ARM dense kernel still
multiplies the full matrices. Removing selected feed-forward channels and exporting smaller aligned tensors could reduce
work, but recovery training and conversion must preserve residual, gating, and tokenizer/output-head constraints.

### Strengths, limitations, compatibility, and decision

Magnitude pruning is cheap; Wanda adds activation sensitivity; SparseGPT uses approximate second-order
reconstruction. These are valuable sparse controls. LLM-Pruner, SliceGPT, and static layer removal offer a more
portable dense-reduction hypothesis but perform architecture surgery. Use sparse pruning only with a selected kernel
and measured break-even point. Use structured pruning when quantization and a native smaller model do not meet the
budget and the exporter accepts modified shapes. Avoid pruning before establishing a dense baseline and recovery set.

## Foundational strategy map

<!-- markdownlint-disable MD013 -->
| Strategy                   | Mechanism                                                                                                                                                                                                                                                                                                                | Main caveat and SLM/edge relevance                                                                                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Magnitude pruning          | Removes smallest absolute weights, globally or locally.                                                                                                                                                                                                                                                                  | Cheap baseline, but magnitude alone ignores activation/context sensitivity. Arbitrary zeros rarely speed mobile dense kernels.                                                      |
| Importance-based pruning   | Scores weights or structures using gradients, Hessian approximations, activations, or output change.                                                                                                                                                                                                                     | Better-informed but calibration- and implementation-sensitive. Score computation may dominate one-shot pruning cost.                                                                |
| Movement pruning           | Learns scores during fine-tuning based on movement toward/away from zero; Sanh et al., NeurIPS 2020 ([paper](https://proceedings.neurips.cc/paper/2020/hash/eae15aabaa768ae4a5993a8a4f4fa6e4-Abstract.html), [archived code](https://github.com/huggingface/transformers/tree/33eef992503689ba1af98090e26d3e98865b2a9b/examples/research_projects/movement-pruning)). | Requires training and was developed around pretrained encoders. Decoder-only applicability and actual sparse acceleration are `TEST-REQUIRED`.                                      |
| Iterative pruning          | Repeated prune/recover cycles.                                                                                                                                                                                                                                                                                           | Can preserve quality better but increases compute and selection bias. Keep one-shot and compute-matched baselines.                                                                  |
| One-shot pruning           | Calibrates and prunes once, often layer by layer.                                                                                                                                                                                                                                                                        | Operationally attractive, but recovery may still be needed and calibration choice matters.                                                                                          |
| Attention-head pruning     | Removes entire heads based on importance.                                                                                                                                                                                                                                                                                | Dense benefit requires rewriting projection shapes; grouped-query/multi-query attention (GQA/MQA), RoPE layout, tensor parallelism, and head divisibility constrain valid removals. |
| Layer/block pruning        | Removes Transformer blocks.                                                                                                                                                                                                                                                                                              | Portable dense speed/memory benefit, but depth reduction can sharply harm reasoning/generation and requires configuration/export updates.                                           |
| Channel/neuron/MLP pruning | Removes hidden or feed-forward dimensions.                                                                                                                                                                                                                                                                               | Can create smaller dense matrices; dimensions should remain aligned to target kernels. Coupled residual and gated-MLP tensors must be pruned consistently.                          |
| Pruning-aware distillation | Uses a teacher during or after pruning to recover behavior.                                                                                                                                                                                                                                                              | Adds teacher compute and loss/design complexity; relevant to domain SLMs where sparse recovery data alone is insufficient.                                                          |
<!-- markdownlint-restore -->

The strategy table moves from inexpensive scoring to more invasive shape changes. Increased sophistication can improve
selection but also increases calibration and recovery cost. Attention-head pruning is especially constrained by
Grouped-Query Attention (GQA), Multi-Query Attention (MQA), rotary position layout, and head divisibility; a logical
head mask does not necessarily create a smaller exported projection.

## Named LLM methods

<!-- markdownlint-disable MD013 MD049 -->
| Method                  | Identity, publication, and official source                                                                                                                                                                                           | Mechanism and claimed contribution                                                                                                                                                                                                                 | Limitations, maturity, compatibility, and project relevance                                                                                                                                                                                                                                                                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SparseGPT               | Frantar and Alistarh, _SparseGPT: Massive Language Models Can Be Accurately Pruned in One-Shot_, ICML 2023 ([paper](https://proceedings.mlr.press/v202/frantar23a.html), [official code](https://github.com/IST-DASLab/sparsegpt)).  | Layer-wise approximate second-order weight reconstruction; supports unstructured and N:M sparsity and a quantization combination. The paper reports at least 50% one-shot sparsity with small loss in its large-model settings (`PAPER-REPORTED`). | **Established research baseline.** Calibration required. Official code initially focuses on OPT/BLOOM; newer architectures need a maintained implementation. Unstructured checkpoint sparsity does not imply wall-clock gain. N:M acceleration depends on exact hardware/library support.                                                                                             |
| Wanda                   | Sun et al., _A Simple and Effective Pruning Approach for Large Language Models_, ICLR 2024 ([paper](https://openreview.net/forum?id=PxoFut3dWW), [official code](https://github.com/locuslab/wanda)).                                | Scores each weight by magnitude multiplied by the norm of its input activation, pruning per output without weight update. Supports unstructured and documented N:M experiments.                                                                    | **Established research baseline; simpler than SparseGPT.** Requires representative calibration activations. It produces sparse tensors, so portable speed is not automatic. Domain text can change activation scores; use domain and mixed calibration ablations.                                                                                                                     |
| LLM-Pruner              | Ma, Fang, and Wang, _LLM-Pruner: On the Structural Pruning of Large Language Models_, NeurIPS 2023 ([paper](https://arxiv.org/abs/2305.11627), [official code](https://github.com/horseee/LLM-Pruner)).                              | Discovers coupled removable structures, scores groups with gradient information, structurally removes them, then applies LoRA recovery.                                                                                                            | **Advanced practical candidate.** Official repository now documents several Llama-family/GQA updates, but also notes repetitive/nonsensical outputs in compressed models (`DOC-VERIFIED`). Recovery data and architecture-specific dependency rules are substantial variables. Dense dimension reduction is promising for edge if exported kernels accept the resulting shapes.       |
| SliceGPT                | Ashkboos et al., _SliceGPT: Compress Large Language Models by Deleting Rows and Columns_, ICLR 2024 ([paper](https://arxiv.org/abs/2401.15024), [official code](https://github.com/microsoft/TransformerCompression)).               | Applies function-preserving orthogonal transformations, then deletes low-significance rows/columns to reduce embedding dimension and replace matrices with smaller dense matrices.                                                                 | **Advanced practical candidate.** Portable dense shape reduction is its main advantage. Paper-reported quality/speed is model- and hardware-specific. Residual architecture assumptions, tied embeddings, export configuration, quantization order, and kernel-friendly dimensions require validation.                                                                                |
| ShortGPT / ShortGPT-gen | Men et al., _ShortGPT: Layers in Large Language Models are More Redundant Than You Expect_, Findings of ACL 2025 ([paper](https://aclanthology.org/2025.findings-acl.1035/), [official code](https://github.com/icip-cas/ShortGPT)). | Uses Block Influence, based on input/output hidden-state similarity, to identify low-impact layers. ShortGPT removes layers statically; ShortGPT-gen introduces generation-oriented dynamic skipping.                                              | **Recent/experimental.** Dense static layer removal is highly portable. The paper itself reports that simple static pruning degrades generative tasks and motivates ShortGPT-gen (`PAPER-REPORTED`). Dynamic skipping needs runtime control-flow support and may not map cleanly to mobile/NPU graphs. Official repository maturity is limited; independent reproduction is required. |
<!-- markdownlint-restore -->

The named-method table is best read by output structure. SparseGPT and Wanda primarily create sparse weights;
LLM-Pruner and SliceGPT aim to create smaller dense structures; ShortGPT explores depth removal and dynamic skipping.
Paper-reported quality does not establish latency because runtime representation and kernels determine execution.

## Hardware reality

1. **Unstructured zeros:** a dense tensor containing zeros has the same dense dimensions and usually the same dense
   kernel work. A sparse representation adds indices/metadata and is beneficial only above hardware- and shape-dependent
   break-even sparsity.
2. **N:M:** 2:4 can accelerate supported NVIDIA paths, but support is dtype, shape, library, and GPU-generation
   dependent. It is not a general ARM, Apple Neural Engine, Android NPU, or mobile GPU guarantee.
3. **Structured shape reduction:** deleting layers or dimensions can reduce ordinary dense work across more runtimes,
   but irregular dimensions may lose vector/tensor-core efficiency. Round candidate widths to backend-friendly multiples
   and benchmark.
4. **Storage:** generic compression can make zero-heavy files smaller without reducing runtime memory after dense
   loading. Report both on-disk encoding and resident representation.
5. **KV cache:** removing layers directly reduces per-layer KV storage; pruning MLP neurons does not. Head pruning can
   reduce KV only if key/value head dimensions are truly reduced in the exported architecture.

## Recovery and ordering

- **Recovery fine-tuning:** compare no recovery, LoRA recovery, and full recovery under equal data. Include general-data
  replay and domain data; otherwise recovery can hide catastrophic forgetting.
- **Pruning plus distillation:** use the unpruned domain model as teacher, with logits/features where
  tokenizer/architecture permits. Teacher imitation cannot restore information absent from the recovery distribution.
- **Pruning plus quantization:** structured prune first, recover, then calibrate and quantize the final shapes. For
  methods jointly modeled in the source implementation, also test the joint path. Quantizing first can corrupt
  importance estimates and may leave no tooling to rewrite compressed tensors.
- **Adapter state:** if the target artifact is merged, merge the trained domain adapter before pruning unless the
  pruning implementation explicitly understands adapter decomposition. Revalidate after every transformation.

## Initial project priority

1. **Controls:** dense baseline, global/local magnitude, and no-op conversion.
2. **Sparse research baselines:** Wanda and SparseGPT at matched unstructured and 2:4 ratios.
3. **Portable edge candidates:** SliceGPT, LLM-Pruner, and static layer removal, each with recovery.
4. **Recent branch:** ShortGPT/ShortGPT-gen only after the official implementation is pinned and generative evaluation
   is reproduced.
5. **Decision rule:** retain a method only if it improves a measured Pareto frontier for domain quality, general
   retention, artifact size, peak memory, TTFT, decode rate, energy, and sustained thermal behavior on an actual target
   runtime.

## Required experiment record

Record method/commit; base and tokenizer revisions; sparsity definition and ratio; global/local scope;
calibration/recovery dataset hashes; score precision; removed indices/shapes; alignment constraints; pruning
time/hardware; recovery steps and seed; serialized format; actual nonzero count and dense parameter count; runtime
kernel; peak memory; latency/throughput at fixed prompts; quality and confidence intervals; and conversion failures.
Mobile claims require physical-device execution. Until then, all deployment outcomes are `PLANNED EXPERIMENT`.

## Recency checks

- `DOC-VERIFIED` 2026-11-23: official repositories remain available for SparseGPT, Wanda, LLM-Pruner, SliceGPT, and
  ShortGPT.
- `TEST-REQUIRED`: current checkpoint support, especially GQA/MQA, fused projections, mixture-of-experts blocks, and
  tied embeddings.
- `TEST-REQUIRED`: sparse acceleration in current PyTorch, TensorRT-LLM, llama.cpp, ExecuTorch, LiteRT-LM, Core ML, and
  device-vendor runtimes; repository support must not be inferred from paper terminology.
