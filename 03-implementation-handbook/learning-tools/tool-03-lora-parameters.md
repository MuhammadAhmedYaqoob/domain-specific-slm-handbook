# TOOL-03 — LoRA trainable-parameter calculator

## Learn before calculating

**Concept:** low-rank adaptation size. **Purpose:** derive LoRA trainable weights from actual projection dimensions and rank.

**Prerequisites:** Python 3.11 and the target module inventory from the base model. **Expected inputs:** module name, input width, output width, rank, and optional base parameter count. **Expected outputs:** per-module and total LoRA parameters with optional percentages. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0003` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** total LoRA A/B parameters for an explicit module inventory. It does not discover modules or certify a PEFT
configuration. **Units:** dimensions, rank, parameter counts, percent. **Assumptions:** each target uses matrices
$A\in\mathbb R^{r\times d_{in}}$, $B\in\mathbb R^{d_{out}\times r}$, with no trainable bias.

## Contract, equation, and edge cases

Repeat `--module NAME:DIN:DOUT:RANK`; optionally supply positive `--base-parameters`. Per target,
$N_i=r_i(d_{in,i}+d_{out,i})$. Output contains IDs, per-module rows, `lora_parameters`, optional
`percent_of_base`, and optional `trainable_percent_of_combined`. Duplicate names, nonpositive dimensions/ranks, or a
nonpositive base fail. Fused QKV must use its actual input/output dimensions; shared adapters must be entered once.

## Complete CLI listing

Intended path: `slm-implementation/tools/lora_parameters.py`; Python 3.11+, offline and read-only.

<!-- extract:LST-TOOL-0003:start -->
<a id="lst-tool-0003"></a>
**LST-TOOL-0003 — LoRA parameter CLI.**

```python
from __future__ import annotations
import argparse, json

def calculate(specs:list[str],base:int|None)->dict:
    if not specs or (base is not None and base<=0): raise ValueError("modules required; base must be positive")
    seen=set(); rows=[]; total=0
    for spec in specs:
        parts=spec.split(":")
        if len(parts)!=4: raise ValueError(f"expected NAME:DIN:DOUT:RANK: {spec}")
        name=parts[0]; din,dout,rank=map(int,parts[1:])
        if not name or name in seen or min(din,dout,rank)<=0: raise ValueError(f"invalid module: {spec}")
        count=rank*(din+dout); seen.add(name); total+=count
        rows.append({"name":name,"input_dim":din,"output_dim":dout,"rank":rank,"parameters":count})
    out={"tool_id":"TOOL-03","listing_id":"LST-TOOL-0003","modules":rows,"lora_parameters":total}
    if base is not None:
        out["percent_of_base"]=100*total/base
        out["trainable_percent_of_combined"]=100*total/(base+total)
    return out

def main()->None:
    p=argparse.ArgumentParser(); p.add_argument("--module",action="append",default=[])
    p.add_argument("--base-parameters",type=int); p.add_argument("--self-test",action="store_true")
    a=p.parse_args()
    if a.self_test:
        r=calculate(["q:4:6:2","v:4:6:2"],1000)
        assert r["lora_parameters"]==40 and r["percent_of_base"]==4
        print(json.dumps({"self_test":"passed"})); return
    print(json.dumps(calculate(a.module,a.base_parameters),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0003:end -->

## Fixtures, commands, interpretation, and limitations

Hand fixture: each $r=2,d_{in}=4,d_{out}=6$ target has 20 parameters; two targets have 40, or 4% of a 1,000-parameter
base. Run `python tools/lora_parameters.py --self-test` and then enumerate actual target shapes. Counts do not imply
quality, optimizer memory, merge support, or latency; compare to framework-reported trainable names and counts.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
