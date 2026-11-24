# Mobile benchmarking, battery, and thermal measurement

## 1. Measure the complete unit

Freeze model, tokenizer/template, artifact and cache precision, runtime build, backend placement, device/OS/driver,
prompt token IDs, output policy, context, device state, and protocol. A change creates a new test unit. Correctness and
safety pass before timing.

## 2. Metrics and formulas

Record monotonic timestamps for request submission, prefill start/end, first model token, first rendered fragment,
every generated token, and completion.

\[
TTFT_{visible}=t_{first\ rendered}-t_{submit}
\]

\[
r_{prefill}=N_{input}/(t_{prefill\ end}-t_{prefill\ start})
\]

\[
r_{decode}=(N_{generated}-1)/(t_{last}-t_{first})
\]

Decode rate requires at least two generated tokens. Report median and p95 inter-token latency. Memory records include
before load, load peak, post-load, prefill peak, early/late decode, maximum context, reset, and unload.

For a declared energy boundary:

\[
E_{token}=(E_{active}-E_{matched\ idle})/N_{generated}.
\]

Whole-device, wall, battery-rail, SoC-rail, and process estimates are not interchangeable. Battery percentage is too
coarse for short trials. If no calibrated measurement exists, omit J/token and report only the available relative
diagnostic.

## 3. Worked illustrative calculation

Suppose a calibrated whole-device meter records 1,800 J during a 15-minute active run and 720 J during a matched idle
run. If the model generated 900 tokens, incremental energy is 1,080 J and the illustrative value is 1.2 J/token.
If early decode is 8.0 tokens/s and late decode is 6.4, sustained ratio is \(6.4/8.0=0.80\). This does not prove thermal
throttling: correlate context length, clocks, thermal state, and background activity.

## 4. Controlled workload

- Short prompt: 64–128 input tokens.
- Medium prompt: 512–1,024 tokens.
- Long prompt: near the approved context boundary.
- Fixed 32, 128, and 512 output-token workloads where safe.
- Cache append, edited history, reset, and rollover cases.
- Domain terminology, units, Unicode, structured output, refusal, and safety prompts.
- A 20-minute sustained run, or another preregistered product-derived duration.

Use greedy decoding for parity/performance, then natural completion for product quality. Forced output controls early-EOS
bias but does not replace realistic usage.

## 5. Step-by-step protocol

1. Verify signatures, hashes, tokenizer/template, cache behavior, quality, and safety.
2. Record SKU, SoC, OS, firmware, installed/available RAM, storage, ambient conditions, case, battery, charger,
   display, radios, background allow-list, threads, and backend.
3. Define cold and warm states, warm-ups, start-temperature band, trial count, and exclusion rules.
4. Randomize variants or use AB/BA blocks; do not always run the candidate after the baseline.
5. Capture per-token events, phase memory, placement, clocks, thermal state, and energy.
6. Count crash, OOM, timeout, cancellation, and invalid output in the denominator.
7. Cool or continue according to the preregistered carryover rule.
8. Archive raw JSONL and external traces by immutable hash.
9. Summarize per-trial throughput, median/IQR, supported tail percentiles, confidence intervals, and failure rate.

## 6. Executable record shape

Use concrete command-line arguments to prevent empty result fields:

```python
import argparse
import json
import time

parser = argparse.ArgumentParser()
parser.add_argument("--artifact-sha256", required=True)
parser.add_argument("--device-sku", required=True)
parser.add_argument("--os-build", required=True)
parser.add_argument("--backend", required=True)
parser.add_argument("--prompt-id", required=True)
args = parser.parse_args()

record = {
    "schema": "edge-benchmark/v1",
    "identity": vars(args),
    "timestamps": {"recorded_monotonic_ns": time.monotonic_ns()},
    "outcome": {"state": "started"},
}
print(json.dumps(record, separators=(",", ":"), sort_keys=True))
```

Extend the record as each measured event occurs and finish it with a terminal outcome. A JSON Schema should require
each metric only when its measurement boundary and outcome make it meaningful; do not encode unknowns as zero.

## 7. Platform instrumentation

On Android, combine in-app monotonic events with Perfetto/System Trace, memory profiler or `dumpsys meminfo`, and the
thermal API. Battery claims require unplugged testing or a declared calibrated external boundary. On iOS, use
`os_signpost`, Time Profiler, Allocations/VM Tracker, Core ML or Metal instruments, and
`ProcessInfo.processInfo.thermalState`. Treat Energy Diagnostics as relative unless calibrated.

## 8. Common failures and validation

Common errors are comparing cold with warm, unequal tokenized prompts, simulator with hardware, burst with sustained,
server batching with batch-one mobile, or vendor TOPS with tokens/s. Do not discard OOM or thermal trials as outliers.

Validate the harness with a fake deterministic engine, monotonic-clock tests, token-count fixtures, known energy
calculations, interrupted trials, and schema validation. Then inspect a sample trace manually and reproduce summary
statistics from raw records.

## Sources

- [Perfetto](https://perfetto.dev/docs/)
- [Android system tracing](https://developer.android.com/topic/performance/tracing)
- [Apple performance and metrics](https://developer.apple.com/documentation/xcode/improving-your-app-s-performance)
- [Apple performance signposts](https://developer.apple.com/documentation/os/recording-performance-data)
