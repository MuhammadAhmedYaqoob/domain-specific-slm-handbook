# TOOL-06 — Inference-latency calculator

## Learn before calculating

**Concept:** latency decomposition. **Purpose:** separate queue-visible TTFT, prefill throughput, decode throughput, and end-to-end time.

**Prerequisites:** Python 3.11 and timestamps from one monotonic clock. **Expected inputs:** seven phase timestamps and actual tokenizer-visible input/generated counts. **Expected outputs:** milliseconds and tokens-per-second metrics with preserved counts. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0006` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** derive latency metrics from measured monotonic nanosecond timestamps. It does not estimate unmeasured
hardware performance. **Units:** nanoseconds, milliseconds, tokens, tokens/second. **Assumptions:** timestamps share one
monotonic clock and boundaries follow CH-31.

## Contract, equation, and edge cases

Inputs are request, prefill start/end, first model token, first visible fragment, last token, and finish timestamps plus
actual input/generated token counts. Ordering must be nondecreasing; input tokens positive; generated tokens at least
two for decode rate. \(TTFT=(t_v-t_0)/10^6\), prefill rate \(=N_p10^9/(t_{pe}-t_{ps})\), decode rate
\(=(N_g-1)10^9/(t_{last}-t_1)\), E2E \(=(t_f-t_0)/10^6\). Zero-duration intervals fail.

## Complete CLI listing

Intended path: `slm-implementation/tools/inference_latency.py`; Python 3.11+, no network or files.

<!-- extract:LST-TOOL-0006:start -->
<a id="lst-tool-0006"></a>
**LST-TOOL-0006 — measured latency CLI.**

```python
from __future__ import annotations
import argparse, json

def calculate(t:list[int],np:int,ng:int)->dict:
    t0,ps,pe,first,visible,last,finish=t
    if any(x<0 for x in t) or t!=sorted(t) or np<=0 or ng<2:
        raise ValueError("ordered nonnegative timestamps, input>0, generated>=2 required")
    if pe==ps or last==first: raise ValueError("prefill/decode duration must be positive")
    return {"tool_id":"TOOL-06","listing_id":"LST-TOOL-0006",
            "first_model_token_ms":(first-t0)/1e6,"ttft_ms":(visible-t0)/1e6,
            "prefill_tokens_per_second":np*1e9/(pe-ps),
            "decode_tokens_per_second":(ng-1)*1e9/(last-first),
            "end_to_end_ms":(finish-t0)/1e6,"input_tokens":np,"generated_tokens":ng}

def main()->None:
    names=("request_ns","prefill_start_ns","prefill_end_ns","first_token_ns",
           "first_visible_ns","last_token_ns","finish_ns")
    p=argparse.ArgumentParser()
    for n in names: p.add_argument("--"+n.replace("_","-"),type=int)
    p.add_argument("--input-tokens",type=int); p.add_argument("--generated-tokens",type=int)
    p.add_argument("--self-test",action="store_true"); a=p.parse_args()
    if a.self_test:
        r=calculate([0,10_000_000,110_000_000,120_000_000,130_000_000,320_000_000,350_000_000],50,5)
        assert r["ttft_ms"]==130 and r["prefill_tokens_per_second"]==500 and r["decode_tokens_per_second"]==20
        print(json.dumps({"self_test":"passed"})); return
    vals=[getattr(a,n) for n in names]
    if any(x is None for x in vals+[a.input_tokens,a.generated_tokens]): p.error("all arguments are required")
    print(json.dumps(calculate(vals,a.input_tokens,a.generated_tokens),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0006:end -->

## Fixtures, commands, interpretation, and limitations

Fixture: 50 tokens/0.1 s = 500 input tok/s; four post-first decode intervals/0.2 s = 20 generated tok/s; TTFT 130 ms;
E2E 350 ms. Run `python tools/inference_latency.py --self-test`. Report queue/network/rendering scope, tokenizer,
workload, repetitions, failures, uncertainty, and cold/warm state alongside results; one trace is not a performance
claim.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
