# TOOL-05 — Mobile model feasibility estimator

## Learn before calculating

**Concept:** mobile capacity budgeting. **Purpose:** distinguish installation storage from active RAM and reserve headroom.

**Prerequisites:** Python 3.11 plus artifact sizes and target-device memory observations. **Expected inputs:** storage/RAM capacities and every package, staging, weights, KV, scratch, and reserve component. **Expected outputs:** storage and RAM totals, headroom, and arithmetic pass flags. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0005` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** evaluate arithmetic storage and RAM budgets from user-supplied measurements/estimates. It makes no
compatibility, latency, energy, thermal, or performance claim. **Units:** bytes and unitless reserve fraction.
**Assumptions:** model, runtime, app, update staging, weights, KV cache, scratch, and OS reserve are non-overlapping
budget lines.

## Contract, equation, and edge cases

All byte inputs are explicit and nonnegative; capacities are positive; `--safety-fraction` is in \([0,1)\).
Installed bytes \(I=M+R+A+U\); active bytes \(X=W+K+S+O\); usable RAM \(C(1-f)\). Output schema includes component
totals, headroom, `storage_budget_pass`, `ram_budget_pass`, and `budget_pass`. A pass means only these arithmetic
budgets fit. Double-counted shared memory, load peaks, compressed downloads, lifecycle pressure, and backend support
remain outside this calculation.

## Complete CLI listing

Intended path: `slm-implementation/tools/mobile_feasibility.py`; Python 3.11+, offline.

<!-- extract:LST-TOOL-0005:start -->
<a id="lst-tool-0005"></a>
**LST-TOOL-0005 — mobile budget CLI.**

```python
from __future__ import annotations
import argparse, json

def calculate(v:list[int],fraction:float)->dict:
    (storage,model,runtime,app,update,ram,weights,kv,scratch,osreserve)=v
    if min(storage,ram)<=0 or min(model,runtime,app,update,weights,kv,scratch,osreserve)<0:
        raise ValueError("capacities positive; components nonnegative")
    if not 0<=fraction<1: raise ValueError("safety fraction must be in [0,1)")
    installed=model+runtime+app+update; active=weights+kv+scratch+osreserve
    usable=int(ram*(1-fraction)); sp=installed<=storage; rp=active<=usable
    return {"tool_id":"TOOL-05","listing_id":"LST-TOOL-0005","installed_bytes":installed,
            "storage_headroom_bytes":storage-installed,"active_bytes":active,"usable_ram_bytes":usable,
            "ram_headroom_bytes":usable-active,"storage_budget_pass":sp,"ram_budget_pass":rp,
            "budget_pass":sp and rp,"performance_claim":False}

def main()->None:
    names=("storage_capacity","model_bytes","runtime_bytes","app_bytes","update_staging_bytes",
           "ram_capacity","weights_resident_bytes","kv_cache_bytes","scratch_bytes","os_reserve_bytes")
    p=argparse.ArgumentParser()
    for n in names: p.add_argument("--"+n.replace("_","-"),type=int)
    p.add_argument("--safety-fraction",type=float); p.add_argument("--self-test",action="store_true"); a=p.parse_args()
    if a.self_test:
        r=calculate([1000,400,100,50,200,800,300,100,50,50],.25)
        assert r["installed_bytes"]==750 and r["usable_ram_bytes"]==600 and r["budget_pass"]
        print(json.dumps({"self_test":"passed"})); return
    vals=[getattr(a,n) for n in names]
    if any(x is None for x in vals) or a.safety_fraction is None: p.error("all arguments are required")
    print(json.dumps(calculate(vals,a.safety_fraction),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0005:end -->

## Fixtures, commands, interpretation, and limitations

Fixture: installed \(400+100+50+200=750\) of 1,000 bytes; usable RAM \(800(1-.25)=600\), active
\(300+100+50+50=500\), so both arithmetic gates pass. Run `python tools/mobile_feasibility.py --self-test`.
A result cannot establish load success, sustained generation, device support, battery use, or thermal behavior; those
require the exact runtime/artifact/device workload and physical measurement.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
