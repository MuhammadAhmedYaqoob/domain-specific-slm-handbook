# CH-17: Mobile Hardware Architecture

<a id="ch17-status"></a>

## Chapter status and evidence boundary

**Research cut-off:** 2026-11-23  
**Evidence:** `DOC-VERIFIED` where tied to official sources in [§17.18](#ch17-sources); `ILLUSTRATIVE` for diagrams and
calculations; `PLANNED EXPERIMENT` for unexecuted procedures.  
**Execution status:** no model conversion, mobile build, accelerator trace, benchmark, energy measurement, or
physical-device test was performed for this chapter. Exact model/runtime/device compatibility and every performance
claim remain `TEST-REQUIRED`.

A release decision concerns a complete tuple:

> **checkpoint + tokenizer + artifact + runtime + backend/delegate + OS + driver + SoC + device thermal envelope**

Changing any member can change correctness, supported precision, memory, initialization cost, and sustained speed. “The
phone has an NPU” is not a deployment result, and vendor peak throughput is not used here as a universal planning
specification.

Cross-links: [Part V index](README.md), [CH-16](ch-16-edge-ai-fundamentals.md),
[CH-18](ch-18-edge-inference-frameworks.md), [CH-19](ch-19-mobile-inference-optimization.md),
[CH-17 assessment](../../13-assessments/ch-17-assessment.md), and the
[evidence standard](../../00-governance/evidence-and-citation-standard.md).

<a id="ch17-objectives"></a>

## 17.1 Learning objectives

After this chapter, the reader should be able to:

1. distinguish an Instruction Set Architecture (ISA), processor, accelerator, API, backend, delegate, runtime, and
   artifact;
2. explain how Arm64 CPUs execute quantized transformer kernels using scalar, Neon, and optional dot-product
   instructions;
3. compare CPU, mobile GPU, and NPU execution without treating nominal throughput as an SLM benchmark;
4. describe Snapdragon/Hexagon/QAIRT, MediaTek/NeuroPilot, Google Tensor/LiteRT, and Apple/Core ML/Metal paths;
5. explain NNAPI's historical role and current Android vendor paths;
6. calculate first-order weight, KV-cache, and bandwidth budgets;
7. identify initialization, RAM-pressure, power, thermal, partition, and quantization constraints; and
8. define evidence-based low-, mid-, and flagship-phone cohorts without fabricated specifications.

<a id="ch17-stack"></a>

## 17.2 Name the layer before comparing technologies

Many comparisons join unlike objects—for example, “Arm versus Core ML” or “Vulkan versus Hexagon.”

<a id="tbl-ch17-01"></a> **Table 17-1 — Mobile inference stack vocabulary (`DOC-VERIFIED`; examples are
non-exhaustive)**

<!-- markdownlint-disable MD013 -->
| Layer                           | Definition                                                         | Examples                                                                           | What it does **not** establish             |
| ------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------- | ------------------------------------------ |
| **ISA**                         | Contract of registers, instructions, memory and exception behavior | AArch64; A64 Advanced SIMD/Neon                                                    | Core count, cache, clock, bandwidth, speed |
| **Processor/microarchitecture** | Physical ISA implementation                                        | Arm-compatible performance and efficiency CPU cores                                | That two Arm64 phones perform alike        |
| **Accelerator**                 | Hardware optimized for a workload class                            | Adreno/Mali/Apple GPU; Hexagon HTP; MediaTek MDLA; Apple Neural Engine; Tensor TPU | App accessibility or model compatibility   |
| **API**                         | Software programming contract                                      | Vulkan, Metal, Core ML, LiteRT, historical NNAPI                                   | Which unit executes every operator         |
| **Backend/execution provider**  | Runtime implementation targeting compute hardware                  | XNNPACK, QNN HTP, Core ML, Vulkan, Metal                                           | Full offload or no fallback                |
| **Delegate/partitioner**        | Claims supported graph regions for a backend                       | ExecuTorch Qualcomm backend; LiteRT vendor path                                    | That processor crossings are cheap         |
| **Runtime**                     | Loads artifacts, allocates, schedules and invokes kernels          | llama.cpp, ExecuTorch, LiteRT-LM, ORT GenAI, Core ML                               | Model architecture or artifact portability |
| **Model artifact**              | Serialized graph, weights and metadata                             | GGUF, `.pte`, `.tflite`, `.litertlm`, `.mlpackage`, QNN context                    | Executability by another runtime or SoC    |
| **Quantization scheme**         | Reduced-precision mapping and grouping rules                       | per-channel INT8; grouped 4-bit weight-only; 8-bit KV                              | Availability of a matching kernel          |
<!-- markdownlint-restore -->

An **NPU** is a category, not a shared ISA. Qualcomm, MediaTek, Google, Apple, and Samsung accelerators have different
compilers, operators, layouts, precisions, and drivers. “INT4” is also incomplete: packing, scale type, group size, zero
point, activation precision, and kernel layout must match.

<a id="fig-16"></a> **FIG-16 — CPU versus GPU versus NPU inference (`ILLUSTRATIVE`).** Alt text: a checkpoint is
exported into a backend-specific artifact, then a runtime partitions work among CPU, GPU, and NPU paths sharing system
memory; unsupported operations may fall back across processor boundaries. Dashed fallback is possible, not guaranteed.
Not measured; arrows show conceptual relationships only.

```text
checkpoint + tokenizer + chat template
                  |
       exporter / quantizer / compiler
                  |
    backend-specific artifact + manifest
                  |
 app -> generation runtime -> partitioner/delegate
                              /       |       \
                       CPU kernels GPU kernels NPU program
                       Arm64/Neon Vulkan/Metal vendor runtime
                              \       |       /
                               shared system DRAM
                                   . . .
                            unsupported-op fallback
```

The diagram centers shared Dynamic Random-Access Memory (DRAM): processors compete for capacity and bandwidth while
display, camera, modem, operating system, and other apps remain active.

<a id="ch17-arm"></a>

## 17.3 Arm, ARM64, AArch64, and Neon

### 17.3.1 Architecture is not a chip

Arm licenses architectures and processor designs. **AArch64** is the 64-bit execution state introduced by Armv8-A;
**A64** is its instruction set. “ARM64” is a common platform/ABI label. These identify a software target, not a
performance tier. Android arm64-v8a binaries run on devices with different cores, caches, memory controllers, thermal
policy, and optional ISA extensions.

The CPU remains essential when an accelerator is selected. It commonly performs tokenization, sampling, scheduling,
allocation, unsupported operators, and fallback. It is the broadest recovery path when a GPU driver or vendor compiler
rejects a graph.

### 17.3.2 Neon and useful vector instructions

Arm Neon, called **Advanced SIMD** in architecture manuals, applies one instruction across values packed into 64- or
128-bit vectors. AArch64 exposes 32 128-bit vector registers. One register can hold sixteen 8-bit integers, eight 16-bit
values, four FP32 values, or two FP64 values.

Relevant operations include:

- vector loads/stores and fused multiply-add for floating-point kernels;
- widening integer multiplication and accumulation for quantized kernels;
- saturating, narrowing, shift, lookup, and permutation instructions for packing/dequantization;
- optional Armv8.2-A signed/unsigned dot products such as `SDOT`/`UDOT`; and
- newer optional matrix-oriented extensions on supported implementations.

`DOC-VERIFIED`: Arm defines A64 Advanced SIMD and `SDOT`; its Neon guide describes the 64/128-bit SIMD model and matrix
multiplication [S1–S3]. **Optional** is crucial: use capability detection or compatible binary variants. Arm64 alone
does not promise every extension.

### 17.3.3 A grouped low-bit CPU kernel

For grouped 4-bit weight-only matrix multiplication, a kernel:

1. reads packed nibbles and group scales;
2. unpacks into wider lanes;
3. combines values with FP16/FP32/INT8 activations;
4. accumulates in a wider type;
5. applies fused or separate transformations; and
6. writes output activations.

The artifact reduces traffic, but unpacking and scaling cost instructions. It speeds execution only when a tuned kernel
uses the representation more cheaply than the avoided memory traffic. A mismatched backend may dequantize to FP16,
increasing temporary memory and erasing the expected advantage.

### 17.3.4 CPU strengths and limits

**Strengths:** widest compatibility; mature low-bit LLM kernels; flexible control flow; effective single-token decode;
reliable fallback.

**Limits:** lower efficiency than a well-supported accelerator for regular dense graphs; strong dependence on cache and
bandwidth; excessive threads can activate power-hungry cores, contend for cache, heat the phone, and reduce sustained
speed. “Use every core” is a hypothesis, not a rule.

<a id="ch17-soc"></a>

## 17.4 Major mobile SoC paths

<a id="tbl-ch17-02"></a> **Table 17-2 — Accelerator paths at the research cut-off (`DOC-VERIFIED`; exact tuples remain
`TEST-REQUIRED`)**

<!-- markdownlint-disable MD013 -->
| Family              | Hardware                                  | Current documented path                                   | Compilation implication                                     | Principal trap                                       |
| ------------------- | ----------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| Qualcomm Snapdragon | Arm CPU, Adreno GPU, Hexagon HTP/NPU      | QAIRT/QNN and Genie; LiteRT, ExecuTorch, ORT integrations | quantized/fixed shapes and HTP-specific contexts may apply  | context for another HTP architecture can be invalid  |
| MediaTek Dimensity  | Arm CPU, mobile GPU, MediaTek NPU         | NeuroPilot; LiteRT/ExecuTorch paths                       | compiler/runtime must match platform and operator set       | “Dimensity” spans incompatible generations           |
| Google Tensor       | Arm CPU, GPU, Tensor TPU                  | Tensor ML SDK integrated with LiteRT                      | current public path is limited to documented device cohorts | owning a Pixel does not prove third-party TPU access |
| Apple silicon       | Arm64 CPU, Apple GPU, Apple Neural Engine | Core ML for CPU/GPU/ANE; Metal or MLX for GPU             | Core ML specializes per device; other paths differ          | `.all` does not prove complete ANE execution         |
<!-- markdownlint-restore -->

### 17.4.1 Qualcomm Snapdragon, Hexagon, and QAIRT

Snapdragon is a platform family. A SoC can combine Arm-compatible CPUs, Adreno GPU, image/signal processors, and a
Hexagon Tensor Processor (HTP). Qualcomm AI Runtime (QAIRT)—whose documentation also uses Qualcomm Neural Network (QNN)
and earlier AI Engine Direct terminology—exposes CPU, GPU, and HTP backends. Genie adds a generative-AI workflow.

`DOC-VERIFIED`: Qualcomm says its documented HTP quant runtime supports quantized 8- and 16-bit networks, while
floating-point/BF16 availability is platform-dependent. It warns that a context binary for a different HTP architecture
can have indeterminate results [S4]. Its LLM tutorial selects a chipset ID because this configures architecture and
optimization defaults [S5].

Treat SDK, SoC ID, HTP architecture, runtime libraries, context binary, quantization, shapes, and firmware as a release
unit. Inspect exact operator/precision tables, profile CPU↔HTP boundaries, preserve a tested CPU fallback, and
separately review SDK redistribution terms. ExecuTorch-QNN, ORT-QNN, and LiteRT-QNN simplify integration but do not make
artifacts portable to another vendor.

### 17.4.2 MediaTek NeuroPilot

MediaTek combines Arm-compatible CPUs, mobile GPUs, and NPUs exposed through **NeuroPilot**. Public software includes
conversion, quantization, compilation, and execution; LiteRT and ExecuTorch document MediaTek paths.

`DOC-VERIFIED`: LiteRT lists specific supported Dimensity SoCs, not every MediaTek phone, and documents AOT and
on-device compilation [S9]. MediaTek's official Genio documentation shows that NeuroPilot and MediaTek Deep Learning
Accelerator (MDLA) versions have fixed operator restrictions and need matching host tools; its generative workflow
quantizes, converts, compiles, then executes through Neuron Runtime [S10]. Genio is not proof of phone support; it
demonstrates why “MediaTek NPU” is not one target.

Query the exact phone/SoC, current public support list, and applicable commercial SDK. Never infer compatibility from
another Dimensity generation.

### 17.4.3 Google Tensor TPU

Google Tensor SoCs include a TPU for on-device inference. Internal Google feature access has not historically implied
equivalent third-party access.

`DOC-VERIFIED`: in May 2026 Google described Tensor ML SDK as Beta, integrated with LiteRT, and listed support for the
Pixel 10 family. The workflow compiles PyTorch/TFLite models, distributes required components, and permits CPU/GPU
fallback [S11]. LiteRT documents Google Tensor AOT support and says the current SDK does not yet provide on-device JIT
compilation [S8].

Gate TPU use by the current SDK device list, operators, license, distribution requirements, and measured placement.
Treat older Pixels as separate CPU/GPU cohorts unless official documentation says otherwise.

### 17.4.4 Apple silicon and Apple Neural Engine

Apple mobile SoCs combine Arm64 CPUs, Apple GPUs, Apple Neural Engine (ANE), media engines, caches, and shared memory.
Third-party apps normally reach ANE through **Core ML**, not a public low-level ANE instruction API. GPU paths use
**Metal** directly or through MLX/other runtimes.

Core ML schedules graph segments across eligible CPU, GPU, and ANE units. `computeUnits` states what may be used, not
exact placement. Apple tooling reports actual layer placement [S12–S14]:

- `.all` permits all eligible units; it does not prove ANE-only execution;
- `.cpuAndNeuralEngine` can avoid GPU contention but can change partitioning;
- `.cpuOnly` is a useful correctness baseline;
- Metal/MLX GPU inference does not use ANE merely because ANE exists; and
- Core ML, Metal, and MLX use distinct artifacts/execution paths.

Core ML state can retain KV tensors across predictions on supported deployment targets, avoiding explicit cache round
trips; verify exact OS and conversion requirements when exporting [S15].

<a id="ch17-gpu"></a>

## 17.5 Mobile GPUs, Vulkan, and Metal

### 17.5.1 GPU behavior

A mobile GPU provides many arithmetic lanes for throughput. It works best with abundant parallel work, coalesced access,
and kernels large enough to amortize launch/synchronization. Transformer **prefill** exposes parallel matrix work;
single-token **decode** uses smaller operations and repeatedly streams weights, often becoming bandwidth- and
launch-limited.

Mobile GPUs usually share system memory rather than owning discrete VRAM. This avoids PCIe copies but does not remove
caches, alignment, synchronization, or contention. “Zero-copy” avoids a copy; it does not make data movement free.

Advantages include FP16/FP32 throughput, programmable coverage, cross-vendor Android access through Vulkan, and native
Apple access through Metal. Limits include driver variability, inconsistent integer/low-bit support, packed-weight
expansion, contention with rendering, heat, and dispatch overhead.

### 17.5.2 Vulkan

Vulkan is a cross-platform graphics/compute API, not an ML runtime, tokenizer, or quantization format. An engine
packages SPIR-V shaders, binds buffers/descriptors, records commands, and dispatches to the GPU.

`DOC-VERIFIED`: Khronos documents compute queues/pipelines and `vkCmdDispatch`. Its Android guidance requires capability
queries because workgroup limits, shared memory, features, and drivers vary; it requires graceful CPU fallback
[S16–S18].

Verify Vulkan version/extensions, numeric types, subgroup features, workgroup and shared-memory limits, buffer
mapping/synchronization, pipeline-cache behavior, and output parity.

### 17.5.3 Metal

Metal is Apple's low-overhead graphics/compute API. A compute function is compiled into a pipeline, buffers are bound,
and thread grids are dispatched through command buffers [S19–S20]. Metal controls the GPU, not ANE.

Pipeline creation is expensive enough to cache; workgroups should follow queried execution width and maximum limits.
Compare a Metal/MLX path with Core ML: the former offers flexible GPU kernels, while Core ML may select ANE and apply
device specialization.

<a id="ch17-android"></a>

## 17.6 Android: NNAPI and current vendor acceleration

### 17.6.1 NNAPI's historical role

Android Neural Networks API (NNAPI) arrived in Android 8.1 as a C API beneath frameworks. It could partition graphs over
vendor CPU, GPU, DSP, or NPU drivers, with CPU execution where a specialized driver was absent.

`DOC-VERIFIED`: Android deprecated NNAPI in Android 15. Google says rapidly changing transformer/diffusion workloads
require frequently updated infrastructure and directs performance-sensitive work to alternatives such as LiteRT.
Android's NNAPI page warns that future devices are expected mostly to use the CPU backend [S6–S7].

Existing tested systems may retain NNAPI temporarily. A new product should not make it the future acceleration contract.

### 17.6.2 Runtime plus vendor backend

```text
application / generation layer
  -> LiteRT-LM, ExecuTorch, ORT GenAI, llama.cpp, MLC, or Genie
     -> portable CPU or GPU backend
     -> optional Qualcomm / MediaTek / Google Tensor / Samsung backend
        -> vendor compiler + runtime + driver + accelerator
```

`DOC-VERIFIED`: LiteRT documents Google Tensor, Qualcomm, MediaTek, Intel, and Samsung NPU integrations. It
distinguishes ahead-of-time (AOT) from on-device compilation, recommends AOT for large known targets to reduce
initialization/memory, and permits fallback or partial delegation [S8]. Qualcomm and MediaTek pages publish explicit SoC
lists [S9].

ExecuTorch and ONNX Runtime likewise expose separate vendor backends. These are multiple integrations, not a universal
Android NPU ABI. Export backend-specialized artifacts, capture partition logs, and define unsupported-device behavior.

<a id="ch17-coreml"></a>

## 17.7 Core ML backends and specialization

At model load, Core ML can:

1. parse and optimize the graph;
2. segment operations by eligible device;
3. compile segments for selected units; and
4. cache specialized artifacts.

`DOC-VERIFIED`: Apple says uncached specialization can be significant; caches are tied to model path/configuration and
can disappear after storage pressure, system update, or model change [S14].

For autoregressive SLMs:

- provide efficient prefill and decode; one-token-only decode makes prompt ingestion poor;
- use stateful KV where supported, or account for explicit cache I/O;
- test shape policies because flexibility can affect backend eligibility;
- inspect Core ML reports/Instruments instead of assuming ANE placement;
- own tokenization, template, sampling, stop detection, and incremental Unicode handling; and
- measure uncached compile, cached load, first prefill, and steady decode separately.

<a id="ch17-memory"></a>

## 17.8 Capacity, bandwidth, and RAM pressure

### 17.8.1 Peak memory

$$
M_{\text{peak}} \approx M_{\text{weights}} + M_{\text{KV}} + M_{\text{activations}} + M_{\text{workspace}} +
M_{\text{compiled}} + M_{\text{runtime}} + M_{\text{app}} + M_{\text{headroom}}.
$$

All terms are bytes. Model-file size is absent because a file may be mapped, copied, decompressed, repacked, or
compiled.

For $P$ parameters at $b$ bits, ideal raw weight storage is:

$$
M_W = \frac{Pb}{8}.
$$

**Example (`ILLUSTRATIVE`).** A 1.5B-parameter model at ideal 4-bit storage uses $1.5\times10^9\times4/8=0.75$ GB
before scales, zero points, alignment, metadata, mixed-precision tensors, repacking, or duplicate buffers. It is a
component lower bound, not “RAM required.”

A conventional decoder KV cache is:

$$
M_{\text{KV}} = L \times T \times 2 \times H_{\text{kv}} \times D_h \times B \times S,
$$

where $L$=layers, $T$=cached tokens, 2=keys plus values, $H_{\text{kv}}$=KV heads, $D_h$=head dimension,
$B$=bytes per cache element, and $S$=sequences.

**Example (`ILLUSTRATIVE`).** With 24 layers, 4 KV heads, dimension 128, FP16 ($B=2$), 4,096 tokens and one sequence:

$$
24\times4096\times2\times4\times128\times2 =201{,}326{,}592\text{ bytes}\approx192\text{ MiB}.
$$

Doubling context doubles this cache. Verified 8-bit KV ideally halves payload, but scales, conversion, quality, and
backend support still matter.

### 17.8.2 Bandwidth

Arithmetic intensity is $I=\text{operations}/\text{bytes moved}$. If decode streams $M_W$ bytes through effective
bandwidth $BW_{\text{eff}}$:

$$
t_{\text{decode}} \gtrsim \frac{M_W}{BW_{\text{eff}}}.
$$

With the illustrative 0.75 GB payload and **illustrative effective** 30 GB/s, traffic alone takes at least 25 ms. This
is not a phone prediction: caches, metadata, repeated reads, activations, contention, fusion, and achieved bandwidth
alter it. It explains why low-bit weights can help decode even when compute is plentiful.

Never substitute peak vendor bandwidth for $BW_{\text{eff}}$; measure the complete runtime at sustained conditions.

### 17.8.3 Unified memory and pressure

Apple describes CPU and GPU sharing physical memory, avoiding discrete-GPU PCIe copies [S13]. Android SoCs also commonly
use shared system DRAM. Unified memory avoids some copies but:

- all clients compete for capacity/bandwidth;
- caches and synchronization remain;
- APIs may allocate private/repacked intermediates;
- accelerator SRAM is limited;
- mapped files consume resident pages when touched; and
- the OS can terminate an app under pressure.

Total phone RAM is not app-available RAM. Reserve measured headroom; inspect resident/proportional/native memory; bound
context, prefill chunks, and workspaces; avoid duplicate model instances during update; test backgrounding and pressure;
and fail safely instead of crashing or silently invoking cloud inference.

<a id="ch17-init"></a>

## 17.9 On-device model initialization

Initialization may open/validate an artifact, hash it, map/read weights, allocate KV/workspaces, repack weights, compile
Vulkan/Metal pipelines, load NPU contexts, specialize Core ML, initialize tokenizers, and warm pages/kernels.

Therefore **install size**, **download size**, **virtual size**, **resident memory**, **peak initialization memory**,
**uncached load**, **cached load**, and **time to first token** are different metrics.

Engineering controls:

- never block the UI thread;
- validate signature/hash/version/ABI before large allocation;
- use AOT compilation for known large-model/SoC cohorts when supported;
- cache compiled assets but tolerate eviction;
- serialize or budget concurrent initialization;
- keep the model loaded only when reuse justifies its resident cost;
- bound warm-up energy; and
- test first install, OS/model update, cache eviction, low storage, and cancellation.

<a id="ch17-power"></a>

## 17.10 Power and thermal design

Phone processors use Dynamic Voltage and Frequency Scaling (DVFS). Frequencies and cores depend on load, battery, skin
temperature, charging, and competing work. A burst benchmark may use an unsustainable power state.

Energy per output token can be estimated:

$$
E_{\text{token}} = \frac{\int_{t_0}^{t_1}(P(t)-P_{\text{idle}})\,dt}{N_{\text{generated}}},
$$

where $P(t)$=device power, $P_{\text{idle}}$=declared baseline, and $N_{\text{generated}}$=generated tokens. Label
OS-counter measurements and uncertainty when external instrumentation is unavailable.

Benchmark at declared ambient/start temperature, case, charge, charging state, and background load. Run to steady state;
log throughput and temperature over time; compare thread counts; cap/cancel output; respond to OS thermal state; and
test GPU contention with UI/camera. Never disable thermal controls.

An NPU may be efficient for a supported graph, but transfers, fallback, or initialization can negate this. End-to-end
sustained latency and energy decide.

<a id="ch17-limits"></a>

## 17.11 Backend limits

<a id="tbl-ch17-03"></a> **Table 17-3 — Common constraints and required proof**

| Constraint                    | Symptom                            | Required verification                        |
| ----------------------------- | ---------------------------------- | -------------------------------------------- |
| unsupported operator/fusion   | partitioning or conversion failure | operator list and partition trace            |
| static context/batch          | excess allocation or rejection     | exported signatures and boundary tests       |
| precision restriction         | rejection or silent widening       | placement/precision report and memory trace  |
| layout/alignment              | repack and high temporary memory   | allocation trace and cold-load peak          |
| tokenizer/template mismatch   | plausible but wrong output         | token-ID/template golden tests               |
| unsupported RoPE/attention/KV | wrong logits or fallback           | source-target logits across cache boundaries |
| custom operation              | CPU island                         | partition report and transfer timing         |
| version coupling              | one-firmware-only behavior         | device/OS/driver matrix and rollback         |
| accelerator contention        | UI jank                            | concurrent rendering/camera test             |
| lifecycle restriction         | cache loss or process death        | pressure/background tests                    |

Autoregressive models are difficult because prefill/decode shapes differ, KV state changes every token, sampling has
control flow, and architectures use GQA/MQA, RoPE, gated MLPs, sliding windows, or MoE. Framework support is not
exact-checkpoint support.

Partial delegation can lose to CPU-only execution when every token crosses processors. Capture placement and compare
against an optimized CPU baseline.

<a id="ch17-quantization"></a>

## 17.12 Hardware controls quantization and runtime choice

Use this order:

1. identify device cohorts;
2. enumerate maintained runtimes/backends;
3. inspect architecture, operator, shape, and precision support;
4. create only formats with real kernels;
5. test quality and tokenizer parity;
6. measure peak memory, cold start, latency, energy, and thermal stability; and
7. retain Pareto-optimal tuples.

<a id="tbl-16"></a> **TBL-16 — CPU versus GPU versus NPU (`STATICALLY REVIEWED`; guidance, not a measured ranking or
universal support statement)**

<!-- markdownlint-disable MD013 -->
| Target                    | Promising first candidate                                     | Why                                           | Reject when                                       |
| ------------------------- | ------------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------- |
| Arm64 CPU/LLM kernels     | grouped 4–6-bit weights; wider activations; optional 8-bit KV | lower weight traffic                          | no exact kernel; dequant dominates; quality fails |
| CPU with INT8 dot kernels | INT8 weights/activations or mixed INT8                        | vector dot products                           | calibration harms quality or tensors widen        |
| Mobile GPU                | FP16 or GPU-native packed low-bit                             | strong half precision, programmable unpacking | expansion/driver/heat loses                       |
| Vendor NPU                | exact vendor INT8/INT16/FP16 recipe                           | compiler maps approved graph                  | fallback, unsupported cache, SoC coupling         |
| Apple Core ML             | supported palettization/quantization and state                | scheduler may use CPU/GPU/ANE                 | selected unit expands compression or misses ANE   |
| Metal/MLX                 | runtime-native grouped format or FP16                         | flexible Apple GPU kernels                    | ANE efficiency required or architecture absent    |
<!-- markdownlint-restore -->

Weight-only, weight-and-activation, and KV-cache quantization change different tensors. Artifact compression expanded at
load is not low-precision execution. Mixed precision can preserve sensitive tensors. Sparsity helps only with matching
patterns and kernels.

Multiple artifacts are often correct: portable CPU GGUF, Vulkan/Metal variant, Core ML package, and SoC-compiled NPU
programs. Share the source checkpoint and evaluation suite rather than forcing one binary everywhere.

<a id="ch17-tiers"></a>

## 17.13 Low-, mid-, and flagship-phone planning

Price and marketing tier vary by market/year. Define tiers by measured workload capability.

<a id="tbl-ch17-05"></a> **Table 17-5 — Capability-based cohorts (`ENGINEERING RECOMMENDATION`; no universal
specifications)**

<!-- markdownlint-disable MD013 -->
| Cohort                       | Admission evidence                                                                                       | Initial strategy                                                                                       | Guardrail                                                                           |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| **Low capability**           | Arm64 CPU passes correctness; accelerator unavailable or loses; limited tested headroom/thermal duration | smallest acceptable model, mature CPU runtime, validated aggressive weight quantization, short context | pre-download check, hard context/memory cap, graceful unsupported result            |
| **Mid capability**           | CPU has headroom; documented GPU/NPU path passes parity and improves one objective                       | compare CPU low-bit with GPU and vendor backend; bounded prefill chunks                                | retain CPU fallback; reject harmful partitioning; thermal degradation policy        |
| **High/flagship capability** | safe measured peak; maintained exact-SoC path; sustained and cold-start tests pass                       | test larger model/context or better-quality quantization; separate optimized artifact                  | exact model/OS gate; concurrent camera/UI tests; “flagship” never bypasses evidence |
<!-- markdownlint-restore -->

Admission records OS/ABI, exact hardware/SoC, backend versions, compute capabilities, artifact compatibility,
memory/storage headroom, quality parity, sustained latency/energy/thermal behavior, and lifecycle recovery.

**Example (`ESTIMATE`).** If measured peak is 1.45 GiB, reserve 0.55 GiB for UI/runtime variation and 25% uncertainty:

$$
M_{\text{admission}}=(1.45+0.55)\times1.25=2.50\text{ GiB}.
$$

Require 2.50 GiB of tested allocatable headroom under the scenario—not “a 4 GB phone.” Total RAM does not prove
headroom.

<a id="ch17-domain"></a>

## 17.14 Specialist offline SLM implications

Hardware optimization does not change the knowledge boundary. Automotive, pharmacy, beekeeping, or repair SLMs rely on
learned parameters and can hallucinate or become stale; the primary architecture does not use runtime retrieval.

Domain text can increase token count and KV work; long diagnostic histories increase prefill/cache; aggressive
quantization can harm rare terminology and refusals; process death loses state; and unsupported devices must not
silently route an offline/privacy promise to cloud inference. Run the same domain, uncertainty, refusal, and safety
evaluation on every compressed/backend variant. A faster artifact that changes a contraindication or safety procedure
fails.

<a id="ch17-verification"></a>

## 17.15 Capability verification procedure

This is a `PLANNED EXPERIMENT`, not an execution claim.

### A. Immutable inventory

Record phone/model, SoC, RAM class, OS/firmware; runtime/exporter/compiler/vendor SDK/backend; checkpoint/tokenizer
revisions; quantization/group/scales/KV precision; artifact hash/size/context/shapes; app ABI/libraries; ambient
temperature, case, battery, charging, display, and background load.

<a id="1719-honest-completion-status"></a>

### B. Static compatibility

1. Query CPU features; do not infer optional instructions from arm64.
2. Query Vulkan/Metal/Core ML or vendor capabilities.
3. compare exact SoC with the current official support list.
4. validate artifact manifest/hash before loading.
5. confirm licenses and redistribution rights.

### C. Numerical and language correctness

1. Compare token IDs for domain text, Unicode, emoji, whitespace, units, and special tokens.
2. Render the training template byte-for-byte.
3. Compare logits for one-token, short/long prefill, cached decode, reset, and context boundary.
4. Run unchanged domain/safety tests before and after quantization.
5. Test stops and incremental UTF-8 decoding.

### D. Prove placement

Capture graph partitions, fallback operations, boundaries, and tensor precision/layout where possible. Compare optimized
CPU-only, GPU, NPU, and mixed paths. Reject acceleration that does not improve the declared objective.

### E. Memory and initialization

Measure artifact/download size, virtual/resident/proportional/peak memory, uncached specialization, cached load, first
prefill, time to first token, steady decode across contexts, unload/recovery, and low-storage/low-memory failure.

### F. Sustained power/thermal behavior

Repeat fixed prompts for 10–30 minutes or to declared steady state. Log throughput, TTFT, power source/counter, battery,
thermal state, available frequency, errors, and temperature. Repeat trials and report dispersion.

### G. Lifecycle/release

Test cancellation, screen lock, background/foreground, activity recreation, calls/notifications, camera/GPU contention,
memory pressure, interrupted/corrupt download, cache eviction, model update, and rollback. Release an exact tested
model/runtime/device cohort, never a runtime brand.

### 17.15.1 Worked hardware-path walkthrough

Assume one transformer decode step contains packed low-bit linear layers, RoPE, normalization, attention over an
existing KV cache, logits, and sampling. Follow the data rather than the processor label:

1. The CPU tokenizes input and starts the runtime. This control work remains even if tensor operators are delegated.
2. A CPU path reads packed weights, unpacks groups, multiplies them with activations using Neon or optional dot-product
   instructions, and accumulates in a wider type. It wins only if its kernel saves more memory traffic than unpacking
   costs.
3. A GPU path first creates/caches pipelines and binds shared-memory buffers. Prefill can expose enough parallel work
   to amortize dispatch; one-token decode may not. “Unified memory” avoids a PCIe copy but still requires residency,
   synchronization, and bandwidth.
4. An NPU path compiles supported graph regions for fixed precision and shape rules. If RoPE or the cache update is
   unsupported, a tensor returns to CPU and may be cast or rearranged at each boundary.
5. The CPU samples the next token and the cycle repeats. A per-token CPU↔NPU crossing that looked small in one graph
   invocation can dominate a 200-token answer.

**Worked decision (`ILLUSTRATIVE`).** A 0.75 GB packed payload and an illustrative 30 GB/s effective path have a
traffic-only lower bound of 25 ms per full payload read. This does not predict 40 token/s: KV reads, scale metadata,
kernel work, cache misses, synchronization, and thermal frequency changes remain. If a GPU trace later shows 18 ms of
compute plus 12 ms of dispatch/synchronization, while an optimized CPU path takes 27 ms end to end, the CPU is the
better decode choice despite the GPU's higher peak arithmetic. Prefill may still use the GPU if an equivalent trace
shows a net gain.

Expected evidence is therefore phase-specific:

| Claim | Evidence needed | Common mistaken inference |
| --- | --- | --- |
| “Neon accelerated” | dispatched kernel and exact CPU features | every Arm64 CPU has the same extensions |
| “GPU accelerated” | operator trace plus synchronized end-to-end timing | a Vulkan device exists |
| “NPU accelerated” | partition report, boundary costs, and counters | an NPU is advertised |
| “4-bit saves RAM” | loaded and peak memory by phase | the file is four bits per parameter |
| “flagship supported” | exact device/OS/runtime cohort passes sustained tests | retail tier predicts compatibility |

<a id="fig-ch17-02"></a> **Figure 17-2 — Hardware-aware release gate (`ILLUSTRATIVE`).**

```text
source model
  -> tokenizer/logit parity --fail--> fix/reject
  -> placement proof --------fail--> CPU path/reject
  -> peak RAM/cold start -----fail--> smaller model/context
  -> domain quality/safety ---fail--> change model/quantization
  -> sustained energy/thermal fail--> retune backend/threads
  -> lifecycle/update --------fail--> harden product
  -> release exact cohort
```

<a id="ch17-conclusions"></a>

## 17.16 Engineering conclusions

1. Arm64 is an ISA/ABI target, not a performance specification.
2. Neon and optional dot products matter only through tuned kernels and runtime dispatch.
3. CPU is the portability baseline and remains involved in accelerator paths.
4. GPU API availability does not prove efficient low-bit transformer kernels.
5. NPUs help only when graph, shape, precision, compiler, driver, and SoC align.
6. Qualcomm, MediaTek, Google, and Apple require distinct software/artifacts.
7. NNAPI is deprecated from Android 15; use maintained explicit vendor paths.
8. Core ML's permitted compute units are not placement evidence.
9. Unified memory removes some copies, not contention or RAM pressure.
10. File size, resident memory, peak initialization memory, and KV growth differ.
11. Quantization is a kernel/hardware decision, not merely file compression.
12. Device tiers must be measured cohorts; sustained behavior matters more than burst peaks.

<a id="ch17-exercises"></a>

## 17.17 Review questions and exercises

1. Explain why AArch64, Neon, Snapdragon, QNN, and GGUF occupy different layers.
2. Why can a 4-bit artifact consume much more than four bits per parameter at runtime?
3. Contrast unified memory with unlimited memory and zero-cost access.
4. Why may prefill favor a GPU while decode remains bandwidth-bound?
5. What does Core ML `.all` prove, and what does it not prove?
6. Why is NNAPI not the recommended foundation for a new Android design?
7. Give three ways partial NPU delegation can lose to CPU.
8. Calculate ideal weight sizes for 0.5B, 1.5B, and 3B parameters at 16, 8, 6, and 4 bits; list omitted overheads.
9. For $L=28$, $H_{\text{kv}}=4$, $D_h=128$, FP16, one sequence, calculate KV at 2,048 and 8,192 tokens;
   recalculate ideal 8-bit payload.
10. A path reads 1.1 GB/token at 22 GB/s effective bandwidth. Compute the traffic-only lower bound and explain why it is
    not a performance prediction.
11. Using official documentation, build a Qualcomm/MediaTek/Google/Apple matrix of backend, precision, OS, and compiler
    requirements; mark unresolved cells `TEST-REQUIRED`.
12. Define a decision rule when GPU wins for one minute but CPU wins after throttling.
13. Propose how to measure processor-boundary cost from a partition report.
14. Define three device cohorts for a 1B model without retail price or total RAM as the sole rule.
15. Design a tokenizer-parity release gate.
16. **Practical assignment:** execute [§17.15](#ch17-verification) on one physical phone for a 0.5B–1.5B model. Submit
    immutable manifests, hashes, raw logs, partition evidence, memory traces, three repeated trials, and a sustained
    run. Label only actual results `EXECUTED` and do not generalize beyond the tuple.

<a id="ch17-sources"></a>

## 17.18 Official source register

Sources are first-party architecture/platform/vendor materials accessed 2026-11-23. `DOC-VERIFIED` verifies
documentation, not runtime behavior.

### Arm

- **[S1] Arm Ltd.** *Arm Architecture Reference Manual for A-profile architecture*, A64 Advanced SIMD.
  <https://developer.arm.com/documentation/ddi0487/>
- **[S2] Arm Ltd.** *Coding for Neon*, Issue 04, 2020.
  <https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Neon%20Programmers%20Guide/102159_0104_01_CodingForNeon.pdf>
- **[S3] Arm Ltd.** *SDOT, A64 instruction reference*. <https://developer.arm.com/documentation/ddi0487/latest/>

### Qualcomm

- **[S4] Qualcomm.** *QAIRT/QNN HTP backend documentation*.
  <https://docs.qualcomm.com/nav/home/htp_backend.html?product=1601111740009302>
- **[S5] Qualcomm.** *LLM Inference on HTP*, updated 2026-07-08.
  <https://docs.qualcomm.com/doc/80-87189-2/topic/genai_builder.html?product=1601111740009302>
- Qualcomm. *Use a GenAI model with Genie*.
  <https://docs.qualcomm.com/doc/80-80020-15B/topic/use-genai-model-with-genie.html>

### Android and LiteRT

- **[S6] Android Developers.** *NNAPI Migration Guide*, updated 2026-03-06.
  <https://developer.android.com/ndk/guides/neuralnetworks/migration-guide>
- **[S7] Android Developers.** *Neural Networks API*. <https://developer.android.com/ndk/guides/neuralnetworks>
- **[S8] Google AI Edge.** *NPU acceleration with LiteRT*. <https://developers.google.com/edge/litert/next/npu>
- **[S9] Google AI Edge.** *Qualcomm NPU with LiteRT*; *MediaTek NPU with LiteRT*.
  <https://developers.google.com/edge/litert/next/qualcomm> and
  <https://developers.google.com/edge/litert/next/mediatek>
- Google AI Edge. *LiteRT-LM overview*. <https://developers.google.com/edge/litert-lm/overview>
- PyTorch. *ExecuTorch backend overview*. <https://docs.pytorch.org/executorch/stable/backends-overview.html>

### MediaTek and Google Tensor

- **[S10] MediaTek.** *IoT AI Hub software architecture*; *AI development resources*.
  <https://genio.mediatek.com/doc/iot-aihub/ai_hub/software_architecture.html> and
  <https://genio.mediatek.com/doc/iot-aihub/ai_hub/related_resource.html>
- MediaTek. *NeuroPilot public documentation*.
  <https://neuropilot-developer.mediatek.com/resources/public/latest/en/docs/readme>
- **[S11] Google Developers Blog.** Patel and Roy, *Google Tensor SDK Beta with LiteRT*, 2026-05-19.
  <https://developers.googleblog.com/en/google-tensor-sdk-beta-with-litert/>

### Apple

- **[S12] Apple.** *Optimize your Core ML usage*, WWDC22-10027.
  <https://developer.apple.com/videos/play/wwdc2022/10027/>
- **[S13] Apple.** *Explore the new system architecture of Apple silicon Macs*, WWDC20-10686.
  <https://developer.apple.com/videos/play/wwdc2020/10686/>
- **[S14] Apple.** *Improve Core ML integration with async prediction*, WWDC23-10049.
  <https://developer.apple.com/videos/play/wwdc2023/10049/>
- **[S15] Apple.** *Deploy ML and AI models on-device with Core ML*; *MLModel.makeState*.
  <https://developer.apple.com/videos/play/wwdc2024/10161/> and
  <https://developer.apple.com/documentation/coreml/mlmodel/makestate()>
- **[S19] Apple.** *Performing calculations on a GPU*.
  <https://developer.apple.com/documentation/metal/performing-calculations-on-a-gpu>
- **[S20] Apple.** *Metal Shading Language Specification*.
  <https://developer.apple.com/metal/Metal-Shading-Language-Specification.pdf>

### Vulkan

- **[S16] Khronos Group.** *Vulkan Specification*. <https://docs.vulkan.org/spec/latest/>
- **[S17] Khronos Group.** *Vulkan Guide—Compute Shaders*. <https://docs.vulkan.org/guide/latest/compute_shaders.html>
- **[S18] Khronos Group.** *Compute on Android*.
  <https://docs.vulkan.org/tutorial/latest/Advanced_Vulkan_Compute/12_Mobile_and_Embedded_Compute/02_android_compute.html>

### Project research dependency

- [Edge runtime survey](../../04-research-comparison-report/edge-runtime-survey.md), researched 2026-11-23. It covers
  formats, tokenizer/KV/streaming behavior, licensing, maintenance, and a wider runtime matrix.

<a id="ch17-open-items"></a>

## 17.19 Verification note

**Authored and documentation-verified:** all prompt-required conceptual topics; architecture diagrams; comparison
tables; calculations; capability procedures; official sources; exercises; and explicit device-tier guidance.

**Not executed:** conversion, code, benchmarks, placement traces, parity tests, energy/thermal measurements,
RAM-pressure tests, and lifecycle tests.

Before production claims: choose immutable model/tokenizer revisions; test representative physical phones; confirm
vendor SDK/license terms; verify operators, shapes, precision, and KV behavior; measure peak memory, initialization,
latency, energy, and sustained thermal behavior; run domain/safety regression for every artifact; and refresh
time-sensitive NNAPI, LiteRT, Tensor SDK, Core ML, and vendor-device-list status.
