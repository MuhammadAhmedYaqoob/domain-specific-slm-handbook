# Runtime and deployment comparison

**Research freeze:** 2026-11-23  
**Status:** authored synthesis of the [edge](edge-runtime-survey.md) and [AWS](aws-capability-survey.md) surveys. No
runtime, conversion, accelerator trace, AWS resource, or physical device was executed.

## Unit of compatibility

Compatibility belongs to an exact tuple:

```text
model revision + tokenizer/template + precision/format + converter
+ runtime version + backend/SDK + device/OS/driver + context/workload
```

An operating-system check mark, supported family name, successful conversion, selectable backend, or official artifact
proves only its narrow documented fact. It does not prove numerical parity, full accelerator placement, acceptable
memory, interactive latency, sustained thermals, or production readiness.

## Lesson: choose separate deployment children from one governed parent

Cloud and edge deployments solve different systems problems. A phone serves one user within fixed power and memory;
cloud serving shares accelerators across concurrent requests and optimizes batching, queueing, and utilization. This
historical divergence explains why one high-precision parent often produces different cloud and edge artifacts rather
than one universal binary.

For example, an automotive model can produce a GGUF 4-bit child for Android CPU, a Core ML or MLX child for iPhone, and
an AWQ or supported half-precision child for an Amazon Web Services (AWS) GPU runtime. The generated behavior should be
tested against the same domain and safety suites, but the packaging, cache design, concurrency, and operational metrics
differ. Artifact diversity is acceptable when lineage, approvals, and rollback bind every child to the same parent.

Use llama.cpp as a portable correctness baseline, then compare one graph or operating-system-native runtime on each
target. On AWS, begin with vLLM and SGLang for a supported GPU model and add specialized systems only for a measured
need. Avoid forcing cloud quantization formats onto phones, inferring Neural Processing Unit (NPU) execution from a
successful export, or adopting Kubernetes before workload evidence justifies it.

## Edge runtime matrix

<!-- markdownlint-disable MD013 -->
| Runtime                   | Platforms/compute                                                         | Artifact and generation layer                                                       | License/maintenance                                      | Best initial role                             | Principal caveat                                                                          |
| ------------------------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | -------------------------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------- |
| llama.cpp/GGML/GGUF       | Android/iOS/Linux; CPU, Metal, Vulkan and documented vendor packages      | GGUF; tokenizer, sampling, KV, streaming integrated                                 | MIT; very active/fast-moving                             | Portable CPU correctness and low-bit baseline | Explicit architecture conversion; metadata/template parity; NPU coverage                  |
| ExecuTorch                | Android/iOS; XNNPACK, Vulkan, Core ML, Qualcomm/MediaTek/Samsung backends | Backend-specialized `.pte`; LLM runners                                             | BSD; active 1.x; high-level mobile LLM APIs experimental | PyTorch-owned heterogeneous export            | Separate artifact per backend and partial delegation                                      |
| LiteRT/LiteRT-LM          | Android/iOS/web/desktop; CPU/GPU, documented Android NPU paths            | `.tflite` graphs or `.litertlm` package; LM layer owns conversation/KV/streaming    | Apache-2.0; active; Swift early preview at freeze        | Android-first supported-model path            | Do not confuse legacy TFLite Interpreter, LiteRT, and LiteRT-LM                           |
| ONNX Runtime Mobile/GenAI | Core ORT Android/iOS; provider-specific CPU/GPU/NPU                       | ONNX/ORT graphs + GenAI config/tokenizer                                            | MIT; mature core, GenAI preview                          | ONNX estate and explicit EP plan              | Generic ONNX is not optimized autoregressive support; GenAI iOS maturity                  |
| Core ML                   | Apple platforms; CPU/GPU/ANE scheduling                                   | `.mlpackage`; app owns tokenizer/sampler/loop unless layered library                | Tools BSD-3; OS terms; first-party                       | Native iOS/ANE candidate                      | `computeUnits=.all` does not prove placement; stateful KV needs current deployment target |
| MLX Swift LM              | iOS/macOS; Apple CPU/GPU                                                  | MLX weights + Swift architecture/tokenizer/generation                               | MIT; active                                              | Rapid Apple GPU implementation                | Exact architecture code; no ANE claim; unified memory still pressures system RAM          |
| MLC LLM                   | Android/iOS/web; OpenCL/Vulkan/Metal                                      | Converted weights, config, compiled model library, tokenizer; integrated generation | Apache-2.0; active                                       | Cross-platform compiler/GPU/web arm           | Toolchain and target library must remain version-compatible                               |
| MNN-LLM                   | Android/iOS; CPU, OpenCL/Vulkan/Metal and evolving vendor paths           | MNN graph/weights/config + `.mtok`; integrated generation                           | Apache-2.0; active                                       | Integrated alternate mobile stack             | Rapidly evolving exporter/backends; newest docs uneven                                    |
| ncnn/ncnn_llm             | Android/iOS/embedded; ARM CPU/Vulkan                                      | ncnn graph/weights; more app-owned orchestration                                    | BSD-3; mature core                                       | Small C++/Vulkan footprint                    | Less turnkey tokenizer/chat/cache/streaming product layer                                 |
| Qualcomm QAIRT/QNN/Genie  | Supported Snapdragon Android cohorts; CPU/GPU/HTP                         | Target binaries/context + tokenizer/config; Genie orchestration                     | Vendor SDK terms                                         | Controlled Snapdragon NPU fleet               | SDK/SoC/firmware/model/tokenizer/shape lock-in                                            |
| WebLLM                    | WebGPU-capable browsers                                                   | MLC/WebGPU assets; integrated web API                                               | Apache-2.0; active                                       | Browser deliverable                           | Browser quotas, caching, lifecycle, memory, iOS behavior                                  |
| OpenVINO GenAI            | Intel/ARM-Linux edge; Intel CPU/GPU/NPU                                   | OpenVINO IR + generation stack                                                      | Apache-2.0; active                                       | Intel laptop/edge target                      | Not evidence of Android phone NPU support                                                 |
<!-- markdownlint-restore -->

