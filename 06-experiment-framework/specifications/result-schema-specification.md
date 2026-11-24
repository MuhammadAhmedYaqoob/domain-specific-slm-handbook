# Result schema lesson

## Question answered

What machine-readable structure prevents ambiguous, incomplete, or silently edited experiment claims?

## Why controls matter

Typed records force units, denominators, failures, hashes, and missing reasons to travel with a number. This prevents
an attractive aggregate from becoming detached from its attempts.

## Common novice mistakes

- Using zero for missing.
- Omitting failed attempts from coverage.
- Embedding units inside numeric strings.
- Mutating a reviewed record in place.

## Complete worked illustrative experiment

Eight attempts contain six successes, one OOM, and one timeout. Coverage is therefore 8 attempted and 6 successful;
success-conditioned latency has `n=6`, while failure rate is `(1+1)/8=25%`. A latency aggregate with `n=8` would be
invalid unless failures had a preregistered numeric penalty.

## Fully populated illustrative JSON

```json
{
  "schema": "experiment-result/v1",
  "record_id": "result-example-001",
  "experiment_id": "exp-example-quant-001",
  "plan_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "analysis_sha256": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
  "status": "INCONCLUSIVE",
  "evidence_label": "ILLUSTRATIVE",
  "run_ids": ["run-example-001", "run-example-002"],
  "coverage": {"SUCCESS": 1, "OOM": 1},
  "aggregates": [{
    "metric_id": "time.ttft_visible",
    "value": 520.0,
    "unit": "ms",
    "direction": "minimize",
    "n": 1,
    "analysis_unit": "device_session",
    "interval": {"level": 0.95, "method": "not-estimable-single-session", "lower": null, "upper": null},
    "missing_reason": "interval unavailable because one successful illustrative session is insufficient"
  }],
  "failures": ["failure-example-oom-001"],
  "deviations": [],
  "decision": "INCONCLUSIVE"
}
```

The IDs and hashes establish identity; coverage retains failure states; each aggregate includes unit, direction,
analysis unit, interval, and missing explanation; decision language is bounded.

## Reusable specification

Use immutable result, run, and trial records in UTF-8 with UTC timestamps and explicit units. Reject duplicate IDs,
malformed hashes, null without reason, unknown enums, coverage mismatch, orphan links, execution claims without raw
bundles, and qualification with a failed gate. Corrections create a superseding record. Conditional performance must
appear beside failure rate, and Pareto fields remain absent until hard-gate qualification.
