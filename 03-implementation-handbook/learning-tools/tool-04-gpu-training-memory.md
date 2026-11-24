# TOOL-04 — GPU training-memory estimator

## Learn before calculating

**Concept:** training-memory accounting. **Purpose:** separate resident weights, trainable state, activations, temporary buffers, and reserve.

**Prerequisites:** Python 3.11 plus framework-reported local parameter counts and measured activation/temp values. **Expected inputs:** local counts, bytes-per-parameter rates, measured byte components, and optional device capacity. **Expected outputs:** component ledger, total GiB, arithmetic headroom, and fit flag. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0004` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** add an explicit per-rank training-memory ledger; it does not predict allocator peaks. **Units:** parameter
counts, bytes/parameter, bytes, GiB. **Assumptions:** user supplies local parameter counts and empirically justified
activation, temporary, and reserve bytes; no hidden optimizer or sharding defaults.

## Contract, equation, and edge cases

Required nonnegative inputs describe local `--resident-parameters`, `--trainable-parameters`, weight/gradient/optimizer
bytes per parameter, activation, temporary, and reserve bytes. At least one resident parameter is required.
$M=N_rw+N_t(g+o)+A+T+R$. Output reports each component, total bytes/GiB, and optional headroom against
`--device-bytes`. Negative values and nonpositive byte rates fail; a device budget below total is reported, not called
an OOM prediction. Shared pages, fragmentation, communication buffers, offload, recomputation, and allocator behavior
must be represented explicitly.

## Complete CLI listing

Intended path: `slm-implementation/tools/gpu_training_memory.py`; Python 3.11+, no GPU access.

<!-- extract:LST-TOOL-0004:start -->
<a id="lst-tool-0004"></a>
**LST-TOOL-0004 — GPU memory ledger CLI.**

```python
from __future__ import annotations
import argparse, json

def calculate(nr:int,nt:int,w:float,g:float,o:float,a:int,t:int,r:int,device:int|None)->dict:
    if nr<=0 or nt<0 or min(w,g,o)<=0 or min(a,t,r)<0 or (device is not None and device<=0):
        raise ValueError("invalid count, rate, component, or device budget")
    parts={"weights_bytes":nr*w,"gradients_bytes":nt*g,"optimizer_bytes":nt*o,
           "activations_bytes":a,"temporary_bytes":t,"reserve_bytes":r}
    total=sum(parts.values())
    out={"tool_id":"TOOL-04","listing_id":"LST-TOOL-0004",**{k:int(v) for k,v in parts.items()},
         "total_bytes":int(total),"total_gib":total/(2**30)}
    if device is not None:
        out["device_bytes"]=device; out["headroom_bytes"]=int(device-total)
        out["arithmetic_fits"]=total<=device
    return out

def main()->None:
    p=argparse.ArgumentParser()
    for n in ("resident_parameters","trainable_parameters","activation_bytes","temporary_bytes","reserve_bytes","device_bytes"):
        p.add_argument("--"+n.replace("_","-"),type=int)
    for n in ("weight_bytes_per_parameter","gradient_bytes_per_parameter","optimizer_bytes_per_parameter"):
        p.add_argument("--"+n.replace("_","-"),type=float)
    p.add_argument("--self-test",action="store_true"); x=p.parse_args()
    if x.self_test:
        z=calculate(100,20,2,2,8,100,50,10,1000)
        assert z["total_bytes"]==560 and z["headroom_bytes"]==440
        print(json.dumps({"self_test":"passed"})); return
    vals=(x.resident_parameters,x.trainable_parameters,x.weight_bytes_per_parameter,
          x.gradient_bytes_per_parameter,x.optimizer_bytes_per_parameter,x.activation_bytes,
          x.temporary_bytes,x.reserve_bytes)
    if any(v is None for v in vals): p.error("all component arguments are required")
    print(json.dumps(calculate(*vals,x.device_bytes),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0004:end -->

## Fixtures, commands, interpretation, and limitations

Fixture: $100·2+20·2+20·8+100+50+10=560$ bytes; against 1,000 bytes, arithmetic headroom is 440 bytes. Run
`python tools/gpu_training_memory.py --self-test`; for real work derive local counts after sharding and measure peak
allocated/reserved memory. `arithmetic_fits` is not a training-success, speed, or hardware claim.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
