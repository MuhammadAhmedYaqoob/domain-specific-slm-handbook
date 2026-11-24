<a id="hardware-memory-reference"></a>

# Appendix B — Hardware and Memory Reference

This appendix provides a comparison framework, not a timeless compatibility matrix. Hardware specifications require
dated vendor evidence; runtime support requires exact-version documentation; performance requires project execution on
the exact target.

The practical question is never “Can this phone run SLMs?” It is “Can this exact artifact, runtime, backend, workload,
and app fit and sustain acceptable behavior on this exact device?”

## Execution classes

<!-- markdownlint-disable MD013 -->
| Class              | Typical strengths                                                       | Common constraints                                                 | Evidence needed                                                |
| ------------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------- |
| CPU                | Broad control flow/operator support, mature tooling, small-batch decode | Bandwidth and vector-kernel limits; core contention                | ISA, thread/affinity policy, kernel path, clocks/thermal state |
| GPU                | Parallel prefill and dense tensor work                                  | Copies, shader/compiler limits, memory pressure, fallback          | Backend, operator coverage, precision, allocation/copy trace   |
| NPU                | Efficient supported low-precision graphs                                | Strict shapes/operators, compilation, partitioning, vendor tooling | Exact SoC/SDK, compiled graph, partition/fallback report       |
| Server accelerator | Training throughput, large memory, distributed collectives              | Cost, availability, interconnect, container/driver coupling        | Device/SKU, topology, driver/runtime, precision, utilization   |
<!-- markdownlint-restore -->

ARM64 Neon and dot-product instructions (`SDOT`/`UDOT`) can accelerate suitable kernels, but ISA presence does not prove
runtime dispatch. Mobile GPU paths may use Metal, Vulkan, or OpenCL. NPU paths may involve QAIRT/QNN HTP,
NeuroPilot/MDLA, Tensor TPU, or Apple ANE through Core ML; product-specific verification remains mandatory.

## Memory ledger

Report these terms separately:

1. packaged artifact and tokenizer;
2. decoded/memory-mapped weight pages;
3. quantization scales, zero points, sparse metadata, and alignment;
4. runtime graph and persistent buffers;
5. prompt/prefill activations;
6. KV cache by active sequence;
7. temporary operator workspace;
8. app/UI/network/telemetry allocations;
9. OS/runtime overhead;
10. observed peak resident or device memory.

For ideal weight payload and KV equations, see the [mathematical reference](appendix-mathematical-reference.md). The
acceptance value is measured peak under the declared worst supported workload, not ideal weight bytes.

### Worked memory ledger

`ILLUSTRATIVE`: suppose a profiler records 0.80 GiB mapped weights, 0.05 GiB quantization metadata, 0.20 GiB KV cache,
0.30 GiB workspaces and activations, and 0.40 GiB app/runtime overhead. The subtotal is 1.75 GiB. If the observed process
peak is 2.05 GiB, report 2.05 GiB and investigate the 0.30 GiB difference. A device with 2.1 GiB “free” memory has
inadequate margin even though the run happened once.

## Training memory

Training can hold master weights, compute weights, gradients, optimizer moments, activations, communication buckets,
workspaces, dataloader buffers, and checkpoints. PEFT reduces trainable gradients/optimizer state but the base still
consumes storage and device/host memory. QLoRA reduces frozen-base storage; actual memory depends on dequantization,
kernels, paging, sequence length, batch, and optimizer.

Record sharding stage, tensor/pipeline/data parallelism, gradient accumulation, activation checkpointing, offload,
sequence packing, precision, and failure headroom.

## Inference phases

- **Load:** verify, map/read, allocate, compile, and initialize.
- **Prefill:** process prompt tokens and create KV state; often parallel and compute intensive.
- **Decode:** generate one token per sequence step while reading weights and KV; often bandwidth/launch sensitive.
- **Unload/background:** release resources and survive application lifecycle transitions.

Report cold/warm load, TTFT, inter-token latency or decode tokens/s, end-to-end latency, prompt/output lengths, batch,
context occupancy, sampling, and concurrency.

## Bandwidth and roofline

The model may be compute-bound, memory-bandwidth-bound, launch-bound, or limited by graph partition and copies. Peak
TOPS/FLOPS numbers use precision and sparsity assumptions that may not match the workload. Use them only as
`VENDOR-REPORTED` specifications, never as application throughput.

## Thermal and energy protocol

Declare ambient conditions, power source, battery state, initial device temperature, cooling, screen/radio state,
background processes, performance mode, warm-up, run duration, repetitions, and throttling observations. Energy per
generated token requires a defined measurement boundary and synchronized workload. Short bursts do not establish
sustained behavior.

## Compatibility record

For each candidate tuple retain:

- model and tokenizer revision/hash;
- source and converted format/hash;
- converter, runtime, backend, compiler, OS, firmware, and driver versions;
- device model and SoC/SKU;
- operator/partition/fallback report;
- supported context, batch, precision, and cache format;
- conversion/load/inference results and logs;
- quality/safety parity result;
- date, owner, and known limitations.

## Feasibility gates

1. Artifact converts and verifies reproducibly.
2. Required operators execute on the intended backend without unacceptable fallback.
3. Peak memory fits with declared safety margin.
4. Sustained TTFT/decode/energy/thermal targets pass.
5. Output quality and safety remain within allowed regression.

Failure means this tuple is unsupported; it does not prove the model or hardware is universally unsuitable.

### Example qualification record

`ILLUSTRATIVE`: “Model M revision h1, converted by C 2.0 to format F, loaded by runtime R 1.4 on device D/OS 18.2,
batch 1, 512-token prompt, 128-token output.” Add fallback, peak memory, cold/warm TTFT, sustained decode distribution,
temperature trend, crashes, and quality parity. Without these qualifiers, “works on mobile” is not actionable.

## Runtime shortlist

Candidate runtimes include llama.cpp/GGUF, ExecuTorch, LiteRT/LiteRT-LM, Core ML, MLX Swift, ONNX Runtime Mobile/GenAI,
MLC LLM, MNN, ncnn, and Qualcomm Genie. The canonical comparison is
[CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md); its documented capabilities are not
executed compatibility claims.
