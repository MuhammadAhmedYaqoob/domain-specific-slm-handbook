# Edge learning and target-selection plan

## Learning objective and prerequisites

The learner will determine whether an exact model–format–runtime–backend tuple is genuinely usable on a named physical
device. Complete CH-02, CH-10–11, CH-16–19, and CH-29–31 first. You should be able to calculate parameter and KV-cache
memory, identify tokenizer/template mismatches, distinguish prefill from decode, and design controlled repeated trials.

Storage fit, runtime-memory fit, successful generation, interactive performance and sustained thermal operation are five
separate gates. A model-file size or simulator result cannot establish the latter four.

The sequence is deliberate. Select devices from accessible inventory rather than aspirational specifications; probe
export before expensive training; reject impossible memory cases by calculation; then measure placement, interaction,
and sustained behavior on physical hardware. This prevents a successful conversion from being mistaken for a usable
mobile product.

## Required target classes

- **EDGE-A0 — Low-end Android:** a currently accessible device selected from the lower supported RAM/performance
  segment, not an assumed specification.
- **EDGE-A1 — Mid-range Android:** the primary breadth target, selected for availability and representative current
  hardware.
- **EDGE-A2 — Flagship Android:** a higher-capability target used to evaluate GPU/NPU paths and larger candidates.
- **EDGE-I — Modern iPhone:** a named supported iPhone and iOS build with signing and instrumentation access.
- **EDGE-L — Consumer laptop:** exact CPU/GPU/NPU, RAM, OS and power mode; useful for offline desktop comparison.
- **EDGE-R — ARM64 edge computer:** a named Linux ARM64 board/system with cooling and power instrumentation.
- **Exploratory:** Raspberry Pi-class, Jetson-class, industrial edge and WebGPU/browser tuples where access and intended
  use justify them.

The final decision framework requires records for the first six classes. Exact SKU selection is delayed until inventory
and access are verified.

For a first capstone, prioritize one mid-range Android device because it represents a useful breadth constraint, then
one modern iPhone to expose platform differences. Add low-end and flagship devices only after the pipeline is stable.
Use the laptop as a debugging and offline comparison target, not as evidence for phone performance. Select an ARM64
board when Linux deployment is a real requirement; Raspberry Pi-class and Jetson-class systems answer different
questions and should not be treated as interchangeable.

## Device selection procedure

1. **Define use contract.** Freeze offline requirement, minimum task quality/safety, maximum artifact/download/storage,
   usable RAM, approved context/output, visible TTFT, sustained decode, reliability, session duration, update/rollback
   and privacy requirements.
2. **Inventory accessible hardware.** Record manufacturer/model/SKU, SoC/CPU/GPU/NPU, installed and available RAM,
   storage, OS/build/firmware, battery health, cooling/case and instrumentation. Use OS/vendor tools; label unavailable
   fields.
3. **Select representative devices.** Prefer devices physically controlled by the project, reproducible release builds
   and measurable thermal/power state. Avoid choosing solely because a vendor benchmark is favorable.
4. **Enumerate runtime tuples.** Pair exact model revision, tokenizer/template, artifact format/quantization, runtime
   commit/release, backend, app build and device. Documentation support is only a shortlist.
5. **Run an early export probe.** Before training, export/load a compatible tiny or untouched candidate; verify
   operators, tokenizer, stop handling, streaming, cancellation and backend placement. Preserve unsupported/fallback
   outcomes.
6. **Estimate memory conservatively.** Include weights, quantization metadata, runtime buffers, tokenizer, KV cache at
   approved context, app/UI and OS headroom. Estimate only to reject impossible tuples.
7. **Qualify on physical hardware.** Run the
   [edge protocol](../06-experiment-framework/protocols/edge-device-benchmark.md) with source/export semantic parity and
   conditioned cold/warm/sustained tests.
8. **Select per class.** Construct separate fronts by exact device/OS/runtime/workload. Never transfer a flagship result
   to mid-range or low-end devices.

## Runtime selection procedure

Candidate paths include llama.cpp/GGUF, ExecuTorch, LiteRT-LM, ONNX Runtime Mobile, Core ML, MLX Swift, MLC LLM, MNN,
NCNN and documented vendor SDKs. For each tuple record:

