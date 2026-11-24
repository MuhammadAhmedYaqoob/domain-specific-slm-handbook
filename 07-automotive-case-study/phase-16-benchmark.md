# Phase 16: Benchmark cloud and edge under controlled conditions

[Previous phase](phase-15-mobile-deployment.md) · [Next phase](phase-17-final-selection.md)

## Learning objectives

- Define comparable workloads and environment controls.
- Separate startup, prefill, decode, sustained, reliability, energy, and cost metrics.
- Preserve raw trials and protocol deviations.

## Theory and purpose

Benchmark numbers are meaningful only with their conditions. TTFT combines queueing, tokenization, prefill, and
scheduling; decode tokens/s describes autoregressive generation; energy needs a declared measurement boundary.
Warm bursts do not predict sustained mobile behavior. Server aggregate throughput is not comparable with
batch-one mobile speed. Raw per-trial data supports confidence intervals and failure accounting.

## Prerequisites

- Exact qualified AWS and mobile release artifacts.
- Fixed prompts stored as token IDs and a declared output policy.
- Instrumentation for monotonic timing, memory, failures, and available energy/thermal counters.

## Step-by-step method

1. Freeze artifact, runtime, backend, hardware, OS, firmware, prompt tokens, context, and decoding.
2. Declare warm-ups, 30 or more measured trials, timeout, failure denominator, and sustained duration.
3. Record startup, TTFT, prefill, per-token timestamps, decode, end-to-end latency, memory, and failures.
4. For mobile, control charge, radios, display, background activity, ambient conditions, and starting thermal state.
5. For AWS, sweep offered load and record queues, scaling, utilization, errors, and dated allocated cost.
6. Summarize distributions and confidence intervals without deleting invalid or failed attempts.

## Worked automotive example

**`ILLUSTRATIVE` benchmark.** Thirty fictional 512-token-input/128-token-output trials give LoRA-Q4 Android p50/p95
TTFT of 1.6/1.9 s, median decode 6.1 tok/s, 2.35 GiB peak memory, 12% sustained slowdown, and 0/30 failures.
INT8 AWS at 2 requests/s gives 0.55/0.72 s TTFT, 32 tok/s single-request decode, and 1/300 failures. The conditions
differ, so the values describe target fitness; they do not make AWS “6.1 times faster.”

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```yaml
schema: automotive-slm.benchmark/v1
evidence: ILLUSTRATIVE
prompt_tokens: 512
generated_tokens: 128
warmups: 3
measured_trials: 30
decoding: {do_sample: false}
mobile:
  charging: false
  sustained_minutes: 20
  failure_denominator: all_attempts
aws:
  offered_load_requests_per_second: 2
  failure_denominator: all_attempts
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` summary.** The nearest-rank calculation is shown explicitly and uses only ten values to remain
readable. The real protocol uses at least 30 trials, preserves every raw event, reports the quantile convention,
and computes uncertainty appropriate to the distribution.

```python
from statistics import median

ttft_ms = [1550, 1610, 1580, 1710, 1900, 1660, 1590, 1620, 1680, 1570]
ordered = sorted(ttft_ms)
p50 = median(ordered)
p95_nearest_rank = ordered[max(0, (95 * len(ordered) + 99) // 100 - 1)]
print({"trials": len(ordered), "ttft_p50_ms": p50, "ttft_p95_ms": p95_nearest_rank})
```

## Expected behavior

Every attempted trial contributes a raw event or explicit failure. Summaries report declared quantiles and conditions, while cloud and mobile results remain in separate comparable strata.

## Common failures and recovery

- **Failure:** Discarding OOMs and timeouts biases both speed and quality.
- **Failure:** Raw tokens/s across different tokenizers does not represent equal text throughput.
- **Failure:** Charging one phone and discharging another changes thermals and power.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Use non-sensitive prompts, stop tests at unsafe thermal states, and record unavailable counters as unavailable rather than estimating them silently.

## Interpret the result

Benchmark evidence is valid only for the exact workload and environment. Phase 17 compares candidates inside compatible target strata.

## Transition

[Phase 17](phase-17-final-selection.md) replaces intuition with hard gates, Pareto dominance, and target-specific utility.
