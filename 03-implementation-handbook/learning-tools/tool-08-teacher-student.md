# TOOL-08 — Teacher–student compression comparison

## Learn before calculating

**Concept:** distillation trade-off arithmetic. **Purpose:** compare a teacher and student without confusing compression with quality proof.

**Prerequisites:** Python 3.11 and one provenance-bearing paired measurement document. **Expected inputs:** shared protocol identity and teacher/student parameters, bytes, and quality metric. **Expected outputs:** compression ratios, quality delta, and metric-direction flag. The calculator performs declared arithmetic only; it cannot substitute for artifact inspection or target measurement.

**Canonical listing:** `LST-TOOL-0008` · **Verification note:** validate with the commands below before operational use.  
**Purpose:** compare provenance-bearing measured teacher/student records without inventing quality or performance.
**Units:** parameter count, artifact bytes, named quality metric units. **Assumptions:** records use the same immutable
evaluation ID and metric definition; a higher/lower direction is supplied.

## Contract, equation, and edge cases

Input is a JSON file containing `provenance` (`experiment_id`, `dataset_sha256`, `protocol_sha256`, `measured_utc`) and
`teacher`/`student` objects with positive `parameters`, positive `artifact_bytes`, and finite `quality`.
`--direction higher|lower` controls quality delta interpretation. Output includes compression ratios, student-minus-
teacher quality delta, and `quality_change_favorable`; it is not a winner declaration. Missing/blank provenance,
nonfinite quality, mismatched experiments (one file is one experiment), and invalid values fail.

## Complete CLI listing

Intended path: `slm-implementation/tools/teacher_student.py`; Python 3.11+, reads one local JSON file.

<!-- extract:LST-TOOL-0008:start -->
<a id="lst-tool-0008"></a>
**LST-TOOL-0008 — teacher/student CLI.**

```python
from __future__ import annotations
import argparse, json, math
from pathlib import Path

def calculate(doc:dict,direction:str)->dict:
    prov=doc.get("provenance",{})
    required=("experiment_id","dataset_sha256","protocol_sha256","measured_utc")
    if any(not isinstance(prov.get(k),str) or not prov[k].strip() for k in required):
        raise ValueError("complete provenance strings required")
    rows={}
    for role in ("teacher","student"):
        x=doc.get(role,{})
        if x.get("parameters",0)<=0 or x.get("artifact_bytes",0)<=0 or not math.isfinite(x.get("quality",math.nan)):
            raise ValueError(f"invalid {role} record")
        rows[role]=x
    delta=rows["student"]["quality"]-rows["teacher"]["quality"]
    return {"tool_id":"TOOL-08","listing_id":"LST-TOOL-0008","provenance":prov,
            "parameter_compression_ratio":rows["teacher"]["parameters"]/rows["student"]["parameters"],
            "artifact_compression_ratio":rows["teacher"]["artifact_bytes"]/rows["student"]["artifact_bytes"],
            "quality_delta_student_minus_teacher":delta,
            "quality_change_favorable":delta>=0 if direction=="higher" else delta<=0}

def main()->None:
    p=argparse.ArgumentParser(); p.add_argument("--input",type=Path)
    p.add_argument("--direction",choices=("higher","lower")); p.add_argument("--self-test",action="store_true"); a=p.parse_args()
    if a.self_test:
        d={"provenance":{k:"x" for k in ("experiment_id","dataset_sha256","protocol_sha256","measured_utc")},
           "teacher":{"parameters":100,"artifact_bytes":200,"quality":.8},
           "student":{"parameters":25,"artifact_bytes":50,"quality":.75}}
        r=calculate(d,"higher"); assert r["parameter_compression_ratio"]==4 and math.isclose(r["quality_delta_student_minus_teacher"],-.05)
        print(json.dumps({"self_test":"passed"})); return
    if a.input is None or a.direction is None: p.error("--input and --direction are required")
    print(json.dumps(calculate(json.loads(a.input.read_text(encoding="utf-8")),a.direction),sort_keys=True))
if __name__=="__main__": main()
```
<!-- extract:LST-TOOL-0008:end -->

## Fixtures, commands, interpretation, and limitations

Fixture: 100/25 and 200/50 both give 4× compression; quality changes from 0.80 to 0.75, delta −0.05. Run
`python tools/teacher_student.py --self-test`, then `--input measured.json --direction higher`. The CLI cannot establish
statistical significance, metric validity, safety, latency, or causal benefit from distillation; retain item-level
paired data and uncertainty.

## Expected behavior, common failures, and exact validation

A valid invocation prints one JSON object and exits zero; malformed or incomplete input exits nonzero without silently inventing defaults. Common failures are wrong units, duplicated inventory entries, values taken from a different artifact or protocol, and interpreting an arithmetic result as measured performance.

Validate in this order: (1) reproduce the hand calculation with a calculator; (2) run the documented self-test or fixture; (3) test one invalid boundary and require nonzero exit; (4) compare JSON fields and units with the contract; (5) for operational use, compare the calculation with framework, artifact, billing, or device evidence and record the discrepancy. Preserve the command, Python version, input file hash when applicable, stdout, stderr, and exit code.
