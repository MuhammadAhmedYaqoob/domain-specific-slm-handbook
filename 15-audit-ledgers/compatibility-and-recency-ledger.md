# Compatibility and recency ledger

## What is checked and why

Compatibility is a property of a complete tuple: model revision, tokenizer, conversion tool, artifact format, runtime,
backend, operating system, hardware, and workload. This ledger checks whether documentation is recent enough to choose
a test and whether that exact tuple has actually run. It matters because “supports 4-bit,” “supports Android,” or
“supports GPU” can each be true while their combination still fails.

`DOC-VERIFIED` means authoritative documentation supported a bounded statement at the research freeze. `Not yet
performed` means this project has no retained exact-tuple run. A passing tuple needs pinned versions, successful
conversion/load, output checks, backend-placement evidence, system measurements, and logs.

## Compatibility records

<!-- markdownlint-disable MD013 -->
| Area | Documentation finding | Exact-tuple status | Validation method |
| --- | --- | --- | --- |
| Qwen, Gemma, Llama, Phi, SmolLM, TinyLlama, MobileLLM, alternatives | CH-03 records candidate repositories, architecture facts, and license caveats | Not yet performed: revisions, files, and export paths are not uniformly frozen; no selected model was loaded | Freeze model/tokenizer revision and file hashes, capture license, load with pinned libraries, run known prompts, and record memory and output |
| Transformers, Datasets, PEFT, TRL, Accelerate, bitsandbytes | Candidate APIs and versions are taught | Not yet performed: no resolved lock set proves the API combinations | Resolve one lock per workflow, install cleanly, then run import, tiny train, save/reload, and resume checks |
| Quantization, pruning, and distillation tools | Methods and candidate libraries are surveyed | Not yet performed: formats and kernels are not interchangeable | Pin source model, producer, artifact schema, runtime, and backend; run parity, quality, memory, latency, and failure tests |
| llama.cpp and GGUF | Official conversion and inference paths are described | Not yet performed for a selected checkpoint and target | Pin commit/build flags; convert once; verify metadata/tokenizer; load on desktop; then test each claimed ABI/backend |
| ExecuTorch | Official LLM/export paths are described | Not yet performed for checkpoint/backend/mobile integration | Export a `.pte`; inspect operators and delegation; compile app bindings; run a physical device cohort |
| LiteRT and LiteRT-LM | Official model and runtime paths are described | Not yet performed; support depends on packaged model and target SoC | Pin release and artifact; convert/package; inspect delegates and fallback; run on each named Android target |
| ONNX Runtime Mobile/GenAI | Official mobile and selected artifact guidance is described | Not yet performed; provider partitioning, tokenizer, cache, and package behavior are unknown | Pin versions; convert/load; record operator placement and fallback; test generation parity and memory on target |
| Core ML and MLX Swift | Official Apple paths are described | Not yet performed; stateful cache, tokenizer, compute placement, and memory are unknown | Pin Xcode/iOS/tool versions; convert; compile; inspect compute units; profile named iPhones/iPads |
| MLC LLM, MNN, NCNN, and vendor accelerators | Candidate paths are surveyed | Not yet performed; no exact support or fallback record | Run a short compatibility study before making any support claim; retain operator, partition, and failure evidence |
| vLLM, SGLang, TensorRT-LLM, llama.cpp server, TGI, ONNX | Serving capabilities are compared | Not yet performed for selected model, quantization, GPU, and container | Pin release and image digest; load exact artifact; run correctness, concurrency, soak, failure, and recovery tests |
| AWS services and accelerators | Official service capabilities are described | Not yet performed for account, Region, quota, image, driver, capacity, or endpoint | Verify immediately before use; deploy in a named Region; preserve IDs, quotas, configuration, logs, cleanup, and billing |
| Android and iOS app stacks | Build and integration blueprints exist | Not yet performed: neither app has a retained signed physical-device run | Produce release builds, sign, install, test lifecycle/offline/update behavior, and benchmark a declared device matrix |
| Prices | Cost formulae exist; no current rate is asserted | Not yet performed: no dated Region-specific price snapshot or bill | Capture official rates within 30 days of decision, record assumptions, then reconcile forecast with actual usage |
<!-- markdownlint-restore -->

## Passing and failing evidence

**Pass:** “Model revision X, converted by tool commit Y to format Z, loaded in runtime A on Android version B and SoC C;
all operators used the stated backend, output met parity tolerance, and retained logs record memory and latency.”

**Fail:** “The runtime supports Android and the model has a 4-bit file, so NPU execution is supported.” The premises do
not establish conversion compatibility, operator coverage, placement, correctness, or performance.

Desktop, simulator, emulator, or one phone results do not qualify another OS, SoC, backend, or device class. Source
context length does not prove converted-artifact context length. Hardware availability does not prove accelerator use;
CPU fallback must be detected.

## Recency and responsibility

Responsibility is unassigned until a named person accepts each test. Recheck mutable model IDs, licenses, libraries,
runtime APIs, formats, AWS services, and mobile claims within 90 days of a publication candidate and immediately after
a relevant change. Recheck prices within 30 days. Record authoritative URL, exact version, access date, supported and
unsupported scope, affected files, and the event that forces another check.
