# Compression implementation

## Concept, purpose, prerequisites, and outputs

Compression trades representation size or computation for deployability. A smaller file is not evidence of lower runtime memory or faster generation: the target kernel may unpack weights, fall back to dense execution, or allocate a dominant KV cache. Every transformation is therefore a new child artifact evaluated against its direct parent.

**Prerequisites:** a qualified high-precision checkpoint, exact tokenizer/template, target runtime support, calibration and recovery snapshots that exclude evaluation roots, and release gates. **Inputs:** parent manifest, operation configuration, tool revision, and target profile. **Outputs:** merged, quantized, pruned, or distilled children with lineage, conversion logs, tensor audits, hashes, and paired evaluation evidence.

### Worked example

Merge the automotive LoRA adapter into a fresh high-precision base, compare attached and merged logits, then independently create Q8 and Q4 GGUF children. Expected behavior is clean loading and bounded parity before quantization, followed by target-specific quality and systems measurements. Validate hashes, tokenizer IDs, fixed greedy outputs, full domain/safety suites, memory phases, TTFT, sustained decode, and executed kernels. Reject a 4-bit child that is smaller but slower or that silently falls back.

## Contract and practical funnel

> Verification note: commands and listings require execution in the stated target environment before operational use.

Use this funnel:

```text
qualified domain checkpoint
  → merge adapter into exact clean high-precision base
  → runtime-native INT8 and one supported 4-bit PTQ baseline
  → stop if deployment budgets pass
  → otherwise compare native-small vs distilled vs structured-pruned dense
  → recover and qualify high-precision child
  → freshly quantize child for each runtime
  → physical/AWS benchmark and target-specific Pareto selection
```

Do not blindly stack methods. Quantization, pruning, distillation, and export are noncommutative. Preserve every parent
and run every hard gate after each transformation. Canonical treatment:
[CH-11 quantization](../02-textbook/part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md),
[CH-12 pruning](../02-textbook/part-04-model-compression-and-optimization/ch-12-pruning-complete-research-and-implementation.md),
[CH-13 distillation](../02-textbook/part-04-model-compression-and-optimization/ch-13-knowledge-distillation-complete-research-and-implementation.md),
and
[CH-15 combined pipelines](../02-textbook/part-04-model-compression-and-optimization/ch-15-combining-optimization-techniques.md).

## Artifact lineage

**Listing COMP-L01 — extraction path `configs/compression.yaml`.**

```yaml
schema: slm-compression/v1
status: PLANNED_EXPERIMENT
source:
  artifact: "artifacts/merged/auto-lora-r1729-20261124"
  artifact_manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  tokenizer_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  chat_template_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
datasets:
  recovery_manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  quantization_calibration_manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  pruning_calibration_manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  sealed_evaluation_manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
stages:
  - { id: merged-fp, operation: merge_clean_base, parent: source }
  - { id: q8, operation: runtime_native_int8, parent: merged-fp }
  - { id: q4, operation: runtime_native_4bit, parent: merged-fp }
  - { id: student-fp, enabled: false, operation: response_distillation, parent: merged-fp }
  - { id: structured-fp, enabled: false, operation: structured_prune_recover, parent: merged-fp }
  - { id: final-q4, operation: runtime_native_4bit, parent: "merged-fp" }
gates:
  critical_safety_failures_max: 0
  domain_score_min: "0.80"
  general_retention_min: "0.80"
  peak_memory_bytes_max: "2147483648"
  ttft_p95_ms_max: "2147483648"
  decode_tokens_s_min: "2147483648"
  operator_fallback_allowed: false
```

Every child manifest records parent ID/hash, operation/config, tool commit, environment/container, command,
calibration/recovery data, conversion logs, tensor types/shapes, artifact files/hashes, evaluation IDs, failures, and
rollback ID.

## Merge before ordinary deployment PTQ

**Listing COMP-L02 — extraction path `src/slm_impl/compress/merge_lora.py`.**

