# Experiment result lesson and template

## Question answered

What does the reviewed evidence establish for one approved plan and exact target tuple?

## Why controls matter

The report must reconcile all attempts and copy the registered decision rule.

## Common novice mistakes

Do not write winner language from a mean alone, omit deviations, or release a
package different from the evaluated artifact.

## Fully populated illustrative example

```yaml
schema: experiment-result/v1
record_id: result-example-int4-001
experiment_id: exp-example-int4-001
plan_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
analysis_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
status: INCONCLUSIVE
evidence_label: ILLUSTRATIVE
hypothesis:
  estimand: INT4 minus FP16 paired domain score
  primary_metric: quality.domain_rubric
  noninferiority_margin: 0.02
comparison: {baseline: fp16-parent, candidate: int4-child, pairing_key: item-root-id}
coverage:
  fp16-parent: {SUCCESS: 20, OOM: 0, TIMEOUT: 0}
  int4-child: {SUCCESS: 19, OOM: 1, TIMEOUT: 0}
primary_result:
  baseline: 0.750
  candidate: 0.742
  effect: -0.008
  confidence_interval: [-0.026, 0.010]
  decision_rule: lower-bound-must-exceed-minus-0.020
  statistical_decision: INCONCLUSIVE
hard_gates:
  identity_rights: passed-example
  critical_safety: passed-example-zero-events-observed
  peak_ram: passed-example-1.6GB-below-2.5GB
  reliability: failed-example-one-OOM-and-upper-bound-above-gate
failures: [failure-example-oom-001]
deviations: [device-entered-moderate-thermal-state-in-session-5]
artifacts: [artifact-example-quantized-001, artifact-example-raw-results-001]
decision: INCONCLUSIVE
bounded_interpretation: the illustrative child reduces peak RAM, but quality and reliability evidence do not qualify it
reviews: {technical: reviewed-example, statistical: reviewed-example, domain_safety: reviewed-example}
created_at: 2026-11-24T08:00:00Z
supersedes_sha256: not-applicable-first-version
```

## Field-by-field explanation

Identity fields bind the plan and analysis; hypothesis and comparison prevent claim drift; coverage reconciles
attempts; primary result shows effect and interval; hard gates remain non-compensatory; failures and deviations explain
limitations; artifacts bind evidence; decision and interpretation are target-bounded; reviews establish status.

## Reusable protocol

Create a result only after an attempt exists. Reconcile run/trial counts, report primary and secondary metrics with
units and intervals, include resource/cost ledgers, confirm release-artifact identity, link every failure, and state
what the evidence does and does not establish. Corrections supersede rather than overwrite.
