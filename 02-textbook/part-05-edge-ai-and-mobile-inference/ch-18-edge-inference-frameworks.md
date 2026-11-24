# Chapter 18 — Edge Inference Frameworks

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

**Research cut-off:** 2026-11-23  
**Evidence status:** `DOC-VERIFIED` from first-party sources unless marked otherwise  
**Execution status:** no conversion, runtime, benchmark, accelerator trace, or physical-device test was performed

## 18.1 The deployment problem

A trained Small Language Model (SLM) is not yet a mobile product. A deployable stack must preserve the model graph,
tokenizer, chat template, sampling rules, stop tokens, and Key-Value (KV) cache while fitting the target operating
system, processor, memory, storage, power, and lifecycle constraints.

An **edge inference framework** loads a deployed model representation, schedules operators, manages state, and exposes
inference to an application. Some frameworks also implement tokenization, sampling, KV-cache management, and streaming;
graph runtimes often require a separate generative layer.

Four claims must never be conflated:

1. a framework supports an operating system;
2. one backend supports an accelerator;
3. the framework implements a model family;
4. one exact checkpoint, tokenizer, quantization, and device work correctly together.

This chapter evaluates **model/runtime/backend/device tuples**, not brand names. `DOC-VERIFIED` means official
documentation states the capability; it does not mean this project reproduced it. Every exact tuple is `TEST-REQUIRED`.

## 18.2 How an autoregressive runtime works

<a id="fig-ch18-19"></a> **FIG-CH18-19 — Runtime-oriented mobile inference pipeline (`ILLUSTRATIVE`; companion to
canonical FIG-19 in CH-20).** Alt text: source assets pass through compatibility audit, conversion, backend lowering,
packaging, prefill, iterative decode, and streamed text. Provenance: author synthesis from the official sources in
§18.14; no measured data.

```text
checkpoint + tokenizer + chat template
                  │
       architecture/operator audit
                  ▼
 conversion → quantization → backend lowering/compilation
                  ▼
 graph/weights + metadata + tokenizer + native runtime
                  ▼
 tokenize → render chat → prefill → KV cache
                                      │
                                      ▼
                          decode → sample → detokenize
                                      │
                              streamed output

Gates: tokenizer • logits • quality • operator placement • memory
       latency • energy • thermals • lifecycle • license • rollback
```

**Prefill** processes the input and creates cache state; **decode** repeatedly processes a new token while reading that
state. A planning estimate for a conventional dense KV cache is

\[ M_{\mathrm{KV}}\approx L\,T\,2\,H_{\mathrm{KV}}\,D\,B, \]

where \(L\) is layer count, \(T\) cached tokens, \(H_{\mathrm{KV}}\) KV heads, \(D\) head dimension, \(B\) bytes per
element, and 2 represents keys and values. For \(L=24,T=4096,H_{\mathrm{KV}}=8,D=128,B=2\), this is about 384 MiB. This
`ESTIMATE` excludes weights, activations, temporary buffers, padding, backend copies, tokenizer, and app memory. File
size is therefore not peak resident memory.

## 18.3 Reading compatibility claims

- A Hugging Face repository is not a universal interchange format. Converters support explicit architectures, operators,
  shapes, and tokenizers.
- A format is not a runtime: GGUF, ONNX, `.tflite`, `.pte`, and `.mlpackage` are artifacts.
- “INT4” is incomplete without grouping, scales, zero points, activation/cache precision, and kernels.
- Selectable GPU/NPU does not prove full offload. Partial delegation can add synchronization and transfers.
- Conversion success does not prove tokenizer, logit, long-context, or task-quality parity.
- Runtime, model, tokenizer, converter, SDK, and training-data licenses are independent.

<a id="tbl-14"></a>

## 18.4 TBL-14 — Edge runtime decision matrix

**Legend:** D = documented; P = preview/experimental; T = exact tuple needs testing; — = no first-party support found,
not proof of impossibility.

