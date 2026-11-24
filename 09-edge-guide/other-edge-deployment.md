# ARM Linux, Jetson, laptops, industrial edge, and WebGPU

## 1. Target map

| Target | Baseline | Accelerator path | Main constraint |
| --- | --- | --- | --- |
| Raspberry Pi/ARM64 | llama.cpp/GGUF CPU | documented Vulkan/vendor path | cooling and bandwidth |
| NVIDIA Jetson | CPU then CUDA | supported TensorRT/runtime | JetPack coupling and shared RAM |
| Laptop | native CPU | CUDA, Metal/MLX, Core ML, OpenVINO | battery and user contention |
| Industrial computer | unprivileged local service | exact vendor backend | lifecycle and safety isolation |
| Browser | WebLLM/WebGPU | WebGPU | quota, adapter limits, tab lifecycle |

## 2. Worked examples

An 8 GiB ARM SBC can start its search with a 0.5B–1.5B low-bit model, but the actual gate is available memory and
sustained cooling. If a 1B artifact uses the illustrative 1.57 GB peak budget from the hardware lesson, it can fit in
principle; storage I/O, bandwidth, and thermal tests determine usability.

On an 8 GiB Jetson, that same model shares memory with CUDA context, display, camera, and application pipelines.
Reserve those workloads first. Record `nvpmodel`; a maximum-power mode can heat and throttle, so compare absolute early
and late rates rather than assuming it wins.

## 3. ARM64 and Raspberry Pi build path

```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp
git checkout "$(git rev-parse HEAD)"
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DGGML_NATIVE=OFF
cmake --build build --config Release -j 4
sha256sum /opt/models/mechanic-q4.gguf
./build/bin/llama-cli \
  -m /opt/models/mechanic-q4.gguf -c 2048 -n 128 \
  --threads 4 -p "Explain a safe first diagnostic check."
```

Pin a reviewed commit instead of using the self-resolved commit in a production build. `GGML_NATIVE=OFF` favors fleet
portability; build a cohort-specific binary only after checking CPU features. Expected behavior is local token output.
Common failures are OOM, illegal instructions, undervoltage, template mismatch, and decreasing throughput as heat rises.

Integration path:

1. Install a 64-bit OS, stable power, adequate storage, and active cooling.
2. Inventory CPU flags, kernel, libc, firmware, and available memory.
3. Verify artifact hash and source-target semantic parity.
4. Sweep thread counts; more threads can increase contention and energy.
5. Run as an unprivileged CLI or loopback-only service under systemd.
6. Add restart limits, watchdog, storage pressure, corruption, and rollback tests.

## 4. Jetson path

Bind module, JetPack, CUDA, cuDNN, TensorRT/runtime, firmware, and power mode as one cohort. Containers pin user space
but do not replace the host kernel or GPU driver.

```bash
uname -a
cat /etc/nv_tegra_release
sudo nvpmodel -q --verbose
free -h
docker run --rm --gpus all --read-only \
  --mount type=bind,src=/opt/models,dst=/models,readonly \
  --security-opt=no-new-privileges \
  nvcr.io/nvidia/l4t-base:r36.4.0 cat /etc/nv_tegra_release
```

The command verifies that a pinned L4T userspace can see the host stack; it does not benchmark inference. Next, build a
supported CUDA runtime or TensorRT engine, compare outputs against CPU, inspect placement/copies, and benchmark with the
full sensor workload.

## 5. Industrial and laptop integration

Keep generative output outside deterministic control:

```text
sensors and safety I/O -> deterministic controller -> read-only observation
-> local SLM suggestion -> schema/policy check -> qualified human
```

Run unprivileged, limit filesystem/device/network access, authenticate local APIs, cap context and concurrency, and use
secure boot plus signed A/B updates where required. On laptops, test AC and battery separately and record power profile,
display, fans, and selected integrated/discrete accelerator.

## 6. WebGPU integration

```javascript
export async function openWebGPU(minimumBufferBytes) {
  if (!globalThis.isSecureContext || !navigator.gpu) {
    throw new Error("WebGPU is unavailable; model download was not started.");
  }
  const adapter = await navigator.gpu.requestAdapter({
    powerPreference: "high-performance",
  });
  if (!adapter || adapter.limits.maxBufferSize < minimumBufferBytes) {
    throw new Error("This adapter does not meet the release limits.");
  }
  const device = await adapter.requestDevice();
  device.lost.then(() => {
    window.dispatchEvent(new Event("local-model-device-lost"));
  });
  return device;
}
```

`maxBufferSize` is not free memory. Before download, check persistent storage quota, verify a signed shard manifest, run
a bounded allocation/compile test, and place generation in a worker. Handle device loss, reload, cache eviction, and
service-worker termination. Display “offline ready” only after a network-denied restart succeeds.

## 7. Validation

For every target, run token/template/logit/cache parity; cold/warm load; TTFT, prefill, and decode; phase memory;
maximum context; CPU/accelerator placement; sustained power/cooling tests; reboot or suspend; process/device loss;
storage/RAM pressure; corruption; interrupted update; rollback; and network denial.

## Sources

- [Raspberry Pi documentation source](https://github.com/raspberrypi/documentation)
- [Jetson platform power and performance](https://docs.nvidia.com/jetson/archives/r36.4.4/DeveloperGuide/SD/PlatformPowerAndPerformance.html)
- [OpenVINO GenAI](https://docs.openvino.ai/2025/openvino-workflow-generative/inference-with-genai.html)
- [WebGPU specification](https://www.w3.org/TR/webgpu/)
- [WebLLM](https://github.com/mlc-ai/web-llm)
