# Edge Large Language Model (LLM) runtime survey

**Research cut-off:** 2026-11-23  
**Scope:** offline autoregressive small language model (SLM) inference on Android, iOS, and adjacent edge targets  
**Evidence status:** `DOC-VERIFIED` unless a statement is explicitly marked `VENDOR-REPORTED`, `STATICALLY REVIEWED`, or
`TEST-REQUIRED`

## 1. Evidence boundary and reading rules

This survey was researched against current first-party documentation, repositories, licenses, and release pages on
2026-11-23. `DOC-VERIFIED` means that the linked official source states or demonstrates the capability. It does **not**
mean that this project reproduced the capability, that every model in a named family exports, or that performance is
acceptable on an arbitrary device. No runtime was executed and no physical mobile device was tested for this survey.

The following distinctions are essential:

- **Framework support is not model support.** A runtime may implement Android, Metal, Vulkan, or an NPU backend while
  still lacking an operator, graph shape, tokenizer, attention variant, or quantization used by a particular checkpoint.
- **An artifact that converts is not necessarily correct.** Logit parity, tokenizer parity, stop-token behavior,
  chat-template behavior, long-context behavior, and quality after quantization require tests.
- **A backend being selectable is not proof of full offload.** Unsupported graph partitions can fall back to CPU,
  sometimes making an accelerated configuration slower or more memory hungry.
- **Model-file size is not peak resident memory.** Weights, Key-Value (KV) cache, activations, temporary buffers,
  tokenizer state, compiled programs, and app memory all count.
- **Vendor benchmark numbers are not portable.** Any performance number in an official release remains `VENDOR-REPORTED`
  unless reproduced under this project's benchmark protocol.
- **Runtime license and model/data license are separate.** A permissively licensed engine does not change checkpoint,
  tokenizer, conversion-tool, SDK, or training-data obligations.

### Comparative lesson: a runtime is an execution contract

Training frameworks optimize flexibility; phones optimize fixed memory, power, and vendor hardware. Edge runtimes arose
to convert model graphs and weights into representations that device kernels can execute. The intuition is that a
runtime is not a universal media player. It is a contract among an exact model, converter, artifact, tokenizer,
Key-Value (KV) cache design, backend, operating system, and device.

llama.cpp interprets supported GGUF model containers with integrated tokenization and generation. ExecuTorch exports
PyTorch graphs into backend-specialized `.pte` programs. LiteRT-LM packages graph execution with language-model
orchestration. Core ML lets Apple schedule compatible operations across CPU, GPU, and Apple Neural Engine (ANE). A
Qwen checkpoint that loads in GGUF on an Android CPU is therefore not automatically compatible with a LiteRT Neural
Processing Unit (NPU) path. Each route uses different conversion, quantization, graph, and tokenizer assumptions.

For an offline mechanic assistant, first establish tokenizer and output correctness with llama.cpp on ARM CPU. Then
compare a graph runtime such as ExecuTorch XNNPACK and an officially supported LiteRT-LM route. Add a Snapdragon NPU
artifact only for a defined device cohort. On iOS, compare llama.cpp Metal, MLX Swift LM, and stateful Core ML. No
performance number is assumed; the exercise isolates portability, native acceleration, and maintenance cost.

CPU-first runtimes maximize coverage but may consume more energy. Compiler and graph runtimes can exploit accelerators
but require backend-specific exports and can silently fall back. Vendor runtimes may deliver the best device-specific
placement while creating the narrowest portability and strongest version coupling. Use a portfolio with a tested
portable fallback and optional cohort-specific accelerators. Avoid selecting from platform check marks, conversion
success, or vendor benchmarks alone.

### Status vocabulary used in the matrices

- **D:** `DOC-VERIFIED` by an official source linked in this report.
- **P:** documented preview, beta, experimental, or otherwise not a stable production API.
- **T:** `TEST-REQUIRED`; compatibility or usable acceleration is unresolved for the exact model/device.
- **—:** no first-party support was found in the reviewed sources; this is not proof that a community integration does
  not exist.

## 2. Executive findings

1. **For the broadest low-friction CPU baseline, llama.cpp/GGUF remains the most practical first experiment.** It has
   direct GGUF loading, extensive low-bit formats, mature command-line/server tooling, an official Android example, iOS
   XCFramework release artifacts, and rapid model-family additions. Its principal production costs are a fast-moving
   API/release cadence, per-architecture conversion validation, and uneven mobile GPU/NPU paths.
2. **For a PyTorch-owned export path and heterogeneous vendor acceleration, ExecuTorch is the strongest general
   integration candidate.** Its `.pte` artifacts are backend-specialized; XNNPACK is the portable CPU baseline, while
   Core ML, Vulkan, Qualcomm, MediaTek, Samsung, and other delegates require separate export and target testing.
3. **LiteRT is the successor to TensorFlow Lite; LiteRT-LM is the LLM orchestration layer.** New work should not treat
   legacy TensorFlow Lite `Interpreter` examples as the current Google path. LiteRT-LM manages conversations,
   tokenization, KV cache, and streaming over LiteRT and has stable Kotlin/C++ APIs, while Swift was still labeled early
   preview in the current official overview.
4. **ONNX Runtime Mobile is a strong graph-runtime option, but autoregressive products should evaluate ONNX Runtime
   GenAI separately.** Core ORT supplies mobile packages and execution providers; ORT GenAI supplies tokenization,
   sampling, generation, KV management, and streaming. The GenAI API remains documented as preview, and mobile NPU
   support is constrained by model-specific export and fixed-shape/quantized requirements.
5. **Core ML is the native iOS deployment anchor, not a complete cross-platform LLM stack.** It can use Apple
   CPU/GPU/Neural Engine and supports stateful KV-cache models on iOS 18+, but applications must supply or adopt
   compatible tokenization, chat, sampling, streaming, and conversion logic. Core ML Tools is open source; the OS
   framework is an Apple platform API.
6. **MLX Swift LM is a compelling Apple-only rapid-development path.** It supplies model implementations, tokenizer
   integrations, streaming detokenization, several KV-cache strategies, and direct Swift app examples. It uses Apple
   GPU/CPU through MLX rather than the Apple Neural Engine (ANE), so Core ML remains the path to test when ANE use or
   Core ML packaging is required.
7. **MLC LLM is the strongest compiler-first cross-platform GPU candidate.** It provides Android/iOS/Web paths, compiled
   model libraries, converted weights, tokenizers, streaming, and explicit context/prefill memory controls. The
   compiler/package toolchain and target-specific artifacts add operational complexity.
8. **MNN and ncnn are no longer only conventional vision runtimes.** Both are actively maintained and now expose
   explicit LLM conversion, low-bit, KV-cache, and mobile GPU work. MNN has the more integrated LLM pipeline and
   emerging Qualcomm Hexagon path; ncnn is attractive for a small C++/Vulkan footprint but has a less turnkey generative
   application layer.
9. **Qualcomm QAIRT/QNN/Genie is the direct Snapdragon path but the least portable.** It can orchestrate CPU/GPU/HTP NPU
   execution and has dedicated GenAI tooling, but artifacts and SDK versions are hardware-specific and commercial SDK
   terms/deployment obligations require a separate legal and release review.
10. **There is no evidence-based universal winner.** The minimum useful bake-off is one CPU-first runtime, one OS-native
    or vendor-accelerated runtime, and one compiler/graph runtime on the exact target devices.

## 3. Platform and accelerator matrix