Rows are `DOC-VERIFIED` summaries plus `STATICALLY REVIEWED` role assignments. Exact tuple support remains
`PLANNED EXPERIMENT`.

The edge table compares ownership and portability. llama.cpp integrates a broad CPU-oriented generation stack;
ExecuTorch, LiteRT, ONNX Runtime, and Core ML emphasize graph export and backend delegation; MLX and MLC provide
framework/compiler alternatives; vendor paths trade portability for accelerator access. “Best initial role” indicates
where to learn first, not which runtime will benchmark fastest.

## AWS/server runtime matrix

<!-- markdownlint-disable MD013 -->
| Runtime/path       | Artifact/hardware                                                                      | Scheduling and precision                                                 | Maintenance posture                             | Initial role                            |
| ------------------ | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------- | --------------------------------------- |
| vLLM               | Supported HF-format models; NVIDIA is lowest-friction AWS path; other platforms differ | Continuous batching, paged KV, multiple model-specific quantizers        | Active                                          | First GPU baseline                      |
| SGLang             | Supported models on several hardware paths                                             | Continuous batching, prefix-aware scheduling, multiple quantizers        | Active/fast-moving                              | Contemporary GPU comparison             |
| TGI                | HF/container path                                                                      | Continuous batching/streaming and quantization paths                     | Vendor labels maintenance mode since 2025-11-12 | Existing compatibility only             |
| llama.cpp server   | GGUF; CPU and multiple accelerator backends                                            | Continuous batching/slot controls; GGUF types                            | Active                                          | CPU/GGUF and cloud-edge parity baseline |
| ONNX Runtime GenAI | ONNX graph/config with explicit Execution Provider                                     | Generation/KV layer; provider quantization                               | GenAI API preview                               | Conditional portability arm             |
| TensorRT-LLM       | NVIDIA/Linux and supported models/GPUs                                                 | Paged attention, in-flight batching, hardware-specific FP8/FP4/INT8/INT4 | Active                                          | Advanced optimization after baseline    |
| vLLM Neuron plugin | Supported Trainium/Inferentia recipes                                                  | Plugin-specific features/precision                                       | Officially beta at freeze                       | Targeted AWS-silicon experiment         |
| SageMaker/Bedrock  | Managed hosting/import contracts                                                       | Service capabilities do not replace runtime qualification                | Current AWS service docs                        | Conditional operations choice           |
<!-- markdownlint-restore -->

