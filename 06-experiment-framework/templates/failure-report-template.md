# Failure report lesson and template

## Question answered

What failed, what evidence supports the observation, how does it affect the claim, and what retry is permitted?

## Why controls matter

Separating facts from causal hypotheses prevents an attractive explanation from
becoming recorded truth.

## Common novice mistakes

Never relabel an OOM as infrastructure noise, delete the attempt, or expose
secrets in a public report.

## Fully populated illustrative example

```yaml
schema: slm-failure/v1
failure_id: failure-example-oom-001
experiment_id: exp-example-int4-001
run_id: run-example-benchmark-002
trial_ids: [trial-example-session-006]
plan_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
variant_id: int4-child
stage: benchmark
classification: resource
severity: cell-blocking
time: {first_observed: 2026-11-24T07:10:00Z, last_observed: 2026-11-24T07:10:03Z}
expected:
  behavior: complete a 4096-token-context request within the 2.5 GB peak-RAM gate
  gate_or_limit: gates.compatibility_memory_reliability
observed:
  behavior: operating system terminated the process after peak RAM exceeded the device limit
  outcome: OOM
  phase: prefill
identity:
  input_artifacts: [{id: artifact-example-quantized-001, sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}]
  config_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  environment_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  hardware_condition_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
reproduction:
  attempts: 1
  reproduced: false
  minimal_steps: [load signed package, submit registered 4096-token workload]
  retry_allowed_by_plan: true
evidence:
  logs: [{uri: artifacts/failure-example-oom-001/os-exit.log, sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}]
  traces: [artifacts/failure-example-oom-001/memory-trace.json]
  output_artifacts: [artifact-example-partial-events-001]
  screenshots_or_diagnostics: [os-exit-reason-memory-pressure]
impact:
  affected_cells: [int4-child-4096-context]
  denominator_treatment: retain as one failed attempted session
  claim_gate: memory and reliability gates cannot pass from this attempt
  data_integrity: completed earlier sessions remain valid
  safety_or_privacy: no exposure observed
analysis:
  suspected_causes:
    - {hypothesis: KV-cache growth exceeded available RAM, evidence_for: [monotonic-memory-trace], evidence_against: [no-independent-reproduction]}
  root_cause_status: suspected
remediation:
  immediate_containment: stop the affected benchmark branch
  proposed_change: run the one plan-authorized exact retry after restoring the start state
  change_requires_new_plan: false
  retest_id: run-example-benchmark-003
resolution:
  status: contained
  resolved_at: 2026-11-24T07:20:00Z
  superseding_evidence_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
unresolved_risk: repeat OOM may disqualify the 4096-token target
reviews: {technical: reviewed-example, safety_security_privacy: not-applicable}
record_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
```

## Field-by-field explanation

Expected versus observed defines the failure; identity and evidence anchor it; reproduction states what was attempted;
impact controls denominators and claims; analysis labels hypotheses; remediation is bounded by the plan; resolution
never erases history.

## Reusable protocol

Create one immutable report per distinct observed failure, link every affected trial, retain original evidence, apply
the registered denominator policy, and use a new run ID for any retry.
