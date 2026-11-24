<a id="part-05"></a>

# Part V — Edge AI and Mobile Inference

This part turns abstract model efficiency into behavior on phones and other constrained targets. It covers CPU/GPU/NPU
execution, runtime formats and operators, conversion, packaging, profiling, and platform integration.

## Conceptual map

A model file is converted into a runtime graph; the runtime partitions operators across available backends; weights,
KV cache, activations, and workspaces occupy memory; prefill and decode stress hardware differently; the mobile app must
survive lifecycle, storage, battery, and thermal constraints. A failure at any link invalidates the exact deployment
tuple, even when the model runs elsewhere.

## Learning outcomes and dependencies

After CH-02 and [Part IV](../part-04-model-compression-and-optimization/README.md), readers should be able to map a
model graph to device hardware, shortlist runtimes, identify fallback, establish memory and thermal envelopes, and
design Android/iOS qualification. This part feeds CH-30–31, CH-36, and CH-47.

## Chapters

1. [CH-16 — Edge AI Fundamentals](ch-16-edge-ai-fundamentals.md): edge constraints and feasibility.
2. [CH-17 — Mobile Hardware Architecture](ch-17-mobile-hardware-architecture.md): ARM CPU, mobile GPU, NPU, memory, and
   bandwidth.
3. [CH-18 — Edge Inference Frameworks](ch-18-edge-inference-frameworks.md): llama.cpp/GGUF, ExecuTorch, LiteRT, Core ML,
   MLX Swift, ONNX Runtime, MLC LLM, MNN, ncnn, and vendor SDKs.
4. [CH-19 — Mobile Inference Optimization](ch-19-mobile-inference-optimization.md): kernels, threading, cache, context,
   batching, and profiling.
5. [CH-20 — Android Deployment](ch-20-android-deployment.md): packaging, native integration, lifecycle, memory, and
   release testing.
6. [CH-21 — iOS Deployment](ch-21-ios-deployment.md): Core ML/Metal-oriented integration, app lifecycle, and
   qualification.
7. [CH-22 — Other Edge Devices](ch-22-other-edge-devices.md): embedded Linux, SBC, industrial, and heterogeneous
   targets.

## Five feasibility gates

Qualify artifact/format, operator/backend coverage, peak memory, performance/energy/thermal behavior, and quality/safety
parity. Failure of any mandatory gate blocks that model-runtime-device combination.

### Practice before continuing

Write a qualification tuple containing model hash, format, converter, runtime/backend, device/OS, context, prompt and
output lengths, and temperature state. If you cannot fill a field, you do not yet have a reproducible mobile claim.

## Cross-links

[Hardware reference](../back-matter/appendix-hardware-and-memory-reference.md) ·
[CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md) ·
[Part IX](../part-09-aws-training-optimization-and-deployment/README.md)
