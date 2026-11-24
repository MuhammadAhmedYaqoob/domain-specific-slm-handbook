# AWS inference benchmark lesson

## Question answered

Does an exact artifact–server–instance tuple meet quality, latency, throughput, reliability, scaling, and allocated-cost gates at a declared offered load?

This is an educational protocol. All numbers below are explicitly illustrative; they are not evidence that this
project ran an experiment.

## Why controls matter

Pin Region, purchase model, image digest, instance, replicas, batching, scheduler, context, load generator, arrival process, warm-up, and observability. Separate queue, service, model, and client timing.

## Common novice mistakes

- Dividing theoretical peak throughput by hourly price.
- Using a closed-loop client for an offered-load slo.
- Omitting failed requests from cost.
- Quoting price without date and region.
- Mixing cold-start and steady-state latency.

## Complete worked illustrative experiment

An illustrative one-hour block costs USD 1.20 compute, USD 0.08
storage/network, and USD 0.12 observability, so allocated cost is USD 1.40.
Of 10,000 attempted requests, 9,800 succeed and produce 1,960,000 tokens.
Cost per success is `1.40/9800=0.0001429` USD; cost per million successful
tokens is `1.40/1.96=0.7143` USD. If p95 TTFT is 900 ms at 8 requests/s but
1,600 ms at 10 requests/s against a 1,000 ms SLO, qualified capacity is no
more than the supported 8-request/s point.

The example decision always follows the preregistered hard gates before optimization. A favorable mean never cancels
a critical safety, rights, compatibility, or reliability failure.

## Reusable protocol

### Inputs and controls

Register immutable artifact, tokenizer, template, dataset, evaluator, code, environment, workload, and seed hashes.
Name the independent variable, controlled variables, blocks, exclusions, missing-data policy, MID, and stopping rules
before confirmation.

### Procedure

1. Provision hashed infrastructure in an isolated tagged environment.
2. Verify identity, authentication, placement, and semantic parity.
3. Retain cold and warm evidence separately.
4. Start synchronized client, service, host, and accelerator telemetry.
5. Run balanced open-loop load blocks through saturation.
6. Exercise scaling, interruption, and rollback.
7. Reconcile usage with dated price or billing evidence.
8. Destroy resources and archive teardown evidence.

### Metrics and calculations

Report quality, provisioning/load time, queue and TTFT tails, prefill/decode, requests and tokens per second, utilization and memory, HTTP/model failures, availability, scaling times, and cost per attempt, success, and million successful tokens. For a higher-is-better metric, use `Δ = candidate − baseline`; percentage improvement is
`100 × (candidate − baseline) / baseline`. Failure rate is `failures / attempted`, never failures/successes.

### Repetition and statistics

Use one smoke attempt only to find defects. Use preregistered independent seeds for training and independent
device/host sessions for systems claims. Pair quality items by source root. Report effect sizes and confidence
intervals; use multiplicity control for a confirmatory family. An interval that crosses the decision boundary is
`INCONCLUSIVE`, not evidence of no effect.

### Failure and decision rules

Retain OOM, crash, timeout, invalid output, unsupported, and budget-stopped attempts. Use `null` with a reason for an
unavailable measurement; zero means measured zero. Qualify only an exact target tuple that passes every hard gate and
the registered practical-effect rule.

## Further reading

[Metric specification](../specifications/metric-specification.md) · [Result schema](../specifications/result-schema-specification.md)