<!-- markdownlint-disable MD013 -->
| Runtime                     |                                                                            Android |                           iOS |                                       CPU |                                                                           Mobile GPU |                                                                  Mobile NPU | Primary deployment artifact                                                     | Evidence                                                                                                                                                                                                                       |
| --------------------------- | ---------------------------------------------------------------------------------: | ----------------------------: | ----------------------------------------: | -----------------------------------------------------------------------------------: | --------------------------------------------------------------------------: | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| llama.cpp / GGML / GGUF     |                                                                                  D |                             D |                                         D | D/T: Vulkan or OpenCL-family/backend dependent; Snapdragon Adreno package documented |       P/T: Snapdragon Hexagon package exists; exact coverage must be tested | `.gguf` plus native library/binary                                              | [llama.cpp README](https://github.com/ggml-org/llama.cpp/), [Android guide](https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/android.md), [releases](https://github.com/ggml-org/llama.cpp/releases)                       |
| ExecuTorch                  |                                                                                  D |                             D |                       D: XNNPACK/portable |                            D/T: Android Vulkan; Apple Core ML/experimental MLX paths |                  D/T: Qualcomm, MediaTek, Samsung and other vendor backends | backend-specialized `.pte` plus tokenizer                                       | [backend overview](https://docs.pytorch.org/executorch/stable/backends-overview.html)                                                                                                                                          |
| LiteRT Runtime              |                                                                                  D |                             D |                                         D |                                                                                    D | D/T: Android vendor NPU paths; no iOS NPU entry in current LiteRT-LM matrix | `.tflite`                                                                       | [LiteRT repository](https://github.com/google-ai-edge/LiteRT), [LiteRT overview](https://developers.google.com/edge/litert)                                                                                                    |
| LiteRT-LM                   |                                                                                  D |                             D |                                         D |                                                                                    D |                                         D/T: Android only in current matrix | `.litertlm` package containing LiteRT graphs and assets                         | [LiteRT-LM overview](https://developers.google.com/edge/litert-lm/overview)                                                                                                                                                    |
| ONNX Runtime Mobile + GenAI |                                                                                  D | P/T for GenAI; D for core ORT |                                         D |                                        D/T: NNAPI, Core ML, QNN GPU depending target |                           D/T: NNAPI/Core ML/QNN; QNN HTP constraints apply | `.onnx`/ORT-format graph(s), `genai_config.json`, tokenizer and provider assets | [mobile guide](https://onnxruntime.ai/docs/tutorials/mobile/), [ORT GenAI](https://onnxruntime.ai/docs/genai/)                                                                                                                |
| Core ML                     |                                                                                  — |                             D |                                         D |                                                                                    D |                          D: Apple Neural Engine, subject to graph placement | `.mlmodel`/`.mlpackage`, compiled by OS/Xcode                                   | [Core ML](https://developer.apple.com/documentation/coreml), [Core ML Tools](https://github.com/apple/coremltools)                                                                                                             |
| MLX / MLX Swift LM          |                                                                                  — |                             D |                                         D |                                                                         D: Apple GPU |                                                                           — | MLX/Safetensors-style weights plus Swift model/tokenizer integration            | [MLX](https://github.com/ml-explore/mlx), [MLX Swift LM](https://github.com/ml-explore/mlx-swift-lm)                                                                                                                           |
| MLC LLM                     |                                                                                  D |                             D | T: not the primary documented mobile path |                                            D: OpenCL/Vulkan on Android, Metal on iOS |                                                   — in reviewed mobile docs | converted MLC weights, `mlc-chat-config.json`, compiled model library           | [MLC LLM README](https://github.com/mlc-ai/mlc-llm/), [compile guide](https://llm.mlc.ai/docs/compilation/compile_models.html)                                                                                                 |
| MNN / MNN-LLM               |                                                                                  D |                             D |                                         D |                                                               D: OpenCL/Vulkan/Metal |                     D/T: Core ML, Hexagon and other backend maturity varies | `.mnn`, weight file, `.mtok`, configs and optional embedding file               | [MNN README](https://github.com/alibaba/MNN/blob/master/README.md), [LLM guide](https://mnn-docs.readthedocs.io/en/latest/transformers/llm.html)                                                                               |
| ncnn / ncnn_llm             |                                                                                  D |                             D |                                         D |                                                                            D: Vulkan |                                                      — in reviewed LLM path | ncnn `.param`/`.bin` and tokenizer/application assets                           | [ncnn README](https://github.com/Tencent/ncnn/blob/master/README.md), [releases](https://github.com/Tencent/ncnn/releases/)                                                                                                    |
| Qualcomm QAIRT/QNN/Genie    |                                                    D on supported Qualcomm targets |                             — |                             D on Qualcomm |                                                                           D: QNN GPU |                                                                  D: QNN HTP | QNN/context or GenAI Transformer binaries, tokenizer and JSON config            | [Genie workflow](https://docs.qualcomm.com/doc/80-80020-15B/topic/use-genai-model-with-genie.html)                                                                                                                             |
| WebLLM                      |                                                      Browser on compatible Android |     Browser on compatible iOS |                                         — |                                                                            D: WebGPU |                                                                           — | MLC/WebGPU model libraries and cached weights                                   | [WebLLM README](https://github.com/mlc-ai/web-llm/blob/main/README.md)                                                                                                                                                         |
| OpenVINO GenAI              | T: ARM64 CPU packaging is documented, app integration is not a primary mobile path |                             — |                                D: x86/ARM |                                                 D on supported Intel/desktop devices |                                 D on Intel Core Ultra NPU, not Android NPUs | OpenVINO IR plus tokenizer/config assets                                        | [OpenVINO GenAI install](https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-genai.html), [requirements](https://docs.openvino.ai/2026/about-openvino/release-notes-openvino/system-requirements.html) |
<!-- markdownlint-restore -->

The platform matrix is an entry-point map. “D” means an official source documents a path, not that an arbitrary
checkpoint runs. The portable CPU column favors llama.cpp, ExecuTorch XNNPACK, LiteRT, and ONNX Runtime; GPU and NPU
columns become increasingly backend- and device-specific. Artifact differences explain why one approved model commonly
produces several release children.

## 4. Format, conversion, tokenizer, KV-cache, and streaming matrix

<!-- markdownlint-disable MD013 -->
| Runtime        | Conversion boundary                                                          | Tokenizer and chat handling                                                                             | KV-cache controls                                                                      | Streaming surface                                                                             | Main compatibility trap                                                                              |
| -------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| llama.cpp      | supported source checkpoints → GGUF; quantization usually follows conversion | GGUF can embed vocabulary/tokenizer metadata and chat template; implementation parity still needs tests | configurable context and K/V types; offload and quantized cache options                | CLI/server and Android Kotlin `Flow`                                                          | a `.gguf` file can load while tokenizer, RoPE, attention, or template semantics are wrong            |
| ExecuTorch     | `torch.export` capture → backend lowering → `.pte`                           | HF JSON, SentencePiece, TikToken and Llama2.c formats in LLM runner                                     | model-owned cache, exported sequence/context metadata; backend recipes vary            | per-token callbacks in Android/iOS/C++ runners                                                | each backend normally needs its own export; unsupported partitions may fall back                     |
| LiteRT-LM      | PyTorch/TensorFlow/JAX/HF path → LiteRT graphs → `.litertlm`                 | package can carry SentencePiece or Hugging Face tokenizer; orchestration applies conversation logic     | LiteRT KV implementation, dynamic NPU growth documented, conversation managed by layer | high-level response APIs; language maturity differs                                           | legacy TFLite examples and new LiteRT/LiteRT-LM APIs are easy to mix incorrectly                     |
| ORT GenAI      | exporter/Optimum path → ONNX graph set and config                            | bundled GenAI tokenizer/chat-template APIs                                                              | GenAI owns cache; provider/model-specific graph layouts                                | `TokenizerStream` and token generation loop                                                   | generic ONNX conversion is insufficient for performant generation and provider-specific caches       |
| Core ML        | PyTorch/TensorFlow → Core ML Tools → ML Program/package                      | application responsibility unless a higher-level library is used                                        | iOS 18+ `MLState` supports in-place stateful KV                                        | application loops over predictions and detokenization                                         | converter success does not prove all operations land on GPU/ANE or generation is numerically correct |
| MLX Swift LM   | supported HF/MLX weights + Swift architecture implementation                 | selectable tokenizer/downloader integration; chat/session helpers                                       | simple, rotating, quantized, chunked, and architecture-specific caches                 | `streamResponse` and streaming detokenizer                                                    | architecture code must match checkpoint; GPU memory remains shared system memory                     |
| MLC LLM        | HF model → MLC weights/config + target-compiled library                      | package builds HF/SentencePiece tokenizer libraries and conversation config                             | context window and prefill chunk explicitly affect cache/temp memory                   | OpenAI-compatible streaming and native SDK callbacks                                          | compiled library, weights, config, tokenizer and runtime versions must remain compatible             |
| MNN-LLM        | PyTorch → ONNX intermediate → MNN graph/weights/config                       | `.mtok` preferred, legacy text tokenizer; chat template may be embedded                                 | reuse and several KV quantization modes documented                                     | native/Python generation; Python `stream=True` writes stdout rather than yielding an iterator | export docs evolve rapidly and English coverage is less uniform                                      |
| ncnn           | PyTorch/ONNX through pnnx/converters → ncnn graph/weights                    | ncnn_llm/application-owned                                                                              | session KV allocator and reuse documented in releases                                  | application/example-owned                                                                     | core runtime is mature, but LLM orchestration and model coverage are less standardized               |
| Qualcomm Genie | AIMET/QNN or GenAI Transformer composer → target binaries/config             | tokenizer assets referenced in Genie JSON; composer export support is model-limited                     | backend/config specific                                                                | high-level Genie execution; app callback behavior must be verified in selected SDK            | SDK, SoC, backend library, quantization, shape, and context binaries form a tightly coupled set      |
<!-- markdownlint-restore -->

This table identifies product responsibilities that are often hidden by a successful demo. Integrated runtimes own more
of tokenization, cache, sampling, and streaming; low-level graph frameworks leave more to the application. More
integration can accelerate development, while explicit ownership can make parity and lifecycle behavior easier to
audit. Neither is inherently superior.

## 5. Runtime profiles

### 5.1 llama.cpp, GGML, and GGUF

#### Relationship

`DOC-VERIFIED`:

- **GGML** is the low-level, dependency-light C/C++ tensor library. It provides portable kernels, quantized tensor
  types, and multiple CPU/GPU/NPU/browser backends. The current GGML repository directs core contributions through
  llama.cpp because that is where the code receives wider integration testing:
  [GGML README](https://github.com/ggml-org/ggml/blob/master/README.md).
- **GGUF** is a model container, not an inference runtime and not a quantization algorithm. It supersedes older
  GGML/GGMF/GGJT file formats and stores tensor data plus extensible typed metadata in an mmap-friendly binary:
  [official GGUF specification](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md).
- **llama.cpp** is the LLM/MLLM inference project built on GGML. It implements model architectures,
  conversion/quantization utilities, tokenization, sampling, KV-cache management, command-line tools, a server, and
  platform examples: [llama.cpp README](https://github.com/ggml-org/llama.cpp/).

#### Platforms and acceleration

`DOC-VERIFIED`:

- Android has an official Android Studio binding/example, NDK build instructions, ARM CPU feature detection, automatic
  prompt formatting, prefill/decode, and token delivery through Kotlin `Flow`:
  [Android guide](https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/android.md).
- Release assets include Android arm64 CPU, Snapdragon CPU/Adreno GPU/Hexagon NPU, and iOS XCFramework packages. This
  verifies published artifacts, not model/backend coverage: [releases](https://github.com/ggml-org/llama.cpp/releases).
- Metal is the standard Apple GPU backend; Vulkan and CUDA are selectable build backends. Multiple backends may be built
  together and selected at runtime: [build guide](https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/build.md).

#### Format, tokenizer, KV cache, and streaming

`DOC-VERIFIED`:

- The GGUF specification defines tokenizer metadata, including embedded vocabulary fields and license metadata. It
  explicitly warns that GGML-vocabulary tokenization can be less accurate than the original tokenizer:
  [GGUF tokenizer section](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md#tokenizer).
- The CLI exposes separate K and V cache data types, including floating-point and quantized choices, and controls cache
  offload: [CLI options](https://github.com/ggml-org/llama.cpp/blob/HEAD/tools/cli/README.md).
- The Android example performs incremental token-to-text handling and buffers incomplete UTF-8 fragments:
  [Android native implementation](https://github.com/ggml-org/llama.cpp/blob/master/examples/llama.android/lib/src/main/cpp/ai_chat.cpp).

#### License, maintenance, and production caveats

`DOC-VERIFIED`: llama.cpp and GGML are MIT licensed. The project publishes very frequent build-number releases and was
active at the research cut-off: [license/repository](https://github.com/ggml-org/llama.cpp/),
[releases](https://github.com/ggml-org/llama.cpp/releases).

`TEST-REQUIRED`:

- Pin a known commit/build and conversion tool; rapid architecture and format evolution can make an unpinned artifact
  pipeline non-reproducible.
- Compare source-tokenizer IDs, decoded strings, special tokens, and the exact chat template against GGUF behavior.
- Run prompt-logit and generated-token parity before and after GGUF quantization.
- Verify which layers actually execute on Vulkan/Metal/Hexagon and whether backend mixing saves latency, energy, and
  peak memory.
- Exercise context exhaustion, cache shifting/sliding, cancellation, model reload, app backgrounding, malformed GGUF
  metadata, and low-memory termination.
- Treat the official Android sample as a reference integration, not a complete production lifecycle, model-update,
  integrity, or telemetry design.

**Best fit:** CPU-first baseline; broad GGUF ecosystem; local desktop/ARM reuse; teams willing to own a native C/C++
integration.  
**Weak fit:** a requirement for a stable vendor NPU abstraction across many Android SoCs.

### 5.2 ExecuTorch

`DOC-VERIFIED`:

- ExecuTorch captures PyTorch models with `torch.export`, lowers/partitions them for a target backend, and serializes a
  `.pte` program. Backend-specialized `.pte` files are normally generated separately:
  [backend overview](https://docs.pytorch.org/executorch/stable/backends-overview.html).
- Android packages provide Java/Kotlin bindings and native runtime libraries. The official guidance recommends XNNPACK
  first for broad compatibility, then Vulkan or vendor accelerators:
  [Android integration](https://docs.pytorch.org/executorch/stable/using-executorch-android.html).
- The documented mobile backend set includes XNNPACK CPU, Android Vulkan GPU, Apple Core ML, Qualcomm AI Engine,
  MediaTek NeuroPilot, Samsung Exynos, and others. Partial delegation can execute unsupported operators on CPU:
  [backend overview](https://docs.pytorch.org/executorch/stable/backends-overview.html).
- The LLM runner supports Hugging Face JSON, SentencePiece, TikToken, and Llama2.c tokenizer formats. Exported metadata
  describes maximum sequence/context lengths and KV behavior:
  [C++ LLM runner](https://docs.pytorch.org/executorch/stable/llm/run-with-c-plus-plus.html).
- Android `LlmModule` and iOS `TextLLMRunner` expose token callbacks, stop, and reset; both high-level mobile LLM APIs
  are documented as experimental:
  [Android LLM runner](https://docs.pytorch.org/executorch/stable/llm/run-on-android.html),
  [iOS LLM runner](https://docs.pytorch.org/executorch/stable/llm/run-on-ios.html).
- ExecuTorch is BSD licensed and had a 1.6.0 release on 2026-11-12:
  [license](https://github.com/pytorch/executorch/blob/main/LICENSE),
  [release](https://github.com/pytorch/executorch/releases/tag/v1.6.0).

`TEST-REQUIRED`:

- Confirm `torch.export` capture for the exact architecture, dynamic-shape policy, custom ops, and attention
  implementation.
- Produce and test one artifact per intended backend; do not assume a Core ML `.pte` is portable to Android or a QNN
  `.pte` to a different Snapdragon generation.
- Collect delegate partition logs. A model that “runs” may cross CPU/accelerator boundaries repeatedly.
- Verify tokenizer and chat-template ownership. Some model-specific tutorials use raw tokenizer assets and note missing
  template utilities.
- Validate cache precision and context length per backend. The Qualcomm Llama example uses static shapes, model
  sharding, mixed precision, and an 8-bit KV recipe and documents high host-memory/export cost:
  [Qualcomm Llama tutorial](https://docs.pytorch.org/executorch/stable/llm/build-run-llama3-qualcomm-ai-engine-direct-backend.html).
- Freeze PyTorch, ExecuTorch, delegate SDK, compiler, and mobile package versions as one release unit.

**Best fit:** PyTorch-centered development; one application targeting several acceleration vendors; teams able to
maintain backend-specific exports.  
**Weak fit:** “convert any Hugging Face checkpoint once and run everywhere.”

### 5.3 LiteRT, LiteRT-LM, and the TensorFlow Lite relationship

#### Relationship and migration status

`DOC-VERIFIED`:

- LiteRT is Google's successor to TensorFlow Lite and is now developed in its own repository. The project directs new
  inference code toward the `CompiledModel` API and states that TensorFlow Lite packages/`Interpreter` paths are in
  maintenance mode for critical security and stability work:
  [LiteRT repository](https://github.com/google-ai-edge/LiteRT).
- Existing `.tflite` artifacts remain central to LiteRT. The relationship is evolutionary at the model-container level
  but not API-neutral: legacy interpreter/delegate code and new compiled-model code should not be mixed casually.
- LiteRT-LM is the LLM orchestration layer over LiteRT. Google explicitly assigns conversation management, KV cache,
  streaming output, multimodal inputs, constrained decoding, and tool calling to LiteRT-LM rather than the generic
  runtime: [LiteRT product overview](https://developers.google.com/edge/litert).

#### Current platform and API state

`DOC-VERIFIED`:

- LiteRT-LM lists Android, iOS, web, desktop, and IoT; CPU and GPU are cross-platform in its current matrix, while NPU
  is listed for Android and preview on Windows, not iOS:
  [LiteRT-LM overview](https://developers.google.com/edge/litert-lm/overview).
- Kotlin, C++, CLI, and Python are labeled stable; Swift and JavaScript are labeled early preview in the same current
  overview.
- The `.litertlm` builder can package LiteRT prefill/decode graphs and a SentencePiece tokenizer; the runtime tokenizer
  interface identifies SentencePiece and Hugging Face tokenizers:
  [builder](https://github.com/google-ai-edge/LiteRT-LM/blob/main/docs/litert_lm_builder.md),
  [tokenizer interface](https://github.com/google-ai-edge/LiteRT-LM/blob/d5374cc9/runtime/components/tokenizer.h).
- Release history documents a LiteRT-based KV implementation, speculative decoding, and dynamic on-demand NPU KV growth:
  [LiteRT-LM releases](https://github.com/google-ai-edge/LiteRT-LM/releases).
- LiteRT and LiteRT-LM are Apache-2.0 licensed. LiteRT 2.3.2 and LiteRT-LM 0.19.1 were current releases near the
  cut-off: [LiteRT releases](https://github.com/google-ai-edge/LiteRT/releases/),
  [LiteRT-LM releases](https://github.com/google-ai-edge/LiteRT-LM/releases).

`TEST-REQUIRED`:

- Decide explicitly between generic `.tflite` execution and `.litertlm` orchestration. A raw LiteRT graph does not
  supply a complete chat loop.
- Verify whether the intended model is supported by the Generative Torch/Hugging Face export path and whether
  prefill/decode signatures match the runtime.
- Validate Swift API churn and binary packaging because Swift remained early preview.
- On Android NPU, test each SoC/OS/vendor library combination. “Unified NPU access” does not establish equal operator
  coverage or artifact portability.
- Compare CPU, GPU, and NPU numerical outputs, first-load compilation, program-cache behavior, memory, thermal
  stability, and fallback.
- Preserve legacy TFLite production systems until the LiteRT replacement passes parity and rollback gates; successor
  status alone is not a migration result.

**Best fit:** Google/Android-first deployment, especially when a supported LiteRT-LM model and vendor acceleration path
already exist.  
**Weak fit:** an immediate stable Swift API requirement or unsupported custom transformer architecture.

### 5.4 ONNX Runtime Mobile and ONNX Runtime GenAI

#### Two-layer stack

`DOC-VERIFIED`:

- Core ONNX Runtime Mobile loads ONNX models on Android and iOS. Official mobile packages provide CPU by default,
  XNNPACK on both platforms, NNAPI on Android, and Core ML on iOS:
  [mobile deployment guide](https://onnxruntime.ai/docs/tutorials/mobile/).
- ONNX Runtime GenAI implements the autoregressive generation layer: preprocessing, tokenizer, logits processing,
  sampling/search, KV-cache management, and constrained generation:
  [ORT GenAI documentation](https://onnxruntime.ai/docs/genai/).
- ORT GenAI exposes incremental detokenization with `TokenizerStream` and a token-by-token generator:
  [GenAI Python API](https://onnxruntime.ai/docs/genai/api/python.html).
- ORT and ORT GenAI are MIT licensed: [ORT source license notice](https://onnxruntime.ai/docs/api/c/onnxruntime__c__api_8h_source.html),
  [ORT GenAI documentation](https://onnxruntime.ai/docs/genai/).

#### Accelerator constraints

`DOC-VERIFIED`:

- NNAPI may partition unsupported operations back to ORT; official guidance warns that partitioning can degrade
  performance: [mobile guide](https://onnxruntime.ai/docs/tutorials/mobile/).
- The QNN Execution Provider targets Qualcomm Android and Windows. Its HTP/NPU backend requires quantized models, fixed
  shapes, and a supported operator subset:
  [QNN EP](https://onnxruntime.ai/docs/execution-providers/QNN-ExecutionProvider.html).
- The Snapdragon GenAI tutorial lists a small explicit model set and requires Qualcomm context binaries plus ONNX
  wrappers and metadata, demonstrating that NPU deployment is not arbitrary ONNX portability:
  [Snapdragon tutorial](https://onnxruntime.ai/docs/genai/tutorials/snapdragon.html).
- ORT GenAI's repository matrix lists Android as supported and iOS on the roadmap/under development, even though release
  pages publish iOS zip assets. The API documentation labels the API preview:
  [ORT GenAI status](https://onnxruntime.ai/docs/genai/),
  [API](https://onnxruntime.ai/docs/genai/api/python.html).

`TEST-REQUIRED`:

- Treat core ORT mobile support and ORT GenAI iOS product readiness as separate decisions.
- Verify exporter and opset versions, external-data handling, graph optimization, provider options, and minimal-build
  operator configuration.
- Measure provider partitioning and transfer overhead. Reject configurations with harmful CPU/accelerator ping-pong.
- Validate static context/batch dimensions for QNN HTP and quantify wasted cache memory if fixed shapes exceed actual
  use.
- Test tokenizer/chat-template parity and stop conditions from `genai_config.json`.
- Pin ORT, ORT GenAI, exporter, QNN SDK, and context binaries together.

**Best fit:** organizations standardized on ONNX; Windows/Android/desktop reuse; explicit Qualcomm or Intel provider
plans.  
**Weak fit:** a stable turnkey iOS generative SDK today.

### 5.5 Core ML and Core ML Tools

`DOC-VERIFIED`:

- Core ML is Apple's on-device model framework and can schedule work across CPU, GPU, and Neural Engine:
  [Core ML Tools README](https://github.com/apple/coremltools/blob/main/README.md).
- Core ML Tools converts and optimizes models from PyTorch/TensorFlow and emits Core ML packages. It is BSD-3-Clause
  licensed; the OS Core ML framework is distributed under Apple's platform terms:
  [Core ML Tools repository](https://github.com/apple/coremltools).
- iOS 18/macOS 15 introduced model state. `MLModel.makeState()` allocates declared state buffers, enabling in-place
  KV-cache updates across predictions:
  [Apple `makeState`](<https://developer.apple.com/documentation/coreml/mlmodel/makestate()>),
  [WWDC stateful-model session](https://developer.apple.com/videos/play/wwdc2024/10159/).
- Core ML Tools 8 added stateful model conversion, SDPA, low-bit weight compression, and representative Llama conversion
  coverage; version 9 added state read/write and newer deployment targets:
  [Core ML Tools releases](https://github.com/apple/coremltools/releases).

`TEST-REQUIRED`:

- Core ML does not itself define the product's tokenizer, chat template, sampler, stop logic, or incremental Unicode
  handling. Select and test those components.
- Inspect Xcode model reports and runtime profiling; `computeUnits = .all` is not proof that the full graph executes on
  ANE.
- Compare fixed-shape, range-shape, enumerated-shape, and state layouts for compile time, memory, and backend placement.
- Verify prefill/decode graph design. A one-token decode graph alone can make prompt ingestion unusably slow.
- Test minimum deployment target: stateful KV requires iOS 18+, while older devices need explicit cache I/O or a
  different runtime.
- Validate compressed-model quality and whether compressed weights remain compressed on the selected compute unit.

**Best fit:** an iOS-only product requiring first-party framework integration and potential ANE use.  
**Weak fit:** shared Android/iOS artifacts or an out-of-box Hugging Face chat engine.

### 5.6 MLX and MLX Swift LM

`DOC-VERIFIED`:

- MLX is an MIT-licensed array framework designed around Apple silicon unified memory. Its documented devices are CPU
  and GPU; it does not claim ANE execution: [MLX README](https://github.com/ml-explore/mlx/blob/main/README.md),
  [unified memory](https://github.com/ml-explore/mlx/blob/main/docs/src/usage/unified_memory.rst).
- MLX Swift LM is the current reusable Swift LLM/VLM package. It supplies model implementations, downloader/tokenizer
  integration protocols, generation/session APIs, and model examples:
  [MLX Swift LM](https://github.com/ml-explore/mlx-swift-lm).
- The official chat example runs on iOS and macOS, supports streaming/cancellation and model caching, and requires iOS
  17+/macOS 14+:
  [MLX Chat example](https://github.com/ml-explore/mlx-swift-examples/blob/main/Applications/MLXChatExample/README.md).
- MLX Swift LM documents simple, rotating, quantized, chunked, and architecture-specific caches and incremental
  detokenization:
  [KV cache guide](https://github.com/ml-explore/mlx-swift-lm/blob/main/skills/mlx-swift-lm/references/kv-cache.md),
  [tokenizer/streaming guide](https://github.com/ml-explore/mlx-swift-lm/blob/main/skills/mlx-swift-lm/references/tokenizer-chat.md).
- The project was actively released at the cut-off and is MIT licensed:
  [3.33.0 release](https://github.com/ml-explore/mlx-swift-lm/releases/tag/3.33.0),
  [repository](https://github.com/ml-explore/mlx-swift-lm).

`TEST-REQUIRED`:

- Confirm the architecture implementation and parameter naming exactly match the selected checkpoint.
- Test model downloader/cache behavior under interrupted download, low disk, upgrade, checksum failure, and app data
  protection.
- Unified memory avoids explicit CPU/GPU copies but does not remove system memory pressure. Measure Jetsam termination
  and resident memory at realistic context.
- Validate KV-cache configuration. The official guide notes that conversion from rotating to quantized cache is not
  implemented; do not assume every cache feature composes.
- Benchmark sustained GPU use, thermal throttling, background/foreground transitions, and contention with UI rendering.
- Compare against Core ML on the same iPhones if ANE placement, lower power, or OS-managed model compilation may matter.

**Best fit:** Swift-native prototyping and Apple-only products that want rapid support for modern model families.  
**Weak fit:** Android, ANE-specific requirements, or a single portable artifact.

### 5.7 MLC LLM

`DOC-VERIFIED`:

- MLC LLM is an Apache-2.0 machine-learning compiler and deployment engine with Android, iOS/iPadOS, desktop, and web
  APIs: [repository](https://github.com/mlc-ai/mlc-llm/).
- Android GPU paths use OpenCL on Adreno/Mali in the top-level matrix; TVM build documentation also supports Vulkan on
  most Android devices. Apple mobile uses Metal: [README](https://github.com/mlc-ai/mlc-llm/blob/main/README.md),
  [TVM build requirements](https://github.com/mlc-ai/mlc-llm/blob/main/docs/install/tvm.rst).
- Deployment requires converted MLC weights and a compiled model library. `mlc-chat-config.json` binds architecture,
  quantization, context, prefill chunk, and target:
  [compile guide](https://llm.mlc.ai/docs/compilation/compile_models.html).
- Android/iOS packaging builds the runtime, model libraries, and Hugging Face/SentencePiece tokenizer libraries.
  Context-window and prefill-chunk overrides control KV-cache and temporary memory:
  [Android SDK](https://github.com/mlc-ai/mlc-llm/blob/main/docs/deploy/android.rst),
  [iOS SDK](https://github.com/mlc-ai/mlc-llm/blob/main/docs/deploy/ios.rst).
- MLCEngine provides an OpenAI-compatible streaming API:
  [quick start](https://github.com/mlc-ai/mlc-llm/blob/main/docs/get_started/quick_start.rst).

`TEST-REQUIRED`:

- Rebuild and pin compiler, TVM, runtime, model library, tokenizer library, and converted-weight revisions as one
  manifest.
- Verify that an architecture appears in current model code, not merely an old model catalog.
- Compare OpenCL and Vulkan on each Android GPU/driver; backend availability does not establish correctness or speed.
- Physical-device testing is mandatory; the official Android guide states the accelerated app is not meaningful in an
  emulator.
- Test first-run compilation/cache, model download, large-file packaging, shader compilation, thermal behavior, and OS
  upgrade/driver regressions.
- Verify context and prefill chunk values against quality and memory; reducing these settings changes usable
  conversation behavior.

**Best fit:** one compiler-driven engine across Android GPU, iOS Metal, desktop, and web.  
**Weak fit:** teams unable to maintain a native compiler/package pipeline.

### 5.8 MNN and MNN-LLM

`DOC-VERIFIED`:

- MNN is Apache-2.0 and supports Android/iOS, ARM/x86 CPU, Metal/OpenCL/Vulkan mobile GPU, and several NPU/delegate
  paths: [README](https://github.com/alibaba/MNN/blob/master/README.md),
  [license](https://github.com/alibaba/MNN/blob/master/LICENSE.txt).
- MNN-LLM exports PyTorch checkpoints through an ONNX intermediate into MNN graph/weight/config assets. Current
  documentation lists low-bit export options and runtime backend configuration:
  [LLM guide](https://mnn-docs.readthedocs.io/en/latest/transformers/llm.html).
- The preferred tokenizer artifact is `.mtok`; the exporter derives it from `tokenizer.json` when available and can
  carry normalization, pretokenizer, BPE/WordPiece/Unigram data, added tokens, chat template, and special-token
  metadata: [tokenizer documentation](https://mnn-docs.readthedocs.io/en/latest/transformers/tokenizer.html).
- Runtime config includes KV reuse, flash attention, and multiple KV quantization modes. Python exposes generation and
  tokenizer calls, but `stream=True` writes native stdout rather than yielding a Python token iterator:
  [LLM guide](https://mnn-docs.readthedocs.io/en/latest/transformers/llm.html),
  [Python LLM API](https://mnn-docs.readthedocs.io/en/stable/pymnn/llm.html).
- 2026 releases added or expanded Vulkan LLM execution and direct Hexagon work, demonstrating active maintenance:
  [MNN releases](https://github.com/alibaba/MNN/releases).

`TEST-REQUIRED`:

- Validate exporter support for the exact architecture and compare the ONNX intermediate, MNN FP model, and quantized
  model.
- Treat backend maturity letters in the README as vendor/project classification, not independent production proof.
- Confirm that the mobile application API exposes cancellable token callbacks suitable for UI; the Python streaming
  behavior is not enough.
- Test English documentation/source alignment where the newest LLM details are documented primarily in Chinese.
- Verify Hexagon support for the exact SoC and attention/quantization mode and measure fallback.
- Exercise KV reuse with altered chat history; cached state and full prompt must not diverge silently.

**Best fit:** integrated Android/iOS engine with strong China/mobile ecosystem and willingness to validate rapidly
evolving LLM tooling.  
**Weak fit:** teams needing uniformly mature English LLM documentation and a stable cross-vendor NPU contract.

### 5.9 ncnn and ncnn_llm

`DOC-VERIFIED`:

- ncnn is a BSD-3-Clause, dependency-light C++ inference framework for mobile/embedded/desktop with ARM CPU and Vulkan
  GPU acceleration: [README](https://github.com/Tencent/ncnn/blob/master/README.md),
  [license](https://github.com/Tencent/ncnn/blob/HEAD/LICENSE.txt).
- Official release artifacts cover Android and Apple platforms, including Vulkan builds. The 2026 release stream added
  LLM-oriented SDPA/FlashAttention, RoPE, mmap/loading improvements, low-bit formats, LLM benchmarks, and session KV
  allocation: [releases](https://github.com/Tencent/ncnn/releases/).
- PyTorch and ONNX conversion is provided through pnnx and converter tooling; the repository links an `ncnn_llm` example
  project rather than presenting the core runtime as a complete chat framework:
  [README](https://github.com/Tencent/ncnn/blob/master/README.md).

`TEST-REQUIRED`:

- Inspect `ncnn_llm` support for the exact model, tokenizer, chat template, sampling policy, stop tokens, and streaming
  callback needs.
- Test Vulkan availability and operator placement on each GPU/driver. Apple Vulkan packages imply a portability layer
  and should be compared with native Metal/Core ML/MLX paths.
- Validate converter output and custom-layer requirements.
- Confirm application-level KV lifecycle, cancellation, prefix reuse, thread safety, and multiple conversations.
- Measure whether the smaller runtime footprint offsets added custom orchestration engineering.

**Best fit:** C++ teams prioritizing small dependencies, ARM CPU/Vulkan, and explicit control.  
**Weak fit:** a turnkey, documented, high-level multi-platform LLM SDK.

### 5.10 Qualcomm QAIRT/QNN and Genie

`DOC-VERIFIED`:

- Qualcomm AI Runtime (QAIRT), historically exposed through Qualcomm AI Engine Direct/QNN terminology, supplies CPU,
  GPU, and HTP/NPU backends on supported Qualcomm targets.
- Genie is the high-level generative framework. Its workflow prepares/quantizes a model, converts it with QNN, then runs
  it with tokenizer and JSON configuration assets:
  [Genie workflow](https://docs.qualcomm.com/doc/80-80020-15B/topic/use-genai-model-with-genie.html),
  [How to use Genie](https://docs.qualcomm.com/doc/80-63442-10/topic/how_to_use_genie.html).
- The Gen AI Transformer composer creates a Genie-consumable binary from a supported model and documents FP16/FP32 and
  several block quantization choices. Tokenizer export support is explicitly limited to named tokenizer families in the
  reviewed version: [composer](https://docs.qualcomm.com/doc/80-63442-10/topic/qnn-genai-transformer-composer.html).
- Android deployment requires ABI-specific Genie/QNN libraries, model binary, tokenizer, configuration, and matching
  runtime setup:
  [Gen AI Transformer guide](https://docs.qualcomm.com/nav/home/genai_genai.html?product=1601111740009302).

`TEST-REQUIRED`:

- Obtain and record the SDK license, redistribution rights, supported-product list, and production deployment terms. The
  public technical pages are not a substitute for legal review.
- Match SDK, backend libraries, device firmware, SoC revision, compiler/context binaries, quantization recipe, and model
  config.
- Confirm supported architecture/tokenizer in the exact SDK release; “Hugging Face model” is not a compatibility
  statement.
- Validate static shapes, maximum context, prompt/decode graph split, KV precision, and memory.
- Compare direct Genie/QNN integration with ExecuTorch-QNN, ORT-QNN, or LiteRT vendor paths. An abstraction can reduce
  app integration work but add partitioning or version lag.
- Build a CPU fallback and an explicit unsupported-device policy; Qualcomm artifacts do not cover non-Qualcomm Android
  phones.

**Best fit:** a controlled Snapdragon device matrix where NPU efficiency justifies vendor coupling.  
**Weak fit:** broad Android portability or open-source-only supply-chain requirements.

## 6. Important actively maintained alternatives

### 6.1 WebLLM

`DOC-VERIFIED`: WebLLM is an Apache-2.0 in-browser engine built on MLC/WebGPU. It provides an OpenAI-compatible API,
streaming, JSON mode, and current model releases: [README](https://github.com/mlc-ai/web-llm/blob/main/README.md),
[releases](https://github.com/mlc-ai/web-llm/releases).

`TEST-REQUIRED`: Browser WebGPU availability, per-origin storage quotas, service-worker lifecycle, tab suspension, model
caching, download resumption, shader compilation, memory limits, and iOS browser behavior must be tested. It is a useful
progressive-web-app or browser target, not a substitute for native Android/iOS NPU access.

### 6.2 OpenVINO GenAI

`DOC-VERIFIED`: OpenVINO GenAI wraps tokenization and the generation loop and supports CPU/GPU/NPU on supported Intel
platforms. ARM64 CPU archives exist, while the documented NPU plugin targets Intel Core Ultra on Windows/Linux:
[install](https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-genai.html),
[NPU guide](https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai/inference-with-genai-on-npu.html).

`TEST-REQUIRED`: Treat it as a laptop/Intel-edge and ARM Linux candidate, not as evidence of Android phone NPU support.
Native Android app packaging, Java/Kotlin API fit, Play Store binary constraints, and mobile thermal behavior need a
dedicated feasibility spike.

### 6.3 MediaPipe LLM Inference

`DOC-VERIFIED`: MediaPipe's historical LLM Inference task remains relevant to older examples, but the reviewed current
Google product documentation directs LLM deployment toward LiteRT-LM. New designs should begin at the
[LiteRT-LM overview](https://developers.google.com/edge/litert-lm/overview) and preserve MediaPipe only where an
existing deployed app has a tested dependency.

`TEST-REQUIRED`: Before maintaining or starting a MediaPipe LLM path, verify the exact package's release status,
supported model list, and migration guidance. No current official evidence reviewed here justifies ranking it as a
separate preferred runtime over LiteRT-LM.

### 6.4 General-purpose Rust/C++ engines

Projects such as Candle and other community native engines can be valuable for embedding, research, or a Rust-owned
stack, but this survey did not find first-party evidence sufficient to rank them above the listed runtimes for both
Android and iOS production LLM deployment. They remain `TEST-REQUIRED` candidates rather than omitted claims of
non-support. A candidate must demonstrate current mobile build instructions, tokenizer parity, KV cache, streaming,
low-bit kernels, app lifecycle handling, and recent releases before entering the main bake-off.

## 7. Maintenance, documentation, and production-readiness assessment

This is an engineering assessment of the official evidence, not a performance ranking.

<!-- markdownlint-disable MD013 -->
| Runtime            | Maintenance signal at cut-off                             | Documentation maturity                                                                     | Production posture                                                                       |
| ------------------ | --------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| llama.cpp          | Very high-frequency releases and active architecture work | Broad but fast-moving; examples and CLI docs are strong                                    | Mature core; mobile app and NPU paths still require substantial product hardening        |
| ExecuTorch         | Regular 1.x releases; multi-vendor contributors           | Strong stable docs, explicit backend status; high-level mobile LLM APIs still experimental | Strong candidate when backend-specific export/testing is funded                          |
| LiteRT / LiteRT-LM | Frequent 2026 releases; explicit successor roadmap        | Strong product docs; migration boundary can be confusing; some language APIs preview       | Strong Android candidate for supported models; iOS API maturity must be checked          |
| ORT Mobile / GenAI | Mature ORT core and active GenAI releases                 | Strong provider docs; GenAI API marked preview and mobile status differs by OS             | Strong graph/runtime foundation; generative layer needs pinned API and mobile validation |
| Core ML            | OS-integrated and first-party                             | High-quality Apple docs, but fewer end-to-end open-model chat recipes                      | Strongest native iOS deployment primitive; application owns generation stack             |
| MLX Swift LM       | Rapid active releases                                     | Good code-centric docs/examples; architecture support evolves quickly                      | Strong Apple prototype/product candidate with GPU-only accelerator scope                 |
| MLC LLM            | Active compiler/runtime project                           | Deep but toolchain-heavy; some mobile examples lag latest hardware                         | Viable cross-platform GPU stack for teams that can own compilation                       |
| MNN                | Active releases with explicit LLM and Hexagon work        | Substantive, newest details often Chinese-first                                            | Promising integrated mobile stack; validate docs/API/backend consistency                 |
| ncnn               | Long-running, active releases and shipped Tencent use     | Excellent core build docs; thinner high-level LLM product docs                             | Mature low-level runtime; more application orchestration is required                     |
| Qualcomm Genie/QNN | Current vendor SDK documentation                          | Detailed but SDK/version/product specific                                                  | Production-capable on controlled Snapdragon targets; highest vendor lock-in              |
| WebLLM             | Active releases                                           | Strong web examples/API docs                                                               | Production candidate for WebGPU web apps, subject to browser/storage constraints         |
| OpenVINO GenAI     | Current 2026 releases/docs                                | Mature for Intel/desktop/edge                                                              | Strong Intel laptop/edge candidate, not a general phone runtime                          |
<!-- markdownlint-restore -->

## 8. Required compatibility and production test plan

No runtime should pass selection from documentation alone. Execute the following on every candidate
artifact/backend/device combination.

### 8.1 Conversion and numerical gates

1. Record immutable source checkpoint and tokenizer revisions, converter/runtime commits, SDK versions, command lines,
   and artifact hashes.
2. Compare source-framework and target-runtime logits on:
   - one token;
   - short prefill;
   - long prefill near the intended context;
   - decode after cache reuse;
   - cache rollover/sliding-window boundary.
3. Run task-quality evaluation before and after conversion/quantization. A logit tolerance alone does not establish
   domain-quality retention.
4. Verify every intended model variant independently: base versus instruct, tied versus untied embeddings, Grouped-Query
   Attention (GQA), Multi-Query Attention (MQA), sliding attention, Mixture of Experts (MoE), multimodal encoders, and
   nonstandard positional encoding.

### 8.2 Tokenizer and conversation gates

1. Compare token IDs for ASCII, Unicode, combining characters, emoji, whitespace, code, domain units, malformed UTF-8
   handling, and special tokens.
2. Compare the rendered chat template byte-for-byte with the training/model-card template.
3. Verify Beginning of Sequence (BOS), End of Sequence (EOS), end-of-turn, padding, tool, and reasoning tokens.
4. Test incremental detokenization across byte/token boundaries; never render replacement characters or duplicate held
   bytes.
5. Test cancellation and restart at every generation stage.

### 8.3 KV-cache and memory gates

1. Measure loaded-model memory, first-prefill peak, steady decode, maximum intended context, conversation reset, and
   model unload.
2. Verify cache reset, prefix reuse, multi-session isolation, and history edit/rollback.
3. Evaluate cache precision independently from weight precision; quantify quality and memory changes.
4. Confirm behavior at context limit: reject, truncate, rotate, shift, or silently corrupt are materially different
   outcomes.
5. Test app background/foreground, device lock, Android activity recreation, and iOS memory pressure.

### 8.4 Accelerator gates

1. Capture partition/operator-placement reports and prove actual CPU/GPU/NPU work distribution.
2. Compare accelerated backend against the runtime's optimized CPU path; reject acceleration that loses latency, energy,
   or stability.
3. Test cold compile/load, warm load, first token, prefill, decode, and sustained 10–30 minute generation.
4. Record device model, SoC, RAM, OS/firmware, driver, battery/charging state, ambient temperature, thread count,
   context, prompt/output lengths, and trial count.
5. Include low-end, mid-range, and flagship devices rather than extrapolating from one reference phone.

### 8.5 Product and supply-chain gates

1. Inventory runtime, converter, tokenizer, model, SDK, and bundled third-party licenses.
2. Sign artifacts, verify hashes before loading, reject path traversal/oversized metadata, and fuzz model parsers where
   feasible.
3. Test interrupted download, insufficient storage, corrupt artifact, incompatible runtime version, and rollback.
4. Verify App Store/Play packaging, native ABI coverage, download-on-demand rules, executable-code restrictions, and
   privacy declarations.
5. Define a safe unsupported-device outcome and never silently route an offline/privacy promise to cloud inference.

## 9. Recommended staged bake-off for this project

### Stage A — portable correctness baseline

1. Convert one 0.5B–1.5B target model to GGUF and run it with llama.cpp on Android ARM CPU and Apple CPU/Metal.
2. Freeze tokenizer/chat-template golden tests and source-versus-runtime logits.
3. Establish common measurements for model size, loaded memory, peak memory, Time to First Token (TTFT), prefill
   tokens/s, decode tokens/s, energy, and quality.

### Stage B — OS-native and graph runtimes

1. Export the same model to ExecuTorch XNNPACK for both OSes.
2. On iOS, compare Core ML stateful KV and MLX Swift LM.
3. On Android, compare LiteRT-LM CPU/GPU and ORT GenAI CPU/XNNPACK where the architecture is officially exportable.

### Stage C — accelerator-specific paths

1. On one supported Snapdragon reference device, compare ExecuTorch-QNN, ORT-QNN, LiteRT's Qualcomm path, and direct
   Genie/QNN only if all can represent the same model and quantization closely enough for a fair comparison.
2. On a MediaTek and a Google Tensor device, test only documented vendor paths; do not infer Snapdragon results.
3. Reject any configuration that depends on broad CPU fallback unless it still wins the end-to-end objective.

### Stage D — compiler and secondary runtimes

1. Add MLC LLM when cross-platform GPU/Web reuse is a product requirement.
2. Add MNN and ncnn when their supported model list, target SoC, or footprint offers a concrete advantage.
3. Add WebLLM only for the browser deliverable and OpenVINO GenAI for Intel/ARM-Linux edge targets.

### Decision rule

Select a **runtime portfolio**, not one universal engine:

- a portable fallback with the widest tested device coverage;
- an Apple-optimized path if iOS quality/efficiency justifies a separate artifact;
- one or more Android vendor paths only for explicitly supported device cohorts;
- a browser or laptop runtime only when those are actual product targets.

Release a model/runtime/device tuple only when it passes quality, memory, latency, energy, lifecycle, licensing, and
rollback gates. Runtime brand, benchmark screenshots, or successful conversion are not release evidence.

## 10. Unresolved questions requiring execution or vendor confirmation

All items below are explicitly `TEST-REQUIRED` or require non-public vendor/legal confirmation:

1. Which selected domain SLM architectures export without graph rewrites to every candidate?
2. Which exact Android SoCs and OS builds provide complete GPU/NPU partitioning?
3. Are accelerator artifacts portable across devices in one SoC family, or must releases be keyed to exact revisions?
4. What are the measured quality effects of each runtime's supported weight and KV-cache quantization?
5. Which runtimes preserve the source tokenizer and chat template exactly?
6. What are peak memory and sustained thermal limits on the project's low-, mid-, and high-tier phones?
7. Does iOS Core ML place enough of each transformer on ANE/GPU to beat MLX or llama.cpp Metal?
8. Is LiteRT-LM Swift sufficiently stable for the intended release date?
9. Is ONNX Runtime GenAI's iOS package/API production-supported for the required feature set despite current status
   wording?
10. What Qualcomm SDK redistribution and update obligations apply to the intended commercial app?
11. Can MNN/ncnn/MLC integrations support cancellable, thread-safe multi-session streaming with the required app
    lifecycle?
12. Which runtime has acceptable cold-start and model-update behavior for multi-gigabyte downloadable artifacts?
13. Can one quantized artifact meet both cloud/AWS and mobile requirements, or are independent quantization/export
    pipelines necessary?
14. What device-specific fallback policy preserves the product's offline and privacy guarantees?

## 11. Official source register

Sources below are first-party project, vendor, or platform materials accessed for this dated survey.

### llama.cpp / GGML / GGUF

- <https://github.com/ggml-org/llama.cpp/>
- <https://github.com/ggml-org/llama.cpp/releases>
- <https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/build.md>
- <https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/android.md>
- <https://github.com/ggml-org/llama.cpp/blob/HEAD/tools/cli/README.md>
- <https://github.com/ggml-org/ggml/blob/master/README.md>
- <https://github.com/ggml-org/ggml/blob/master/docs/gguf.md>

### ExecuTorch

- <https://github.com/pytorch/executorch/>
- <https://github.com/pytorch/executorch/releases>
- <https://github.com/pytorch/executorch/blob/main/LICENSE>
- <https://docs.pytorch.org/executorch/stable/backends-overview.html>
- <https://docs.pytorch.org/executorch/stable/using-executorch-android.html>
- <https://docs.pytorch.org/executorch/stable/llm/getting-started.html>
- <https://docs.pytorch.org/executorch/stable/llm/run-with-c-plus-plus.html>
- <https://docs.pytorch.org/executorch/stable/llm/run-on-android.html>
- <https://docs.pytorch.org/executorch/stable/llm/run-on-ios.html>
- <https://docs.pytorch.org/executorch/stable/llm/export-custom-llm.html>

### LiteRT / LiteRT-LM

- <https://github.com/google-ai-edge/LiteRT>
- <https://github.com/google-ai-edge/LiteRT/releases/>
- <https://github.com/google-ai-edge/LiteRT-LM/>
- <https://github.com/google-ai-edge/LiteRT-LM/releases>
- <https://developers.google.com/edge/litert>
- <https://developers.google.com/edge/litert-lm/overview>
- <https://developers.google.com/edge/litert-lm/api_overview>
- <https://github.com/google-ai-edge/LiteRT-LM/blob/main/docs/litert_lm_builder.md>

### ONNX Runtime

- <https://onnxruntime.ai/docs/tutorials/mobile/>
- <https://onnxruntime.ai/docs/build/eps.html>
- <https://onnxruntime.ai/docs/execution-providers/NNAPI-ExecutionProvider.html>
- <https://onnxruntime.ai/docs/execution-providers/QNN-ExecutionProvider.html>
- <https://onnxruntime.ai/docs/genai/tutorials/snapdragon.html>
- <https://onnxruntime.ai/docs/genai/api/python.html>
- <https://onnxruntime.ai/docs/genai/>
- <https://onnxruntime.ai/docs/genai/reference/config.html>
- <https://onnxruntime.ai/docs/api/c/onnxruntime__c__api_8h_source.html>

### Apple Core ML and MLX

- <https://developer.apple.com/documentation/coreml>
- <https://developer.apple.com/documentation/coreml/mlmodel/makestate()>
- <https://developer.apple.com/videos/play/wwdc2024/10159/>
- <https://developer.apple.com/videos/play/wwdc2024/10161/>
- <https://github.com/apple/coremltools>
- <https://github.com/apple/coremltools/releases>
- <https://github.com/ml-explore/mlx>
- <https://github.com/ml-explore/mlx-swift-lm>
- <https://github.com/ml-explore/mlx-swift-lm/releases>
- <https://github.com/ml-explore/mlx-swift-examples>

### MLC LLM and WebLLM

- <https://github.com/mlc-ai/mlc-llm/>
- <https://llm.mlc.ai/docs/compilation/compile_models.html>
- <https://github.com/mlc-ai/mlc-llm/blob/main/docs/deploy/android.rst>
- <https://github.com/mlc-ai/mlc-llm/blob/main/docs/deploy/ios.rst>
- <https://github.com/mlc-ai/web-llm>
- <https://github.com/mlc-ai/web-llm/releases>

### MNN and ncnn

- <https://github.com/alibaba/MNN>
- <https://github.com/alibaba/MNN/releases>
- <https://mnn-docs.readthedocs.io/en/latest/transformers/llm.html>
- <https://mnn-docs.readthedocs.io/en/latest/transformers/tokenizer.html>
- <https://mnn-docs.readthedocs.io/en/stable/pymnn/llm.html>
- <https://github.com/Tencent/ncnn>
- <https://github.com/Tencent/ncnn/releases/>
- <https://github.com/Tencent/ncnn/blob/HEAD/LICENSE.txt>

### Qualcomm and additional runtimes

- <https://docs.qualcomm.com/doc/80-80020-15B/topic/use-genai-model-with-genie.html>
- <https://docs.qualcomm.com/doc/80-63442-10/topic/how_to_use_genie.html>
- <https://docs.qualcomm.com/doc/80-63442-10/topic/qnn-genai-transformer-composer.html>
- <https://docs.qualcomm.com/nav/home/genai_genai.html?product=1601111740009302>
- <https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-genai.html>
- <https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai.html>
- <https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai/inference-with-genai-on-npu.html>

## 12. Verification status

**Research status:** `DOC-VERIFIED` survey completed against official sources as of 2026-11-23.  
**Execution status:** no model conversion, runtime execution, benchmark, mobile build, accelerator trace, or
physical-device test was performed. Every exact model/runtime/device compatibility and every production recommendation
remains subject to the gates in Sections 8–10.
