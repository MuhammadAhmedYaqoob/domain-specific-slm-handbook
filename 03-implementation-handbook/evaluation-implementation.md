# Evaluation implementation

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

## Concept, purpose, prerequisites, and outputs

Evaluation estimates behavior under a frozen protocol; it does not certify a model in the abstract. The evaluated unit is the full model, tokenizer, template, decoding policy, runtime, hardware, and workload tuple. Pairing candidates on the same scenario families reduces irrelevant variation and exposes regressions.

**Prerequisites:** sealed independent items, approved scoring rubrics and thresholds, immutable artifacts, blinded reviewers where needed, and a registered statistical plan. **Inputs:** evaluation items, generation records, artifact identities, reviewer decisions, and benchmark trials. **Outputs:** append-only raw generations, deterministic scores, paired effects and intervals, failure counts, scorecards, and signed decisions.

### Worked example

`auto-q4-parity-20261124` compares a merged parent with its Q4 child on the same root families. Generation receives prompts only; scoring later joins references. Expected behavior is one raw record per attempt, including OOM and timeout outcomes. Validate scorer fixtures by hand, inspect rendered token IDs, include every failed attempt in denominators, bootstrap root families rather than turns, and reject promotion when any non-compensatory safety gate fails.

## Contract

> Verification note: commands and listings require execution in the stated target environment before operational use.

Canonical definitions and statistical cautions are in
[CH-29](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-29-model-quality-evaluation.md),
domain suites in
[CH-30](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md),
and systems benchmarking in
[CH-31](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md).

## Evaluation architecture

```text
frozen blueprint + sealed items + approved authorities
                     │
frozen artifact + template + decoding + runtime
                     ▼
append-only raw generation (no references exposed)
                     ▼
deterministic parsers/metrics + blinded expert review
                     ▼
paired scenario-level statistics + contamination audit
                     ▼
non-compensatory quality/safety/system gates
                     ▼
signed report or explicit rejection
```

Keep inference and scoring separate. Generation workers receive prompts, never private answers/rubrics. Preserve raw
output exactly; regenerated text is different evidence.

## Evaluation item and manifest schemas

**Listing EVAL-L01 — extraction path `schemas/evaluation.schema.json`.**

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "required": ["id", "root_family_id", "construct", "prompt", "scoring", "strata", "provenance"],
  "properties": {
    "id": { "type": "string", "minLength": 1 },
    "root_family_id": { "type": "string", "minLength": 1 },
    "construct": { "type": "string", "minLength": 1 },
    "prompt": {
      "type": "object",
      "required": ["messages"],
      "properties": {
        "messages": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "required": ["role", "content"],
            "properties": {
              "role": { "enum": ["system", "user", "assistant"] },
              "content": { "type": "string" }
            }
          }
        }
      }
    },
    "scoring": {
      "type": "object",
      "required": ["type"],
      "properties": {
        "type": { "enum": ["exact_match", "label", "json_schema", "expert_rubric"] },
        "accepted": { "type": "array", "items": { "type": "string" } },
        "label": { "type": "string" },
        "rubric_id": { "type": "string" },
        "critical_failures": { "type": "array", "items": { "type": "string" } }
      }
    },
    "strata": {
      "type": "object",
      "required": ["domain", "risk_tier", "difficulty"],
      "properties": {
        "domain": { "type": "string" },
        "risk_tier": { "enum": ["low", "moderate", "high", "critical"] },
        "difficulty": { "enum": ["D1", "D2", "D3", "D4", "D5"] }
      }
    },
    "provenance": {
      "type": "object",
      "required": ["authority_ids", "independence_review"],
      "properties": {
        "authority_ids": { "type": "array", "minItems": 1, "items": { "type": "string" } },
        "independence_review": { "enum": ["passed", "failed", "awaiting_review"] }
      }
    }
  }
}
```

**Listing EVAL-L02 — extraction path `configs/evaluation.yaml`.**

```yaml
schema: slm-evaluation/v1
status: PLANNED_EXPERIMENT
experiment_id: "auto-q4-parity-20261124"
dataset:
  path: "data/evaluation/sealed.jsonl"
  manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  split: sealed_test
  grouping_key: root_family_id
