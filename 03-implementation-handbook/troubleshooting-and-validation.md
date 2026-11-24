# Troubleshooting and validation

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

## Concept, purpose, prerequisites, and outputs

Troubleshooting is controlled comparison between the last qualified parent and the first failing child. Preserving identities and changing one factor prevents an apparent fix from concealing data leakage, incompatible kernels, or a different model.

**Prerequisites:** immutable artifacts, raw logs, hashes, reproducible commands, safe rollback, and access to the relevant test fixture or target. **Inputs:** failure record, parent and child manifests, environment probe, first bad item, and traces. **Outputs:** a minimal reproduction, supported root cause, corrective change, rerun evidence, and release or rollback decision.

### Worked example

If a Q4 export emits role markers, compare the training, parent, converter, desktop runtime, and target-runtime token IDs for one golden conversation. The first boundary with different IDs owns the fault. Expected behavior after correction is identical special-token placement and bounded greedy parity. Validate with clean reload, multi-turn reset, EOS handling, full quality/safety regression, and target benchmark; do not accept a string-only comparison.

## Contract

> Verification note: commands and listings require execution in the stated target environment before operational use.

## Universal incident record

**Listing VALID-L01 — extraction path `schemas/failure-record.example.json`.**

```json
{
  "schema": "slm-failure/v1",
  "failure_id": "00000000-0000-4000-8000-000000000001",
  "utc": "2026-01-15T12:00:00Z",
  "stage": "data|train|merge|compress|export|load|prefill|decode|update|serve",
  "artifact_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "parent_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "data_manifest_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "config_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "environment_lock_sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "command": ["illustrative-exact-value"],
  "hardware": { "device": "illustrative-exact-value", "ram": "illustrative-exact-value", "driver_os": "illustrative-exact-value" },
  "symptom": "validation loss increased after step 120",
  "first_bad_batch_or_item": "illustrative-id-001",
  "stdout_uri": "<path+sha256>",
  "stderr_uri": "<path+sha256>",
  "trace_uri": "<path+sha256-or-null>",
  "reproduction_count": 0,
  "hypotheses": [],
  "tests": [],
  "resolution": null,
  "rollback_artifact": "illustrative-id-001"
}
```

Never store credentials, personally identifiable information, private prompts, or unsafe raw content in general logs.
Restrict and redact diagnostic artifacts while retaining enough hashed identity to reproduce.

## Diagnosis method

1. Stop promotion and preserve the last qualified parent.
2. Verify hashes, exact command, environment, hardware, available memory, and data IDs.
3. Reproduce once under the same condition; do not loop on destructive or expensive failures.
4. Reduce to the smallest failing stage/item without changing semantics.
5. Compare parent versus child at token IDs, logits, generation, quality, and runtime placement.
6. Change one factor and record the result.
7. Fix the responsible high-precision parent or configuration.
8. Rerun all downstream gates; never assume the fix is local.
9. Roll back if hard evidence is incomplete.

## Data failures

<!-- markdownlint-disable MD013 -->
| Symptom                        | Likely cause                      | First evidence                | Safe correction                         |
| ------------------------------ | --------------------------------- | ----------------------------- | --------------------------------------- |
| empty PDF text                 | scan/encryption/font map          | page render + parser warnings | quarantine; OCR reviewed pages          |
| scrambled columns              | wrong reading order               | block coordinates/page image  | layout-aware extraction and review      |
| wrong number/unit              | OCR/locale ambiguity              | source crop + raw cell        | preserve raw; expert adjudication       |
| source appears in all splits   | split after chunking              | lineage/root report           | regroup and rebuild snapshots           |
| huge duplicate cluster         | boilerplate/transitive chaining   | pair samples/cluster graph    | remove boilerplate; validate thresholds |
| unknown-rights row accepted    | quality score overrode hard gate  | source registry decision      | revoke snapshot and descendants         |
| PII/secrets emitted            | inadequate detection/minimization | restricted source trace       | quarantine, incident/takedown, rebuild  |
| contradiction silently removed | majority/recency heuristic        | scoped claim versions         | preserve both; qualified adjudication   |
| snapshot changes in place      | mutable path/process              | file hash history             | publish new immutable version           |
<!-- markdownlint-restore -->

Validation: prove source withdrawal identifies every descendant; prove evaluation roots never enter any
training/calibration view; manually audit high-risk quantities and warnings.

## Environment and dependency failures