<!-- markdownlint-disable MD013 -->
| Runtime                  | Android/iOS                  | CPU       | GPU                   | NPU                             | Artifact                        | Generative layer                  | License               | Main boundary                                                  |
| ------------------------ | ---------------------------- | --------- | --------------------- | ------------------------------- | ------------------------------- | --------------------------------- | --------------------- | -------------------------------------------------------------- |
| llama.cpp/GGML           | D/D                          | D         | Metal, Vulkan D/T     | Snapdragon package P/T          | GGUF                            | integrated                        | MIT                   | explicit architectures; fast-moving conversion/backend support |
| ExecuTorch               | D/D                          | XNNPACK D | Vulkan/Core ML D/T    | Qualcomm, MediaTek, Samsung D/T | backend `.pte`                  | runners; mobile APIs experimental | BSD                   | backend-specific exports and partial delegation                |
| LiteRT/LiteRT-LM         | D/D                          | D         | D/T                   | documented Android paths D/T    | `.tflite`/`.litertlm`           | LiteRT-LM                         | Apache-2.0            | supported exporter/model set; Swift early preview              |
| ORT Mobile/GenAI         | D/core D; GenAI iOS P/T      | D         | provider-specific D/T | NNAPI/Core ML/QNN D/T           | ONNX/ORT + config               | ORT GenAI preview                 | MIT                   | generic ONNX is insufficient for efficient generation          |
| Core ML                  | —/D                          | D         | D                     | ANE D/T                         | `.mlpackage`                    | app-owned                         | Tools BSD-3; OS terms | Apple-only; placement and generation stack need proof          |
| MLX Swift LM             | —/D                          | D         | Apple GPU D           | —                               | MLX/weights                     | integrated                        | MIT                   | exact Swift architecture; no claimed ANE path                  |
| MLC LLM                  | D/D                          | T mobile  | OpenCL/Vulkan/Metal D | — reviewed                      | weights+config+compiled library | integrated                        | Apache-2.0            | compiler/package compatibility                                 |
| MNN-LLM                  | D/D                          | D         | OpenCL/Vulkan/Metal D | Hexagon/Core ML paths D/T       | MNN assets + `.mtok`            | integrated                        | Apache-2.0            | evolving exporter/backends/docs                                |
| ncnn                     | D/D                          | D         | Vulkan D              | — reviewed                      | `.param`/`.bin`                 | app/example-owned                 | BSD-3                 | less turnkey LLM orchestration                                 |
| Qualcomm QAIRT/QNN/Genie | supported Qualcomm Android/— | D         | D                     | HTP D                           | context/binary+config           | Genie                             | SDK terms             | SoC/SDK/firmware lock-in                                       |
| WebLLM                   | browser-dependent            | —         | WebGPU D              | —                               | web/MLC assets                  | integrated                        | Apache-2.0            | browser quota/lifecycle/WebGPU                                 |
| OpenVINO GenAI           | edge/ARM Linux T; —          | D         | Intel D               | Intel NPU D                     | OpenVINO IR                     | integrated                        | Apache-2.0            | not evidence of phone-NPU support                              |
<!-- markdownlint-restore -->

### A defensible selection rule

First reject any candidate that cannot prove exact architecture/tokenizer representation, target OS/ABI, acceptable
licensing, required context without termination, offline streaming/cancellation, quality parity, and safe
unsupported-device behavior. Then score only measured criteria:

\[ S_r=\sum_j w_j s_{rj},\quad \sum_jw_j=1. \]

Illustrative weights are quality .25, memory .20, energy/thermals .15, TTFT/prefill .10, decode .10, device coverage
.08, lifecycle .05, maintenance .04, and legal fit .03. Keep raw results and a Pareto frontier; safety and quality
should be non-compensable hard gates in high-risk domains.

## 18.5 Runtime profiles

### 18.5.1 llama.cpp, GGML, and GGUF

`DOC-VERIFIED`: GGML is the low-level C/C++ tensor library; GGUF is an mmap-friendly typed model container; llama.cpp
supplies model implementations, conversion/quantization, tokenization, sampling, cache, CLI/server, and mobile examples
[S1–S7].

- **Boundary:** only architectures implemented by the pinned converter/runtime; metadata cannot add execution semantics.
- **Platforms/compute:** official Android example and iOS XCFramework; CPU, Metal, Vulkan, and published Snapdragon
  variants. Hexagon/Adreno coverage is T.
- **Formats/quantization/custom ops:** source→GGUF→optional GGML quantization. Unsupported operators require C/C++
  implementation.
