# Hardware and environment lesson

## Question answered

What exact hardware, software, and physical state produced a measurement?

## Why controls matter

Device temperature, power mode, page cache, drivers, and runtime flags can change performance enough to reverse a
ranking. “Android” or “GPU” is not an environment identity.

## Common novice mistakes

- Recording a marketing family instead of an exact SKU and OS build.
- Omitting available RAM, charging state, ambient temperature, or cache state.
- Assuming backend selection proves NPU placement.
- Comparing a cold candidate with a warm candidate.

## Complete worked illustrative experiment

Device A runs five warm trials at 18.0 token/s near 25 °C. The same package runs at 14.4 token/s after thermal state
changes. The decline is `(14.4-18.0)/18.0=-0.20`, or 20%. Without state capture, that thermal effect could be
misattributed to a model change. These are illustrative values.

## Fully populated illustrative manifest

```yaml
schema: slm-environment/v1
environment_id: env-android-lab-001
captured_at: 2026-11-24T05:00:00Z
class: android
hardware:
  device_or_instance: ExamplePhone-X1-128GB
  cpu: {model: ExampleSoC-8core, topology: 1-plus-3-plus-4}
  accelerators: [ExampleGPU-v2, ExampleNPU-v1]
  ram: {installed_bytes: 8589934592, available_before_bytes: 5368709120}
software:
  os_build: Android-16-example-build-001
  kernel_firmware_drivers: {kernel: 6.1-example, gpu_driver: example-42}
  container_digest: not-applicable-physical-device
  lockfile_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
runtime:
  name_revision: example-runtime-1.4.0-commit-a1b2c3d
  backend_build_flags: arm64-neon-release
state:
  power_mode: balanced
  ambient_c: 23.0
  charging: false
raw_inventory: {uri: artifacts/env-android-lab-001.json, sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}
missing: [npu_utilization-counter-not-exposed-by-example-driver]
```

`environment_id` names the immutable capture; `hardware` identifies compute and available memory; `software` fixes
the OS and dependency state; `runtime` fixes executable behavior; `state` captures physical conditions;
`raw_inventory` links evidence; `missing` explains unavailable fields.

## Reusable specification

Capture exact CPU/accelerator topology, memory, storage, OS, kernel, drivers, code and image digests, runtime and build
flags, monitoring clocks, and raw inventory. Training adds distributed topology, precision, checkpoint policy, power
cap, and accelerator-hours. Edge adds battery, charging, display/radios, background policy, case, ambient and thermal
state. AWS adds Region, instance, tenancy, purchase model, AMI, storage/network, replicas, autoscaling, and load
generator. Differences must be blocked, modelled, or labelled conditional.
