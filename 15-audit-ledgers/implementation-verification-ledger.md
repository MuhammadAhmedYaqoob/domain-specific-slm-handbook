# Implementation verification ledger

## What is checked and why

This ledger asks whether a learner can move from a code listing to a repeatable result. It checks exact source bytes,
dependencies, inputs, commands, environment, assertions, outputs, failures, and target hardware. This matters because
syntactically convincing ML and deployment code can fail through version conflicts, unsupported operators, memory
limits, wrong chat templates, data leakage, or device-specific behavior.

`UNVERIFIED IMPLEMENTATION` means the instructions are useful but no retained target run proves them. `EXECUTED`
means the exact revision ran on the named target with logs and outputs. `REPRODUCED` additionally requires an
independent repeat. A code block, screenshot, or upstream tutorial does not move a listing to either higher status.

## Verification records

<!-- markdownlint-disable MD013 MD060 -->
| Area | What the learner can use now | Current record | Complete validation method |
| --- | --- | --- | --- |
| Environment | Candidate Python, ML-library, container, and hardware setup guidance | Not yet performed: no resolved lock, image digest, or clean-install log | Freeze OS/driver/CUDA and package versions; create lock and SBOM; build from a clean machine; import every dependency; save commands, logs, and image digest |
| Data pipeline | Acquisition, schema, cleaning, deduplication, splitting, lineage, and rights checks | Not yet performed on a project corpus | Approve source rights; hash inputs; run schema, PII, duplicate, contamination, and leakage tests; inspect samples; retain rejected records and reports |
| Training | CPT, SFT, full tuning, LoRA, QLoRA, preference, checkpoint, and resume procedures | Not yet performed: no project checkpoint or training log | Run a tiny fixture, overfit test, save/reload, deterministic evaluation, interruption/resume test, then a controlled full run; retain configs, seeds, metrics, failures, and hashes |
| Compression | Quantization, pruning, distillation, adapter merge, and combined workflows | Not yet performed on an exact artifact | Freeze source model and producer; create artifact; check load and output parity; measure quality, size, memory, latency, and failures on each claimed runtime/target |
| Evaluation | Scorers, result schemas, statistics, plots, and Pareto procedures | Not yet performed with a sealed benchmark | Unit-test scorers on known fixtures; freeze and hash test data; repeat runs; report uncertainty and exclusions; have a statistical reviewer check conclusions |
| AWS | Container, infrastructure, endpoint, monitoring, canary, rollback, cleanup, and cost procedures | Not yet performed in a named account and Region | Deploy exact image/model; test auth, health, load, failure, canary, rollback, and teardown; retain CloudWatch records, resource IDs, dates, and reconciled billing |
| Android | GGUF/llama.cpp, Kotlin/Compose/JNI, streaming, storage, and update path | Not yet performed: no release build or device result | Pin NDK/runtime/model; build release ABI packages; sign and install; test offline inference, lifecycle, corruption, update/rollback, memory, latency, battery, and thermal behavior on named phones |
| iOS | SwiftUI/runtime integration, packaging, streaming, storage, and update path | Not yet performed: no signed install or device profile | Pin Xcode/iOS/runtime/model; build and sign; install on named devices; test offline lifecycle, compute placement, memory, latency, battery, thermal behavior, and rollback |
| Other edge | ARM Linux, Raspberry Pi, Jetson, laptop, and WebGPU procedures | Not yet performed on named targets | Record board/device, OS, runtime, model, backend, power mode, and workload; run correctness and sustained-performance tests; retain raw measurements |
| Release controls | Registries, manifests, checksums, signing, integrity, promotion, revocation, and rollback designs | Not yet exercised end to end | Sign a frozen artifact; verify before load; simulate corruption and revoked versions; promote, roll back, and recover while retaining the audit trail |
<!-- markdownlint-restore -->

## Eleven learning tools

`TOOL-01`–`TOOL-11` cover model size, quantization memory, LoRA parameters, training memory, mobile feasibility,
inference latency, KV-cache memory, teacher/student comparison, quality/compression plots, AWS training cost, and
cloud/mobile comparison. All 11 are documented and registered as `LST-TOOL-0001`–`LST-TOOL-0011`.

**Execution status:** 0/11 have retained run records; validation is not yet performed. For each tool, extract the exact
listing, hash it, run syntax and static checks, test a hand-calculated fixture, test zero/negative/oversized or missing
inputs as applicable, compare output with assertions, and record environment and reviewer.

## Passing and failing evidence

**Pass:** an Android listing is extracted unchanged, built with pinned Gradle/NDK/runtime versions, installed on a
named phone, run offline with a hashed model, and accompanied by build logs, output assertions, peak memory, latency,
and a reviewer record.

**Fail:** the chapter includes complete Kotlin and C++ code and an upstream project says Android is supported, but
there is no project build, install, or device log. The correct status remains `UNVERIFIED IMPLEMENTATION`.

## How the repository advances

Assign a named person only when work begins; until then responsibility is **unassigned**. Register every executable
listing with a stable ID and extraction markers. Freeze dependencies, model, data, runtime, OS, hardware, and workload.
Preserve failed attempts as well as successes. A changed tuple requires re-execution or a documented equivalence
argument. Independent repetition is required before using `REPRODUCED`.
