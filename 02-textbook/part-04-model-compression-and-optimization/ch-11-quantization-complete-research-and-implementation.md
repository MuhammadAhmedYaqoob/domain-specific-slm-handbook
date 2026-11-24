# CH-11: Quantization — Complete Research and Implementation

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

## 11.1 Purpose, scope, and evidence boundary

Quantization represents model tensors with fewer bits or with a more useful distribution of representable values. The
engineering objective is not merely a smaller checkpoint: it is an acceptable quality–memory–latency–energy operating
point on a named runtime and device. Quantization can reduce storage and memory bandwidth, but it accelerates inference
only when kernels consume the encoded representation efficiently. A “4-bit model” can therefore be smaller yet slower
than an FP16 model that uses highly optimized matrix-multiplication kernels.

This chapter covers historical scalar quantization, modern large-language-model (LLM) post-training quantization (PTQ),
quantization-aware training (QAT), native low-bit architectures, server GPUs, and ARM/mobile deployment. It assumes the
memory model introduced in [CH-10](ch-10-the-fundamentals-of-model-compression.md) and follows the project
[evidence standard](../../00-governance/evidence-and-citation-standard.md).

**Evidence status, 2026-11-23.** Research and official interfaces cited here are `PAPER-REPORTED` or `DOC-VERIFIED`. No
code listing in this chapter was executed, no model was quantized, and no AWS or physical edge benchmark was run for
this chapter. Every listing is `UNVERIFIED IMPLEMENTATION`; every proposed measurement and graph is a
`PLANNED EXPERIMENT`. Reported paper results are not project results.

## 11.2 The problem quantization solves

A dense model with $N$ parameters stored at $b$ bits per parameter has an idealized weight payload

$$
M_\text{weights}=\frac{Nb}{8}\ \text{bytes}.
$$

For an illustrative 1.5-billion-parameter model, payloads are 6.0 GB at FP32, 3.0 GB at FP16/BF16, 1.5 GB at INT8, and
0.75 GB at a nominal 4 bits. These are decimal, idealized values—not file sizes or resident memory. Scales, zero points,
codebooks, alignment, tensor metadata, embeddings left at higher precision, runtime workspaces, activations, and the
key–value (KV) cache add overhead. A model may fit storage but fail to load into available random-access memory (RAM),
or fit RAM yet decode too slowly or overheat.

Quantization addresses three related costs:

1. **Storage and transfer:** fewer bytes to publish, download, verify, and map.
2. **Memory capacity and bandwidth:** fewer weight and possibly activation/KV bytes moved per operation.
3. **Arithmetic:** integer or low-precision floating-point kernels may execute more operations per cycle and use less
   energy.

The third benefit is conditional. Weight-only formats often unpack or dequantize blocks into FP16/BF16 registers before
multiply-accumulate. They can still win by reducing memory traffic, especially in single-user decode, but nominal bit
width does not predict speed by itself.

## 11.3 Mathematical foundations

### 11.3.1 Uniform affine quantization

Uniform quantization places reconstruction levels at equal intervals. For a real value $x$, scale $s>0$, integer
zero point $z$, and integer range $[q_{\min},q_{\max}]$,

$$
q=\operatorname{clamp}\left(\operatorname{round}\left(\frac{x}{s}\right)+z,\ q_{\min},q_{\max}\right), \qquad
\hat{x}=s(q-z).
$$

Here $q$ is the stored code and $\hat{x}$ is the reconstructed approximation. Quantization error is $e=x-\hat{x}$.
For a non-clipped value rounded to its nearest level, $|e|\le s/2$. Assuming error is uniformly distributed within a
bin—an approximation that often fails for correlated weights—the mean squared error (MSE) is $s^2/12$.

For asymmetric unsigned $b$-bit min–max quantization:

$$
s=\frac{x_{\max}-x_{\min}}{2^b-1},\qquad
z=\operatorname{clamp}\left(\operatorname{round}\left(q_{\min}-\frac{x_{\min}}{s}\right),q_{\min},q_{\max}\right).
$$

**Numerical example.** Quantize $[-1.0,-0.2,0.3,1.4]$ to unsigned 3-bit codes $0\ldots7$. Then
$s=(1.4-(-1.0))/7=0.342857$, and $z=\operatorname{round}(2.9167)=3$. The codes are approximately $[0,2,4,7]$,
reconstructing $[-1.0286,-0.3429,0.3429,1.3714]$. The nonzero zero point uses the code space efficiently for the
offset range, but $-0.2$ incurs a visible error.

### 11.3.2 Symmetric versus asymmetric

Symmetric quantization uses a range centered on zero, normally $z=0$, such as signed INT8 $[-127,127]$:

$$
s=\frac{\max(|x_{\min}|,|x_{\max}|)}{127}.
$$

It simplifies kernels and exactly represents zero. It can waste codes when the distribution is strongly shifted.
Asymmetric quantization fits both endpoints and is often useful for nonnegative or skewed activations, but requires
zero-point correction and metadata. “Symmetric” does not by itself imply signed storage, nor does “asymmetric” imply
higher accuracy on every tensor.

**Numerical example.** For $[-1,3]$, symmetric signed 3-bit quantization using $[-3,3]$ has $s=1$, while an
asymmetric 3-bit range has $s=4/7$. Asymmetric quantization offers finer spacing over this skewed interval. If the
target kernel only accelerates symmetric operands, however, the numerically attractive representation may lose system
performance.

### 11.3.3 Non-uniform and floating formats

Non-uniform quantization allocates levels unevenly. A codebook maps each low-bit index to a learned or fixed
reconstruction value:

$$
q=\arg\min_{k\in\{0,\ldots,K-1\}} |x-c_k|,\qquad \hat{x}=c_q.
$$

NormalFloat 4 (NF4) uses levels designed for approximately normal pretrained-weight distributions. Additive quantization
reconstructs a vector from a sum of codewords from multiple codebooks. Low-bit floating formats allocate bits among
sign, exponent, and mantissa and therefore give non-uniform spacing with dynamic range. These formats can better match a
distribution but need specialized packing, lookup, scaling, and kernels.

### 11.3.4 Clipping and range selection

Min–max scaling can devote most levels to rare outliers. Clipping to $[a,b]$ introduces saturation error outside the
interval but reduces rounding error inside it. Calibration searches or estimates a range using MSE, percentile,
entropy/Kullback–Leibler divergence, or task-aware reconstruction objectives. The best local tensor MSE is not
guaranteed to maximize end-task quality.

<a id="fig-08"></a> **FIG-08 — Quantization error and calibration (`ILLUSTRATIVE`).**

```text
Observed values
     │
     ├── rare outliers ─────────────────────────────────┐
     │                                                  │
     ▼                                                  ▼
[collect histogram/statistics] → [choose clip range] → [derive scale / zero point]
                                      │
                                      ▼
real x → divide by scale → round/codebook lookup → clamp → packed q
                                                        │
                                                        ▼
                             kernel dequantizes/uses q → reconstructed x̂
                                                        │
                                                        ▼
                                      local error e=x-x̂ → layer/output error
```

_Caption and alt text:_ Calibration converts representative tensor distributions into clipping and encoding parameters; <!-- markdownlint-disable-line MD013 MD049 -->
outliers create a trade-off between clipping and coarse bins. This is an educational flow, not measured data. Units
depend on the tensor.

## 11.4 Granularity: where scales apply

<!-- markdownlint-disable MD013 -->
| Granularity     | Parameters shared by                           | Fidelity and overhead                                                           | Typical use                                 |
| --------------- | ---------------------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------- |
| Per-tensor      | entire tensor                                  | Lowest metadata and simplest kernels; one outlier can enlarge every bin         | Activations or older deployment stacks      |
| Per-channel     | one output/input channel, as defined by layout | Better tracks channel ranges; more scales                                       | INT8 linear/conv weights; AWQ-style scaling |
| Per-group/block | contiguous group, often 32–256 weights         | Strong low-bit fidelity; scale/zero-point overhead and layout-sensitive kernels | INT4/INT3/INT2 weight-only formats          |
| Per-token       | activation vector for each token               | Adapts to token-dependent magnitude; runtime reduction overhead                 | Dynamic activation quantization             |
| Per-row/vector  | matrix row or vector                           | Balances adaptation and kernel structure                                        | LLM.int8 vector-wise operations             |
<!-- markdownlint-restore -->