- **Memory:** mmap helps loading, not cache/buffer peaks; context, K/V types, and cache offload are configurable.
- **Tokenizer/KV/streaming:** GGUF can embed tokenizer/template metadata; its specification warns tokenization can
  differ. Configurable K/V types and Android Kotlin `Flow` are documented.
- **Maturity:** MIT, highly active, broad but fast-moving docs. Pin commits.

Best first use: portable CPU correctness baseline. Weak fit: one stable cross-vendor Android NPU abstraction.

### 18.5.2 ExecuTorch

`DOC-VERIFIED`: `torch.export` captures a PyTorch program, which is lowered/partitioned and serialized as
backend-specific `.pte` [S8–S15].

- **Boundary:** capture, shapes, custom attention, and backend operators must all succeed.
- **Compute:** Android/iOS; XNNPACK CPU, Android Vulkan, Apple Core ML, Qualcomm, MediaTek, Samsung, and other
  documented delegates. Fallback can be partial.
- **Quantization/custom ops:** backend recipes differ; custom ops require registration and possibly backend kernels.
- **Memory:** measure program, backend allocations, static shapes, cache, and export-host costs separately.
- **Tokenizer/KV/streaming:** runner supports HF JSON, SentencePiece, TikToken, and Llama2.c formats; exported metadata
  describes cache/context; callbacks/reset/stop are documented, while high-level mobile LLM APIs are experimental.
- **Maturity:** BSD, active 1.x, strong backend-status documentation.

Best fit: PyTorch-owned pipelines able to maintain one artifact per backend.

### 18.5.3 LiteRT, LiteRT-LM, and TensorFlow Lite

`DOC-VERIFIED`: LiteRT succeeds TensorFlow Lite. `.tflite` remains central, but new work is directed to `CompiledModel`;
legacy `Interpreter` packages are maintenance paths. LiteRT-LM adds conversation, tokenizer, KV cache, and streaming
[S16–S20].

- **Boundary:** generic `.tflite` execution is not a chat loop; `.litertlm` packages compatible prefill/decode graphs
  and assets.
- **Compute:** Android/iOS CPU/GPU; current LiteRT-LM matrix lists NPU for Android, not iOS.
- **Quantization/custom ops:** exporter/backend-specific; custom ops/delegates can restrict portability.
- **Memory/KV:** context, prefill temporaries, compilation cache, and KV dominate. Releases document LiteRT KV and
  dynamic NPU growth.
- **Tokenizer/streaming:** SentencePiece/HF assets; LiteRT-LM owns conversation and streaming. Kotlin/C++/CLI/Python
  stable; Swift early preview at cut-off.
- **Maturity:** Apache-2.0, active, good docs.

Do not rewrite a tested TFLite app merely because of successor branding; migrate behind parity and rollback gates.

### 18.5.4 ONNX Runtime Mobile and GenAI

`DOC-VERIFIED`: ORT Mobile executes graphs; ORT GenAI adds tokenizer, sampling/search, KV management, and token
streaming [S21–S27].

- **Boundary:** explicit exporter/model support, opset, external-data, cache graph, and Execution Provider (EP) support.
- **Compute:** Android/iOS core; CPU/XNNPACK, Android NNAPI, iOS Core ML, and Qualcomm QNN. QNN HTP requires quantized
  fixed-shape supported graphs. GenAI iOS remained P/T.
- **Quantization/custom ops:** provider preparation matters; custom ops/minimal builds require packaging and can block
  delegation.
- **Memory:** fixed max shapes may reserve excess cache; measure provider and transfer allocations.
- **Tokenizer/KV/streaming:** GenAI config owns tokenizer/template/cache and provides `TokenizerStream`.
- **Maturity:** MIT; mature core, GenAI API preview.

Best fit: an ONNX estate with an explicit provider plan—not arbitrary ONNX export.

### 18.5.5 Core ML

`DOC-VERIFIED`: Apple's framework can schedule supported operations on CPU/GPU/Apple Neural Engine (ANE); iOS 18/macOS
15 stateful models can update KV state in place [S28–S32].

