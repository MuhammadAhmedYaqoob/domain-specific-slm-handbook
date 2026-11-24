# Experiment plan lesson and template

## Question answered

What will be tested, held constant, measured, and decided before outcomes are visible?

## Why controls matter

A frozen plan prevents outcome-driven changes.

## Common novice mistakes

Common mistakes are vague hypotheses, mutable model aliases, test-set selection,
unspecified failure handling, and a sample size chosen after looking at variance.

## Fully populated illustrative example

```yaml
schema: slm-experiment-plan/v1
status: ILLUSTRATIVE
experiment:
  id: exp-example-int4-001
  title: Example INT4 non-inferiority on a midrange Android profile
  protocol: ../protocols/quantization-benchmark.md
  intended_use: educational offline automotive explanations
  prohibited_uses: [autonomous repair control, safety-system bypass]
hypothesis:
  null: INT4 quality loss is at least 0.02 or no deployment MID is achieved
  alternative: INT4 quality loss is below 0.02 and peak RAM improves by at least 0.5 GB
  estimand: INT4 minus FP16 paired domain score
  primary_metric: quality.domain_rubric
  test: noninferiority
  mid_or_margin: {value: 0.02, unit: score, rationale: approved educational tolerance}
variants:
  baseline: {id: fp16-parent, artifact_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"}
  candidates: [{id: int4-child, parent_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa", factors: {weight_type: int4, group_size: 128}}]
variables:
  independent: [weight representation]
  controlled: [model parent, tokenizer, prompts, decoding, runtime, device state]
  blocks: [device-day-session]
snapshots:
  validation: {id: auto-validation-v1, manifest_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}
  sealed_tests: [{id: auto-sealed-v1, manifest_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa", access: sealed}]
  evaluator: {rubric_metric_registry_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}
  code_environment: {commit: a1b2c3d4, environment_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"}
hardware:
  target_environment_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
workload:
  manifest_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  decoding: {strategy: greedy, max_new_tokens: 128, stop_policy_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}
repetitions:
  generation_samples: 1
  systems_pilot: 5
  systems_confirmation: 20
statistics:
  confidence: 0.95
  analysis_unit: source-root-and-device-session
  pairing_key: item-root-id
  method: root-clustered-paired-bootstrap-and-device-block-bootstrap
  resampling: {unit: root-or-session, replicates: 10000, seed: 1729}
  multiplicity: single-primary-contrast
  missing_failure_policy: retain all failures; exclude only registered instrumentation faults
  analysis_code_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
gates:
  identity_rights: all hashes and rights records verified
  quality_retention_safety: quality LCB above -0.02 and zero critical failures
  compatibility_memory_reliability: peak RAM below 2.5 GB and failure UCB below 5 percent
  performance_energy_cost: at least one registered deployment MID
budget:
  device_hours: 12
  conversion_cpu_hours: 4
outputs:
  raw_root: artifacts/exp-example-int4-001
  required_records: [run, trial, failure, result, review]
approvals:
  technical: approved-example
  statistical: approved-example
  domain_safety: approved-example
  rights_privacy: approved-example
```

## Field-by-field explanation

`experiment` bounds the claim; `hypothesis` defines what can falsify it; `variants` fixes identities; `variables`
separates the changed factor from controls and blocks; `snapshots` prevents data and evaluator drift; `hardware` and
`workload` define the target; `repetitions` and `statistics` define uncertainty; `gates` prevent compensation;
`budget` limits exposure; `outputs` requires evidence; `approvals` records review.

## Reusable protocol

Copy the schema shape, replace every illustrative value with the approved experiment's concrete value, validate all
hashes and links, obtain reviews, then hash the canonical plan. Remove inapplicable fields by recording an explicit
`not-applicable` reason rather than leaving them blank.
