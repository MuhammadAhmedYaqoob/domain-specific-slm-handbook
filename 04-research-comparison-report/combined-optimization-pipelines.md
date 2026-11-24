# Combined optimization pipelines

## Purpose and evidence status

This document turns the methods in [PEFT](peft-methods.md), [quantization](quantization-methods.md),
[pruning](pruning-methods.md), and [distillation](distillation-methods.md) into testable workflows. It covers every
optimization sequence named in `prompt.txt`. These are engineering hypotheses, not measured recommendations. Every
outcome is `PLANNED EXPERIMENT` until executed with raw results.

The governing rule is **one transformation at a time, evaluate after every gate, and stop when the next transformation
leaves the useful Pareto frontier**. Combining techniques compounds numerical error, distribution shift, conversion
risk, and provenance complexity.

## Lesson: optimization order is part of the method

The problem is not merely whether techniques are individually compatible. Each stage changes the artifact consumed by
the next stage. Quantizing before structural pruning can distort importance scores and leave no supported shape-editing
path. Merging a Low-Rank Adaptation (LoRA) adapter after low-bit deployment quantization can add rounding error or fail
entirely. The historical growth of separate training and inference toolchains is why ordering must be explicit.

Use a high-precision parent as the recoverable source of truth. Adapt it, merge adapters when the release runtime
requires a dense artifact, verify parity, then apply deployment quantization. If the result is still too large or slow,
return to a fresh high-precision parent for distillation or structured pruning, recover quality, and quantize that new
architecture. Never requantize an already quantized child.

Illustrative walkthrough: a Quantized LoRA (QLoRA) automotive model fits training memory but not a target phone.
Reconstruct and merge it against the exact base, compare attached and merged outputs, and preserve the merged parent.
Create separate 8-bit and runtime-native 4-bit children. If neither meets latency, compare a native smaller model,
distilled student, and structured-pruned candidate. Stop when one passes quality, safety, memory, and latency gates;
additional compression has no product value merely because it is possible.

## Common baseline and release gates

Freeze:

- exact base model, tokenizer, chat template, license, and immutable revisions;
- domain train/validation/test and general-retention datasets;
- quantization/pruning calibration set separate from evaluation;
- prompt lengths, generation settings, seeds, trial counts, and target hardware;
- quality, safety, memory, latency, energy, thermal, artifact, and conversion metrics.

After each stage, test domain quality, general retention, safety/refusal correctness, deterministic smoke prompts, and
artifact integrity. A pipeline is deployable only if the final artifact loads and runs in the **actual** target runtime.
File-size reduction alone is not a release gate.

## Pipeline comparison