No throughput, cost, or superiority claim is made.

The server table separates general baselines from specialized optimizers. vLLM and SGLang are first comparisons because
they expose modern scheduling with broad model ecosystems. llama.cpp tests CPU/GGUF and cloud-edge parity.
TensorRT-LLM and AWS Neuron can be valuable when their narrower hardware paths solve a measured bottleneck. TGI remains
relevant to existing deployments but its documented maintenance posture lowers its priority for new work.

## Deployment target comparison

<!-- markdownlint-disable MD013 -->
| Dimension            | Offline edge/mobile                                              | AWS cloud                                                          |
| -------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------ |
| Availability/privacy | Local and offline; model stored on device                        | Network/service dependent; centralized access controls             |
| Compute envelope     | Fixed RAM/bandwidth/power/thermals                               | Scalable CPU/GPU/accelerator capacity                              |
| Artifact             | GGUF, `.pte`, `.litertlm`, ONNX, Core ML, MLX, vendor binary     | HF weights, GGUF, ONNX, TensorRT/Neuron/service-specific           |
| Concurrency          | Usually one interactive user                                     | Batching, queueing, multi-tenant concurrency                       |
| Primary limits       | Load peak, KV context, battery, sustained heat, ABI/SoC coverage | GPU memory/KV pool, utilization, queue/tail latency, idle cost     |
| Update model         | Signed download/bundle, compatibility manifest, rollback         | Central blue-green/canary deployment                               |
| Observability        | Privacy-limited app telemetry and local benchmark                | Rich host/runtime/request metrics, with privacy controls           |
| Failure mode         | OOM/Jetsam/process death, unsupported device, corrupt update     | Capacity/quota, model load, queue overload, region/service failure |
<!-- markdownlint-restore -->

The deployment table explains why a shared quality evaluation does not imply a shared systems benchmark. Edge tests
must include load peaks, battery, heat, lifecycle, and unsupported devices. Cloud tests must include concurrency,
queueing, tail latency, scaling, failure recovery, and all material costs. Both need tokenizer, safety, provenance, and
rollback verification.

The same high-precision parent can produce different approved cloud and edge children. One artifact is not a requirement
and often is not possible.

## Tokenizer, KV cache, and streaming gates

Every candidate must demonstrate:

1. exact tokenizer IDs and decoded text for ASCII, whitespace, domain codes/units, Unicode normalization, combining
   characters, emoji, special tokens, and malformed boundaries;
2. byte-identical intended chat template, BOS/EOS/EOT/padding, and stop behavior;
3. source-versus-target logits where feasible and deterministic greedy token checks;
4. prefill and decode paths, not decode-only demos;
5. cache allocation, reset, edited history, prefix reuse, context exhaustion, cache precision, and session isolation;
6. incremental UTF-8 detokenization, streaming backpressure, cancellation, restart, and app/service lifecycle;
7. operator/delegate placement showing actual CPU/GPU/NPU partition and fallback.

## Runtime selection by target

### Android

Start with llama.cpp ARM CPU where the exact architecture is supported. Compare ExecuTorch XNNPACK and a documented
LiteRT-LM path. Add Vulkan/GPU only after CPU parity. Test Qualcomm/MediaTek/Google Tensor vendor paths independently;
Snapdragon evidence does not transfer to another SoC.

### iOS

Compare llama.cpp Metal, MLX Swift LM, and stateful Core ML where the OS/device target supports it. ExecuTorch is the
cross-platform graph arm. Measure Apple Neural Engine placement rather than inferring it from conversion or `.all`
compute units.

### ARM Linux/Raspberry Pi/laptop

