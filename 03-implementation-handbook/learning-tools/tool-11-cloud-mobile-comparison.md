# TOOL-11 — Cloud-versus-mobile inference comparison

## Learn before calculating

**Concept:** deployment-placement comparison. **Purpose:** compare cloud and mobile observations under one workload without declaring a universal winner.

**Prerequisites:** Python 3.11 and paired measured records from a common protocol. **Expected inputs:** shared workload identity plus success, TTFT, decode, energy, and cost fields. **Expected outputs:** latency/throughput deltas, failure rates, and available energy/cost deltas. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0011` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** normalize and compare two provenance-bearing measured inference records under one workload. It does not
predict performance, rank deployments, or recommend a winner. **Units:** milliseconds, generated tokens/second,
joules/request, currency/request; unavailable measures are JSON `null`, never fabricated zero.

## Contract, equation, and edge cases

Input JSON requires shared `experiment_id`, `workload_sha256`, `quality_protocol_sha256`, `measured_utc`, and `currency`,
plus `cloud` and `mobile`. Each side requires nonblank artifact/runtime/hardware IDs, positive finite `attempts`,
`successes` in \([0,attempts]\), positive finite `ttft_ms` and `decode_tokens_per_second`; `energy_j_per_request` and
`cost_per_request` are null or nonnegative finite. Output gives mobile-minus-cloud latency/throughput deltas and failure
rates; energy/cost deltas are emitted only when both are measured. It makes no “faster,” “better,” or feasibility claim.

## Complete CLI listing

Intended path: `slm-implementation/tools/cloud_mobile_comparison.py`; Python 3.11+, reads one local JSON file.

<!-- extract:LST-TOOL-0011:start -->
<a id="lst-tool-0011"></a>
**LST-TOOL-0011 — measured placement comparison CLI.**

```python
from __future__ import annotations
import argparse, json, math
from pathlib import Path

def finite(v:object)->bool: return isinstance(v,(int,float)) and math.isfinite(v)
def calculate(d:dict)->dict:
    shared=("experiment_id","workload_sha256","quality_protocol_sha256","measured_utc","currency")
    if any(not isinstance(d.get(k),str) or not d[k].strip() for k in shared):
        raise ValueError("complete shared provenance required")
    rows={}
    for side in ("cloud","mobile"):
        x=d.get(side,{})
        if any(not isinstance(x.get(k),str) or not x[k].strip() for k in ("artifact_id","runtime_id","hardware_id")):
            raise ValueError(f"incomplete {side} identity")
        if not all(finite(x.get(k)) for k in ("attempts","successes","ttft_ms","decode_tokens_per_second")):
            raise ValueError(f"invalid {side} metric")
        if x["attempts"]<=0 or not 0<=x["successes"]<=x["attempts"] or x["ttft_ms"]<=0 or x["decode_tokens_per_second"]<=0:
            raise ValueError(f"out-of-range {side} metric")
        for k in ("energy_j_per_request","cost_per_request"):
            if x.get(k) is not None and (not finite(x[k]) or x[k]<0): raise ValueError(f"invalid {side} {k}")
        rows[side]=x
    c,m=rows["cloud"],rows["mobile"]
    out={"tool_id":"TOOL-11","listing_id":"LST-TOOL-0011",
         "experiment_id":d["experiment_id"],"workload_sha256":d["workload_sha256"],
         "ttft_delta_mobile_minus_cloud_ms":m["ttft_ms"]-c["ttft_ms"],
         "decode_delta_mobile_minus_cloud_tokens_per_second":m["decode_tokens_per_second"]-c["decode_tokens_per_second"],
         "cloud_failure_rate":1-c["successes"]/c["attempts"],
         "mobile_failure_rate":1-m["successes"]/m["attempts"],"performance_claim":False}
    for key,label in (("energy_j_per_request","energy_delta_mobile_minus_cloud_j_per_request"),
                      ("cost_per_request","cost_delta_mobile_minus_cloud_per_request")):
        out[label]=None if c.get(key) is None or m.get(key) is None else m[key]-c[key]
    return out

def main()->None:
    p=argparse.ArgumentParser(); p.add_argument("--input",type=Path); a=p.parse_args()
    if a.input is None: p.error("--input is required")
    print(json.dumps(calculate(json.loads(a.input.read_text(encoding="utf-8"))),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0011:end -->

## Fixtures, commands, interpretation, and limitations

Hand fixture: cloud/mobile TTFT 100/140 ms gives +40 ms; decode 20/8 tok/s gives −12 tok/s; 9/10 and 8/10 successes
give failure rates 0.1 and 0.2. These are arithmetic fixtures, not platform results. Run
`python tools/cloud_mobile_comparison.py --input measured-comparison.json`. Comparability still requires identical
prompts, tokenizer-visible token counts, output policy, quality/safety gates, trial state, and uncertainty. Cloud
network scope and mobile thermal duration must be documented; deltas alone do not select placement.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