<!-- markdownlint-disable MD013 -->
| Sequence                                       | Order and rationale                                                                                                                                                             | Compatibility and cost                                                                                                                                                            | Expected deployment value and stop condition                                                                                                                                           |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Fine-tune → quantize                           | Train domain model in supported training precision; merge adapters if applicable; calibrate and PTQ the final weights; evaluate.                                                | Simple and broadly supported. Quantization can erase small fine-tuning changes, especially at very low bits. Keep the unquantized domain checkpoint.                              | **Core practical.** Real storage/runtime value when target low-bit kernels exist. Stop at the highest precision meeting device limits if lower bits leave the quality/safety frontier. |
| QLoRA → merge → quantize                       | Train adapters through an NF4/FP4 frozen base; save adapter; reconstruct/merge against the exact high-precision base; validate merged outputs; freshly quantize for deployment. | QLoRA's training representation is not automatically the deployment format. Merge may need substantial host/GPU memory. LoftQ/PiSSA variants have additional base/residual rules. | **Core practical.** Lowers training memory while allowing GGUF/runtime-native deployment. Reject if merge equivalence or provenance cannot be demonstrated.                            |
| Prune → recovery fine-tune                     | Score and prune the domain-adapted model; rewrite sparse/structured artifact; recover using domain plus general replay; evaluate.                                               | Recovery requires training compute. Unstructured pruning needs sparse kernels; structured pruning needs architecture-aware shape edits.                                           | **Advanced practical.** Prefer structured candidates for portable edge gains. Stop if wall-clock latency/memory does not improve on target hardware.                                   |
| Prune → distill                                | Prune student, then use the original unpruned domain model as teacher during recovery.                                                                                          | Logit/feature compatibility depends on tokenizer and shapes. Teacher execution raises cost. A response-only teacher can still supply validated sequences.                         | Can recover behavior better than hard labels (`HYPOTHESIS`). Compare against equal-compute recovery SFT.                                                                               |
| Distill → quantize                             | Distill into a smaller dense student; select checkpoint; calibrate and quantize it.                                                                                             | Quantizer support must be checked for the student architecture. Smaller models can be more quantization-sensitive than large teachers.                                            | **Core edge candidate.** Architectural reduction plus low-bit storage can be portable. Compare to a native small model of equal size.                                                  |
| Distill → prune → recovery                     | Distill to a deployable student; prune it; recover against expert data and/or teacher.                                                                                          | Three training/evaluation stages and compounded loss. Student redundancy may be lower, leaving less safe pruning headroom.                                                        | Research branch for budgets not met by distill→quantize. Stop if pruning does not improve measured runtime.                                                                            |
| Compression → QAT                              | Produce final pruned/distilled architecture; initialize quantization-aware training for the exact target recipe; recover; export.                                               | QAT fake-quant operators and backend quantization specs must match export. Training cost and version coupling are high.                                                           | Useful when PTQ misses quality gates and the runtime has a stable QAT path. Do not run generic QAT with no target kernel/export.                                                       |
| Structured prune → quantize                    | Remove layers/dimensions; recover; calibrate the resulting shapes; quantize.                                                                                                    | Pruned dimensions must satisfy packing/vector alignment. Some quantizers assume canonical architecture shapes.                                                                    | **Advanced practical.** Potentially portable dense and precision reduction. Verify converter metadata and kernels before expensive recovery.                                           |
| Native small model vs compressed large model   | Train/adapt a native SLM and compare with distilled/pruned larger-origin candidates under the same artifact/RAM budget.                                                         | Pretraining data and architecture differ, so parameter count alone is not a controlled variable.                                                                                  | Mandatory decision branch. Native models have simpler tooling; compressed students may inherit stronger capabilities. Neither is presumed superior.                                    |
| Mixed precision / layer-sensitive quantization | Profile quantization error; keep embeddings, output head, or sensitive blocks at higher precision; quantize others.                                                             | Runtime must support mixed tensor types without conversion overhead. Calibration can overfit.                                                                                     | Useful when uniform low-bit fails. Count actual effective bits and kernel transitions.                                                                                                 |
| Joint pruning + distillation                   | Learn masks/structures while matching a teacher, or alternate prune and KD updates.                                                                                             | Highest implementation complexity; loss weights, schedules, and mask constraints interact. Sparse deployment still needs kernels.                                                 | Research only until sequential baselines are strong. Require an ablation proving the joint formulation adds value.                                                                     |
| Combined training/inference optimization       | Select architecture, attention/KV design, PEFT, compression, runtime, context cap, threading/offload, and packaging jointly.                                                    | Risk of changing too many variables. Use staged elimination and retain per-stage controls.                                                                                        | Final production optimization. Runtime settings can dominate theoretical compression; benchmark sustained use, not only first run.                                                     |
<!-- markdownlint-restore -->

The pipeline table is a causal map. Core paths have simple rollback and broad tool support. Advanced paths can reduce
dense architecture cost but add recovery and converter constraints. Research paths combine several interacting losses
or masks and therefore require strong sequential baselines. “Expected deployment value” states a hypothesis and a stop
condition, not a benchmark result.

## Recommended staged funnel

### Stage 0: feasibility

Run the untouched model in each candidate cloud/edge runtime. Record conversion failures before training. A model
unsupported by the intended mobile stack should not become the sole baseline.

### Stage 1: domain adaptation

Compare SFT with LoRA and QLoRA; include full fine-tuning only where feasible. Select by domain/general quality and
training resource use, not by trainable-parameter count alone.

### Stage 2: first deployment quantization

Merge the selected adapter and compare a high-precision artifact, INT8, and one mature 4-bit runtime-native format. This
establishes whether quantization alone meets the device budget.

### Stage 3: architecture reduction

If the model remains too slow or large, compare:

- a natively smaller pretrained checkpoint;
- a distilled dense student;
- structured pruning with recovery;
- unstructured/N:M pruning only on hardware with a verified matching sparse kernel.

### Stage 4: second compression

Quantize the best architecture-reduced candidates from fresh high-precision recovered checkpoints. Do not requantize a
quantized artifact. Consider target-specific QAT only when PTQ fails quality gates.

### Stage 5: runtime tuning

Tune context cap, KV-cache precision where supported, prefill/decode threads, CPU/GPU/NPU partition, memory mapping, and
model packaging. Keep model weights fixed while tuning runtime variables.

## Compatibility matrix

