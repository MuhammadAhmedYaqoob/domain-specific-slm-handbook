# Quantization methods

## Scope and evidence policy

This survey covers all quantization concepts and named methods in `prompt.txt`. Sources were checked on 2026-11-23.
Empirical statements are labeled `PAPER-REPORTED` or `VENDOR-REPORTED`; documented interfaces are `DOC-VERIFIED`. None
is a project benchmark. Quality, memory, throughput, latency, energy, and physical-device compatibility are
`TEST-REQUIRED` on the selected checkpoint, runtime, and hardware.

Quantization reduces numerical precision. It may reduce file size and memory traffic, but speed requires kernels that
consume the encoded format efficiently. A nominal “4-bit model” also contains scales, zero points, metadata, possible
outliers, and sometimes higher-precision tensors, so effective bits per weight and runtime memory exceed the nominal
value.

## Lesson: make numbers cheaper without confusing encoding with execution

### Problem, intuition, and history

Language-model weights are usually trained with floating-point representations chosen for stable optimization, not for
cheap deployment. Quantization maps those values to fewer representable levels. Historically, 8-bit integer execution
became common in conventional neural networks; language-model outliers and very large matrices then motivated methods
such as LLM.int8(), GPTQ, Activation-Aware Weight Quantization (AWQ), SmoothQuant, and runtime-specific 4-bit formats.
The intuition is to use a coarser ruler while protecting values or layers for which rounding causes disproportionate
error.

### Mechanism and concrete example

With affine quantization, a real value is divided by a scale, rounded into an integer code, and reconstructed by the
same scale and a zero point. Per-group quantization gives different groups their own rulers, usually improving fidelity
while adding metadata. Suppose a domain model fits phone storage in 4-bit form but fails at long context. Weight
quantization has addressed only the weights; the Key-Value (KV) cache can still dominate memory. The next experiment is
therefore shorter context or separately supported cache quantization—not an invented claim that “4-bit” solves all
memory.

### Strengths, limitations, compatibility, and decision

Post-Training Quantization (PTQ) is fast and should be tried before Quantization-Aware Training (QAT). QAT is justified
when PTQ narrowly fails quality and a stable target export recipe exists. Integer 8-bit is the conservative portable
baseline; mature runtime-native 4-bit is the first aggressive edge experiment. GPTQ/AWQ are strong cloud candidates
where their exact kernels exist. GGUF is a container ecosystem, not one algorithm. NormalFloat 4 (NF4) is primarily a
QLoRA training representation, not a universal phone format. Avoid choosing by nominal bit width: qualify effective
size, resident memory, prefill, decode, quality, energy, and thermal behavior on the actual backend.

## Fundamentals and formats

- **Uniform quantization:** maps values to evenly spaced levels. A common affine form is
  $q=\operatorname{clamp}(\operatorname{round}(x/s)+z)$, with scale $s$ and zero point $z$; dequantization is
  $\hat{x}=s(q-z)$.
- **Symmetric/asymmetric:** symmetric schemes normally center levels around zero and can omit a nonzero zero point;
  asymmetric schemes fit offset ranges but add metadata/handling.
- **Granularity:** per-tensor uses one scale; per-channel and per-group use more scales, usually improving fidelity at
  metadata and kernel cost.
- **Static/dynamic:** static quantization estimates ranges before deployment from calibration data; dynamic quantization
  determines some activation ranges during execution.
- **Post-training quantization (PTQ):** quantizes an already-trained model, possibly with calibration or reconstruction.
- **Quantization-aware training (QAT):** simulates or uses quantization during training so weights adapt to quantization
  error. It costs training compute and does not guarantee target-kernel support.
- **Weight-only / weight-and-activation (W*A*) / KV-cache:** these target different memory and compute components.
  Weight-only compression does not shrink activations or KV cache.
- **Mixed precision:** leaves sensitive tensors/layers at higher precision. Its benefit depends on a runtime that
  supports the mixture without excessive conversions.

