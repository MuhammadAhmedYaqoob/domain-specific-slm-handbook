# Runtime record tutorial

A runtime record describes one pinned software release and the exact model–artifact–backend–device combinations tested
with it. “The runtime supports Android” is documentation; “this artifact executed correctly on this phone” requires a
retained run.

## Worked example: llama.cpp on an Android phone

This is a realistic **illustrative record**, not a claim that the described project run occurred.

### Runtime and build identity

- **Record ID:** `llamacpp-android-arm64-commit-to-freeze`
- **Runtime:** llama.cpp with the application owning chat orchestration
- **Owner:** ggml-org
- **Official repository:** https://github.com/ggml-org/llama.cpp
- **Version:** full commit must be selected and recorded before building; never use mutable `master` as an identity
- **Status:** active project according to repository activity checked 2026-11-23
- **License:** MIT; verify the pinned repository license before distribution
- **Target:** Android 14, arm64-v8a, named Snapdragon phone
- **Build:** Android NDK and CMake versions, compiler flags, dependency lock, command, and log must be retained
- **Security:** review advisories, submodules, downloaded tools, native library hashes, and SBOM at the pinned commit

### Platform and model evidence

| Tuple | Documentation state | Execution state | What would prove it |
| --- | --- | --- | --- |
| Android arm64 CPU, GGUF, supported decoder model | Official examples exist | Not executed in this illustration | Build log, artifact hashes, output, and trace |
| Vulkan backend on the same phone | Backend is selectable in some builds | Placement not established | Backend trace showing offload and fallback |
| Four-bit GGUF artifact | Format is accepted for supported architectures | Quality not established | Conversion provenance, parity, and evaluation runs |

The model row must name the exact checkpoint revision, tokenizer hash, derived GGUF hash, architecture features,
maximum tested context, required operators, and staged state.

### Conversion, generation, and cache

- **Converter:** pin the converter to the same llama.cpp commit and retain the exact command.
- **Quantization:** record weight encoding, group/block details, effective bits per weight, activation/accumulation
  dtype, and KV-cache precision.
- **Tokenizer and template:** identify whether the artifact, runtime, or app owns each; run golden token and rendered
  prompt tests.
- **Streaming:** test Unicode boundaries, cancellation, stop tokens, simultaneous sessions, and thread safety.
- **KV cache:** record allocation, precision, context-limit behavior, reset isolation, memory growth, and cached versus
  full-prefix parity.

### Validation and performance protocol

- **Reference:** pinned high-precision framework checkpoint and tokenizer.
- **Numerical checks:** prefill logits, cached decode, greedy tokens, and tolerance rationale.
- **Quality checks:** identical domain, general, and safety suites before and after conversion.
- **Workload:** fixed prompt/output lengths, context, batch, concurrency, trials, warm-up, threads, and offload.
- **Device state:** exact device/SoC, OS, RAM, battery, charging state, ambient temperature, background activity, and
  thermal state.
- **Results:** link run IDs for startup, TTFT, prefill, decode, inter-token latency, peak memory, energy, temperature,
  sustained performance, fallback, crashes, and OOMs.

### Product decision

- **Candidate role:** portable Android CPU baseline.
- **Current decision:** documentation candidate only; no compatibility or performance conclusion until execution.
- **Release blockers:** pinned build, terms review, model parity, quality floor, sustained physical-device performance,
  integrity enforcement, lifecycle tests, privacy review, and rollback.
- **Recheck:** whenever the runtime, converter, model, NDK, OS, driver, or target device changes.

## Field-by-field instructions

1. **Identity:** pin the runtime, converter, dependencies, SDK, drivers, and build output. Hash source and binaries.
2. **Environment:** retain OS, architecture, compiler, flags, bindings, image digest, firmware, lockfile, SBOM, command,
   and build log.
3. **Platform matrix:** make each row independently testable. “Documented,” “built,” “executed,” and “placement
   captured” are separate facts.