<!-- markdownlint-disable MD013 -->
| Symptom                        | Cause                              | Diagnostic                            | Correction                                       |
| ------------------------------ | ---------------------------------- | ------------------------------------- | ------------------------------------------------ |
| no matching wheel              | unsupported Python/OS/architecture | resolver output and index metadata    | choose supported lane or build reviewed wheel    |
| CUDA unavailable               | driver/runtime/container mismatch  | `nvidia-smi`, torch CUDA versions     | align documented stack; rebuild image            |
| bitsandbytes fails             | unsupported backend/kernel         | minimal import and 4-bit linear smoke | use supported GPU/backend or LoRA/full           |
| API argument missing           | library drift                      | installed version/source signature    | update one pin and all affected listings/tests   |
| lock hash mismatch             | mutable input/cache compromise     | recompute trusted source hash         | stop; rebuild lock from reviewed input           |
| iOS build attempted on Windows | platform requirement               | host OS                               | move build/signing to qualified macOS/Xcode host |
<!-- markdownlint-restore -->

Do not solve compatibility by removing version pins or enabling unreviewed remote checkpoint code.

## Training failures

### Out of memory (OOM)

Identify phase: model load, first forward, backward, optimizer, evaluation, save, merge, or generation. Record host/GPU
memory timeline and exact sequence/batch.

Reduction order:

1. kill unrelated processes and confirm no leaked model copies;
2. reduce microbatch while preserving effective batch via accumulation;
3. reduce sequence length only if the product/data contract permits;
4. enable activation checkpointing and supported memory-efficient attention;
5. use LoRA/QLoRA or optimizer/sharding appropriate to the hypothesis;
6. offload only after measuring transfer and stability;
7. select a smaller model rather than silently changing the experiment.

Re-run the intended max-length batch and a full optimizer step. Checkpoint file size is not a VRAM estimate.

### Zero, NaN, or spiking loss

```python
# Verification note: not executed here — extraction path scripts/inspect_batch.py
import torch
def inspect(batch, model):
    labels=batch["labels"]
    print({"shape":list(labels.shape),"active":int(labels.ne(-100).sum()),
           "min_id":int(batch["input_ids"].min()),
           "max_id":int(batch["input_ids"].max()),
           "vocab":model.config.vocab_size})
    if not labels.ne(-100).any(): raise RuntimeError("all labels masked")
    if batch["input_ids"].max()>=model.config.vocab_size:
        raise RuntimeError("token id outside vocabulary")
```

Check in order: malformed/empty target, assistant mask, truncation, out-of-range IDs, exploding gradients, learning
rate/scheduler, FP16 overflow, corrupt sample, and distributed reduction. Replay the first bad batch in FP32 or BF16
where supported. Lowering learning rate without finding a masking/data bug can hide the cause.

### Template and tokenizer faults

Symptoms: role markers in output, continuation of user text, no stop, gibberish after export, unexpectedly high loss.

Exact checks:

1. hash tokenizer files and `chat_template.jinja`;
2. render one golden multi-turn conversation in training, evaluation, AWS, Android, and iOS;
3. compare token IDs including BOS/EOS/end-of-turn;
4. verify `add_generation_prompt` policy;
5. print assistant loss mask token by token;
6. verify converter metadata and runtime template;
7. clear KV state between conversations.

String-equivalent prompts are insufficient if IDs differ.

### LoRA/QLoRA and resume faults

<!-- markdownlint-disable MD013 -->
| Symptom                   | Cause                                           | Check                                       |
| ------------------------- | ----------------------------------------------- | ------------------------------------------- |
| no improvement            | zero/wrong adapter targets                      | trainable names, A/B gradients, update norm |
| merge mismatch            | wrong base revision/dtype/double merge          | hashes and attached-vs-merged logits        |
| QLoRA export fails        | NF4 training artifact treated as serving format | clean high-precision merge then target PTQ  |
| resume jumps/loss differs | optimizer/scheduler/sampler state absent        | four-step uninterrupted vs 2+resume         |
| full and LoRA unfair      | different tokens/search/checkpoint policy       | run manifests and supervised-token counts   |
<!-- markdownlint-restore -->

Reject an adapter that cannot identify its exact base, targets, rank/alpha/scaling, initialization/conversion metadata,
tokenizer, and template.

## Compression and conversion failures