- **Boundary:** pinned Core ML Tools operations, shapes, state, and deployment target.
- **Quantization/custom ops:** Core ML Tools supports compression; custom layers may prevent ANE/GPU placement.
- **Memory:** compare fixed/range/enumerated shapes and explicit versus stateful cache.
- **Tokenizer/streaming:** application-owned tokenizer, template, sampler, stops, Unicode streaming, and loop.
- **Maturity:** first-party stable framework; Tools BSD-3, OS terms separate.

`computeUnits = .all` permits scheduling; it does not prove ANE execution. Best fit: dedicated iOS artifact where
profiling validates placement.

### 18.5.6 MLX and MLX Swift LM

`DOC-VERIFIED`: MLX targets Apple CPU/GPU unified memory; Swift LM supplies model implementations, tokenizer
integration, generation, streaming, and simple/rotating/quantized/chunked caches [S33–S37].

- **Boundary:** exact architecture code and parameter/config match.
- **Quantization/custom ops:** MLX-specific representation; extensions add Swift/Metal maintenance.
- **Memory:** unified memory avoids some copies but not system pressure or Jetsam termination.
- **NPU:** no claimed ANE execution.
- **Maturity:** MIT, rapid active releases, strong code-centric examples.

Compare it against Core ML and llama.cpp Metal on the same iPhones; documentation cannot rank them.

### 18.5.7 MLC LLM

`DOC-VERIFIED`: compiler-first Android/iOS/web stack with converted weights, `mlc-chat-config.json`, target-compiled
library, tokenizer assets, and streaming engine [S38–S42].

- **Compute:** Android OpenCL/Vulkan and iOS Metal documented; do not infer mobile NPU or CPU quality.
- **Boundary/custom ops:** architecture must exist in current MLC code; extension is compiler engineering.
- **Memory:** context and prefill chunk control cache/temporary memory but also usable behavior.
- **Maturity:** Apache-2.0, active, deep but toolchain-heavy.

Pin TVM/compiler/runtime/library/tokenizer as one manifest.

### 18.5.8 MNN and MNN-LLM

`DOC-VERIFIED`: PyTorch→ONNX→MNN export, CPU plus Metal/OpenCL/Vulkan, low-bit options, `.mtok`, KV reuse/quantization,
and active Hexagon work are documented [S43–S48].

- **Boundary:** validate every conversion stage and backend; project maturity labels are not independent proof.
- **Custom ops/memory:** custom backends can reduce portability; record mmap, separated weights, cache, and backend
  buffers.
- **Tokenizer/streaming:** `.mtok` can carry BPE/WordPiece/Unigram, normalization, added tokens, template, and specials.
  Python streaming writes stdout; cancellable mobile callbacks remain T.
- **Maturity:** Apache-2.0, active; newest docs can be Chinese-first.

### 18.5.9 ncnn

`DOC-VERIFIED`: dependency-light C++ ARM/Vulkan runtime; releases add SDPA, RoPE, low-bit, mmap, LLM benchmarks, and
session KV [S49–S52].

- **Boundary:** pnnx/converters and layers must cover the model; `ncnn_llm` owns more orchestration.
- **Custom ops/tokenizer/streaming:** possible but application engineering is required.
- **NPU:** no reviewed first-party LLM NPU path.
- **Maturity:** BSD-3, mature low-level docs; thinner high-level chat docs.

### 18.5.10 Qualcomm QAIRT/QNN/Genie

`DOC-VERIFIED`: Qualcomm CPU/GPU/HTP stack; Genie orchestrates QNN/composer-produced binaries with tokenizer and JSON
configuration [S53–S56].

- **Boundary:** supported SDK model/tokenizer list; SDK, SoC, firmware, backend, context binary, shapes, and
  quantization are coupled.
- **Custom ops/memory:** QNN packages increase coupling; inspect static context, cache precision, and prompt/decode
  split.
- **License:** obtain redistribution/update terms; public docs are not legal approval.

Compare direct Genie with ExecuTorch-QNN, ORT-QNN, and documented LiteRT Qualcomm paths on one device. Best fit:
controlled Snapdragon fleets.

## 18.6 Active alternatives

- **WebLLM:** `DOC-VERIFIED` Apache-2.0 MLC/WebGPU browser engine with streaming [S57–S58]. Quota, service-worker/tab
  lifecycle, shader compilation, caching, memory, and iOS browser behavior are T.