<!-- markdownlint-disable MD013 -->
| Format              | Role and caveats                                                                                                                                                                           |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| FP32                | Reference/storage/training precision; approximately 4 bytes per scalar before overhead. Usually excessive for edge inference.                                                              |
| TF32                | NVIDIA Ampere+ matrix-multiply execution mode using FP32-range inputs with reduced mantissa; not a compact checkpoint format and not a mobile target.                                      |
| FP16                | Approximately 2 bytes per scalar; broad GPU support, narrower dynamic range than BF16. Mobile support is backend-specific.                                                                 |
| BF16                | Approximately 2 bytes with FP32-like exponent range; strong modern accelerator support, but not universally fast on mobile CPUs/NPUs.                                                      |
| FP8                 | E4M3/E5M2-family 8-bit floating formats are accelerator- and framework-specific. Primarily cloud training/inference; edge portability is `TEST-REQUIRED`.                                  |
| INT8                | Mature CPU/GPU/NPU target. Weight-only and W8A8 are distinct. Per-channel weights are common; activation outliers remain important.                                                        |
| INT4 / INT3 / INT2  | Increasing compression and error sensitivity. Practical acceleration depends on packed kernels and group layout; many runtimes dequantize into higher precision during multiplication.     |
| NF4                 | Non-uniform 4-bit type introduced by QLoRA for normally distributed pretrained weights. Primarily a bitsandbytes training/storage representation, not a universal mobile execution format. |
| FP4                 | A family name, not one universal encoding. Backend-specific exponent/mantissa and block scaling must be recorded.                                                                          |
| Block/group formats | Store low-bit blocks plus scale/metadata, including many GGUF K/IQ types. Exact encoding, effective bits per weight, and kernel support matter more than a shorthand such as “Q4.”         |
<!-- markdownlint-restore -->

The format table separates arithmetic meaning from marketing shorthand. FP16 and BF16 occupy similar storage but have
different numeric ranges and hardware support. INT4, FP4, and “Q4” identify families, not interchangeable encodings.
Block formats trade metadata and reconstruction behavior against fidelity. A valid comparison records the exact type,
group size, accumulation precision, tensor exclusions, and kernel.

## Named algorithms and ecosystems

