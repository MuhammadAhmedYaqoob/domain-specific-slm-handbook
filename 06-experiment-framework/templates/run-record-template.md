# Run record lesson and template

## Question answered

What exactly happened in one attempted training, conversion, evaluation, benchmark, or packaging action?

## Why controls matter

Attempt-level records preserve failures and resource use.

## Common novice mistakes

Do not overwrite retries, infer success from exit code, or leave missing
telemetry unexplained.

## Fully populated illustrative example

```yaml
schema: slm-run-record/v1
run_id: run-example-benchmark-001
experiment_id: exp-example-int4-001
plan_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
variant_id: int4-child
attempt: 1
stage: benchmark
evidence_label: ILLUSTRATIVE
operator: {id: operator-example-01, role: benchmark-engineer}
time:
  scheduled_at: 2026-11-24T06:00:00Z
  started_at: 2026-11-24T06:01:00Z
  ended_at: 2026-11-24T06:31:00Z
  monotonic_clock: CLOCK_MONOTONIC
identity:
  inputs: [{artifact_id: artifact-example-quantized-001, sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb", verified_before: true}]
  config_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  code_commit: a1b2c3d4
  environment_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  workload_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
command:
  argv_redacted: [benchmark-app, --variant, int4-child, --sessions, "5"]
  command_bundle_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  working_directory_revision: a1b2c3d4
seeds: {training: not-applicable, sampler: not-applicable, generation: 101, systems: 202}
conditions:
  hardware_environment_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  available_ram_before_bytes: 5368709120
  ambient_start_power_background: 23C; battery 80 percent; not charging; allow-list active
  cold_warm_cache_state: one cold session followed by four registered warm sessions
resources:
  allocated_accelerators: [ExampleNPU-v1]
  budget_caps: {device_minutes: 60}
  observed_summary: {device_minutes: 30, peak_ram_bytes: 1717986918}
trials:
  expected: 5
  written: 5
  bundle_uri: artifacts/run-example-benchmark-001/trials.jsonl
  bundle_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
logs:
  stdout: {uri: artifacts/run-example-benchmark-001/stdout.log, sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}
  stderr: {uri: artifacts/run-example-benchmark-001/stderr.log, sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"}
  telemetry: [artifacts/run-example-benchmark-001/trace.json]
outputs: [artifact-example-raw-results-001]
deviations: [device-entered-moderate-thermal-state-in-session-5]
outcome:
  status: SUCCESS
  exit_code: 0
  failure_id: not-applicable
  stop_reason: all-registered-sessions-completed
integrity:
  input_hashes_verified: true
  output_manifest_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  record_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
review:
  completeness: passed-example
  protocol_conformance: passed-with-documented-deviation-example
```

## Field-by-field explanation

Identity and command fields make the action reproducible; seeds and conditions describe stochastic and physical
state; resources capture cost; trials and logs link evidence; deviations preserve comparability limits; outcome
records termination; integrity and review prevent an exit code from being mistaken for validated success.

## Reusable protocol

Create the record before launch, verify inputs, finalize after termination, reconcile expected and written trials,
hash outputs, and create a new record for every retry. Sensitive material belongs in access-controlled artifacts.