```python
from __future__ import annotations
import argparse, hashlib, json
from pathlib import Path
import torch
from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer

def tree_hash(root: Path) -> dict[str,str]:
    out={}
    for path in sorted(p for p in root.rglob("*") if p.is_file()):
        out[path.relative_to(root).as_posix()]=hashlib.sha256(path.read_bytes()).hexdigest()
    return out

p=argparse.ArgumentParser()
p.add_argument("--base",required=True); p.add_argument("--revision",required=True)
p.add_argument("--adapter",type=Path,required=True); p.add_argument("--out",type=Path,required=True)
a=p.parse_args()
tok=AutoTokenizer.from_pretrained(a.base,revision=a.revision,trust_remote_code=False)
base=AutoModelForCausalLM.from_pretrained(
    a.base,revision=a.revision,torch_dtype=torch.bfloat16,trust_remote_code=False).eval()
attached=PeftModel.from_pretrained(base,a.adapter).eval()
batch=tok(["A fixed non-sensitive parity prompt."],return_tensors="pt")
batch={k:v.to(attached.device) for k,v in batch.items()}
with torch.inference_mode(): before=attached(**batch).logits.float().cpu()
merged=attached.merge_and_unload(safe_merge=True).eval()
with torch.inference_mode(): after=merged(**batch).logits.float().cpu()
max_abs=float((before-after).abs().max())
if not torch.allclose(before,after,rtol=1e-3,atol=1e-3):
    raise RuntimeError(f"merge mismatch max_abs={max_abs}")
a.out.mkdir(parents=True,exist_ok=False)
merged.save_pretrained(a.out,safe_serialization=True); tok.save_pretrained(a.out)
(a.out/"merge_manifest.json").write_text(json.dumps({
    "status":"execution_requires_review","base":a.base,"revision":a.revision,
    "adapter":str(a.adapter),"max_abs_logit_error":max_abs,
    "files":tree_hash(a.out)},indent=2,sort_keys=True),"utf-8")
```

The tolerance is illustrative and must be justified by dtype/model. Compare many fixed prompts, greedy token IDs,
domain/safety metrics, and clean-process reload. For QLoRA, reload the original high-precision base; do not repeatedly
merge/unmerge quantized codes.

## Quantization

Quantize independently for each runtime:

| Target                   | First candidates                                             |
| ------------------------ | ------------------------------------------------------------ |
| AWS GPU low-batch decode | BF16/FP16 plus serving-engine-supported AWQ/GPTQ INT4        |
| AWS throughput/prefill   | BF16 plus runtime-native W8A8/FP8 where hardware supports it |
| Android/ARM/Linux CPU    | exact llama.cpp GGUF Q8/Q5/Q4 types                          |
| Android/iOS accelerator  | runtime/vendor-native INT8/INT4 with partition evidence      |
| Apple portable control   | GGUF CPU/Metal; separate Core ML/MLX/ExecuTorch experiment   |

NF4 is a QLoRA training representation, not a universal mobile format. “INT4” is incomplete without algorithm,
group/block, scales, zero points, exclusions, KV precision, serializer, runtime, and kernel.

**Listing COMP-L03 — extraction path `scripts/build_gguf.sh`.**

```bash
set -euo pipefail
: "${LLAMA_CPP_COMMIT:?required}"
: "${SOURCE_MODEL:?required}"
git clone https://github.com/ggml-org/llama.cpp.git third_party/llama.cpp
git -C third_party/llama.cpp checkout "$LLAMA_CPP_COMMIT"
cmake -S third_party/llama.cpp -B third_party/llama.cpp/build \
  -DGGML_NATIVE=OFF -DCMAKE_BUILD_TYPE=Release
cmake --build third_party/llama.cpp/build --config Release -j
python third_party/llama.cpp/convert_hf_to_gguf.py "$SOURCE_MODEL" \
  --outfile artifacts/exports/model-f16.gguf --outtype f16
third_party/llama.cpp/build/bin/llama-quantize \
  artifacts/exports/model-f16.gguf artifacts/exports/model-q8_0.gguf Q8_0
third_party/llama.cpp/build/bin/llama-quantize \
  artifacts/exports/model-f16.gguf artifacts/exports/model-q4_k_m.gguf Q4_K_M
sha256sum artifacts/exports/*.gguf > artifacts/exports/SHA256SUMS
third_party/llama.cpp/build/bin/llama-cli -m artifacts/exports/model-q4_k_m.gguf \
  -p "Return exactly: READY" -n 8 --seed 1729
```

Confirm available type names with the pinned build. Never requantize a quantized child. Build the same commit for each
target ABI/backend and retain compiler flags.

Calibration must match deployed domains, templates, prompt lengths, symbols/units, multi-turn state, and rare activation
tails while excluding all evaluation roots. Use one common control set for method comparisons and declare
method-specific variants separately.

For GPTQ/AWQ/SmoothQuant/HQQ/AQLM/other methods:

1. pin the official repository or runtime-integrated quantizer;
2. verify exact architecture and producer/consumer compatibility;
3. use a separate locked environment;
4. record group size, activation order, clipping/scaling, exclusions, calibration IDs/tokens, quantization time/peak
   memory, and kernel;
5. retain unsupported cells rather than changing the model;
6. re-run source versus quantized quality and target benchmark.

QAT is attempted only after PTQ is the isolated blocker and the fake-quant recipe exactly matches the deployment
backend's signedness, granularity, groups, accumulators, exclusions, and packing.

## Pruning