4. **Model compatibility:** name the exact checkpoint and artifact hashes. Progress through conversion, load,
   execution, numerical validation, quality validation, and production qualification.
5. **Quantization:** record the algorithm, serialized format, kernel, all dtypes, granularity, exclusions, calibration,
   effective width, and conversion cost.
6. **Generation and cache:** test tokenization, templates, streaming, stops, cancellation, concurrency, cache parity,
   context limits, and session isolation.
7. **Performance:** define boundaries and device state before measuring. Separate prefill from decode and file size from
   loaded and peak memory.
8. **Lifecycle:** test app backgrounding, interruption, corruption, low storage, unsupported devices, integrity,
   rollback, logs, and privacy.
9. **Unknowns:** name the official page, build inspection, trace, or physical-device test that will resolve them.

## Reusable copy

### Identity and environment

- **Record ID:** `runtime-2.4.1-full-commit-target-backend`
- **Layers:** Example generation wrapper over Example Runtime 2.4.1
- **Repository and revision:** Official URL and full commit hash
- **Release and access:** 2026-10-15 release; accessed 2026-11-24
- **Status and source:** Active according to the official release page
- **License:** Exact SPDX identifier, version, and pinned license URL
- **Hashes:** Source archive, build tools, native libraries, and package SHA-256 values
- **Environment:** Android 15 arm64-v8a, named device/SoC, NDK 29, CMake version, flags, firmware, and driver
- **Build evidence:** Lockfile, SBOM, exact command, full log, package size, and supported ABIs

### Compatibility and conversion

- **Checkpoint:** `organization/examplelm-1b-instruct` at full commit
- **Artifact:** `examplelm-1b-q4-v1.gguf` with SHA-256 and source manifest
- **Backend:** CPU vector kernels with four threads; Vulkan candidate tested separately
- **State:** Executes; numerical and quality validation still required
- **Conversion:** Converter commit, exact command/configuration, calibration dataset hash, duration, hardware, and peak
  memory
- **Numerics:** Four-bit weight encoding, FP16 activations and accumulation, FP16 KV cache, group details, exclusions,
  and effective bits per weight
- **Operators:** Exact custom or unsupported operations and graph rewrites

### Validation, performance, and product

- **Token contract:** Tokenizer/template owner, hashes, golden test, Unicode streaming, stops, and cancellation
- **Cache:** Layout, allocation, precision, reset, context-limit behavior, parity, and measured growth
- **Reference:** Exact framework/checkpoint/precision tuple and test-corpus hash
- **Parity:** Tolerances, prefill/decode results, greedy tokens, and retained failures
- **Quality:** Identical domain, general, and safety evaluation run IDs
- **Protocol:** Versioned benchmark with device state, workload, trials, warm-up, and measurement boundaries
- **Results:** Run IDs for startup, TTFT, prefill, decode, memory, energy, thermal, fallback, and failures
- **Lifecycle:** App integration package, minimum OS/ABI, integrity, interruption, low-storage, rollback, privacy, and
  unsupported-device tests
- **Decision:** Candidate until numerical, quality, sustained performance, security, and lifecycle gates pass
- **Recheck:** Runtime, converter, model, SDK, OS, driver, hardware, or license change

### Completion checks

- [ ] Runtime, converter, backend, SDK, dependencies, model, tokenizer, and artifacts are immutable.
- [ ] Documentation, conversion, load, execution, parity, quality, and production evidence are separate.
- [ ] Graph placement and fallback are traced.
- [ ] Prefill, decode, loaded memory, peak memory, energy, and sustained thermal behavior are measured separately.
- [ ] Lifecycle, integrity, rollback, privacy, and unsupported-device behavior pass before release.
- [ ] Every unknown names a verification action.

Cross-links: [model checkpoint record](model-checkpoint-record-template.md) ·
[source record](source-record-template.md) ·
[edge runtime survey](../04-research-comparison-report/edge-runtime-survey.md)
