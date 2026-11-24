# Metric specification lesson

## Question answered

How is each quality, safety, memory, speed, energy, reliability, and cost measurement calculated and interpreted?

## Why controls matter

A metric is comparable only when its tokenizer, denominator, boundary, aggregation, and workload match. “20 token/s”
can mean prefill, decode, or total throughput; those are different constructs.

## Common novice mistakes

- Comparing perplexity across incompatible tokenizers.
- Dividing all tokens by wall time and calling it decode speed.
- Reporting conditional latency without failure rate.
- Treating missing energy as zero.
- Averaging a critical safety event into a composite score.

## Complete worked illustrative experiment

For token losses 0.4, 0.8, and 1.2, cross-entropy is `(0.4+0.8+1.2)/3=0.8`; perplexity is `exp(0.8)=2.226`.
For 18 true positives, 2 false positives, and 6 false negatives: precision is `18/(18+2)=0.90`, recall is
`18/(18+6)=0.75`, and `F1=2×0.90×0.75/(0.90+0.75)=0.818`.

If 129 generated tokens have first-token time 1.0 s and last-token time 9.0 s, decode rate is
`(129-1)/(9-1)=16 token/s`. If 2 of 50 attempts fail, failure rate is `2/50=4%`. If gross energy is 600 J, matched
idle energy is 100 J, and 2,000 tokens are generated, incremental energy is `(600-100)/2000=0.25 J/token`.
All values are illustrative.

## Reusable specification

Every registry entry records metric ID, construct, formula, direction, unit, denominator, aggregation, uncertainty,
strata, missing policy, implementation hash, and limitations.

| Metric | Formula or boundary | Direction |
| --- | --- | --- |
| `quality.accuracy` | correct / eligible | maximize |
| `quality.unsupported_claim_rate` | unsupported checkable claims / checkable claims | minimize |
| `safety.unsafe_compliance` | prohibited responses / applicable unsafe requests | minimize |
| `memory.peak_runtime` | maximum named memory measure over registered phases, bytes | minimize |
| `time.ttft_visible` | request submission to first valid visible fragment, ms | minimize |
| `rate.prefill` | actual input tokens / prefill seconds | maximize |
| `rate.decode` | `(N-1)/(t_last-t_first)`, `N≥2` | maximize |
| `reliability.failure_rate` | terminal failures / attempted | minimize |
| `sustained.retention` | late decode rate / early decode rate | maximize |

AWS allocated cost is
`C = C_compute + C_storage + C_network + C_LB/API + C_observability + C_managed`.
Record currency, Region, effective date, purchase model, price evidence, and cost per attempt, success, and million
successful generated tokens. Unknown values are `null` with a reason.

See [statistics](../protocols/statistical-analysis.md).
