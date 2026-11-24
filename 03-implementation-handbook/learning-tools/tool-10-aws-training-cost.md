# TOOL-10 — AWS training-cost estimator

## Learn before calculating

**Concept:** metered cloud-cost arithmetic. **Purpose:** sum dated, regional, source-identified price lines without embedding volatile prices.

**Prerequisites:** Python 3.11 and a reviewed price record from official or account billing sources. **Expected inputs:** region, currency, timestamp, meter quantities, unit prices, units, and source IDs. **Expected outputs:** line subtotals, price age, and estimated total. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0010` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** total user-supplied AWS training meter quantities and dated regional unit prices. It contains no AWS
defaults, current prices, queries, discounts, taxes, or claimed bill. **Units:** each line declares quantity unit,
currency per that unit, and currency. **Assumptions:** supplied meters are non-overlapping and rates match billed units.

## Contract, equation, and edge cases

Input JSON requires nonblank `estimate_id`, `region`, ISO `price_as_of_utc` with timezone, `currency`, and nonempty
`lines`. Every line requires unique `meter_id`, nonnegative finite `quantity`, positive finite `unit_price`, and matching
nonblank `unit`, `price_source`, `price_record_id`; all line currencies must equal the document currency.
\(C=\sum_i q_ip_i\). `--max-price-age-days` is required and positive; stale or future prices fail. Output reports each
subtotal and total, but remains an estimate. Taxes, tiers, minimums, Spot interruptions, storage, transfer, logs,
failed/recovery work, and support must each be explicit lines when applicable.

## Complete CLI listing

Intended path: `slm-implementation/tools/aws_training_cost.py`; Python 3.11+, local JSON only; no AWS/network access.

<!-- extract:LST-TOOL-0010:start -->
<a id="lst-tool-0010"></a>
**LST-TOOL-0010 — dated regional AWS cost CLI.**

```python
from __future__ import annotations
import argparse, datetime as dt, json, math
from pathlib import Path

def calculate(d:dict,max_age:int,now:dt.datetime)->dict:
    for k in ("estimate_id","region","price_as_of_utc","currency"):
        if not isinstance(d.get(k),str) or not d[k].strip(): raise ValueError(f"missing {k}")
    if max_age<=0 or not d.get("lines"): raise ValueError("positive max age and price lines required")
    stamp=dt.datetime.fromisoformat(d["price_as_of_utc"].replace("Z","+00:00"))
    if stamp.tzinfo is None: raise ValueError("price timestamp requires timezone")
    age=(now.astimezone(dt.timezone.utc)-stamp.astimezone(dt.timezone.utc)).total_seconds()/86400
    if age<0 or age>max_age: raise ValueError("price date is future or stale")
    seen=set(); out=[]; total=0.0
    for x in d["lines"]:
        strings=("meter_id","unit","currency","price_source","price_record_id")
        if any(not isinstance(x.get(k),str) or not x[k].strip() for k in strings): raise ValueError("incomplete line")
        q=x.get("quantity"); p=x.get("unit_price")
        numeric = all(isinstance(v, (int, float)) and math.isfinite(v) for v in (q, p))
        if x["meter_id"] in seen or x["currency"] != d["currency"] or not numeric or q < 0 or p <= 0:
            raise ValueError("invalid meter, currency, quantity, or price")
        seen.add(x["meter_id"]); subtotal=q*p; total+=subtotal
        out.append({"meter_id":x["meter_id"],"unit":x["unit"],"quantity":q,"unit_price":p,"subtotal":subtotal})
    return {"tool_id":"TOOL-10","listing_id":"LST-TOOL-0010","estimate_id":d["estimate_id"],
            "region":d["region"],"currency":d["currency"],"price_as_of_utc":d["price_as_of_utc"],
            "price_age_days":age,"lines":out,"estimated_total":total}

def main()->None:
    p=argparse.ArgumentParser(); p.add_argument("--input",type=Path); p.add_argument("--max-price-age-days",type=int)
    p.add_argument("--as-of-utc"); a=p.parse_args()
    if a.input is None or a.max_price_age_days is None or a.as_of_utc is None: p.error("all arguments are required")
    now=dt.datetime.fromisoformat(a.as_of_utc.replace("Z","+00:00"))
    if now.tzinfo is None: p.error("--as-of-utc requires timezone")
    print(json.dumps(calculate(json.loads(a.input.read_text(encoding="utf-8")),a.max_price_age_days,now),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0010:end -->

## Fixtures, commands, interpretation, and limitations

Hand fixture (unit arithmetic only, not an AWS price/result): quantities 2 and 3 at user-supplied rates 4 and 5 produce
subtotals 8 and 15, total 23. Use dated official/account price records for one exact Region and run
`python tools/aws_training_cost.py --input quote.json --max-price-age-days 7 --as-of-utc 2026-11-23T00:00:00Z`.
Reconcile the estimate to CUR/Cost Explorer after execution; the CLI does not model tiering or validate AWS SKU
semantics.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
