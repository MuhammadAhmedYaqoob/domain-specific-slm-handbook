# Benchmark table lesson and template

## Question answered

How can comparable quality and deployment evidence be summarized without hiding uncertainty, failures, or target
conditions?

## Why controls matter

One table must describe one target and workload contract.

## Common novice mistakes

Do not mix tokenizers without qualification, omit failed attempts, or rank
conditional values as universal results.

## Fully populated illustrative example

Target: ExamplePhone X1, Android example build, runtime 1.4, greedy decoding, 512 input tokens, 128 output tokens,
battery operation at 23 °C. All values are illustrative.

| Variant | Domain score [95% CI] | Critical failures | Peak RAM GB | TTFT p95 ms | Decode token/s | Attempts: success/OOM |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| FP16 parent | 0.750 [0.710, 0.790] | 0 | 3.0 | 540 | 18.0 | 20/0 |
| INT4 child | 0.742 [0.700, 0.784] | 0 | 1.6 | 500 | 17.5 | 19/1 |

The child quality point delta is `0.742-0.750=-0.008`; RAM reduction is `(3.0-1.6)/3.0=46.7%`. The OOM remains visible,
so the child is not automatically preferred.

## Field-by-field explanation

`Variant` resolves to an artifact hash in the machine record; score includes uncertainty; critical failures remain
separate; memory names the peak boundary; TTFT states a tail; decode uses generated-token timing; attempts expose
reliability.

## Reusable protocol

Begin with protocol, plan/result hashes, exact target, workload, power/thermal state, metric registry, interval method,
and evidence label. Add quality/safety, storage/memory/reliability, interaction/sustained performance, energy,
conversion, training resources, and dated cost sections. Every unavailable value must have a machine-readable `null`
reason. Link raw trials, environments, failures, analysis, and lineage.