<!-- markdownlint-disable MD013 -->
| Method                  | Identity, publication, and official source                                                                                                                                                                                                                                                                        | Mechanism and claimed contribution                                                                                                                                                                            | Calibration, limitations, maturity, and edge/cloud relevance                                                                                                                                                                                                                                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LLM.int8()              | Dettmers et al., *LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale*, NeurIPS 2022 ([paper](https://arxiv.org/abs/2208.07339), [bitsandbytes](https://github.com/bitsandbytes-foundation/bitsandbytes), [Transformers docs](https://huggingface.co/docs/transformers/quantization/bitsandbytes)). | Vector-wise INT8 matmul while routing activation outlier dimensions through higher precision. Designed to preserve quality in large models with emergent outliers.                                            | **Established, GPU-centric.** No representative calibration set is central to the load-time path, but threshold behavior is model-dependent. Useful for memory-constrained cloud inference/training; not a native Android/iOS format. Current hardware support is broader than early CUDA-only releases, but exact backend performance is `TEST-REQUIRED`. |
| bitsandbytes 8-/4-bit   | Official library and current Transformers integration ([repo](https://github.com/bitsandbytes-foundation/bitsandbytes), [docs](https://huggingface.co/docs/transformers/quantization/bitsandbytes)).                                                                                                              | Provides `Linear8bitLt`, `Linear4bit`, LLM.int8(), FP4/NF4, double quantization, and low-bit optimizers. The common 4-bit path supports QLoRA-style adapter training.                                         | **Established for PEFT workflows.** `DOC-VERIFIED`: Transformers uses `BitsAndBytesConfig`. It is not itself an edge interchange format. Quantized training memory savings do not establish final mobile latency.                                                                                                                                          |
| GPTQ                    | Frantar et al., *GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers*, ICLR 2023 ([paper](https://arxiv.org/abs/2210.17323), [official code](https://github.com/IST-DASLab/gptq)).                                                                                                  | Layer-wise, approximate second-order reconstruction quantization; commonly weight-only 2–4 bit with groups and optional activation ordering.                                                                  | **Established/core cloud experiment.** Needs representative calibration activations. Multiple modern implementations/formats called “GPTQ” are not automatically binary-compatible. Strong CUDA ecosystem, but mobile runtime support is limited relative to GGUF. Quantization time and kernel choice must be recorded.                                   |
| AWQ                     | Lin et al., *AWQ: Activation-aware Weight Quantization*, MLSys 2024 Best Paper ([paper](https://arxiv.org/abs/2306.00978), [official code](https://github.com/mit-han-lab/llm-awq)).                                                                                                                              | Uses activation statistics to identify salient weights and searches per-channel scaling that protects them before low-bit weight-only quantization.                                                           | **Established/core practical.** Requires calibration data and supported model modules. Official TinyChat targets edge GPU scenarios, but this is not proof of support on arbitrary phones. CUDA/cloud tooling is comparatively mature; verify runtime-specific AWQ layouts.                                                                                |
| SmoothQuant             | Xiao et al., *SmoothQuant*, ICML 2023 ([paper](https://proceedings.mlr.press/v202/xiao23c.html), [official code](https://github.com/mit-han-lab/smoothquant)).                                                                                                                                                    | Applies an equivalent channel-wise rescaling that migrates activation quantization difficulty into weights, enabling W8A8 execution. Paper reports speed/memory gains in its server setup (`PAPER-REPORTED`). | **Established for server INT8.** Requires calibration to choose smoothing/ranges and an INT8 GEMM stack. The paper's CUTLASS/FasterTransformer path is not mobile evidence. Potentially relevant to NPU export only when the converter preserves the transform and operators.                                                                              |
| GGUF quantization       | GGUF is a model container/ecosystem used by llama.cpp, not one algorithm ([llama.cpp repo](https://github.com/ggml-org/llama.cpp), [quantization docs](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md)).                                                                              | Converts supported architectures to GGUF and packs tensors in runtime-specific types such as Q4_K_M; an importance matrix can guide mixed tensor quantization.                                                | **Core edge practical.** Strong CPU/Metal/Vulkan-oriented deployment ecosystem. Architecture conversion, tokenizer/chat metadata, and quant type must match a current llama.cpp build. Requantizing an already quantized model can degrade quality (`DOC-VERIFIED`). “GGUF” alone does not identify precision.                                             |
| HQQ                     | Badri and Shaji, *Half-Quadratic Quantization*, 2023 technical release ([official code](https://github.com/dropbox/hqq), [technical article](https://dropbox.github.io/hqq_blog/)).                                                                                                                               | Optimizes quantization parameters with a half-quadratic formulation and does not require calibration data. Supports low-bit weight-only and PEFT paths.                                                       | **Advanced practical.** Official docs distinguish quality-favored axis choices from fast-kernel-compatible layouts (`DOC-VERIFIED`). Paper/venue evidence is less conventional than GPTQ/AWQ. CUDA/Triton/ATen maturity does not imply phone support.                                                                                                      |
| AQLM                    | Egiazarian et al., *Extreme Compression of Large Language Models via Additive Quantization*, ICML 2024 ([paper](https://arxiv.org/abs/2401.06118), [official code](https://github.com/Vahe1994/AQLM)).                                                                                                            | Learns additive multi-codebook representations, including joint block optimization, aimed at extreme 2–3-bit compression.                                                                                     | **Advanced/experimental deployment.** Requires calibration and comparatively expensive optimization. Official CPU/GPU kernels exist; architecture and platform coverage is narrower than INT8/GGUF. Attractive under severe storage/RAM limits, but quantization time, effective bit rate, and mobile kernels are `TEST-REQUIRED`.                         |
| SpQR                    | Dettmers et al., *SpQR: A Sparse-Quantized Representation for Near-Lossless LLM Weight Compression*, ICLR 2024 ([paper](https://arxiv.org/abs/2306.03078), [official code](https://github.com/Vahe1994/SpQR)).                                                                                                    | Quantizes most weights to 3–4 bits while storing sensitive outliers in a sparse higher-precision representation.                                                                                              | **Advanced.** Needs calibration/second-order information and a runtime handling both dense low-bit and sparse outlier components. Paper reports near-lossless compression and GPU acceleration in its tested models (`PAPER-REPORTED`), not universal behavior. Smaller SLMs can be more sensitive; mobile kernel maturity is low.                         |
| QuIP                    | Chee et al., *QuIP: 2-Bit Quantization of Large Language Models With Guarantees*, NeurIPS 2023 ([paper](https://arxiv.org/abs/2307.13304), [official code](https://github.com/Cornell-RelaxML/QuIP)).                                                                                                             | Uses incoherence processing/rotations so weights and Hessian are easier to quantize, targeting very low bit widths with theoretical guarantees.                                                               | **Research/experimental.** Calibration and specialized encoding/kernels are required. QuIP# supersedes it empirically in its authors' lineage; retain QuIP for historical and ablation value.                                                                                                                                                              |
| QuIP#                   | Tseng et al., *QuIP#: Even Better LLM Quantization with Hadamard Incoherence and Lattice Codebooks*, ICML 2024 ([paper](https://openreview.net/forum?id=9BrydUVcoe), [official code](https://github.com/Cornell-RelaxML/quip-sharp)).                                                                             | Adds randomized Hadamard incoherence processing, E8 lattice codebooks, and inter-layer fine-tuning for extreme weight-only quantization.                                                                      | **Experimental/advanced.** Official repository states it is no longer actively developed in favor of QTIP (`DOC-VERIFIED`, checked 2026-11-23). CUDA kernels exist, but GPL-3.0 licensing and limited edge portability are material. Do not start a production mobile pipeline without a maintenance/license review.                                       |
| BitNet / native low-bit | Wang et al., *BitNet: Scaling 1-bit Transformers for Large Language Models* ([paper](https://arxiv.org/abs/2310.11453)); Ma et al., *The Era of 1-bit LLMs: All Large Language Models are in 1.58 Bits* ([paper](https://arxiv.org/abs/2402.17764)); [Microsoft bitnet.cpp](https://github.com/microsoft/BitNet). | Trains architectures with binary or ternary weights (for BitNet b1.58, values conceptually in {-1,0,1}) rather than merely PTQ-compressing a conventional dense checkpoint.                                   | **Experimental but important.** Native low-bit training changes the model/training recipe and should not be compared as a drop-in quantizer. bitnet.cpp documents CPU/NPU-oriented inference, but checkpoint availability, training reproducibility, supported architectures, and task quality require direct verification.                                |
<!-- markdownlint-restore -->

The methods table should be read as a map from failure mode to remedy. LLM.int8() protects activation outliers;
GPTQ reconstructs layers using approximate second-order information; AWQ uses activation statistics to protect salient
weights; SmoothQuant moves activation difficulty into weights for W8A8 execution. Extreme codebook and sparse-outlier
methods seek lower effective bit rates but narrow the compatible kernel set. Native BitNet-style models alter training
and architecture and therefore belong in a separate model comparison, not in a PTQ row.

## Calibration and compatibility

Calibration data should match the intended domain, language, sequence lengths, chat template, and activation
distribution without overlapping evaluation. Record its immutable snapshot and sample/token count. GPTQ, AWQ,
SmoothQuant, AQLM, SpQR, QuIP/QuIP#, importance-matrix GGUF, and static activation quantization all depend on
calibration in different ways; one calibration set is useful for control but may not be optimal for each method.

Important non-equivalences:

- 4-bit **training storage** (for example NF4 QLoRA) is not necessarily a 4-bit **inference kernel**.
- Weight-only INT4 does not imply INT4 activations or INT4 accumulation.
- Quantized file size is not peak runtime memory; include allocator, graph, activations, KV cache, tokenizer, and
  conversion buffers.
- Prefill and decode exercise kernels differently. Report both.
- KV-cache quantization is runtime/model-specific and should be evaluated separately by context length, quality,
  latency, and memory.

## Initial AWS and edge priorities

1. **Portable baseline:** BF16/FP16 where supported and INT8.
2. **Cloud GPU:** bitsandbytes for training, then compare GPTQ, AWQ, SmoothQuant/W8A8, and the serving runtime's native
   quantizer.
3. **ARM/mobile baseline:** current llama.cpp GGUF quant types, plus a runtime-native INT8/INT4 export for ExecuTorch or
   LiteRT-LM if the exact architecture is supported.
4. **Extreme compression research:** AQLM, SpQR, and QuIP# only after their target kernels and licenses fit the
   deployment.
5. **Native low-bit branch:** evaluate BitNet checkpoints as separate model candidates, not compressed copies of the
   baseline.

Current official docs confirm (`DOC-VERIFIED`, 2026-11-23) that llama.cpp converts then quantizes GGUF; ExecuTorch
exports supported LLMs to `.pte` ([docs](https://docs.pytorch.org/executorch/stable/llm/getting-started.html)); and
LiteRT Torch can export documented Hugging Face architectures to `.litertlm` with runtime-specific recipes
([docs](https://developers.google.com/edge/litert/conversion/pytorch/genai)). These are support statements, not
successful conversions in this project.

## Required experiment record

For each method: source and commit/version; base model/tokenizer revision; calibration hash and preprocessing; tensor
exclusions; group size/granularity; scale/zero-point type; nominal and effective bits per weight; quantization wall time
and hardware; artifact size; load and peak memory; TTFT; prefill/decode throughput; output equivalence checks; domain
and general quality; context lengths; energy/thermal conditions; and kernel/runtime build. All results are
`PLANNED EXPERIMENT` until raw outputs are retained.