Unstructured zeros are a quality experiment unless the final runtime preserves a sparse representation and executes a
compatible kernel. Mobile-first candidates should physically reduce dense layers, multilayer perceptron (MLP) width,
hidden width, or architecture-valid attention/KV groups.

**Listing COMP-L04 — extraction path `src/slm_impl/compress/prune_control.py`.**

```python
from __future__ import annotations
import argparse, json, math
from pathlib import Path
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

EXCLUDED=("embed_tokens","lm_head","norm")

@torch.no_grad()
def dense_zero_control(model,ratio: float) -> dict:
    audit={}
    for name,module in model.named_modules():
        if isinstance(module,torch.nn.Linear) and not any(x in name for x in EXCLUDED):
            n=math.floor(module.weight.numel()*ratio)
            if not n: continue
            idx=torch.topk(module.weight.abs().reshape(-1),n,largest=False).indices
            mask=torch.ones(module.weight.numel(),dtype=torch.bool,device=module.weight.device)
            mask[idx]=False; module.weight.mul_(mask.reshape_as(module.weight))
            audit[name]={"shape":list(module.weight.shape),
                         "zeros":int((module.weight==0).sum()),"elements":module.weight.numel()}
    return audit

p=argparse.ArgumentParser()
p.add_argument("--model",required=True); p.add_argument("--revision",required=True)
p.add_argument("--ratio",type=float,required=True); p.add_argument("--out",type=Path,required=True)
a=p.parse_args()
if not 0<a.ratio<0.8: raise ValueError("ratio outside experiment bound")
model=AutoModelForCausalLM.from_pretrained(
    a.model,revision=a.revision,torch_dtype=torch.float16,trust_remote_code=False)
tok=AutoTokenizer.from_pretrained(a.model,revision=a.revision,trust_remote_code=False)
before=sum(p.numel() for p in model.parameters()); audit=dense_zero_control(model,a.ratio)
after=sum(p.numel() for p in model.parameters())
if before!=after: raise RuntimeError("dense-zero control changed logical parameters")
a.out.mkdir(parents=True,exist_ok=False); model.save_pretrained(a.out,safe_serialization=True)
tok.save_pretrained(a.out)
(a.out/"pruning_audit.json").write_text(json.dumps({
    "status":"execution_requires_review","kind":"dense_zero_no_speed_claim",
    "requested_ratio":a.ratio,"parameters_before":before,"parameters_after":after,
    "details":audit},indent=2),"utf-8")
```

This control intentionally does not create sparse storage or speed. For SparseGPT, Wanda, LLM-Pruner, SliceGPT,
ShortGPT, or FLAP, pin the official repository, isolate dependencies, save masks/retained indices and calibration IDs,
and use the common evaluation harness.

Structured surgery must preserve:

- gated MLP coupling: gate/up rows and down columns use the same retained indices;
- grouped-query/multi-query attention head grouping, rotary layout, and divisibility;
- residual width across every connected projection and normalization;
- tied input/output embeddings;
- configuration, tensor-parallel alignment, and exporter constraints.

Run `no recovery`, compute-matched LoRA recovery, full recovery, and optionally teacher-guided recovery. Report
scoring/pruning/recovery compute. Compare against a native dense model at the same measured deployment budget.

## Distillation

Response/sequence distillation is the first practical method because teacher text can be retokenized by the student.
Every teacher output is a candidate derivative, linked to independent evidence and reviewed. Direct logit KD normally
requires identical vocabulary ordering, tokenization, template, and positions.

**Listing COMP-L05 — extraction path `src/slm_impl/compress/kd_loss.py`.**

```python
from dataclasses import dataclass
import torch
import torch.nn.functional as F

@dataclass
class KDConfig:
    temperature: float=2.0
    hard_weight: float=0.5
    soft_weight: float=0.5

def aligned_causal_kd(student,teacher,batch,cfg: KDConfig):
    if cfg.temperature<=0: raise ValueError("temperature must be positive")
    labels=batch["labels"]; valid=labels.ne(-100)
    if not valid.any(): raise ValueError("no supervised tokens")
    s=student(input_ids=batch["input_ids"],attention_mask=batch["attention_mask"]).logits[:,:-1]
    with torch.inference_mode():
        teacher.eval()
        t=teacher(input_ids=batch["input_ids"],attention_mask=batch["attention_mask"]).logits[:,:-1]
    target=labels[:,1:]; mask=valid[:,1:]
    if s.shape!=t.shape: raise ValueError("teacher/student tokens are not aligned")
    hard=F.cross_entropy(s.reshape(-1,s.size(-1)),target.reshape(-1),ignore_index=-100)
    temp=cfg.temperature
    token_kl=F.kl_div(F.log_softmax(s.float()/temp,dim=-1),
                      F.softmax(t.float()/temp,dim=-1),
                      reduction="none").sum(-1)
    soft=token_kl.masked_select(mask).mean()*temp**2
    return cfg.hard_weight*hard+cfg.soft_weight*soft,{"hard":hard.detach(),"soft":soft.detach()}
```