- **OpenVINO GenAI:** `DOC-VERIFIED` generation stack for supported Intel CPU/GPU/NPU and ARM64 CPU archives [S59–S61].
  It is an Intel laptop/edge candidate, not evidence of Android NPU.
- **MediaPipe LLM Inference:** historical Google path; current reviewed guidance points new work to LiteRT-LM. Existing
  apps must verify exact package status and migration guidance.
- **Candle/other Rust/C++ engines:** admit to the bake-off only with current mobile builds, exact model/tokenizer
  support, KV, streaming/cancellation, low-bit kernels, lifecycle evidence, license, and active maintenance.

## 18.7 Cross-cutting release gates

### Custom operators

A custom op requires a correct kernel, registration/ABI packaging, backend implementations or explicit fallback, and
maintenance across releases. A CPU custom op can make a graph load while destroying accelerator efficiency. Record every
custom op and placement.

### Tokenizer and chat

Golden tests must compare IDs and decoded text for ASCII, domain terms, whitespace, units, code, Unicode normalization,
combining marks, emoji, and special tokens; compare rendered chat bytes; verify BOS/EOS/EOT/padding; test incremental
UTF-8 boundaries. Never invent a template to make output look plausible.

### KV, streaming, and lifecycle

Measure loaded, prefill-peak, steady-decode, max-context, reset, and unload memory. Test cache reset, edited history,
prefix reuse, context exhaustion, multi-session isolation, cancellation, backpressure, Android activity recreation, iOS
backgrounding, and memory pressure.

### Domain quality and safety

After conversion and quantization, rerun terminology, factual accuracy, structured output, uncertainty, refusal,
general-retention, and hallucination tests. Offline execution does not cure stale learned knowledge. Pharmaceutical and
safety-critical repair output still requires qualified verification; this project's primary inference path remains
no-runtime-RAG.

## 18.8 Practical examples

All listings are `UNVERIFIED IMPLEMENTATION`; check APIs against pinned releases.

**Listing 18.1 — llama.cpp conversion probe**

```powershell
git clone https://github.com/ggml-org/llama.cpp.git
Set-Location llama.cpp
git checkout aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
python .\convert_hf_to_gguf.py C:\models\source `
  --outfile C:\artifacts\model-f16.gguf --outtype f16
cmake -B build
cmake --build build --config Release
.\build\bin\Release\llama-quantize.exe `
  C:\artifacts\model-f16.gguf C:\artifacts\model-q.gguf `
  q4_k_m
