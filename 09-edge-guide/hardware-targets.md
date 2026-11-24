# Hardware tiers and memory budgets

## 1. Concepts: CPU, GPU, and NPU

A CPU provides broad operator coverage, mature control flow, and the most portable correctness baseline. ARM64 Neon
vectorizes arithmetic; dot-product and i8mm instructions help only when both the processor and selected kernel support
them. A GPU exposes high parallel throughput and bandwidth through Metal, Vulkan, CUDA, or a runtime delegate, but
decode's small matrix operations and backend transfers can limit benefit. An NPU is optimized for supported tensor
graphs at specific shapes and precisions. Its advertised TOPS does not prove that a decoder, dynamic KV cache, or
tokenizer can run there.

Unified memory removes some explicit copies, not capacity competition. CPU, GPU/NPU, display, OS, app, weights,
workspace, and KV cache still share bandwidth and memory pressure.

## 2. Capability tiers

| Tier | Admission evidence | Starting strategy |
| --- | --- | --- |
| Low | CPU path fits with headroom; accelerator is absent or unhelpful | 0.1B–0.5B low-bit, short context |
| Mid | CPU passes and one qualified accelerator or stronger CPU meets interaction gates | 0.5B–1.5B low-bit |
| High | Exact backend has measured memory and sustained headroom | 1B–4B first; larger only by evidence |

These ranges are search bands, not promises. Tier assignment uses available memory under workload, not retail price or
installed RAM. Keep explicit unsupported states and never silently route an offline request to cloud.

## 3. Memory model

For \(N\) parameters and effective \(b_w\) bits per weight,
\(M_W=N b_w/8\). Effective bits include group scales, zero points, metadata, and high-precision tensors.

For a conventional decoder KV cache,

\[
M_{KV}=2LBT H_{kv}d_hb_{kv}
\]

where \(L\) is layer count, \(B\) batch, \(T\) cached tokens, \(H_{kv}\) KV heads, \(d_h\) head width, and \(b_{kv}\)
bytes per element. Add activations, prefill workspace, backend programs, tokenizer, runtime, UI, temporary conversion,
and a pressure-tested margin.

### Worked device example

For the 1B example in the course overview, weights are 562.5 MB and ideal KV is 192 MiB at 4,096 tokens. Suppose the
target has 2.2 GiB available after the OS and background allow-list. A planning ledger allocates 0.56 GB weights,
0.20 GB KV, 0.32 GB workspace, 0.18 GB app/runtime, and 0.31 GB margin: 1.57 GB. The remaining 0.63 GB is useful
pressure reserve. Every non-formula number is illustrative; measure load, prefill, decode, reset, and update peaks.

Storage needs more than the active model. A side-by-side update may require:

\[
S_{peak}=S_{active}+S_{previous}+S_{partial}+S_{staged}+S_{unpacked}+S_{margin}.
\]

## 4. Build a hardware cohort

1. Record exact manufacturer SKU, SoC, RAM, storage, ABI, OS build, firmware, and driver.
2. Read CPU features at runtime; do not ship a native-tuned binary to an unqualified fleet.
3. Record current available memory under the actual app workload.
4. Verify the SoC in the backend's current support documentation.
5. Define maximum input, output, and total context for this cohort.
6. Run the portable CPU model, then separately export and test GPU/NPU variants.
7. Capture backend partitions, fallbacks, precision, and transfer boundaries.
8. run maximum-context and 20-minute sustained tests before assigning a tier.

An inventory record should contain concrete values and omit unknown optional fields rather than representing missing
measurements as zero. Record measurements in a separate benchmark result linked by artifact hash.

```yaml
schema: edge-hardware-cohort/v1
cohort_id: pixel-class-arm64-android-15-cpu
device:
  manufacturer_model_sku: lab-device-01
  soc: recorded-from-device-inventory
software:
  os_build: recorded-build-fingerprint
compute:
  abi: arm64-v8a
  backend: cpu
policy:
  context_tokens_maximum: 2048
  output_tokens_maximum: 256
```

The values `lab-device-01` and `recorded-*` are safe example labels, not physical claims. Replace the complete example
record during inventory; the production schema should reject those training labels.

## 5. Other device classes

- **Raspberry Pi/ARM SBC:** use a 64-bit OS, stable power, SSD when appropriate, active cooling, and ARM CPU baseline.
- **Jetson:** bind module, JetPack, CUDA, TensorRT/runtime, power mode, and shared-memory budget as one cohort.
- **Laptop:** test AC and battery separately; record power profile and selected integrated/discrete accelerator.
- **Industrial edge:** add watchdog, brownout recovery, secure boot, extended-temperature, and A/B update tests.
- **Browser/WebGPU:** qualify adapter limits, persistent quota, cache eviction, and tab/device-loss behavior.
- **Microcontroller:** use it for sensing and deterministic interlocks; ordinary MCU memory cannot host a conversational
  SLM. A 100M-parameter 4-bit weight payload alone is 50 MB before scales and runtime.

## 6. Expected behavior, failures, and validation

Expected behavior: admission code selects only an artifact whose measured storage, RAM, context, backend, and sustained
profile match the cohort.

Common failures include illegal instructions from an over-tuned binary, OS kills despite “enough total RAM,” NPU
fallback, GPU regressions from transfers, Pi undervoltage, and Jetson throttling in an aggressive power mode.

Validate with phase-specific allocation traces, context sweeps, backend placement traces, CPU comparison, memory
pressure, process restart, and controlled sustained runs. Requalify after changing model, runtime, OS, driver, or case.

## Sources

- [Arm ABI specifications](https://github.com/ARM-software/abi-aa)
- [Android NNAPI migration](https://developer.android.com/ndk/guides/neuralnetworks/migration-guide)
- [Apple Core ML](https://developer.apple.com/documentation/coreml)
- [NVIDIA Jetson power management](https://docs.nvidia.com/jetson/archives/r36.4.4/DeveloperGuide/SD/PlatformPowerAndPerformance.html)