Unit-test identical distributions, vocabulary mismatch, all-masked input, causal shift, temperature, reduction, and
mixed precision before any real model. Full logits are storage-expensive; top-k caches must retain token IDs, values,
all-logit log-sum-exp or residual mass, teacher/tokenizer/template hashes, precision, and random spot-checks.

Required experiment arms:

1. student trained directly on expert data;
2. student trained on independently validated teacher examples;
3. mixed expert and teacher data;
4. aligned soft targets where technically feasible;
5. selected distilled student freshly quantized;
6. same-size native pretrained/adapted student control.

Distillation can transfer teacher errors and does not guarantee speed. Profile the untrained student architecture on
target hardware before spending teacher-generation budget.

## Combined sequence matrix

<!-- markdownlint-disable MD013 -->
| Sequence                  | Correct order                                                  | Stop condition                    |
| ------------------------- | -------------------------------------------------------------- | --------------------------------- |
| LoRA/QLoRA → PTQ          | train adapter → clean merge → parity → fresh target PTQ        | merge mismatch or no target gain  |
| prune → recover → PTQ     | score FP → surgery/pack → recover FP → qualify → calibrate PTQ | unsupported graph or quality gate |
| distill → PTQ             | train/qualify student FP → student calibration → target PTQ    | native-small dominates            |
| distill → prune → recover | qualify student → one moderate pruning arm → recover           | no benefit at first ratio         |
| compression → QAT         | finalize architecture → exact-target fake quant → convert      | PTQ was not isolated blocker      |
| structured prune × PTQ    | 2×2 factorial: neither/prune/PTQ/both                          | combined child dominated          |
| joint prune + KD          | only after sequential controls                                 | instability/export failure        |
<!-- markdownlint-restore -->

Prune before quantizing unless a verified joint method requires another order. Quantization calibration is stale after
pruning, recovery, distillation, or adapter merge.

## Stage runner and validation

**Listing COMP-L06 — extraction path `src/slm_impl/compress/stage.py`.**

```python
from __future__ import annotations
import argparse, hashlib, json, subprocess, time
from pathlib import Path

def digest(path: Path) -> str:
    h=hashlib.sha256()
    with path.open("rb") as f:
        for block in iter(lambda:f.read(1<<20),b""): h.update(block)
    return h.hexdigest()

p=argparse.ArgumentParser()
p.add_argument("--stage",required=True); p.add_argument("--parent-manifest",type=Path,required=True)
p.add_argument("--child-manifest",type=Path,required=True); p.add_argument("--record",type=Path,required=True)
p.add_argument("command",nargs=argparse.REMAINDER); a=p.parse_args()
if not a.command: raise SystemExit("command required after --")
if not a.parent_manifest.is_file(): raise FileNotFoundError(a.parent_manifest)
started=time.time()
proc=subprocess.run(a.command,capture_output=True,text=True,check=False)
row={"stage":a.stage,"status":"EXECUTED_REQUIRES_REVIEW",
     "parent_manifest_sha256":digest(a.parent_manifest),
     "command":a.command,"started":started,"ended":time.time(),
     "returncode":proc.returncode,"stdout":proc.stdout,"stderr":proc.stderr}
a.record.parent.mkdir(parents=True,exist_ok=True)
a.record.write_text(json.dumps(row,indent=2),"utf-8")
if proc.returncode or not a.child_manifest.is_file():
    raise RuntimeError("stage failed; preserve parent and rollback")
```

Production execution must isolate commands, protect logs, validate/sign child manifests, and run gates before
advancement.

After every stage:

1. validate manifest ancestry and hashes;
2. clean-process load and finite-logit smoke test;
3. compare tokenizer/template IDs and fixed greedy outputs;
4. run complete domain, general-retention, uncertainty/refusal, and safety suites;
5. measure artifact/download bytes, post-load and peak memory, conversion time/peak, cold/warm TTFT, prefill, sustained
   decode, failures, energy/thermal where instrumented;
6. prove intended kernel/operator placement;
7. compare direct parent to isolate the stage delta;
8. retain raw trials and rollback artifact.

Never infer speed from parameters, zeros, bits, or file size. Use the
[evaluation implementation](evaluation-implementation.md) and
[CH-31 benchmark contract](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md).

## Completion record

The lineage contract, merge, GGUF PTQ, quantizer integration rules, pruning control/structured constraints, recovery,
response/logit distillation, combined ordering, stage runner, commands, stop conditions, and validation gates are
authored. No model was merged, pruned, distilled, quantized, converted, or benchmarked.