Use llama.cpp as the portable baseline; add ONNX/OpenVINO where the processor/provider supports it. Jetson should
compare llama.cpp CUDA and an appropriate NVIDIA server/edge stack on exact JetPack/software versions.

### AWS

Start with one right-sized NVIDIA EC2 candidate and compare vLLM/SGLang. Add llama.cpp for GGUF/CPU intent. Use
SageMaker when managed lifecycle is worth its contract. Add TensorRT-LLM, Trainium/Inferentia, Bedrock import, ECS, or
EKS only for a measured requirement.

## AWS service prioritization

<!-- markdownlint-disable MD013 -->
| Class                     | Services                                                                                                              |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Essential baseline        | S3, IAM, KMS, VPC/security groups, CloudWatch, CloudTrail; EC2 or SageMaker training/hosting                          |
| Conditional essential     | ECR with containers; Infrastructure as Code before repeatable/shared environments; Secrets Manager when secrets exist |
| Optional growth           | ECS, Step Functions, API Gateway, SageMaker Model Registry/endpoints, Bedrock supported customization/import          |
| Defer until measured need | EKS, FSx for Lustre, multi-node P/Trainium clusters, elaborate multi-model serving                                    |
<!-- markdownlint-restore -->

Availability is a dated tuple of service, feature, instance, Region, Availability Zone, account quota, and capacity. No
dollar rate is copied; pricing must be captured from official AWS sources with Region, date, SKU/unit, and attached
resource costs.

## Common qualification protocol

1. Freeze model/tokenizer/template, source/export tool, runtime/backend/SDK, artifact hashes, target hardware/OS/driver,
   precision/cache/context, workload, and licenses.
2. Run source-framework goldens.
3. Convert/package and retain complete logs, including documented failures.
4. Prove tokenizer/template and numerical/behavior parity.
5. Capture operator placement and reject harmful fallback.
6. Measure cold and warm load, artifact/download size, loaded and phase-peak memory, KV growth, TTFT, prefill, decode,
   end-to-end latency, errors, and sustained trials.
7. On devices, record charging, battery, ambient/start temperature, clocks where available, background state, and 10–30
   minute thermal behavior.
8. On AWS, record Region/AZ/instance, image digest, driver/runtime, concurrency/arrival process, queue time, tail
   latency, utilization, load/scale behavior, and every material cost input.
9. Rerun domain/general/safety suites after conversion/quantization.
10. Test corrupt artifact, interrupted update/download, low storage, context exhaustion, cancellation, backgrounding,
    restart, blue-green/canary, and complete-tuple rollback.

## Project bake-off shortlist

- **Portable edge:** llama.cpp CPU/GGUF.
- **Cross-platform graph:** ExecuTorch XNNPACK.
- **Android:** LiteRT-LM supported route; one vendor accelerator only after portable baselines.
- **Apple:** MLX Swift LM and Core ML, compared with llama.cpp Metal.
- **Cloud GPU:** vLLM and SGLang.
- **Cloud/edge artifact parity:** llama.cpp server.
- **Conditional:** ORT GenAI for ONNX requirements, MLC for cross-platform GPU/web, TensorRT-LLM for measured NVIDIA
  bottleneck.

Select a portfolio, not a universal winner.

## Evidence limits and direct references

No row establishes that an arbitrary Hugging Face model exports, that a backend performs full offload, or that any
candidate meets production quality, memory, latency, energy, thermal, reliability, license, or cost requirements.

- [Detailed edge runtime survey](edge-runtime-survey.md)
- [Detailed AWS capability survey](aws-capability-survey.md)
- [CH-18: Edge inference frameworks](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- [CH-35: AWS inference serving](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-35-aws-inference-serving.md)
- [Edge runtime selection guide](../09-edge-guide/runtime-selection.md)
- [AWS service selection guide](../08-aws-guide/service-selection.md)
- [AWS benchmark protocol](../06-experiment-framework/protocols/aws-inference-benchmark.md)
- [Edge benchmark protocol](../06-experiment-framework/protocols/edge-device-benchmark.md)