If group size is $G$, a 4-bit group with one FP16 scale has effective storage near $4+16/G$ bits per weight before
alignment and other metadata. At $G=32$, that is at least 4.5 bits; at $G=128$, 4.125 bits. Adding a zero point or
double-quantized scale changes the result. Always report **effective bits per weight**, not only nominal data bits.

Granularity is semantic as well as numeric. A scale axis incompatible with the packed kernel may require repacking,
fallback, or a slower generic kernel. HQQ documentation, for example, distinguishes quality-favored axis choices from
fast-kernel-compatible layouts (`DOC-VERIFIED`).

## 11.5 When quantization parameters are determined

### Static and dynamic quantization

**Static quantization** fixes weight and activation parameters before inference using calibration data. It removes most
online range-selection work and supports integer-only graphs, but distribution shift can cause clipping. **Dynamic
quantization** stores quantized weights while determining activation scales during inference, often per token or batch.
It adapts to inputs but adds reductions, scaling, and conversion overhead. “Dynamic” is not synonymous with PTQ: dynamic
range selection can be part of a PTQ runtime recipe.

### PTQ

PTQ starts from a trained checkpoint. Basic PTQ rounds independently; advanced PTQ reconstructs layer outputs, uses
approximate Hessian information, rescales channels, rotates features, protects outliers, or optimizes codebooks. PTQ is
the usual first production experiment because it avoids retraining, but calibration-sensitive methods still consume GPU
memory and hours of wall time at larger scales.

### QAT

QAT exposes the model to quantization during training. A fake-quantizer commonly computes quantized/dequantized values
in the forward pass while the straight-through estimator approximates gradients through non-differentiable rounding:

$$
\frac{\partial\,\operatorname{round}(x)}{\partial x}\approx 1 \quad\text{inside the unclipped range.}
$$

QAT can recover quality at INT8 and difficult low-bit settings and can learn clipping/scales. It requires training data,
optimizer state, careful schedules, and an exact target format. Training with a simulated format that the deployment
runtime packs differently is not a deployment result. Full-model QAT is costly; low-rank QAT and quantization-aware
adapters are practical research branches.

## 11.6 What is quantized

### Weight-only quantization

Weights are packed to low precision; activations and accumulation usually remain FP16/BF16/FP32. Weight-only INT4 is
attractive for memory-bandwidth-bound decode. It does not reduce activation or KV-cache memory and may not accelerate
high-batch prefill.

### Weight-and-activation quantization

Notation such as W8A8 means 8-bit weights and activations; it does not specify signedness, granularity, accumulation
precision, clipping, or kernel. W8A8 can turn matrix multiplication into native integer/FP8 accelerator operations and
is often more relevant to server throughput or NPUs. Activation outliers are the main difficulty. W4A16 is weight-only
in practical effect; W4A8, W4A4, and lower combinations are increasingly hardware- and method-specific.

### Activation-only quantization

Activation-only compression is uncommon as the primary model format because weights dominate storage and are reused, but
activation quantization is useful for communication, temporary-memory reduction, and accelerator graph requirements.
Dynamic per-token INT8/FP8 activation quantization is a current server strategy.

### KV-cache quantization

For $L$ layers, batch $B$, sequence length $T$, $H_{kv}$ KV heads, head dimension $D$, and $b$ bits per
cache scalar, the ideal cache payload is

$$
M_{KV}=2LBTH_{kv}D\frac{b}{8}.
$$

The factor 2 is for keys and values. With $L=24$, $B=1$, $T=4096$, $H_{kv}=8$, $D=128$, an FP16 cache is
$2\times24\times4096\times8\times128\times2=402{,}653{,}184$ bytes (384 MiB). INT8 ideally halves it; 4-bit ideally
quarters it. Metadata, alignment, cache pages, and temporary dequantization add overhead.

Keys and values need not tolerate the same precision. Cache quantization can change long-context quality and add decode
overhead, so test it independently at several context lengths. A weight-quantized result says nothing about KV
precision.

### Mixed precision and sensitive layers

Mixed precision retains embeddings, normalization, output heads, selected transformer blocks, outlier channels, or KV
tensors at higher precision. Sensitivity can be estimated by reconstruction error, Hessian approximations, activation
statistics, or ablation. Mixed precision usually improves quality but complicates kernels and may create conversion
boundaries. The artifact manifest must list every exclusion.

## 11.7 Error propagation, outliers, and model quality

For a linear layer $y=Wx$, quantized $\hat W=W+\Delta W$, and $\hat x=x+\Delta x$,

$$
\hat y-y=\Delta W x+W\Delta x+\Delta W\Delta x.
$$

The first term is weight error amplified by the input, the second activation error amplified by the original weight, and
the third their interaction. Residual additions can preserve or compound errors; attention softmax can magnify changes
that reorder close logits; layer normalization can partially rescale errors but is not a cure. Errors are structured and
input-dependent, not independent white noise.

Outliers appear as unusually large weights, activation channels, or token-dependent values. At low bits they either
consume range or clip. Major responses include:

- route outlier dimensions through higher precision (LLM.int8());
- rescale channels to move difficulty between activations and weights (SmoothQuant, AWQ);
- preserve sparse sensitive weights (SpQR);
- use rotations/incoherence processing to spread outliers (QuIP, QuIP#, SpinQuant);
- optimize reconstruction with approximate second-order information (GPTQ);
- increase precision or reduce group size for sensitive layers.

Quality must be measured at several levels: weight/output reconstruction, validation cross-entropy and perplexity,
general tasks, domain factual and reasoning tasks, calibration/refusal behavior, and long-context tests. A small
perplexity change can conceal a safety-critical regression. For automotive or pharmacy models, quantized candidates must
pass the same expert-reviewed safety suite as the source model; quantization cannot justify relaxed release thresholds.

## 11.8 Precision and encoding formats

<a id="fig-07"></a> **FIG-07 — Precision memory comparison (`ESTIMATE`).** Alt text: for a fixed parameter count, ideal
payload grows linearly with bits per parameter from lower-bit formats through INT4, INT8, FP16/BF16, and FP32; scales,
zero points, alignment, codebooks, and runtime buffers are separate overhead. Formula: ideal payload bytes = parameter
count × nominal bits ÷ 8. Not measured; actual artifacts and runtime memory require target-specific measurement.

```text
fixed parameter count P
  lower-bit → INT4 → INT8 → FP16/BF16 → FP32
  P×b/8 ideal payload bytes, increasing with nominal bit width b
  + separately reported scales / zero-points / alignment / codebooks / runtime buffers
```

<a id="tbl-06"></a> **TBL-06 — Quantization formats. (`ESTIMATE / DOC-VERIFIED`)** Sizes are ideal payloads. Performance
and compatibility are target-specific.

<!-- markdownlint-disable MD013 -->
| Format                        |                                                                              Payload | Numeric character                         | Main engineering role                                 | Critical caveat                                  |
| ----------------------------- | -----------------------------------------------------------------------------------: | ----------------------------------------- | ----------------------------------------------------- | ------------------------------------------------ |
| FP32                          |                                                                              32 bits | IEEE floating point                       | Reference inference, training, sensitive operations   | High storage/bandwidth                           |
| TF32                          | 32-bit storage/input with reduced multiply mantissa on supported NVIDIA Tensor Cores | Execution mode, FP32 exponent range       | NVIDIA Ampere-or-newer matrix math                    | Not a compressed checkpoint or mobile format     |
| FP16                          |                                                                              16 bits | 5-bit exponent, 10-bit fraction           | Common GPU inference/training and accumulation target | Narrower exponent range than BF16                |
| BF16                          |                                                                              16 bits | 8-bit exponent, 7-bit fraction            | Stable training and modern accelerator inference      | Mobile CPU/NPU speed is backend-specific         |
| FP8 E4M3/E5M2 families        |                                             8 bits plus scaling in practical systems | Range/precision trade-off                 | Modern cloud accelerators; W8A8-style paths           | Encoding and scaling recipe are not universal    |
| INT8                          |                                              8 bits plus scales/optional zero points | Uniform integer                           | Mature CPU/GPU/NPU W8A8 or weight-only                | “INT8” omits granularity and kernel              |
| INT4                          |                                                           4 bits plus group metadata | Uniform integer                           | Core weight-only compression; some NPU/server paths   | Often unpacked/dequantized; effective rate >4    |
| INT3/INT2                     |                                                               3/2 bits plus metadata | Uniform integer                           | Extreme weight-only research/deployment               | Packing, quality, and kernels become restrictive |
| NF4                           |                                                     4-bit codebook plus block scales | Non-uniform, normal-distribution-oriented | QLoRA base-weight storage/training                    | Not a universal mobile inference type            |
| FP4                           |                                    Usually sign/exponent/mantissa with block scaling | Non-uniform floating family               | Emerging accelerator-specific inference/training      | “FP4” does not identify one encoding             |
| GGUF Q/K/IQ block types       |                                                               Mixed block structures | Runtime-defined integer/codebook formats  | llama.cpp CPU/Metal/Vulkan ecosystem                  | Record exact type, build, and importance matrix  |
| Additive/lattice codebooks    |                                                             Often 2–4 effective bits | Vector/codebook reconstruction            | AQLM, QuIP# and related extreme compression           | Specialized encoding and kernels                 |
| Binary/ternary native weights |                                     1 bit or ~1.58-bit ternary alphabet conceptually | Native training constraint                | BitNet-family architectures                           | Not drop-in PTQ of an ordinary checkpoint        |
<!-- markdownlint-restore -->

FP8 and FP4 names require a complete encoding record: exponent/mantissa allocation, finite-only/special-value behavior,
block size, scale precision, scale update rule, accumulation type, and hardware. Likewise, sub-4-bit claims must include
metadata in effective bit rate.

## 11.9 Major methods and ecosystems

### Historical progression

Classical neural-network quantization established fixed-point INT8 inference, calibration, and QAT for convolutional and
recurrent networks. Transformer scale and activation outliers motivated vector-wise mixed precision, channel smoothing,
layer reconstruction, groupwise weight-only formats, rotations, and learned codebooks. Current practice is an ecosystem
of algorithm + serializer + kernel + runtime, not an interchangeable “quantize” operation.

<a id="tbl-07"></a> **TBL-07 — Quantization algorithms: identity, mechanism, and evidence.**

<!-- markdownlint-disable MD013 -->
| Method                    | Core mechanism                                                                                                | Calibration/training                                                                     | Credible role and limitations                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| LLM.int8()                | Vector-wise INT8 matmul with outlier dimensions computed at higher precision                                  | No representative calibration set in common load-time path; threshold is model-dependent | Established GPU-centric 8-bit memory path. Not a native phone artifact. Dettmers et al., NeurIPS 2022 (`PAPER-REPORTED`)           |
| bitsandbytes 8/4-bit      | `Linear8bitLt`, `Linear4bit`, LLM.int8, NF4/FP4, nested/double quantization                                   | Load-time conversion; QLoRA trains adapters over frozen 4-bit weights                    | Established PEFT ecosystem; not an edge interchange format. NF4 inference speed is not implied by QLoRA memory savings             |
| GPTQ                      | Layerwise approximate second-order reconstruction with error compensation                                     | Representative calibration activations; potentially substantial quantization time        | Established CUDA/cloud weight-only 2–4-bit family; implementations/layouts called GPTQ are not automatically compatible            |
| AWQ                       | Activation-aware channel scaling protects salient weights before groupwise low-bit quantization               | Small representative calibration set and scale search                                    | Established 4-bit cloud/edge-GPU method. TinyChat support is not arbitrary-phone support                                           |
| SmoothQuant               | Equivalent channel scaling moves activation difficulty into weights                                           | Calibration for smoothing/ranges                                                         | Established W8A8 server method. Needs integer kernels; server CUTLASS evidence does not establish mobile performance               |
| GGUF/llama.cpp quantizers | Convert supported architecture to GGUF, then pack tensors in exact Q\*/K/IQ types; optional importance matrix | Some types need no data; importance-matrix path uses representative data                 | Core ARM/mobile CPU baseline. GGUF is a container, not one algorithm; never report “GGUF 4-bit” without exact type                 |
| HQQ                       | Half-quadratic optimization of quantization parameters                                                        | No calibration dataset required                                                          | Advanced practical fast quantization; kernel-compatible axes matter; mobile kernels remain limited                                 |
| AQLM                      | Additive multi-codebook representation and joint block optimization                                           | Calibration plus expensive optimization                                                  | Credible extreme 2–3-bit research; narrower kernels/architecture support                                                           |
| SpQR                      | Quantize most weights and retain sensitive outliers sparsely at high precision                                | Calibration and approximate second-order information                                     | Near-lossless claim is paper-reported; sparse+dense kernel and metadata complicate mobile deployment                               |
| QuIP                      | Incoherence processing plus second-order quantization                                                         | Calibration and specialized encoding                                                     | Important 2-bit research with guarantees; superseded empirically in authors’ lineage                                               |
| QuIP#                     | Hadamard incoherence, E8 lattice codebooks, inter-layer tuning                                                | Calibration, optimization, CUDA kernels                                                  | Advanced experimental. Official repository states development moved toward QTIP; GPL-3.0 license is material                       |
| QTIP                      | Trellis-coded quantization with incoherence processing                                                        | Calibration/optimization; specialized kernels                                            | Current credible successor research in this lineage; production and mobile maturity remain `TEST-REQUIRED`                         |
| OmniQuant                 | Learnable weight clipping and equivalent transformations                                                      | Blockwise calibration optimization                                                       | Reproducible low-bit PTQ research; conversion time and runtime format remain implementation-specific                               |
| AutoRound                 | Sign-gradient-based rounding optimization                                                                     | Representative calibration samples and short optimization                                | Practical low-bit PTQ candidate in Intel/Transformers ecosystems; validate exact export/kernel path                                |
| SpinQuant                 | Learned rotations reduce outlier effects before quantization                                                  | Rotation optimization/training                                                           | Credible rotation-based research; deployment needs fused or efficiently folded transforms                                          |
| torchao/Quanto ecosystems | PyTorch-native quantizer, tensor subclasses/export integrations, multiple low-bit recipes                     | Recipe-dependent                                                                         | Useful modern implementation layers, not single algorithms. API/backend support must be pinned                                     |
| BitNet / BitNet b1.58     | Train binary/ternary-weight transformers natively                                                             | Training from scratch or a compatible native recipe                                      | Separate architecture candidate, not PTQ. bitnet.cpp documents CPU/NPU-oriented execution; checkpoint/task evidence must be tested |
<!-- markdownlint-restore -->

### Method-by-method deployment characteristics

**TBL-CH11-03 — Required operational comparison. (`STATICALLY REVIEWED`)** “Quality” summarizes the intended mechanism,
not a guaranteed result. `Measure` means no portable result exists.

<!-- markdownlint-disable MD013 -->
| Method               | Quality / storage                                                                   | Runtime memory                                                    | Calibration and quantization time                                  | Latency / throughput                                                               | Hardware, kernels, mobile                                                              | Maturity                                     |
| -------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | -------------------------------------------- |
| LLM.int8()           | Designed to retain quality versus naive INT8; ~8-bit weights plus outlier path      | Lower weight memory; activations/outliers remain higher precision | No dataset; load conversion/time must be measured                  | Can be slower for smaller models or large outlier traffic; measure prefill/decode  | bitsandbytes CUDA and documented additional backends; no standard Android/iOS artifact | Established cloud/training                   |
| bitsandbytes NF4/FP4 | Strong QLoRA storage use; quality depends on model and compute dtype                | 4-bit frozen weights, adapters and dequant buffers                | No task calibration; load/convert time                             | Inference can be limited by dequant kernels; not evidence of mobile speed          | GPU/host backend-specific library kernels                                              | Established PEFT, not mobile serving         |
| GPTQ                 | Often strong 4-bit reconstruction; 3/2-bit more sensitive; group metadata           | Low packed-weight memory plus workspace/KV                        | Calibration required; second-order pass can be costly              | Fast only with compatible Marlin/ExLlama/Triton-style kernels/layouts              | Primarily CUDA; weak general-phone portability                                         | Established cloud, fragmented formats        |
| AWQ                  | Protects activation-salient weights; 4-bit target common                            | Similar groupwise INT4 overhead                                   | Calibration and scale search; usually lighter than full retraining | Strong with AWQ-aware GEMM/TinyChat/vLLM-style kernels; measure                    | CUDA and selected edge GPU paths; not universal CPU/NPU                                | Established practical                        |
| SmoothQuant          | Targets W8A8 while retaining model behavior through exact rescaling before rounding | Reduces weights and activations where graph remains INT8          | Calibration for $\alpha$/ranges; moderate transform cost         | Can improve batched server throughput with INT8 Tensor Core/CUTLASS/TensorRT paths | Server GPU/CPU; mobile NPU only after supported export                                 | Mature server INT8                           |
| GGUF Q\*/K/IQ        | Quality varies materially by exact tensor mix, block type, and importance matrix    | mmap may lower load copies; KV/runtime separate                   | Conversion usually practical; importance matrix adds calibration   | Mature CPU/Metal/Vulkan kernels; speed is type/device-specific                     | Strong ARM CPU/mobile baseline; GPU offload varies                                     | High practical maturity in llama.cpp         |
| HQQ                  | Low-bit quality through optimized scales/zeros; no universal retention claim        | Packed weights plus dequant workspace                             | No calibration; fast-quantization objective, measure wall time     | ATen/Triton/custom kernel dependent                                                | GPU/CPU integrations; limited phone-native path                                        | Advanced practical                           |
| AQLM                 | Designed for high quality at extreme effective bit rates                            | Very low weights; codebooks and decode workspace                  | Calibration and expensive codebook optimization                    | Specialized CUDA/CPU kernels; throughput not portable                              | Narrower hardware and no mature general mobile path                                    | Advanced/experimental                        |
| SpQR                 | Preserves sensitive outliers; sparse metadata raises effective rate                 | Low-bit dense + sparse high-precision component                   | Calibration/second-order; relatively costly                        | Requires efficient fused sparse+dense execution                                    | Research GPU kernels; low mobile maturity                                              | Advanced research                            |
| QuIP / QuIP#         | Extreme 2-bit objective; quality claims are checkpoint/task-specific                | Very low weights plus codebooks/transforms                        | Calibration and substantial optimization                           | Specialized CUDA kernels/layout                                                    | Poor mobile portability; QuIP# maintenance/license concerns                            | Research; QuIP# superseded by QTIP direction |
| QTIP                 | Extreme compression with trellis coding                                             | Low weight payload plus metadata                                  | Calibration/optimization required                                  | Specialized kernels; measure                                                       | GPU research path; mobile unresolved                                                   | Emerging research                            |
| OmniQuant            | Learns clipping/transforms to improve low-bit PTQ                                   | Recipe-dependent                                                  | Calibration optimization can be lengthy                            | Export/kernel determines speed                                                     | Research integrations; no universal mobile format                                      | Advanced research                            |
| AutoRound            | Optimized rounding across common low-bit exports                                    | Recipe/export-dependent                                           | Short calibration optimization, still nonzero                      | Backend format determines runtime                                                  | Intel/cloud ecosystems and supported exports; test mobile separately                   | Advanced practical                           |
| SpinQuant            | Rotations reduce outliers, potentially improving W/A low bits                       | Transform/fusion overhead matters                                 | Optimization/training required                                     | Unfused rotations can erase gains                                                  | Best where transforms fold/fuse into target kernels                                    | Emerging research                            |
| BitNet native        | Potential native low-bit compute/storage; not comparable at fixed source weights    | Architecture-dependent                                            | Full native training cost, not PTQ time                            | bitnet.cpp/vendor claims require reproduction                                      | CPU and selected accelerator work; model support limited                               | Experimental architecture                    |
<!-- markdownlint-restore -->

No table cell supplies a project benchmark. Accuracy change, artifact size, resident memory, quantization wall time,
time to first token (TTFT), prefill throughput, decode throughput, energy, and sustained thermal behavior must be
recorded for every method that enters the common experiment.

## 11.10 GGUF, block formats, and interchange traps

GGUF stores tensors and typed metadata for the llama.cpp ecosystem. Quantization types such as Q4_0, Q4_K_M, Q5_K_M,
Q8_0, and IQ families have different block layouts and mixed-tensor policies. Names and availability evolve with
llama.cpp; pin the commit and inspect the current quantizer help. Conversion must start from a supported source
checkpoint; requantizing an already quantized artifact adds error and is explicitly discouraged in official
documentation (`DOC-VERIFIED`).

Block quantization amortizes one or more scales over a group. It improves compression but couples the artifact to
assumptions about block size, interleaving, endianness, alignment, and kernel tile shape. GPTQ, AWQ, GGUF, HQQ, and
“INT4” files cannot be interchanged merely because they all use four data bits. A valid interchange record includes
tensor layout, group size, scale/zero format, packing order, excluded tensors, and runtime/kernel version.

## 11.11 Calibration as a dataset-engineering task

A calibration set should resemble intended inputs without overlapping evaluation. For a domain SLM, stratify by domain,
language, chat turn structure, prompt length, technical symbols/units, and safety class. Include ordinary and difficult
examples, but do not tune on benchmark answers.

Recommended immutable record:

- source dataset revision and license;
- record IDs and SHA-256 manifest;
- exact tokenizer revision and chat template;
- random seed and sampling code;
- number of records, tokens, and length distribution;
- domain/category proportions;
- truncation, packing, padding, and masking behavior;
- proof of train/evaluation decontamination.

Use one **common control calibration set** to compare methods, then optionally a method-tuned set as a separate
ablation. GPTQ, AWQ, SmoothQuant, AQLM, SpQR, QuIP/QuIP#, QTIP, OmniQuant, AutoRound, importance-matrix GGUF, and static
activation quantization consume calibration differently; equal sample count does not imply equal optimization effort.
HQQ and common bitsandbytes load-time paths do not require a representative calibration set.

Calibration failure signals include clipped activation tails, a small set of layers with large reconstruction error,
sharp domain perplexity degradation, repeated or malformed generations, refusal/safety regressions, and long-context
degradation. Fixes include more representative data, smaller groups, higher precision for sensitive modules, adjusted
clipping, or a different algorithm—not silently selecting the evaluation set.

## 11.12 AWS versus ARM/mobile guidance

**TBL-CH11-04 — Deployment-oriented choice. (`STATICALLY REVIEWED`)**

<!-- markdownlint-disable MD013 -->
| Target                             | First baselines                                                             | Advanced candidates                                                          | Avoid assuming                                                         |
| ---------------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| AWS NVIDIA GPU, low batch          | BF16/FP16; AWQ/GPTQ INT4 using the serving engine’s supported kernel        | FP8 on supported accelerators; HQQ/AutoRound; AQLM/QTIP research             | A smaller artifact improves TTFT or decode without kernel support      |
| AWS NVIDIA GPU, high batch/prefill | BF16; W8A8 SmoothQuant or runtime-native INT8/FP8                           | W4A8/FP8 recipes supported by TensorRT-LLM, vLLM, SGLang, or selected engine | Weight-only decode gains transfer to high-batch prefill                |
| AWS CPU                            | INT8 and runtime-native weight-only blocks                                  | GGUF Q4/Q5/Q8, hardware-specific INT4                                        | GPU-format GPTQ/AWQ is efficient on CPU                                |
| Android/ARM CPU                    | llama.cpp GGUF exact Q4/Q5/Q8 types; ExecuTorch XNNPACK/runtime-native INT8 | MNN/ncnn/LiteRT-LM supported exports; KV INT8/4-bit ablation                 | NF4 or GPTQ files run natively because they are “4-bit”                |
| Android GPU/NPU                    | Runtime/vendor-supported symmetric INT8 or INT4 artifact for exact SoC      | LiteRT/ExecuTorch/ORT QNN/Qualcomm Genie after partition proof               | One NPU artifact works across Snapdragon, Tensor, MediaTek, and Exynos |
| iPhone/iPad                        | llama.cpp Metal/GGUF baseline; Core ML or MLX supported quantization        | Stateful Core ML low-bit weights/KV, ExecuTorch backend-specific export      | Apple Neural Engine executes arbitrary GGUF/MLX graphs                 |
| Raspberry Pi/ARM Linux             | GGUF CPU Q4/Q5/Q8                                                           | Runtime-native INT8/INT4; accelerator-specific path if present               | File fit implies interactive speed or sustained thermals               |
<!-- markdownlint-restore -->

Cloud artifacts and edge artifacts should normally be generated independently from the same immutable higher-precision
source. AWS GPU kernels may favor AWQ/GPTQ/FP8 layouts; ARM CPU kernels may favor GGUF blocks; NPUs may require
symmetric per-channel INT8, fixed shapes, and a vendor compiler. Forcing one artifact across targets can sacrifice both
performance and supportability.

On mobile, test four separate gates:

1. **Storage:** installation/download and free-space margin.
2. **Load:** peak resident set size, memory mapping, and operating-system termination.
3. **Interactive performance:** cold/warm TTFT, prefill, decode, and cancellation.
4. **Sustainability:** energy per token, battery drain, skin temperature, and throttling over 10–30 minutes.

CPU kernels are the portability baseline. GPUs can improve throughput but compete for unified memory and UI rendering.
NPUs can be efficient only if graph coverage, shape constraints, quantization recipe, and vendor runtime match the exact
device. Capture operator-placement reports; a selectable accelerator is not proof of offload.

## 11.13 A complete common-baseline experiment

### Research question and baseline

`PLANNED EXPERIMENT`: Which feasible quantization method gives the best quality–memory–latency trade-off for one
domain-specialized SLM on one AWS GPU and representative ARM/mobile targets?

Select **one immutable, publicly downloadable, license-approved 0.5B–1.5B decoder-only instruction checkpoint that is
supported by all methods retained after a compatibility spike**. Do not hard-code a fashionable model name before
confirming support. Record repository ID, exact commit hash, tokenizer files, license text, chat template, architecture
config, and source artifact SHA-256. If no single checkpoint is supported by every candidate, preserve the baseline and
report a missing cell; do not substitute another model and call the comparison common.

### Candidate matrix

- source FP32 where feasible for quality reference;
- BF16 or FP16 deployment baseline;
- INT8: bitsandbytes/LLM.int8 on AWS, GGUF Q8_0 or runtime-native INT8 on edge;
- 4-bit: bitsandbytes NF4 as a training/memory control, GPTQ, AWQ, HQQ, and exact GGUF Q4 type;
- 3/2-bit: one supported GPTQ or HQQ setting plus AQLM/QTIP only after kernel validation;
- SmoothQuant W8A8 on a supported server runtime;
- optional QAT candidate using the exact runtime-native target;
- KV cache at FP16, INT8, and 4-bit where the same runtime supports it.

Methods are not forced into unsupported runtimes. The experiment compares both **algorithm effects under common
controls** and **deployable model/runtime tuples**. These are separate analyses.

### Controlled variables

Keep source checkpoint, tokenizer/chat template, calibration IDs, evaluation records, prompts, stop rules, context
lengths, output limits, sampling (prefer greedy for parity), batch size, thread count, runtime build, warm-up policy,
trial count, power mode, and ambient conditions fixed within each hardware comparison. Pin:

- Python, PyTorch, Transformers, quantizer, CUDA/driver, compiler, runtime, and kernel versions;
- random seeds;
- quantization group size and activation order where comparable;
- number of calibration tokens and optimization steps;
- AWS instance type/GPU model and device clocks or power policy where observable;
- edge device model, system-on-chip (SoC), RAM, OS/firmware, charging state, battery level, temperature, and background
  activity.

### Quality suite

1. Validation negative log-likelihood and perplexity on fixed general and domain corpora.
2. Exact-match/F1 or task-specific deterministic metrics.
3. Expert-reviewed domain factuality and diagnostic sequencing.
4. Safety-critical contraindication, escalation, and refusal tests.
5. General-capability retention.
6. Long-context retrieval-from-prompt and instruction-following tests—not runtime retrieval-augmented generation.
7. Paired output review blind to model variant.

Report paired bootstrap 95% confidence intervals where appropriate and seed/trial variation. Perplexity must use the
identical tokenizer and token accounting.

### Systems suite

Measure artifact bytes; effective bits/weight; conversion wall time and peak host/GPU memory; load time; loaded and peak
resident memory; cold and warm TTFT; prefill tokens/s; decode tokens/s; end-to-end latency for fixed input/output
lengths; CPU/GPU/NPU utilization; energy/token where measurable; and sustained thermal performance. Test at least short,
medium, and maximum intended context. Separate first-run compilation from steady state.

Run at least one warm-up followed by 30 timed trials for stable server latency and enough mobile trials to characterize
thermal phases; the exact count can be increased after a pilot variance analysis. Preserve every raw trial. Randomize
candidate order when thermal carryover is possible and return devices to a declared temperature band.

### Acceptance and analysis

Define thresholds before running: maximum allowable domain/safety regression, RAM budget, artifact budget, p95 TTFT,
minimum sustained decode rate, and energy target. A candidate is deployable only if it passes quality, compatibility,
memory, and stability gates. Construct Pareto fronts rather than a single weighted ranking. A candidate is dominated if
another is no worse on every selected objective and strictly better on at least one.

### Illustrative walkthrough: interpreting a completed experiment

The following numbers are deliberately invented for teaching and are **not measurements, estimates, or expected
performance**. Suppose one device test produced:

| Candidate | Domain score | Peak RAM | Decode rate | Observation |
| --- | ---: | ---: | ---: | --- |
| FP16 | 90 | 3.4 GiB | 8 tok/s | quality reference |
| INT8 | 90 | 2.0 GiB | 10 tok/s | passes every gate |
| INT4-A | 88 | 1.2 GiB | 13 tok/s | fastest, but below quality floor of 89 |
| INT4-B mixed | 90 | 1.5 GiB | 11 tok/s | restores sensitive layers |
| INT3 | 82 | 1.0 GiB | 9 tok/s | slower despite smaller payload |

Interpretation proceeds in this order:

1. Reject INT4-A and INT3 because a hard quality floor cannot be compensated by speed or RAM.
2. INT8 and INT4-B both dominate FP16 on the shown systems metrics while matching its rounded domain score; uncertainty
   intervals and safety strata would still need to confirm that apparent tie.
3. INT4-B is the lower-memory survivor; INT8 is the faster survivor. Neither universally dominates the other.
4. INT3 demonstrates why nominal bit width is not a latency ranking: packing, dequantization, or fallback can make a
   smaller representation slower.
5. The table still cannot select a production artifact without artifact size, TTFT, prefill, energy, thermal,
   long-context, failure, and operator-placement evidence.

This is how every method table in the chapter should be read: the mechanism predicts which resource _might_ improve;
the measured model–format–runtime–hardware tuple determines whether it did.

### Result schema

```json
{
  "evidence": "PLANNED_EXPERIMENT",
  "run_id": "ch11-example-run",
  "source_model_revision": "record-an-immutable-commit",
  "tokenizer_sha256": "record-a-sha256",
  "calibration_manifest_sha256": "record-a-sha256-or-none",
  "method": "awq",
  "format": { "weight_bits": 4, "activation_bits": 16, "group_size": 128 },
  "quantizer_version": "record-the-executed-version",
  "runtime_and_kernel": "record-the-executed-build",
  "hardware": { "device": "record-the-device", "os": "record-the-os" },
  "workload": { "input_tokens": 512, "output_tokens": 128, "batch": 1 },
  "quality": { "domain_score": null, "general_score": null, "perplexity": null },
  "systems": {
    "artifact_bytes": null,
    "loaded_memory_bytes": null,
    "peak_memory_bytes": null,
    "ttft_ms": null,
    "prefill_tokens_s": null,
    "decode_tokens_s": null,
    "energy_j_per_token": null
  },
  "raw_results_uri": null,
  "artifact_sha256": null
}
```

## 11.14 Implementation listings

### Listing CH11-L01 — Reproducible environment contract

**Status:** `UNVERIFIED IMPLEMENTATION`. **Purpose:** create a pinned experiment environment after replacing
illustrative values with versions verified together on the target date. **Hardware:** Linux x86-64 host; NVIDIA GPU only for
CUDA candidates. **Security:** use a clean virtual environment, review packages, avoid untrusted pickle checkpoints, and
pin hashes in the final lock file.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip

# Create requirements-ch11.lock only after resolving this exact hardware stack
# in a disposable compatibility environment. The lock must contain hashes.
python -m pip install --require-hashes -r requirements-ch11.lock

python -m pip freeze --all > environment.freeze.txt
python - <<'PY'
import hashlib, pathlib, platform, torch
print(platform.platform())
print("torch", torch.__version__, "cuda", torch.version.cuda)
for p in sorted(pathlib.Path("manifests").glob("*")):
    print(p, hashlib.sha256(p.read_bytes()).hexdigest())
PY
```

Expected failures include incompatible CUDA wheels, quantizer forks that support different architectures, renamed APIs,
unavailable kernels, and insufficient temporary disk/VRAM. The command becomes executable only after the reviewed lock
file exists.

### Listing CH11-L02 — Framework-independent evaluation harness

**Status:** `UNVERIFIED IMPLEMENTATION`. **Purpose:** measure generated-token latency without hiding synchronization or
token accounting. **Inputs:** a model adapter implementing `encode`, `prefill_and_generate`, and synchronization; JSONL
prompts with immutable IDs. **Output:** raw JSONL, one record per trial.

```python
from __future__ import annotations
import json, time, uuid
from pathlib import Path

def benchmark(adapter, prompts, output_path: str, warmups: int = 3, trials: int = 30):
    rows = []
    for prompt in prompts[:warmups]:
        adapter.prefill_and_generate(prompt["text"], max_new_tokens=16, greedy=True)
        adapter.synchronize()

    for prompt in prompts:
        input_ids = adapter.encode(prompt["text"])
        for trial in range(trials):
            adapter.reset()
            adapter.synchronize()
            t0 = time.perf_counter_ns()
            event = adapter.prefill_and_generate(
                prompt["text"], max_new_tokens=128, greedy=True,
                return_first_token_event=True
            )
            event.wait()
            adapter.synchronize()
            t_first = time.perf_counter_ns()
            output_ids = adapter.finish()
            adapter.synchronize()
            t_end = time.perf_counter_ns()
            generated = len(output_ids)
            rows.append({
                "run_id": str(uuid.uuid4()),
                "prompt_id": prompt["id"],
                "trial": trial,
                "input_tokens": len(input_ids),
                "generated_tokens": generated,
                "ttft_ms": (t_first - t0) / 1e6,
                "end_to_end_ms": (t_end - t0) / 1e6,
                "decode_tokens_s": (
                    (generated - 1) / ((t_end - t_first) / 1e9)
                    if generated > 1 and t_end > t_first else None
                )
            })
    Path(output_path).write_text(
        "".join(json.dumps(r, sort_keys=True) + "\n" for r in rows),
        encoding="utf-8"
    )
```

The adapter must use backend-specific synchronization (`torch.cuda.synchronize`, runtime callback/fence, or blocking
API). A host timer around asynchronous GPU launch is invalid. This listing does not measure prefill separately unless
the runtime exposes a prefill-completion event; add that event rather than inferring prefill from TTFT.

### Listing CH11-L03 — llama.cpp edge artifact and benchmark pattern

**Status:** `UNVERIFIED IMPLEMENTATION`. **Purpose:** convert the immutable source once and quantize exact GGUF types.
**Prerequisites:** a pinned llama.cpp commit built for the host and later for each target. **Output:** hashed GGUF
artifacts and raw benchmark logs.

```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp
git checkout aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
cmake -S . -B build -DGGML_NATIVE=OFF -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j

python convert_hf_to_gguf.py ./models/source \
  --outfile artifacts/baseline-f16.gguf --outtype f16

# Confirm exact available type names with this pinned build before running.
./build/bin/llama-quantize artifacts/baseline-f16.gguf \
  artifacts/baseline-q8_0.gguf Q8_0
./build/bin/llama-quantize artifacts/baseline-f16.gguf \
  artifacts/baseline-q4_k_m.gguf Q4_K_M

sha256sum artifacts/*.gguf > artifacts/SHA256SUMS
./build/bin/llama-bench -m artifacts/baseline-q4_k_m.gguf \
  -p 512 -n 128 -b 1 -r 5 > raw/llama-bench-q4_k_m.txt
```

Do not infer Android/iOS results from the desktop command. Build the same pinned commit for each ABI/backend, preserve
compile flags, and test tokenizer/chat-template parity before timing. Exact command flags can change; use the pinned
commit’s help and documentation.

### Listing CH11-L04 — Quality–memory and quality–latency plots

**Status:** `UNVERIFIED IMPLEMENTATION`. **Purpose:** render measured results only. **Input:** validated Parquet with
one row per aggregate candidate/device. The script intentionally fails on missing measurements.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_parquet("results/validated-aggregates.parquet")
needed = {
    "candidate", "hardware", "domain_score_mean", "domain_score_ci95",
    "peak_memory_gib", "decode_tokens_s_median", "decode_tokens_s_ci95"
}
missing = needed - set(df.columns)
if missing or df[list(needed - {"candidate", "hardware"})].isna().any().any():
    raise ValueError(f"Missing or null measured fields: {missing}")

for hardware, g in df.groupby("hardware"):
    fig, ax = plt.subplots(figsize=(8, 5))
    ax.errorbar(g.peak_memory_gib, g.domain_score_mean,
                yerr=g.domain_score_ci95, fmt="none", capsize=3)
    ax.scatter(g.peak_memory_gib, g.domain_score_mean)
    for _, r in g.iterrows():
        ax.annotate(r.candidate, (r.peak_memory_gib, r.domain_score_mean))
    ax.set(xlabel="Peak resident memory (GiB; measured)",
           ylabel="Domain score (pre-registered scale)",
           title=f"Quality versus peak memory — {hardware}")
    ax.grid(alpha=.25)
    fig.tight_layout()
    fig.savefig(f"figures/quality-memory-{hardware}.svg")

    fig, ax = plt.subplots(figsize=(8, 5))
    ax.errorbar(g.decode_tokens_s_median, g.domain_score_mean,
                xerr=g.decode_tokens_s_ci95, yerr=g.domain_score_ci95,
                fmt="none", capsize=3)
    ax.scatter(g.decode_tokens_s_median, g.domain_score_mean)
    for _, r in g.iterrows():
        ax.annotate(r.candidate, (r.decode_tokens_s_median, r.domain_score_mean))
    ax.set(xlabel="Sustained decode throughput (tokens/s; median)",
           ylabel="Domain score (pre-registered scale)",
           title=f"Quality versus decode throughput — {hardware}")
    ax.grid(alpha=.25)
    fig.tight_layout()
    fig.savefig(f"figures/quality-latency-{hardware}.svg")
```

## 11.15 Required graph specifications—no fabricated data

<a id="fig-ch11-15"></a> **FIG-CH11-15 — Quality versus memory footprint specification (`PLANNED EXPERIMENT`; canonical
FIG-15 is owned by CH-32).** Alt text: the planned chart will compare reviewed quality against measured peak memory for
exact artifact, runtime, workload, and target tuples; no points exist.

- One panel per hardware/runtime; do not join points measured on different devices.
- X-axis: measured peak resident memory in GiB, beginning at zero when visually practical.
- Y-axis: pre-registered domain score with 95% confidence interval.
- Point label: method, exact format, group size, runtime/kernel.
- Shape: weight-only, W+A, or native low-bit; color: effective bits/weight.
- Secondary annotation: artifact GiB, not a second axis.
- Add a Pareto frontier only after validating all measurements.
- Source: raw run IDs and aggregate script hash embedded in figure metadata.

<a id="fig-ch11-14"></a> **FIG-CH11-14 — Quality versus latency specification (`PLANNED EXPERIMENT`; canonical FIG-14 is
owned by CH-32).** Alt text: the planned chart will compare reviewed quality against measured latency for exact
artifact, runtime, workload, and target tuples; no points exist.

- Prefer x-axis sustained decode tokens/s (higher is better) or p95 latency in milliseconds (lower is better); never mix
  them.
- Use fixed input/output lengths and batch size; create separate prefill and decode panels.
- Include horizontal quality acceptance threshold and vertical performance threshold defined before results.
- Display uncertainty from repeated trials and paired quality resampling.
- Never place paper-reported and project-measured points on the same empirical series.

Until raw results exist, these are specifications only. Empty axes or invented “representative” points would be
misleading and must not be published.

## 11.16 Troubleshooting

<!-- markdownlint-disable MD013 -->
| Symptom                                             | Likely cause                                                                        | Diagnostic                                                                    | Corrective action                                                                     |
| --------------------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Artifact is small but process is out of memory      | KV cache, conversion copy, mmap page residency, workspace, or fallback tensors      | Measure load, first prefill, steady decode, and max-context memory separately | Reduce context/batch; quantize KV if validated; use mmap; select lower-memory runtime |
| 4-bit is slower than FP16                           | Unpack/dequant overhead, unsupported kernel, tiny matrices, GPU fallback            | Capture kernel/operator placement and profiler trace                          | Use supported layout/group size or FP16; do not select by file size                   |
| Perplexity spikes                                   | Wrong tokenizer/template, calibration mismatch, over-aggressive clipping/group size | Compare source token IDs/logits and per-layer reconstruction                  | Fix parity; smaller groups; higher precision for sensitive layers                     |
| Outputs repeat or terminate incorrectly             | EOS/chat-template mismatch rather than quantization alone                           | Golden token/template and greedy-generation parity                            | Correct metadata/template before judging quantizer                                    |
| INT8 NPU graph runs mostly on CPU                   | Unsupported operators, dynamic shapes, wrong quantization contract                  | Delegate partition report                                                     | Re-export to documented static/symmetric recipe or reject path                        |
| GPTQ/AWQ artifact will not load                     | Serializer/kernel layout mismatch                                                   | Compare producer/runtime versions and format metadata                         | Use runtime’s supported quantizer/exporter as a pinned pair                           |
| NF4 model is not faster                             | NF4 selected for training memory, no native inference kernel                        | Profile dequant and matrix kernels                                            | Export merged model to a serving-native format                                        |
| Quality passes short prompts but fails long context | KV precision, accumulated error, cache rollover, positional mismatch                | Sweep context lengths and cache boundary tests                                | Raise KV precision, fix cache logic, or cap supported context                         |
| Mobile starts fast then slows                       | Thermal throttling or power policy                                                  | Record temperature, clocks, power, and throughput over time                   | Reduce threads/offload, use efficient kernel, set honest sustained target             |
| Metrics differ across methods                       | Different tokenizer, prompt truncation, stop conditions, or generated count         | Audit raw IDs and workload manifest                                           | Re-run under one canonical harness                                                    |
| Quantizer crashes during calibration                | Unsupported module, insufficient VRAM/RAM, sequence too long                        | Preserve traceback and per-stage memory                                       | Use supported architecture, reduce calibration batch, offload carefully               |
| Quantized safety behavior regresses                 | Sensitive logits/ranking changed                                                    | Paired safety review and confidence intervals                                 | Reject candidate or retain sensitive modules at higher precision                      |
<!-- markdownlint-restore -->

## 11.17 Decision framework for domain SLMs

1. Start with the source BF16/FP16 quality baseline and a portable INT8 baseline.
2. Establish tokenizer, chat-template, and logit parity before quantization.
3. For AWS GPU, test only formats with first-class kernels in the intended serving engine; compare weight-only INT4 for
   low-batch decode with W8A8/FP8 for throughput.
4. For ARM/mobile, establish GGUF CPU and a runtime-native INT8 path, then add exact INT4 and accelerator exports.
5. Separate weight, activation, and KV-cache ablations.
6. Reject candidates that miss safety, quality, memory, or sustained-performance thresholds even if their files are
   smallest.
7. Select separate cloud and edge artifacts when kernels demand it.
8. Preserve a signed manifest linking every released artifact to source, calibration, converter, runtime, evaluation,
   and raw benchmark evidence.

Domain specialization does not eliminate stale knowledge, hallucination, or conflicting learned facts. Quantization can
perturb confidence and token ranking; it cannot add missing knowledge. Pharmacy and safety-critical repair releases
require qualified expert review, uncertainty/escalation behavior, and non-autonomous intended-use limits after every
quantization change.

## 11.18 Review questions and engineering exercises

1. Derive scale and zero point for asymmetric unsigned INT8 over $[-0.7,2.1]$, quantize $-0.7,0,0.9,2.1$, and
   calculate reconstruction error.
2. Explain why per-channel INT8 can outperform per-tensor INT8 numerically yet fail to improve latency.
3. For a 2B-parameter model with group size 64, 4-bit codes, FP16 scales, and FP16 zero points, compute the minimum
   effective bits/weight before alignment.
4. Compute KV-cache payload for a named architecture at 2K, 8K, and 32K tokens in FP16, INT8, and 4-bit. State every
   assumption.
5. Contrast static activation quantization, dynamic per-token quantization, PTQ, and QAT without treating them as
   mutually exclusive categories.
6. Explain why NF4 QLoRA storage is not evidence that NF4 is the best Android inference format.
7. Design a calibration split for an automotive model containing diagnostics, units, multi-turn chats, and safety
   warnings. Prevent evaluation leakage.
8. Compare GPTQ, AWQ, SmoothQuant, AQLM, and GGUF for one AWS GPU and one ARM phone. Mark unsupported cells rather than
   inventing compatibility.
9. Inspect a quantized artifact and calculate effective bits/weight from actual tensor and metadata bytes.
10. Implement CH11-L02 for one runtime, prove synchronization correctness, and retain raw trials.
11. Run a layer-sensitivity ablation that restores one layer at a time to FP16; pre-register the selection rule.
12. Evaluate FP16 versus quantized KV caches at multiple context lengths and report paired quality changes.
13. Produce FIG-14 and FIG-15 from measured data, including uncertainty and raw-result provenance.
14. Write a release recommendation where the smallest artifact is rejected because of quality, fallback, thermals, or
    maintenance risk.
15. Distinguish BitNet native low-bit training from PTQ of a conventional transformer and design a fair comparison at
    equal deployment constraints.

## 11.19 Primary sources and authoritative documentation

Accessed or recency-checked 2026-11-23 unless otherwise stated.

1. Dettmers, T. et al. (2022). “LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale.” _NeurIPS 2022_. <!-- markdownlint-disable-line MD013 MD049 -->
   arXiv:2208.07339. <https://arxiv.org/abs/2208.07339> — vector-wise INT8 and outlier decomposition; official
   implementation in bitsandbytes. Limit: reported large-model/server results do not establish SLM/mobile behavior.
2. Frantar, E. et al. (2023). “GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers.” _ICLR <!-- markdownlint-disable-line MD013 MD049 -->
   2023_. arXiv:2210.17323. <https://arxiv.org/abs/2210.17323> and <https://github.com/IST-DASLab/gptq> — approximate <!-- markdownlint-disable-line MD013 MD049 -->
   second-order layer reconstruction. Limit: runtime results depend on later kernels and formats.
3. Lin, J. et al. (2024). “AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration.” _MLSys <!-- markdownlint-disable-line MD013 MD049 -->
   2024_. arXiv:2306.00978. <https://arxiv.org/abs/2306.00978> and <https://github.com/mit-han-lab/llm-awq> — <!-- markdownlint-disable-line MD013 MD049 -->
   activation-aware saliency/scaling and TinyChat. Limit: supported architectures/hardware are finite.
4. Xiao, G. et al. (2023). “SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models.”
   _ICML 2023_, PMLR 202. <https://proceedings.mlr.press/v202/xiao23c.html> and <!-- markdownlint-disable-line MD013 MD049 -->
   <https://github.com/mit-han-lab/smoothquant> — equivalent smoothing for W8A8. Limit: server stack evidence is not
   mobile evidence.
5. Dettmers, T. et al. (2023). “QLoRA: Efficient Finetuning of Quantized LLMs.” _NeurIPS 2023_. arXiv:2305.14314. <!-- markdownlint-disable-line MD013 MD049 -->
   <https://arxiv.org/abs/2305.14314> — NF4, double quantization, paged optimizers. Limit: fine-tuning memory results do
   not determine serving format.
6. bitsandbytes Foundation. Repository and Transformers integration.
   <https://github.com/bitsandbytes-foundation/bitsandbytes> and
   <https://huggingface.co/docs/transformers/quantization/bitsandbytes> — `DOC-VERIFIED` interfaces; exact
   backend/version must be pinned.
7. llama.cpp / GGML. Repository, quantization documentation, and GGUF specification.
   <https://github.com/ggml-org/llama.cpp>,
   <https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md>,
   <https://github.com/ggml-org/ggml/blob/master/docs/gguf.md> — `DOC-VERIFIED`; rapidly evolving types require a commit
   pin.
8. Badri, H. and Shaji, A. (2023). “HQQ: Half-Quadratic Quantization of Large Language Models.” Technical release.
   <https://github.com/dropbox/hqq> and <https://dropbox.github.io/hqq_blog/> — calibration-free optimization.
   Limitation: evidence/venue is less conventional than peer-reviewed methods.
9. Egiazarian, V. et al. (2024). “Extreme Compression of Large Language Models via Additive Quantization.” _ICML 2024_. <!-- markdownlint-disable-line MD013 MD049 -->
   arXiv:2401.06118. <https://arxiv.org/abs/2401.06118> and <https://github.com/Vahe1994/AQLM> — additive codebooks.
   Limit: expensive optimization and specialized kernels.
10. Dettmers, T. et al. (2024). “SpQR: A Sparse-Quantized Representation for Near-Lossless LLM Weight Compression.”
    _ICLR 2024_. arXiv:2306.03078. <https://arxiv.org/abs/2306.03078> and <https://github.com/Vahe1994/SpQR> — sparse <!-- markdownlint-disable-line MD013 MD049 -->
    outlier retention. “Near-lossless” remains paper-reported for tested conditions.
11. Chee, J. et al. (2023). “QuIP: 2-Bit Quantization of Large Language Models With Guarantees.” _NeurIPS 2023_. <!-- markdownlint-disable-line MD013 MD049 -->
    arXiv:2307.13304. <https://arxiv.org/abs/2307.13304> and <https://github.com/Cornell-RelaxML/QuIP>.
12. Tseng, A. et al. (2024). “QuIP#: Even Better LLM Quantization with Hadamard Incoherence and Lattice Codebooks.”
    _ICML 2024_. <https://openreview.net/forum?id=9BrydUVcoe> and <https://github.com/Cornell-RelaxML/quip-sharp> — <!-- markdownlint-disable-line MD013 MD049 -->
    repository maintenance and GPL-3.0 status must be considered.
13. Wang, H. et al. (2023). “BitNet: Scaling 1-bit Transformers for Large Language Models.” arXiv:2310.11453.
    <https://arxiv.org/abs/2310.11453>. Ma, S. et al. (2024). “The Era of 1-bit LLMs: All Large Language Models are in
    1.58 Bits.” arXiv:2402.17764. <https://arxiv.org/abs/2402.17764>. Microsoft bitnet.cpp:
    <https://github.com/microsoft/BitNet> — native low-bit architecture evidence, not ordinary PTQ.
14. Shao, W. et al. (2024). “OmniQuant: Omnidirectionally Calibrated Quantization for Large Language Models.” _ICLR <!-- markdownlint-disable-line MD013 MD049 -->
    2024_. arXiv:2308.13137. <https://arxiv.org/abs/2308.13137> — learnable clipping and equivalent transformations. <!-- markdownlint-disable-line MD013 MD049 -->
15. Liu, Z. et al. (2024). “SpinQuant: LLM Quantization with Learned Rotations.” arXiv:2405.16406.
    <https://arxiv.org/abs/2405.16406> — rotation-based outlier mitigation; runtime fusion is a deployment concern.
16. PyTorch AO documentation/repository. <https://github.com/pytorch/ao> and <https://docs.pytorch.org/ao/> —
    `DOC-VERIFIED` implementation ecosystem, not one quantization algorithm.
17. ExecuTorch LLM documentation. <https://docs.pytorch.org/executorch/stable/llm/getting-started.html> —
    backend-specialized `.pte` export; exact model/backend support is `TEST-REQUIRED`.
18. Google LiteRT-LM overview and PyTorch conversion documentation.
    <https://developers.google.com/edge/litert-lm/overview> and
    <https://developers.google.com/edge/litert/conversion/pytorch/genai> — documented edge paths, not project execution.
19. NVIDIA mixed-precision and TensorRT-LLM documentation. <https://docs.nvidia.com/deeplearning/transformer-engine/>
    and <https://nvidia.github.io/TensorRT-LLM/> — FP8/low-bit behavior is accelerator and release specific.
20. Jacob, B. et al. (2018). “Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only
    Inference.” _CVPR 2018_. arXiv:1712.05877. <https://arxiv.org/abs/1712.05877> — foundational affine integer <!-- markdownlint-disable-line MD013 MD049 -->
    quantization and QAT treatment. Limit: convolutional-network evidence predates current generative-transformer
    workloads.
21. Micikevicius, P. et al. (2022). “FP8 Formats for Deep Learning.” arXiv:2209.05433.
    <https://arxiv.org/abs/2209.05433> — E4M3/E5M2 format proposal and scaling evidence. Limit: practical behavior
    depends on accelerator and framework implementation.
22. Cornell RelaxML. QTIP official repository and paper links. <https://github.com/Cornell-RelaxML/qtip> — current
    trellis-quantization implementation lineage; exact paper status, kernels, license, and maintained model list must be
    rechecked before adoption.
23. Intel. AutoRound official repository and documentation. <https://github.com/intel/auto-round> — sign-gradient-based
    rounding and supported export integrations. This is implementation documentation; quality, conversion time, and
    target-kernel results remain `TEST-REQUIRED`.

## 11.20 Verification note

The listings, experiment, and illustrative result walkthrough are unexecuted and provide no measured quality, size,
memory, latency, throughput, energy, or thermal result. Resolve one compatible stack, archive raw evidence, and validate
the actual exported artifact before drawing a deployment conclusion.

## Cross-links

- [Part index](README.md)
- [CH-10: Compression fundamentals](ch-10-the-fundamentals-of-model-compression.md)
- [Edge runtime survey](../../04-research-comparison-report/edge-runtime-survey.md)
- [Quantization methods research survey](../../04-research-comparison-report/quantization-methods.md)
- [CH-11 assessment](../../13-assessments/ch-11-assessment.md)
- [Reproducibility standard](../../00-governance/reproducibility-standard.md)
