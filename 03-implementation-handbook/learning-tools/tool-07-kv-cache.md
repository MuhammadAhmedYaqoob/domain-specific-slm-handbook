# TOOL-07 — KV-cache memory estimator

## Learn before calculating

**Concept:** attention-state growth. **Purpose:** calculate how architecture, context, batch, and precision determine dense KV payload.

**Prerequisites:** Python 3.11 and verified layer/head/cache configuration. **Expected inputs:** layers, query/KV heads, head dimension, tokens, batch, and cache element bytes. **Expected outputs:** attention mode, grouping ratio, bytes per token, and total cache MiB. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0007` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** estimate dense K/V payload for MHA, GQA, or MQA from explicit architecture fields. It excludes allocator
and runtime overhead. **Units:** layers, heads, dimensions, tokens, batch, bytes/element, bytes, MiB. **Assumptions:** K
and V each store one element per layer, batch item, token, KV head, and head dimension.

## Contract, equation, and edge cases

Required positive values: `--layers`, `--query-heads`, `--kv-heads`, `--head-dim`, `--tokens`, `--batch`,
`--bytes-per-element`; KV heads must not exceed query heads and query heads must divide evenly by KV heads.
\(B=2LBSH_{kv}D_hE\). Output identifies attention mode (`MHA`, `GQA`, `MQA`), grouping ratio, bytes, MiB, and bytes per
added token. Sliding windows, cache quantization metadata, alignment, cross-attention, prefix sharing, and temporary
attention workspace are not included.

## Complete CLI listing

Intended path: `slm-implementation/tools/kv_cache.py`; Python 3.11+, offline.

<!-- extract:LST-TOOL-0007:start -->
<a id="lst-tool-0007"></a>
**LST-TOOL-0007 — KV-cache CLI.**

```python
from __future__ import annotations
import argparse, json

def calculate(l:int,hq:int,hkv:int,d:int,s:int,b:int,e:float)->dict:
    if min(l,hq,hkv,d,s,b)<=0 or e<=0 or hkv>hq or hq%hkv:
        raise ValueError("positive values and evenly grouped kv_heads <= query_heads required")
    per=2*l*b*hkv*d*e; total=per*s
    mode="MHA" if hkv==hq else ("MQA" if hkv==1 else "GQA")
    return {"tool_id":"TOOL-07","listing_id":"LST-TOOL-0007","attention_mode":mode,
            "queries_per_kv_head":hq//hkv,"bytes_per_added_token":int(per),
            "total_bytes":int(total),"total_mib":total/(2**20)}

def main()->None:
    names=("layers","query_heads","kv_heads","head_dim","tokens","batch")
    p=argparse.ArgumentParser()
    for n in names: p.add_argument("--"+n.replace("_","-"),type=int)
    p.add_argument("--bytes-per-element",type=float); p.add_argument("--self-test",action="store_true"); a=p.parse_args()
    if a.self_test:
        r=calculate(2,4,2,8,10,1,2)
        assert r["attention_mode"]=="GQA" and r["bytes_per_added_token"]==128 and r["total_bytes"]==1280
        print(json.dumps({"self_test":"passed"})); return
    vals=[getattr(a,n) for n in names]
    if any(x is None for x in vals+[a.bytes_per_element]): p.error("all arguments are required")
    print(json.dumps(calculate(*vals,a.bytes_per_element),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0007:end -->

## Fixtures, commands, interpretation, and limitations

Fixture: \(2·2·1·10·2·8·2=1,280\) bytes; one extra token costs 128 bytes. Run
`python tools/kv_cache.py --self-test`. Confirm architecture config and runtime cache layout; measured peak memory can
be materially larger and context acceptance does not prove long-context quality.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