```

illustrative values are intentional: never guess a supported architecture or quantizer.

**Listing 18.2 — Core ML stateful loop shape**

```swift
// Requires a compatible generated stateful model and deployment target.
let config = MLModelConfiguration()
config.computeUnits = .all
let model = try GeneratedModel(configuration: config)
let state = model.makeState()
for input in decodeInputs {
    let output = try model.prediction(input: input, using: state)
    stream(incrementalDetokenize(output.nextToken))
}
```

This does not prove placement, prefill efficiency, or tokenizer correctness.

Every artifact should carry immutable source/tokenizer/template/converter/runtime/backend revisions, target OS/ABI/SoC
policy, exact quantization and KV type, context limit, hashes, and validation-record ID. Sign and verify it before
loading; never silently route an offline promise to cloud inference.

### 18.8.1 Worked runtime-selection walkthrough

Suppose the product must run one 1B–2B domain SLM offline on mid-range Android phones and modern iPhones, stream text,
support cancellation, and retain a portable fallback. There is no requirement for browser execution or a single
artifact across both platforms.

**Step 1 — Apply hard gates.** Reject any path that cannot represent the exact architecture, tokenizer, chat template,
2K product context, arm64 target, and redistribution terms. This happens before scoring performance. A successful
generic ONNX or Core ML conversion is not enough unless prefill, decode, cache, and text semantics also pass.

**Step 2 — Establish a correctness baseline.** If the architecture is implemented, use llama.cpp/GGUF CPU on Android
and CPU/Metal on iOS as a broad baseline. This choice is about coverage and inspectability, not presumed speed.

**Step 3 — Add platform hypotheses.**

- On Android, test LiteRT-LM or ExecuTorch only for a documented exporter/model path; add QNN/Genie only for a named
  Snapdragon cohort.
- On iOS, test Core ML when ANE eligibility is a product objective and MLX Swift when its exact architecture is
  implemented. Core ML needs an app-owned generation loop; MLX does not imply ANE use.
- Do not select MLC merely because it spans both platforms. Select it if the team accepts compiler/runtime lockstep and
  cross-platform GPU execution is a real requirement.

**Step 4 — Compare complete paths.** For each candidate, compare tokenizer IDs, template bytes, logits, domain quality,
cold and warm load, TTFT, decode, peak memory, placement, energy, thermal retention, cancellation, and lifecycle. A
runtime that wins decode but fails atomic cancellation is not the product winner.

**Step 5 — Choose a portfolio.** A defensible result may be GGUF/CPU as the Android fallback, a qualified vendor
artifact for one Android cohort, and Core ML or MLX on iOS. Shared source weights and tests matter more than forcing a
universal binary.

Expected outcomes are explicit:

| Outcome | Interpretation | Next action |
| --- | --- | --- |
| converter rejects architecture | unsupported tuple, not a malformed model by assumption | choose supported runtime/model or implement support |
| logits diverge before quantization | semantic export defect | stop; inspect layout, RoPE, cache, and tokenizer |
| accelerator has many CPU islands | partial delegation | measure boundaries; compare CPU-only |
| long context alone OOMs | cache/reservation problem | lower context, change cache/architecture, or runtime |
| one phone succeeds | evidence for that tuple only | expand the cohort matrix deliberately |

This sequence converts the framework catalog into a decision: requirements → hard gates → baseline → one platform
hypothesis at a time → measured portfolio.

## 18.9 Rigorous bake-off protocol

<a id="fig-ch18-23"></a> **FIG-CH18-23 — Runtime selection workflow (`ILLUSTRATIVE`; canonical FIG-23 is owned by
CH-31).** Alt text: model, platform, backend, format, and product requirements pass through compatibility and parity
gates before a runtime is selected.

```text
docs/license → conversion → tokenizer/logit parity → domain quality
     → memory/context → placement/fallback → sustained device trials
     → lifecycle/corruption/rollback → Pareto + release decision