<!-- markdownlint-disable MD013 -->
| Symptom                           | Likely cause                               | Evidence                          | Correction                                               |
| --------------------------------- | ------------------------------------------ | --------------------------------- | -------------------------------------------------------- |
| 4-bit slower than FP16            | dequant/unpack or fallback                 | profiler/operator trace           | supported layout/kernel or retain FP16                   |
| small file, high RAM              | expansion, copy, KV/workspace              | phase memory timeline/dtypes      | mmap, context cap, native format                         |
| quality collapse                  | wrong template/calibration/aggressive bits | source IDs/logits/layer errors    | parity fix, representative calibration, higher precision |
| GPTQ/AWQ won't load               | producer/consumer layout mismatch          | exact versions/group metadata     | supported paired exporter/runtime                        |
| pruned model not faster           | dense zeros/unsupported sparsity           | tensor shapes and executed kernel | structured dense rewrite or reject speed claim           |
| structured model won't load       | GQA/MLP/tie/config mismatch                | shape dependency audit            | restore coupled dimensions/config                        |
| acceptable perplexity, bad safety | tail sensitivity hidden                    | paired safety item outputs        | reject/recover; do not average away                      |
| long context only fails           | KV precision/cache/position mismatch       | context sweep                     | increase KV precision/fix cache/cap context              |
<!-- markdownlint-restore -->

### Conversion bisect

Compare these immutable boundaries:

```text
adapter attached → merged FP → converter source load → exported FP
→ quantized artifact → desktop runtime → target runtime
```

At each boundary run the same token IDs, one-step logits where exposed, fixed greedy outputs, and clean reload. The
first divergence owns the fault.

Unsupported operator handling:

1. capture converter and partition report;
2. identify exact operator, shape, dtype, and dynamic constraint;
3. determine whether fallback is correct but slower, or unsupported;
4. use a documented alternate export/architecture;
5. never claim NPU/GPU acceleration from backend selection alone.

## Mobile failures

### Load crash or operating-system kill

Collect Android exit reason/logcat/Perfetto/memory or iOS crash/MetricKit/Instruments evidence without private prompts.
Distinguish corrupt package, ABI/native symbol, memory limit, file protection, mmap, temporary copies, context
allocation, and watchdog/UI blocking.

Corrective order:

- verify manifest/signature/hash and free-space/update peak;
- load on background worker/actor without blocking UI;
- measure load peak, post-load, first prefill, and max context;
- eliminate duplicate buffers and release old context before swap;
- lower context/batch or choose a smaller target-qualified artifact;
- test memory warning/pressure and process relaunch.

### Streaming corruption or frozen UI

Use incremental UTF-8 decoding; do not assume token bytes form complete characters. Serialize native engine access,
deliver UI updates on the main thread, bound update frequency, and test cancellation during prefill/decode.

### Thermal slowdown

Record token timestamps, context position, clocks/utilization, thermal state/sensor, ambient, battery/charging, and
background activity. A late slowdown can be context growth, not heat. Re-run randomized variants from the same
start-temperature band. Do not disable platform thermal safeguards.

Mitigations after diagnosis: fewer threads, different CPU affinity, reduced GPU offload, shorter approved
context/output, efficient kernel/model, and honest sustained service level.

### Offline/update failure

Test airplane mode after installation, interrupted range download, wrong content length, corrupt chunk, no space,
process death before/after activation, signature failure, incompatible runtime, revoked version, and rollback.
Activation is atomic only after complete verification. Never delete the prior approved package before the new package
passes post-activation health checks.

## AWS failures

<!-- markdownlint-disable MD013 -->
| Symptom                | First checks                               | Recovery                                                   |
| ---------------------- | ------------------------------------------ | ---------------------------------------------------------- |
| task won't place       | capacity provider, quota, GPU resource, AZ | provision qualified capacity; do not change model silently |
| image pull fails       | digest, ECR auth/endpoints, IAM            | fix execution role/network; retain digest                  |
| readiness fails        | load logs, model hash, disk/RAM/GPU        | keep target out of service; rollback                       |
| OOM under load         | queue/batch/KV/concurrency and GPU memory  | cap admission, reduce batch/context, scale out             |
| poor TTFT              | queue, cold load, tokenization, prefill    | separate stages; warm capacity; tune batching              |
| errors during deploy   | draining/health/start period               | halt rollout; route to prior task definition               |
| autoscaling oscillates | wrong metric/cooldown                      | scale on offered load/queue/SLO with bounds                |
| cost appears too low   | idle/storage/network/LB/logging omitted    | billing allocation and success denominator                 |
| private data in logs   | unsafe logging                             | stop collection, incident response, redact/retention fix   |
<!-- markdownlint-restore -->