<!-- markdownlint-disable MD013 -->
| Combination                                 | Default assessment           | Reason                                                                                                                   |
| ------------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| LoRA merge + PTQ                            | Compatible with verification | Merge additive updates first; quantizer sees final dense weights.                                                        |
| QLoRA checkpoint directly to mobile         | Not assumed                  | bitsandbytes NF4 training representation may not be accepted by mobile runtime.                                          |
| LoftQ + bitsandbytes QLoRA                  | Documented narrow path       | Current PEFT docs specify linear targets and particular safetensors/4-bit workflows; final edge format remains separate. |
| Unstructured prune + ordinary dense runtime | Usually no speed benefit     | Dense kernels still process full shapes; serialized zeros may only compress on disk.                                     |
| 2:4 prune + NVIDIA sparse kernel            | Potentially compatible       | Requires supported GPU generation, dtype, dimensions, library, and invocation.                                           |
| Structured prune + ordinary dense runtime   | Potentially portable         | Smaller matrices/layer count use dense kernels, subject to converter and alignment support.                              |
| Feature KD across different tokenizers      | High risk                    | Token positions and vocabularies do not align directly. Response KD is simpler.                                          |
| Distill then PTQ                            | Generally compatible         | Quantize the final student; smaller students may be more sensitive.                                                      |
| PTQ then structural prune                   | Avoid by default             | Low-bit weights distort scores and shape rewriting often lacks support.                                                  |
| PTQ then adapter merge                      | Avoid by default             | In-place low-bit merge is backend-specific and may cause extra error.                                                    |
| Prune then QAT                              | Target-dependent             | QAT must preserve masks/shapes and export to the exact runtime.                                                          |
| GGUF quantization + non-llama.cpp runtime   | Not assumed                  | GGUF container/types require runtime support; format name alone is insufficient.                                         |
<!-- markdownlint-restore -->

The compatibility table gives defaults, not universal laws. A combination marked “potentially compatible” still depends
on exact architecture, group size, tensor alignment, runtime, and hardware. A combination marked “avoid by default” may
be technically possible in one backend, but it increases error or removes a clean attribution path and therefore needs
a specific justification.

## Practical experiment matrix

Use successive halving rather than a full Cartesian product:

1. **Adaptation screen:** base, LoRA, QLoRA, and feasible full FT.
2. **Quantization screen:** winner(s) at high precision, INT8, and two supported 4-bit recipes.
3. **Architecture screen:** native smaller model, distilled student, SliceGPT/LLM-Pruner candidate, and one layer-pruned
   candidate.
4. **Combination screen:** architecture winners at INT8 and selected 4-bit.
5. **Physical-device finalists:** no more than three artifacts per target class, tested with repeated cold/warm runs and
   sustained generation.

Primary endpoints: domain correctness and safety gates. Secondary Pareto axes: peak resident memory, artifact/download
size, startup, TTFT, prefill, decode, energy per token where measurable, thermal throttling, and engineering/maintenance
cost.

## Failure attribution

Retain checkpoints after adaptation, merge, pruning, recovery/distillation, and quantization. If quality drops:

- compare adapter and merged outputs to isolate merge error;
- compare pre/post-prune in high precision to isolate structural loss;
- compare recovered high precision to isolate recovery;
- compare recovered and quantized outputs to isolate quantization;
- compare framework and target-runtime outputs to isolate conversion/kernel error.

Do not tune on the held-out domain test set. A calibration set may guide quantization/pruning but must not be used to
report final quality.

## Hardware recommendations

- **Training/adaptation:** CUDA GPU where bitsandbytes/PEFT paths are used; CPU offload only when measured and
  reproducible.
- **Cloud inference:** benchmark the exact AWS GPU/CPU and serving runtime; GPTQ/AWQ/W8A8/sparsity support differs.
- **ARM/mobile CPU:** prioritize supported dense small models and GGUF/runtime-native low-bit kernels.
- **Mobile GPU/NPU:** use only converter-supported architectures and quantization recipes; paper support for “edge” is
  insufficient.
- **Apple:** test current Metal/Core ML/ExecuTorch paths on named devices; do not infer Apple Neural Engine execution
  from successful conversion.

## Recency and test-required register

Checked 2026-11-23 against current PEFT, Transformers/bitsandbytes, llama.cpp, ExecuTorch, and LiteRT documentation plus
the original method sources cited in the companion surveys. Before execution, pin current releases/commits and re-check:

- exact model architecture export support;
- quantized adapter merge semantics;
- sparse kernel availability and invocation;
- supported quantization type/group/alignment;
- tokenizer and chat-template preservation;
- runtime KV-cache precision;
- mobile GPU/NPU delegation rather than silent CPU fallback;
- model/API licenses and synthetic-output terms.

No benchmark, physical-device run, successful conversion, or quality-retention claim is made here.
