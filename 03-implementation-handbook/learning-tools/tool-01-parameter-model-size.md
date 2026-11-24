# TOOL-01 — Parameter-count and model-size calculator

## Learn before calculating

**Concept:** storage arithmetic. **Purpose:** turn tensor shapes and encoding width into a transparent lower-bound storage calculation.

**Prerequisites:** Python 3.11 and a deduplicated inventory of tensor shapes. **Expected inputs:** repeated named shapes, bits per parameter, and explicit metadata bytes. **Expected outputs:** parameter count plus payload, overhead, total bytes, and MiB. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0001` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** sum explicitly supplied tensor element counts and estimate packed weight bytes. It does not inspect a
checkpoint, detect tied aliases, or predict resident memory. **Units:** parameter count; bits/parameter; bytes and MiB
($2^{20}$ bytes). **Assumptions:** each named shape is unique; dimensions are positive integers; one global bit width;
metadata bytes are supplied separately.

## Contract, equation, and edge cases

Input is one or more `--shape NAME:DIMxDIM...`, required `--bits`, and optional nonnegative `--overhead-bytes`.
Output schema is `{"tool_id","listing_id","parameter_count","payload_bytes","overhead_bytes","total_bytes","total_mib"}`.
For unique tensors $i$, $N=\sum_i\prod_jd_{ij}$ and
$B=\lceil N b/8\rceil+B_\text{overhead}$. Empty/zero/negative dimensions, duplicate names, nonpositive bit widths,
and negative overhead fail. Tied tensors must be listed once; padding, scales, tokenizer, runtime, and sparse encoding
must be entered as overhead or measured separately.

## Complete CLI listing

Intended path: `slm-implementation/tools/parameter_model_size.py`; entry point is `main`; Python 3.11+, no network,
secrets, writes, or cleanup. Run from the extraction root.

<!-- extract:LST-TOOL-0001:start -->
<a id="lst-tool-0001"></a>
**LST-TOOL-0001 — parameter/model-size CLI.**

```python
from __future__ import annotations
import argparse, json, math

def calculate(specs: list[str], bits: float, overhead: int) -> dict:
    if not specs or bits <= 0 or overhead < 0:
        raise ValueError("shapes required; bits > 0 and overhead >= 0")
    names, total = set(), 0
    for spec in specs:
        name, sep, raw = spec.partition(":")
        if not sep or not name or name in names:
            raise ValueError(f"invalid or duplicate shape: {spec}")
        dims = [int(x) for x in raw.lower().split("x")]
        if not dims or any(d <= 0 for d in dims):
            raise ValueError(f"dimensions must be positive: {spec}")
        names.add(name); total += math.prod(dims)
    payload = math.ceil(total * bits / 8)
    size = payload + overhead
    return {"tool_id":"TOOL-01","listing_id":"LST-TOOL-0001","parameter_count":total,
            "payload_bytes":payload,"overhead_bytes":overhead,"total_bytes":size,
            "total_mib":size/(2**20)}

def self_test() -> None:
    r = calculate(["a:2x3", "b:4"], 16, 8)
    assert r["parameter_count"] == 10 and r["payload_bytes"] == 20
    assert r["total_bytes"] == 28

def main() -> None:
    p=argparse.ArgumentParser()
    p.add_argument("--shape", action="append", default=[])
    p.add_argument("--bits", type=float)
    p.add_argument("--overhead-bytes", type=int, default=0)
    p.add_argument("--self-test", action="store_true")
    a=p.parse_args()
    if a.self_test: self_test(); print(json.dumps({"self_test":"passed"})); return
    if a.bits is None: p.error("--bits is required")
    print(json.dumps(calculate(a.shape,a.bits,a.overhead_bytes),sort_keys=True))
if __name__ == "__main__": main()
```
<!-- extract:LST-TOOL-0001:end -->

## Fixtures, commands, interpretation, and limitations

Hand fixture: $2\times3+4=10$ parameters; at 16 bits, payload $=20$ bytes; plus 8 bytes gives 28 bytes.
Run `python tools/parameter_model_size.py --self-test` and
`python tools/parameter_model_size.py --shape a:2x3 --shape b:4 --bits 16 --overhead-bytes 8`.
`total_mib` is an arithmetic storage estimate, not an artifact or RAM measurement. Compare the result to serialized
file bytes and alias-aware framework counts before use.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
