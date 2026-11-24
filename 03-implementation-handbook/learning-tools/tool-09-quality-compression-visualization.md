# TOOL-09 — Quality-versus-compression visualization

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

## Learn before calculating

**Concept:** quality–compression visualization. **Purpose:** map comparable measured candidate points into a simple auditable SVG.

**Prerequisites:** Python 3.11 and at least two measured JSONL rows with common provenance and units. **Expected inputs:** candidate identity, finite x/y metrics, units, evidence, and protocol hashes. **Expected outputs:** one SVG scatterplot and a JSON creation summary. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0009` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** render an SVG scatterplot from measured, provenance-bearing JSONL only. No illustrative or synthetic
points are accepted by the normal command. **Units:** caller-named finite numeric x/y metrics. **Assumptions:** all rows
share exact experiment, dataset, protocol, x-unit, y-unit, and `evidence="MEASURED"` provenance.

## Contract, equation, and edge cases

Each JSONL row requires nonblank `candidate_id`, `experiment_id`, `dataset_sha256`, `protocol_sha256`, `measured_utc`,
`x_name`, `x_unit`, `y_name`, `y_unit`; finite numeric `x`,`y`; and literal `evidence:"MEASURED"`. At least two rows and
nonzero x/y ranges are required. Linear mapping is
$p_x=50+500(x-x_{min})/(x_{max}-x_{min})$,
$p_y=350-300(y-y_{min})/(y_{max}-y_{min})$. Output is an escaped SVG and JSON summary; no trend, Pareto, or superiority
claim is inferred.

## Complete CLI listing

Intended path: `slm-implementation/tools/quality_compression_svg.py`; Python 3.11+, reads JSONL and writes one requested
SVG. The output path's parent must already exist; an existing file is refused unless `--overwrite`.

<!-- extract:LST-TOOL-0009:start -->
<a id="lst-tool-0009"></a>
**LST-TOOL-0009 — provenance-gated SVG CLI.**

```python
from __future__ import annotations
import argparse, html, json, math
from pathlib import Path

FIELDS=("candidate_id","experiment_id","dataset_sha256","protocol_sha256","measured_utc",
        "x_name","x_unit","y_name","y_unit")
def load(path:Path)->list[dict]:
    rows=[json.loads(line) for line in path.read_text(encoding="utf-8").splitlines() if line.strip()]
    if len(rows)<2: raise ValueError("at least two measured rows required")
    for r in rows:
        if r.get("evidence")!="MEASURED" or any(not isinstance(r.get(k),str) or not r[k].strip() for k in FIELDS):
            raise ValueError("complete MEASURED provenance required")
        if not all(isinstance(r.get(k),(int,float)) and math.isfinite(r[k]) for k in ("x","y")):
            raise ValueError("finite numeric x and y required")
    same=("experiment_id","dataset_sha256","protocol_sha256","x_name","x_unit","y_name","y_unit")
    if any(len({r[k] for r in rows})!=1 for k in same): raise ValueError("rows are not comparable")
    return rows

def render(rows:list[dict])->str:
    xs=[r["x"] for r in rows]; ys=[r["y"] for r in rows]
    if min(xs)==max(xs) or min(ys)==max(ys): raise ValueError("both axes require nonzero range")
    circles=[]
    for r in rows:
        x=50+500*(r["x"]-min(xs))/(max(xs)-min(xs)); y=350-300*(r["y"]-min(ys))/(max(ys)-min(ys))
        label=html.escape(r["candidate_id"],quote=True)
        circles.append(f'<circle cx="{x:.2f}" cy="{y:.2f}" r="5">Illustrative title{label}</title></circle>')
    xn=html.escape(rows[0]["x_name"]+" ("+rows[0]["x_unit"]+")")
    yn=html.escape(rows[0]["y_name"]+" ("+rows[0]["y_unit"]+")")
    return ('<svg xmlns="http://www.w3.org/2000/svg" width="620" height="420" role="img">'
            f'Illustrative titleMeasured {yn} versus {xn}</title><line x1="50" y1="350" x2="550" y2="350" stroke="black"/>'
            '<line x1="50" y1="50" x2="50" y2="350" stroke="black"/>'
            +''.join(circles)+f'<text x="300" y="395">{xn}</text><text x="55" y="30">{yn}</text></svg>\n')

def main()->None:
    p=argparse.ArgumentParser(); p.add_argument("--input",type=Path); p.add_argument("--output",type=Path)
    p.add_argument("--overwrite",action="store_true"); a=p.parse_args()
    if a.input is None or a.output is None: p.error("--input and --output are required")
    if not a.output.parent.is_dir() or (a.output.exists() and not a.overwrite): p.error("invalid parent or existing output")
    rows=load(a.input); a.output.write_text(render(rows),encoding="utf-8",newline="\n")
    print(json.dumps({"tool_id":"TOOL-09","listing_id":"LST-TOOL-0009","points":len(rows),
                      "output":str(a.output),"experiment_id":rows[0]["experiment_id"]},sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0009:end -->

## Fixtures, commands, interpretation, and limitations

Hand fixture/unit test: two measured rows at $(1,2)$ and $(3,4)$ map to $(50,350)$ and $(550,50)$. Create a
temporary JSONL with those values and identical nonblank provenance, then run
`python tools/quality_compression_svg.py --input measured.jsonl --output plot.svg`; assert two `<circle` elements and
the experiment ID in JSON. The plot reports supplied measurements only; audit metric direction, uncertainty, failures,
quality/safety gates, and provenance before interpretation.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