artifacts:
  - { id: parent, path: "artifacts/merged/auto-lora-r1729-20261124", sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef" }
  - { id: candidate, path: "artifacts/exports/auto-q4/model-q4_k_m.gguf", sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef" }
tokenizer_revision: "08d7ac0eedd64c9d751783ed8d2178834c21b879"
chat_template_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
generation:
  temperature: 0.0
  top_p: 1.0
  max_new_tokens: 256
  repetitions: 1
statistics:
  confidence: 0.95
  bootstrap_unit: root_family_id
  bootstrap_replicates: 10000
  seed: 1729
gates:
  critical_safety_failures_max: 0
  unsupported_claim_rate_max: 0.02
  general_retention_noninferiority_margin: 0.03
```

The paths and hashes demonstrate a complete configuration shape; recompute the
hashes for extracted artifacts. The numeric thresholds are tutorial policy
examples, not measured acceptance recommendations. Establish production
thresholds from intended use, consequence, prevalence, safeguards, and
qualified review before opening the sealed test.

## Raw generation record

Each attempt, including failures, produces one line:

```json
{
  "schema": "slm-eval-output/v1",
  "experiment_id": "illustrative-id-001",
  "artifact_id": "candidate",
  "artifact_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "item_id": "eval-001",
  "root_family_id": "scenario-001",
  "sample_id": 0,
  "rendered_prompt_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "input_tokens": null,
  "requested_output_tokens": 256,
  "generated_tokens": null,
  "output": "",
  "stop_reason": null,
  "status": "success|oom|timeout|crash|invalid",
  "elapsed_ns": null,
  "runtime_manifest_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
}
```

`null` means unavailable, never zero. Do not drop OOM, timeout, crash, malformed output, or early termination from
denominators.

## Deterministic scoring and paired statistics

**Listing EVAL-L03 — extraction path `src/slm_impl/evaluate/score.py`.**

```python
from __future__ import annotations
import argparse, hashlib, json, math, random
from collections import defaultdict
from pathlib import Path

def digest(path: Path) -> str:
    h=hashlib.sha256()
    with path.open("rb") as f:
        for block in iter(lambda:f.read(1<<20),b""): h.update(block)
    return h.hexdigest()

def normalize(text: str) -> str:
    return " ".join(text.casefold().split())

def load_jsonl(path: Path) -> list[dict]:
    rows=[]; seen=set()
    with path.open(encoding="utf-8") as f:
        for number,line in enumerate(f,1):
            row=json.loads(line)
            key=(row["artifact_id"],row["item_id"],row.get("sample_id",0))
            if key in seen: raise ValueError(f"{path}:{number}: duplicate {key}")
            seen.add(key); rows.append(row)
    return rows

def score(row: dict) -> float:
    if row["status"]!="success": return 0.0
    spec=row["scoring"]; kind=spec["type"]
    if kind=="exact_match":
        return float(normalize(row["output"]) in {normalize(x) for x in spec["accepted"]})
    if kind=="label":
        return float(row["parsed_label"]==spec["label"])
    if kind=="expert_rubric":
        if row.get("adjudicated_score") is None: raise ValueError("rubric score missing")
        if row.get("critical_failure"): return 0.0
        return float(row["adjudicated_score"])/float(spec["max_score"])
    raise ValueError(f"unsupported scoring type {kind}")

def grouped(rows: list[dict]) -> dict:
    out=defaultdict(lambda:defaultdict(list))
    for row in rows: out[row["root_family_id"]][row["artifact_id"]].append(score(row))
    return out

def paired_bootstrap(groups: dict,a: str,b: str,reps: int,seed: int) -> dict:
    ids=sorted(k for k,v in groups.items() if a in v and b in v)
    if not ids: raise ValueError("no paired scenario groups")
    effects={k:sum(groups[k][b])/len(groups[k][b])
               -sum(groups[k][a])/len(groups[k][a]) for k in ids}
    observed=sum(effects.values())/len(ids); rng=random.Random(seed); draws=[]
    for _ in range(reps):
        sample=rng.choices(ids,k=len(ids))
        draws.append(sum(effects[k] for k in sample)/len(sample))
    draws.sort()
    return {"paired_groups":len(ids),"effect_b_minus_a":observed,
      "percentile_95_ci":[draws[math.floor(.025*reps)],
                          draws[min(reps-1,math.ceil(.975*reps)-1)]]}

p=argparse.ArgumentParser()
p.add_argument("scored_jsonl",type=Path); p.add_argument("--a",required=True)
p.add_argument("--b",required=True); p.add_argument("--reps",type=int,default=10000)
p.add_argument("--seed",type=int,default=1729); a=p.parse_args()
rows=load_jsonl(a.scored_jsonl)
print(json.dumps({"input_sha256":digest(a.scored_jsonl),
  **paired_bootstrap(grouped(rows),a.a,a.b,a.reps,a.seed)},sort_keys=True))
```

Before use, build fixtures for all-correct/all-wrong, duplicates, missing pairs, normalization, failures, clustered
variants, and hand-computed bootstrap cases. Add confusion metrics, Wilson/exact intervals, McNemar tests,
schema/stratum checks, and multiple-comparison policy to the production analyzer.

## Quality scorecard

Never use one leaderboard score. Report at least:

| Axis              | Metrics and evidence                                                                    |
| ----------------- | --------------------------------------------------------------------------------------- |
| language modeling | fixed-tokenizer negative log-likelihood/perplexity                                      |
| discrete tasks    | accuracy, exact match, per-class precision/recall/F1, confusion                         |
| factual support   | supported/contradicted/unsupported/unverifiable claims and response incidence           |
| diagnosis         | evidence use, alternative hypotheses, next-action utility, ordering, unnecessary action |
| uncertainty       | explicit event definition, Brier/NLL, reliability, selective risk/coverage              |
| robustness/OOD    | invariance and sensitivity pairs, worst variant, time/product/region shifts             |
| multi-turn        | state retention, correction, contradiction, first failure turn                          |
| retention         | paired domain-independent suite with non-inferiority margins                            |
| safety/refusal    | unsafe compliance, missed escalation, over-refusal, benign-answer rate                  |
| structured output | parse, schema, field, evidence-span, and semantic correctness                           |
| deployment parity | source versus exported artifact on identical prompts                                    |

Perplexity does not prove factuality or safety. Semantic similarity does not prove truth and cannot override units,
negation, identifiers, dates, or critical failures.

### Claim review

Segment open answers into minimal checkable claims. Two qualified reviewers label each supported, contradicted,
unsupported, unverifiable, or non-factual against versioned authorities. Preserve authority/date/jurisdiction/product
scope, evidence span, reviewer disagreement, and adjudication. Report both claim-weighted rate and fraction of responses
containing at least one problematic claim.

### Diagnostic rubric

Use observable behavior, not hidden chain-of-thought:

- confirmed observations remain distinct from hypotheses;
- multiple plausible hypotheses are maintained;
- the next check is feasible, discriminating, and ordered after safety prerequisites;
- new evidence updates the plan;
- uncertainty and escalation are specific;
- premature replacement, dose change, or unsafe action is a critical failure.

Counterfactual pairs change one decisive fact and require the decision to change. Delete decisive evidence to detect
hindsight leakage.

### Human/expert review

Freeze rubric anchors and reviewer qualifications. Blind artifact identity, randomize response order, permit
ties/both-good/both-bad/insufficient-evidence, collect independent ratings, and adjudicate critical disagreements.
Report raw agreement, confusion, and agreement statistics by dimension. Agreement is not correctness; authoritative
adjudication remains necessary.

An automated model judge may triage only after validation against qualified humans by domain, risk, length, language,
and candidate family. It cannot be the sole critical gate or be the same judge that generated/trained the candidate.

## Contamination controls

Evaluation roots, text, answers, rubrics, embeddings, statistics, and prompts must not enter:

- CPT, SFT, preference, or distillation;
- teacher/judge generation or few-shot prompts;
- quantization/pruning calibration;
- prompt/template/decoding optimization;
- checkpoint, threshold, or compression selection.

Audit exact, normalized, near-duplicate, OCR/revision, translation/paraphrase, shared source/scenario/product,
synthetic-template, answer/rationale/rubric, teacher/judge, temporal, and analyst-feedback leakage. Hash matching is
insufficient. Once an item guides development, retire it from sealed evaluation.

## Systems benchmark contract

Measure the complete model/runtime/hardware tuple:

- exact artifact/download/install/staging/rollback bytes;
- pre-load, load peak, post-load, prefill/decode peak, and max-context memory using the platform-native named metric;
- process-cold and warm startup;
- first model token and first visible text (TTFT) separately;
- prefill input tokens/s and decode generated tokens/s separately;
- end-to-end latency and inter-token latency distribution;
- CPU/GPU/NPU placement/utilization where measurable;
- crash/OOM/timeout/invalid output over all attempts;
- gross and incremental energy per token only with a declared validated boundary;
- ambient/start temperature, thermal state/sensors, clocks, early/late sustained throughput;
- conversion time, peak host/GPU memory, temporary disk;
- AWS offered load, queueing, batching, errors, scaling, and dated allocated cost.

**Listing EVAL-L04 — extraction path `schemas/benchmark-trial.example.json`.**

```json
{
  "schema": "slm-benchmark/v1",
  "evidence": "PLANNED_EXPERIMENT",
  "run_id": "00000000-0000-4000-8000-000000000001",
  "trial_index": 0,
  "identity": { "artifact_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa", "runtime_revision": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa", "app_or_container_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" },
  "target": {
    "class": "android|ios|linux|aws",
    "device_or_instance": "illustrative-exact-value",
    "os_build": "illustrative-exact-value",
    "backend": "illustrative-exact-value"
  },
  "conditions": {
    "ambient_c": null,
    "start_temperature_c": null,
    "battery_percent": null,
    "charging": null,
    "background_policy": "illustrative-exact-value"
  },
  "workload": {
    "prompt_id": "illustrative-id-001",
    "input_tokens": null,
    "requested_output_tokens": 128,
    "generated_tokens": null,
    "batch": 1,
    "concurrency": 1
  },
  "timing": {
    "startup_ms": null,
    "first_model_token_ms": null,
    "ttft_visible_ms": null,
    "prefill_ms": null,
    "decode_ms": null,
    "e2e_ms": null
  },
  "resources": {
    "memory_metric_name": "RSS|PSS|footprint",
    "memory_after_load_bytes": null,
    "peak_runtime_memory_bytes": null
  },
  "energy_thermal": {
    "measurement_boundary": "unavailable",
    "gross_energy_j": null,
    "energy_per_generated_token_j": null,
    "thermal_trace_uri": null
  },
  "outcome": { "status": "success|crash|oom|timeout|cancelled|invalid_output", "failure_phase": null },
  "raw_manifest_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
}
```

Use one row per trial. Fixed-output performance prompts and natural-completion quality prompts are separate workloads.
Record actual tokenizer IDs/counts, template, stop rules, cache state, context position, warmups, randomized order,
operating-system build, available memory, charge/power/background/display/radio state, and start-temperature band.

### Timing formulas

For prompt tokens \(N_p\), generated tokens \(N_g\), prefill start/end \(t_{ps},t_{pe}\), first token \(t_1\), and
final generated token \(t_g\):

```text
prefill_tokens_s = N_p / (t_pe - t_ps)
decode_tokens_s  = (N_g - 1) / (t_g - t_1), N_g >= 2
visible_TTFT     = first_visible_fragment - request_submit
```

Synchronize asynchronous accelerators. Do not call total tokens divided by end-to-end time “decode throughput.”

### Platform evidence

- **Android:** release/profileable physical app; monotonic in-app events; Perfetto/System Trace; `dumpsys meminfo`;
  thermal API; operator partition; long unplugged energy run or external instrument.
- **iOS:** signed Release physical build; `os_signpost`; Instruments Time Profiler/Allocations/VM Tracker/Core ML/Metal;
  `ProcessInfo.thermalState`; external energy boundary or report unavailable.
- **Linux/edge:** monotonic events; `/usr/bin/time -v`, `/proc`/PSS/cgroups; named thermal/hwmon sensors;
  RAPL/Jetson/NVIDIA counters with scope.
- **AWS:** client, service, queue, model, and accelerator timing; fixed offered load; CloudWatch/DCGM where configured;
  actual dated regional billing allocation.

Simulator/emulator evidence is functional only. Backend selection is not proof of GPU/NPU execution.

## Statistical and decision procedure

1. Pre-register primary/secondary metrics, scenario-level unit, thresholds/margins, exclusions, trial count, warm-up,
   confidence method, multiplicity, and failure policy.
2. Use identical items and pairing keys. Bootstrap scenario/root families, not correlated turns/variants as independent
   rows.
3. Report effects and confidence intervals, raw counts, discordant pairs, failures, and all prespecified regressions.
4. Keep natural-prevalence, balanced diagnostic, and rare-critical views separate.
5. Build Pareto fronts only among hard-gate-qualified candidates measured under comparable target conditions.
6. If evidence is missing or conditions differ materially, report “not established” or separate strata.

Do not directly rank different tokenizers by raw tokens/s, cold against warm, burst against sustained, emulator against
phone, server aggregate throughput against mobile batch-one, different energy boundaries, different AWS
region/date/purchase model, or external paper/vendor numbers against project measurements.

## Commands and exact validation

```powershell
# Verification note: not executed here
python -m jsonschema -i data/evaluation/sealed.jsonl schemas/evaluation.schema.json
python -m slm_impl.evaluate.generate --config configs/evaluation.yaml `
  --out results/raw/auto-q4-parity-20261124.jsonl
python src/slm_impl/evaluate/score.py `
  results/raw/auto-q4-parity-20261124-scored.jsonl --a parent --b candidate `
  --reps 10000 --seed 1729
```

The first command does not validate JSONL as a stream in every CLI version; production must validate line by line with
the frozen library. Exact release validation:

1. verify artifact/data/template/runtime hashes;
2. run known-answer metric fixtures;
3. prove references/rubrics are absent from generation input/logs;
4. inspect rendered tokens and source/export parity;
5. execute deterministic metrics and blinded expert review;
6. audit contamination and reviewer authorization;
7. compute paired intervals at the registered unit;
8. include all failures and protocol deviations;
9. repeat full quality/safety suite on every export/precision/backend;
10. archive raw records, traces, adjudications, scripts, locks, and report hash.

## Completion record

The schemas, metric portfolio, claim/diagnostic/human review, contamination controls, statistical protocol, systems
metric dictionary, platform procedures, commands, and exact release validation are authored. No output, score, interval,
expert judgment, cloud/device measurement, price, or release decision exists.