```

**Status:** `PLANNED EXPERIMENT`.

1. Freeze model/tokenizer/template revisions, commands, environments, seeds, hashes, host, devices, OS/firmware,
   quantization, cache, prompts, sampling, and raw-result schema.
2. Establish source-framework tokenizer, logits, greedy tokens, and domain-quality goldens.
3. **Stage A:** use llama.cpp CPU/GGUF as a portable baseline where supported; test Android arm64 and Apple CPU/Metal
   separately.
4. **Stage B:** test ExecuTorch XNNPACK; on iOS compare Core ML and MLX; on Android compare documented LiteRT-LM and ORT
   GenAI paths.
5. **Stage C:** on one supported Snapdragon compare QNN paths only when model, context, cache, and quantization are
   sufficiently comparable. Repeat vendor work independently for other SoCs.
6. **Stage D:** add MLC for cross-platform GPU/web requirements; MNN/ncnn for a concrete model, footprint, or hardware
   hypothesis; WebLLM/OpenVINO only for real browser/edge targets.
7. Compare tokenizer IDs/template bytes and logits at one token, short/long prefill, cached decode, and cache boundary.
   A numerical pass is not a quality pass.
8. Measure artifact bytes, load time, resident and peak memory, TTFT, prefill/decode rate, energy, thermal state,
   crashes, and quality. Separate cold/warm and run 10–30 minute sustained trials.
9. Capture operator-placement reports. Reject harmful fallback even if the model “runs.”
10. Use declared warm-ups and repeated trials; report median, dispersion, paired quality effects, confidence intervals
    where applicable, raw data, and failures. Never average unlike devices/contexts/quantizations without qualification.

Release only an approved model+tokenizer+runtime+backend+device cohort. Select a portfolio—portable fallback, Apple
path, explicit Android vendor cohorts, and browser/laptop path if needed—not a universal winner.

## 18.10 Troubleshooting

<!-- markdownlint-disable MD013 -->
| Symptom                  | Likely cause                                           | Action                                              |
| ------------------------ | ------------------------------------------------------ | --------------------------------------------------- |
| Conversion rejects model | unsupported architecture/op/shape/name                 | verify exact pinned support; do not relabel         |
| Fluent nonsense          | tokenizer/template/tensor/RoPE/cache error             | compare IDs, prompt bytes, logits, greedy tokens    |
| Slow first token         | cold compile, paging, shader creation, prefill         | separate load, compile, prefill, decode             |
| Accelerator slower       | fallback, transfers, small work, thermals              | inspect placement; compare identical CPU path       |
| OOM at context           | KV, prefill temporaries, static max allocation, copies | measure phase peaks and cache/shapes                |
| Broken Unicode           | nonincremental detokenization                          | buffer incomplete bytes; test emoji/combining marks |
| Wrong stopping           | EOS/EOT/template/max-length mismatch                   | compare special IDs and prompt                      |
| Cross-session leakage    | cache reset/isolation failure                          | adversarial reset and multi-session tests           |
| One-device-only success  | ABI/driver/SoC/OS artifact coupling                    | explicit compatibility manifest and fallback        |
| Background crash         | reclaimed memory/native handle/worker                  | lifecycle cancellation and reconstruction tests     |
<!-- markdownlint-restore -->

## 18.11 Exercises

1. Distinguish GGML, GGUF, and llama.cpp.
2. Explain why valid ONNX is insufficient evidence for efficient ORT GenAI.
3. Explain TensorFlow Lite, LiteRT, and LiteRT-LM without treating them as synonyms.
4. Why is Core ML not a complete chat stack? Why is MLX not an ANE claim?
5. Compute KV memory for \(L=28,H_{\mathrm{KV}}=4,D=128,T=8192,B=2\); list four excluded allocations.
6. Build tokenizer golden vectors for part numbers, units, drug names, Unicode, code, and chat specials.
7. Design a test distinguishing NPU gains from CPU fallback/transfer overhead.
8. Define hard gates and weighted criteria for a low-end Android product.
9. **Practical assignment (`PLANNED EXPERIMENT`):** run §18.9 on one checkpoint explicitly supported by two runtimes.
   Submit manifests, commands, traces, raw data, failures, hashes, quality results, and Pareto analysis. A documented
   conversion failure is valid; silently changing models is not.

## 18.12 Verification note

The required frameworks, boundaries, platforms, compute targets, formats, quantization/conversion, custom ops, memory,
tokenizer, KV, streaming, licenses, maintenance, documentation, limitations, matrix, protocol, examples,
troubleshooting, and exercises are authored. No framework is claimed fastest, smallest, or production-ready. Exact
compatibility, quality, memory, placement, latency, energy, thermals, lifecycle behavior, LiteRT-LM Swift/ORT GenAI iOS
readiness, and Qualcomm legal terms remain unresolved until executed or confirmed.

## 18.13 Cross-links

- [Part index](README.md)
- [Chapter 17](ch-17-mobile-hardware-architecture.md)
- [Chapter 19](ch-19-mobile-inference-optimization.md)
- [Android deployment](ch-20-android-deployment.md)
- [iOS deployment](ch-21-ios-deployment.md)
- [Assessment](../../13-assessments/ch-18-assessment.md)
- [Evidence standard](../../00-governance/evidence-and-citation-standard.md)
- [Research survey](../../04-research-comparison-report/edge-runtime-survey.md)

## 18.14 Official sources

- **[S1–S7] llama.cpp/GGML/GGUF:** <https://github.com/ggml-org/llama.cpp/> ;
  <https://github.com/ggml-org/llama.cpp/releases> ; <https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/build.md> ;
  <https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/android.md> ;
  <https://github.com/ggml-org/llama.cpp/blob/HEAD/tools/cli/README.md> ;
  <https://github.com/ggml-org/ggml/blob/master/README.md> ; <https://github.com/ggml-org/ggml/blob/master/docs/gguf.md>
- **[S8–S15] ExecuTorch:** <https://github.com/pytorch/executorch/> ; <https://github.com/pytorch/executorch/releases> ;
  <https://github.com/pytorch/executorch/blob/main/LICENSE> ;
  <https://docs.pytorch.org/executorch/stable/backends-overview.html> ;
  <https://docs.pytorch.org/executorch/stable/using-executorch-android.html> ;
  <https://docs.pytorch.org/executorch/stable/llm/run-with-c-plus-plus.html> ;
  <https://docs.pytorch.org/executorch/stable/llm/run-on-android.html> ;
  <https://docs.pytorch.org/executorch/stable/llm/export-custom-llm.html>
- **[S16–S20] LiteRT/LiteRT-LM:** <https://github.com/google-ai-edge/LiteRT> ;
  <https://github.com/google-ai-edge/LiteRT/releases/> ; <https://developers.google.com/edge/litert> ;
  <https://developers.google.com/edge/litert-lm/overview> ;
  <https://github.com/google-ai-edge/LiteRT-LM/blob/main/docs/litert_lm_builder.md>
- **[S21–S27] ONNX Runtime:** <https://onnxruntime.ai/docs/tutorials/mobile/> ;
  <https://onnxruntime.ai/docs/build/eps.html> ;
  <https://onnxruntime.ai/docs/execution-providers/NNAPI-ExecutionProvider.html> ;
  <https://onnxruntime.ai/docs/execution-providers/QNN-ExecutionProvider.html> ;
  <https://onnxruntime.ai/docs/genai/> ; <https://onnxruntime.ai/docs/genai/api/python.html> ;
  <https://onnxruntime.ai/docs/genai/tutorials/snapdragon.html>
- **[S28–S37] Apple:** <https://developer.apple.com/documentation/coreml> ;
  <https://developer.apple.com/documentation/coreml/mlmodel/makestate()> ;
  <https://developer.apple.com/videos/play/wwdc2024/10159/> ; <https://github.com/apple/coremltools> ;
  <https://github.com/apple/coremltools/releases> ; <https://github.com/ml-explore/mlx> ;
  <https://github.com/ml-explore/mlx/blob/main/docs/src/usage/unified_memory.rst> ;
  <https://github.com/ml-explore/mlx-swift-lm> ; <https://github.com/ml-explore/mlx-swift-lm/releases> ;
  <https://github.com/ml-explore/mlx-swift-examples>
- **[S38–S42] MLC:** <https://github.com/mlc-ai/mlc-llm/> ; <https://llm.mlc.ai/docs/compilation/compile_models.html> ;
  <https://github.com/mlc-ai/mlc-llm/blob/main/docs/deploy/android.rst> ;
  <https://github.com/mlc-ai/mlc-llm/blob/main/docs/deploy/ios.rst> ;
  <https://github.com/mlc-ai/mlc-llm/blob/main/docs/get_started/quick_start.rst>
- **[S43–S52] MNN/ncnn:** <https://github.com/alibaba/MNN> ; <https://github.com/alibaba/MNN/releases> ;
  <https://mnn-docs.readthedocs.io/en/latest/transformers/llm.html> ;
  <https://mnn-docs.readthedocs.io/en/latest/transformers/tokenizer.html> ;
  <https://mnn-docs.readthedocs.io/en/stable/pymnn/llm.html> ; <https://github.com/alibaba/MNN/blob/master/LICENSE.txt>
  ; <https://github.com/Tencent/ncnn> ; <https://github.com/Tencent/ncnn/releases/> ;
  <https://github.com/Tencent/ncnn/blob/HEAD/LICENSE.txt> ; <https://github.com/pnnx/pnnx>
- **[S53–S61] Qualcomm/alternatives:**
  <https://docs.qualcomm.com/doc/80-80020-15B/topic/use-genai-model-with-genie.html> ;
  <https://docs.qualcomm.com/doc/80-63442-10/topic/how_to_use_genie.html> ;
  <https://docs.qualcomm.com/doc/80-63442-10/topic/qnn-genai-transformer-composer.html> ;
  <https://docs.qualcomm.com/nav/home/genai_genai.html?product=1601111740009302> ; <https://github.com/mlc-ai/web-llm> ;
  <https://github.com/mlc-ai/web-llm/releases> ;
  <https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-genai.html> ;
  <https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai.html> ;
  <https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai/inference-with-genai-on-npu.html>