- supported architecture/operators and tokenizer/chat-template path;
- conversion commands, source/runtime revisions and artifact hash;
- CPU/GPU/NPU placement, partitions and fallbacks;
- quantization/KV/context support and memory mapping/loading behavior;
- Android/iOS/Linux bindings, licensing and app-package impact;
- streaming, cancellation, lifecycle, thread/affinity and error handling;
- maintenance status, security advisories and rollback compatibility.

A backend is “supported” only after the exact package runs and placement is evidenced on the exact device.

## Controlled measurement

Freeze battery band/health, charging and power mode, ambient or named start thermal state, case/cooling, display/radios,
background allow-list, free storage/RAM, app lifecycle, warm-up/cache state, threads/affinity and workload. Record
actual input/output token counts.

Measure:

- installed artifact and peak update bytes;
- process-cold startup, model ready, first model token and first visible fragment;
- prefill and decode tokens/s, inter-token and end-to-end latency;
- loaded and peak memory by phase; OOM/crash/timeout/invalid output;
- context boundary, cancellation, background/foreground, unload/reload and memory pressure;
- 10–30 minute sustained rate and thermal-state transition;
- energy/token only with valid device-level instrumentation; otherwise `null`;
- battery change only over a sufficiently long controlled run;
- corrupted/interrupted update, signature rejection, atomic activation and rollback;
- full domain/general/safety regression on the exact release package.

## Worked feasibility example

**Illustrative example:** a 1.5B-parameter artifact at an effective 4.5 bits per parameter has a theoretical weight
payload near 0.79 GiB before format metadata. Assume 0.20 GiB metadata/tokenizer, 0.60 GiB KV cache at the approved
context, 0.55 GiB runtime buffers, 0.35 GiB app/UI, and 1.0 GiB OS safety headroom; the planning requirement is about
3.49 GiB. These are not measured project values. Replace each component with the converter report, runtime allocation
trace, approved context formula, app profile, and observed OS headroom on the named device. Reject the tuple if the
measured peak or memory-pressure test violates the use contract, even when the model file fits storage.

## Suggested laboratory schedule

**Illustrative six-session sequence:** inventory and requirement freeze; untouched tiny export; full artifact
conversion and parity; cold/warm/lifecycle testing; 20-minute sustained thermal run; update corruption and rollback.
Repeat each performance condition enough times for the preregistered interval. Substitute measured conversion and
device-conditioning times; do not shorten cooldown or repetition merely to match the example schedule.

## Hard gates

1. Identity, license, signature and artifact hashes pass.
2. Source/export tokenizer/template and semantic/safety parity pass.
3. Package installs offline or through the approved signed-download path and leaves required storage headroom.
4. Peak RAM and context fit without disallowed fallback, OOM or OS termination.
5. Interaction and sustained-performance limits pass under conditioned trials.
6. Critical-safety, quality, retention and reliability limits pass.
7. App lifecycle, cancellation, update, corruption and rollback pass.
8. Placement claims have trace evidence; missing energy/NPU telemetry is disclosed.

Simulator/emulator runs may close app-function defects but cannot qualify memory, performance, energy or thermal gates.

## Target records and dependencies

- [Low-end Android](../14-decision-framework/targets/low-end-android-phone.md)
- [Mid-range Android](../14-decision-framework/targets/mid-range-android-phone.md)
- [Flagship Android](../14-decision-framework/targets/flagship-android-phone.md)
- [Modern iPhone](../14-decision-framework/targets/modern-iphone.md)
- [Consumer laptop](../14-decision-framework/targets/consumer-laptop.md)
- [ARM edge device](../14-decision-framework/targets/arm-edge-device.md)
- [Edge guide](../09-edge-guide/README.md)
- [Watchlist W-06–W-09](fast-moving-topics-watchlist.md)

## Selection rule

Select the simplest qualified tuple per class after hard-gate filtering and uncertainty-aware Pareto analysis. Report
“no qualified candidate” when appropriate. Device specs from a product page are `VENDOR-REPORTED`; project runtime
results require `EXECUTED` evidence.

## Learner exit test

For one Android or iPhone, demonstrate signed release installation, offline generation, source/export parity, measured
loaded and peak memory, cold and warm TTFT, sustained decode, cancellation, background/foreground behavior, corrupted
update rejection, and rollback. Explain every fallback and state which claims remain unavailable because the device
does not expose valid telemetry.
