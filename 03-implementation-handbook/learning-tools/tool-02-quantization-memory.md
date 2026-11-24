# TOOL-02 — Quantization-memory calculator

## Learn before calculating

**Concept:** quantized encoding overhead. **Purpose:** show why a four-bit code stream occupies more than exactly four bits per parameter.

**Prerequisites:** Python 3.11 and the quantizer group/metadata contract. **Expected inputs:** parameter count, weight bits, group size, scale bytes, zero-point bytes, and fixed metadata. **Expected outputs:** group count, code bytes, metadata bytes, effective bits per parameter, and total MiB. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0002` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** estimate encoded weight storage including group scales/zero points and fixed metadata. It does not model
runtime dequantization or activations. **Units:** parameters, bits, bytes, MiB. **Assumptions:** every complete or partial
group stores one scale and optional zero point; parameter codes are densely bit-packed.

## Contract, equation, and edge cases

Required inputs are positive `--parameters`, `--weight-bits`, `--group-size`, `--scale-bytes`; `--zero-bytes` and
`--metadata-bytes` are nonnegative. Output keys are `tool_id`, `listing_id`, `groups`, `code_bytes`,
`quant_metadata_bytes`, `fixed_metadata_bytes`, `total_bytes`, `effective_bits_per_parameter`, and `total_mib`.
$G=\lceil N/g\rceil$, $B=\lceil Nb/8\rceil+G(s+z)+m$, $b_\text{effective}=8B/N$. Invalid values fail. This
deliberately counts a partial final group and does not infer alignment, high-precision tensors, or compression.

## Complete CLI listing

Intended path: `slm-implementation/tools/quantization_memory.py`; Python 3.11+ standard library; read-only, offline.

<!-- extract:LST-TOOL-0002:start -->
<a id="lst-tool-0002"></a>
**LST-TOOL-0002 — quantization-memory CLI.**

```python
from __future__ import annotations
import argparse, json, math

def calculate(n:int,bits:float,group:int,scale:int,zero:int,meta:int)->dict:
    if n<=0 or bits<=0 or group<=0 or min(scale,zero,meta)<0:
        raise ValueError("positive n/bits/group and nonnegative byte fields required")
    groups=math.ceil(n/group); code=math.ceil(n*bits/8)
    qm=groups*(scale+zero); total=code+qm+meta
    return {"tool_id":"TOOL-02","listing_id":"LST-TOOL-0002","groups":groups,
            "code_bytes":code,"quant_metadata_bytes":qm,"fixed_metadata_bytes":meta,
            "total_bytes":total,"effective_bits_per_parameter":8*total/n,
            "total_mib":total/(2**20)}

def main()->None:
    p=argparse.ArgumentParser()
    for name in ("parameters","group_size","scale_bytes","zero_bytes","metadata_bytes"):
        p.add_argument("--"+name.replace("_","-"),type=int)
    p.add_argument("--weight-bits",type=float); p.add_argument("--self-test",action="store_true")
    a=p.parse_args()
    if a.self_test:
        r=calculate(10,4,4,2,1,5)
        assert (r["groups"],r["code_bytes"],r["quant_metadata_bytes"],r["total_bytes"])==(3,5,9,19)
        print(json.dumps({"self_test":"passed"})); return
    required=(a.parameters,a.weight_bits,a.group_size,a.scale_bytes,a.zero_bytes,a.metadata_bytes)
    if any(x is None for x in required): p.error("all numeric arguments are required")
    print(json.dumps(calculate(*required),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0002:end -->

## Fixtures, commands, interpretation, and limitations

Hand fixture: 10 four-bit codes use 5 bytes; group size 4 makes 3 groups; $3(2+1)=9$ metadata bytes; fixed 5 gives
19 bytes and 15.2 effective bits/parameter. Run `python tools/quantization_memory.py --self-test` or supply all six
numeric arguments. A result estimates an encoding contract only; verify actual artifact bytes, tensor exceptions,
alignment, and runtime-expanded memory.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