Safe rollback:

1. stop traffic promotion;
2. route to the last signed qualified image + model manifest + runtime config;
3. drain/cancel new requests without mixing model versions in one response;
4. verify readiness and artifact identity;
5. run critical smoke/safety tests;
6. monitor error/latency;
7. preserve failed artifacts/logs and open incident record.

Do not delete the failed evidence or overwrite the release tag.

## Automated static validation

**Listing VALID-L02 — extraction path `scripts/validate_repository.py`.**

<!-- markdownlint-disable MD013 -->
````python
from __future__ import annotations
import ast, re, sys
from pathlib import Path

ROOT=Path(sys.argv[1] if len(sys.argv)>1 else ".").resolve()
errors=[]
for path in ROOT.rglob("*.md"):
    text=path.read_text("utf-8")
    fence=None
    language=""
    body=[]
    for number,line in enumerate(text.splitlines(),1):
        marker=re.match(r"^(`{3,})([A-Za-z0-9_-]*)\s*$",line)
        if fence is None and marker:
            fence=marker.group(1); language=marker.group(2).casefold(); body=[]
        elif fence is not None and line==fence:
            if language=="python":
                try: ast.parse("\n".join(body),filename=f"{path}:{number}")
                except SyntaxError as exc: errors.append(str(exc))
            fence=None; language=""; body=[]
        elif fence is not None:
            body.append(line)
    if fence is not None: errors.append(f"{path}: unclosed {fence} fence")
    for match in re.finditer(r"\[[^\]]+\]\(([^)]+)\)",text):
        target=match.group(1)
        if target.startswith(("http://","https://","#","mailto:")): continue
        file_part=target.split("#",1)[0]
        if file_part and not (path.parent/file_part).resolve().exists():
            errors.append(f"{path}: missing link {target}")
    forbidden=("TO"+"DO","TB"+"D","fill "+"this in")
    for token in forbidden:
        if token.casefold() in text.casefold():
            errors.append(f"{path}: forbidden illustrative value phrase {token}")
if errors:
    raise SystemExit("\n".join(errors))
print("static repository checks passed")
````
<!-- markdownlint-restore -->

This simple checker does not parse Markdown fully and its nearby-label heuristic can produce false positives. Pin and
run a dedicated Markdown/link linter in release CI, then manually review external links, anchors, listing IDs,
extraction paths, and language syntax.

## Acceptance matrix

| Gate        | Exact pass evidence                                                       |
| ----------- | ------------------------------------------------------------------------- |
| environment | clean hash-locked install, imports, forward/backward, save/reload         |
| data        | rights/security/PII/lineage/dedup/contradiction/split reports             |
| training    | masks, stable run/resume, immutable checkpoints, quality/safety/retention |
| merge       | exact base; attached/merged logit + greedy parity; clean reload           |
| compression | direct-parent paired quality; target format/kernel; no hidden fallback    |
| export      | tokenizer/template/EOS/context parity; corruption/overflow behavior       |
| quality     | paired intervals, strata, contamination audit, qualified review           |
| safety      | no critical failures under approved policy; refusal utility also passes   |
| systems     | storage/load/peak RAM, failures, cold/warm/sustained metrics              |
| mobile      | physical cohort, offline, lifecycle, update, rollback, thermal/energy     |
| AWS         | auth/rate/load/scale/privacy/alarms/canary/rollback/dated cost            |
| release     | signed canonical manifest, hashes, notices/model card, prior rollback     |

If a counter is unavailable, record `null` and a reason. If evidence is missing, the decision is `NOT QUALIFIED`; never
fill it with a planning estimate.

## Final validation commands

```powershell
# Verification note: not executed here
python scripts/validate_repository.py .
python -m compileall src scripts
python -m pytest -q
python -m pip check
git diff --check
```

Run environment-specific formatters, type checkers, JSON/YAML schema fixtures, Markdown lint/link checking, Android
unit/instrumented tests, iOS unit/UI tests, container scan, artifact signature verification, and the full
evaluation/benchmark protocols. Preserve command output and exit codes.

## Completion record

The diagnosis method, failure record, data/environment/training/compression/mobile/AWS fault trees, safe rollback,
static validator, acceptance matrix, and final commands are authored. No fault injection, test, lint, link check, build,
rollback drill, or target validation was executed by these listings.
