# Training implementation

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

## Concept, purpose, prerequisites, and outputs

Training changes model behavior by optimizing a declared objective; it does not automatically create comprehensive or safe expertise. The practical goal is to isolate what each update contributes: CPT changes next-token modeling on domain text, SFT teaches target responses, LoRA limits trainable weights, QLoRA lowers training memory, and preference optimization changes relative response likelihoods.

**Prerequisites:** a qualified environment, immutable base model and tokenizer, rendered train/validation records, a sealed evaluation suite, sufficient compute, and approved stop criteria. **Inputs:** `configs/train.yaml`, model artifacts, dataset manifests, and seed. **Outputs:** checkpoints or adapters, tokenizer/template files, trainer state, run record, logs, hashes, and evaluation-ready artifacts.

### Worked example

The example run `auto-lora-r1729-20261124` trains rank-16 LoRA adapters with an effective batch of 16 from microbatch 1 and accumulation 16. First run `--smoke` for two steps, inspect active assistant labels, confirm finite loss and nonzero adapter gradients, save, reload, and resume. Expected behavior is that only declared adapter parameters update. Validation compares trainable names and counts, performs a two-batch overfit test, checks attached-versus-reloaded logits, and rejects empty masks, non-finite values, missing lineage, or sealed-evaluation leakage.

## Contract and decision order

> Verification note: commands and listings require execution in the stated target environment before operational use.

Recommended order:

1. Evaluate the exact base and, separately, its official instruction checkpoint.
2. Run supervised fine-tuning (SFT) on the base with LoRA as the first adaptation baseline.
3. Compare full fine-tuning and QLoRA under the same token budget if resources justify them.
4. Run continued pretraining (CPT) only when held-out domain evidence identifies a broad distribution/knowledge gap;
   compare `base→SFT` against `base→CPT→same SFT`.
5. Run preference optimization only when the SFT model already produces correct candidates and a measured ranking/policy
   problem remains.
6. Distill only to meet an architecture/deployment target; see
   [compression implementation](compression-implementation.md).

Canonical explanations:
[CH-05 CPT](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md),
[CH-06 SFT](../02-textbook/part-02-training-and-domain-adaptation/ch-06-supervised-fine-tuning.md),
[CH-07 preference](../02-textbook/part-02-training-and-domain-adaptation/ch-07-preference-optimization-and-post-training.md),
and [CH-09 LoRA](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md).

## Input schemas

CPT JSONL:

```json
{
  "id": "cpt-001",
  "text": "Illustrative rights-approved training sentence.",
  "root_family_id": "source-family-17",
  "authority_tier": "A"
}
```

SFT JSONL:

<!-- markdownlint-disable MD013 -->
```json
{
  "id": "sft-001",
  "domain": "automotive",
  "task": "diagnostic_dialogue",
  "messages": [
    {
      "role": "system",
      "content": "Distinguish evidence from hypotheses."
    },
    {
      "role": "user",
      "content": "A code appeared. Which part should I replace?"
    },
    {
      "role": "assistant",
      "content": "A code identifies a monitored condition, not a failed part. Verify the applicable evidence before replacement."
    }
  ]
}
```
<!-- markdownlint-restore -->

Preference JSONL:

```json
{
  "id": "pref-001",
  "prompt": [
    {
      "role": "user",
      "content": "A code appeared. Replace the named part?"
    }
  ],
  "chosen": [
    {
      "role": "assistant",
      "content": "No conclusion follows from the code alone; verify evidence and the applicable procedure."
    }
  ],
  "rejected": [
    {
      "role": "assistant",
      "content": "Replace it immediately."
    }
  ],
  "rubric_id": "repair-safety-v1",
  "root_family_id": "case-001"
}
```

These are `ILLUSTRATIVE`. Preference means rubric-relative ranking, not truth. Ties and both-unacceptable cases must
remain outside strict pair training unless a documented policy converts them.

## Run configuration

**Listing TRAIN-L01 — extraction path `configs/train.yaml`.**

```yaml
schema: slm-training/v1
status: PLANNED_EXPERIMENT
run_id: "auto-lora-r1729-20261124"
seed: 1729
model:
  repository: "artifacts/base-model"
  revision: "08d7ac0eedd64c9d751783ed8d2178834c21b879"
  tokenizer_revision: "08d7ac0eedd64c9d751783ed8d2178834c21b879"
  trust_remote_code: false
data:
  train: "data/curated/automotive-sft-2026-11-24-v1/train.jsonl"
  validation: "data/curated/automotive-sft-2026-11-24-v1/train.jsonl"
  manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
  excluded_evaluation_manifest_sha256: "a8893ab0ed79847b756e21f77778b638d16c3b3e826d0a52177fd1a53dfbf5ef"
objective:
  mode: "sft-lora" # cpt|sft-full|sft-lora|sft-qlora
  assistant_only_loss: true
  sequence_length: 2048
optimization:
  learning_rate: 0.00002
  epochs: 1.0
  microbatch: 1
  gradient_accumulation: 16
  warmup_ratio: 0.03
  weight_decay: 0.0
  gradient_checkpointing: true
  precision: bf16
  eval_steps: 100
  save_steps: 100
  logging_steps: 10
lora:
  rank: 16
  alpha: 32
  dropout: 0.05
  targets: all-linear
resume_from_checkpoint: null
output: "artifacts/checkpoints/auto-lora-r1729-20261124"
```

All numeric hyperparameters are starting hypotheses, not recommendations. CPT generally needs its own conservative
learning-rate/token-budget sweep rather than copying SFT values.

## Unified CPT/SFT/full/LoRA/QLoRA runner

**Listing TRAIN-L02 — extraction path `src/slm_impl/train/run.py`.**

```python
from __future__ import annotations
import argparse, hashlib, json, os, platform, subprocess
from pathlib import Path
import torch, yaml
from datasets import load_dataset
from peft import LoraConfig, TaskType, get_peft_model, prepare_model_for_kbit_training
from transformers import (
    AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig,
    DataCollatorForLanguageModeling, Trainer, TrainingArguments, set_seed)
from trl import SFTConfig, SFTTrainer

def file_hash(path: Path) -> str:
    h=hashlib.sha256()
    with path.open("rb") as f:
        for block in iter(lambda:f.read(1<<20),b""): h.update(block)
    return h.hexdigest()

def git_commit() -> str:
    return subprocess.run(["git","rev-parse","HEAD"],check=True,
                          capture_output=True,text=True).stdout.strip()

def main() -> None:
    p=argparse.ArgumentParser()
    p.add_argument("--config",type=Path,required=True)
    p.add_argument("--smoke",action="store_true")
    a=p.parse_args(); cfg=yaml.safe_load(a.config.read_text("utf-8"))
    mode=cfg["objective"]["mode"]; seed=int(cfg["seed"]); set_seed(seed)
    model_cfg=cfg["model"]; revision=model_cfg["revision"]
    if not cfg["model"]["repository"] or not cfg["data"]["train"]:
        raise RuntimeError("model repository and training data are required")
    if file_hash(Path(cfg["data"]["train"])) == file_hash(Path(cfg["data"]["validation"])):
        raise RuntimeError("train and validation files are identical")
    tok=AutoTokenizer.from_pretrained(
        model_cfg["repository"],revision=model_cfg["tokenizer_revision"],
        trust_remote_code=False)
    if tok.pad_token_id is None:
        if tok.eos_token_id is None: raise RuntimeError("padding policy unresolved")
        tok.pad_token=tok.eos_token
    qlora=mode=="sft-qlora"
    quant=BitsAndBytesConfig(
        load_in_4bit=True,bnb_4bit_quant_type="nf4",
        bnb_4bit_use_double_quant=True,
        bnb_4bit_compute_dtype=torch.bfloat16) if qlora else None
    dtype={"bf16":torch.bfloat16,"fp16":torch.float16,
           "fp32":torch.float32}[cfg["optimization"]["precision"]]
    model=AutoModelForCausalLM.from_pretrained(
        model_cfg["repository"],revision=revision,trust_remote_code=False,
        torch_dtype=dtype,quantization_config=quant)
    ds=load_dataset("json",data_files={
        "train":cfg["data"]["train"],"validation":cfg["data"]["validation"]})
    opt=cfg["optimization"]; max_steps=2 if a.smoke else -1
    common=dict(
        output_dir=cfg["output"],learning_rate=float(opt["learning_rate"]),
        num_train_epochs=float(opt["epochs"]),max_steps=max_steps,
        per_device_train_batch_size=int(opt["microbatch"]),
        per_device_eval_batch_size=int(opt["microbatch"]),
        gradient_accumulation_steps=int(opt["gradient_accumulation"]),
        warmup_ratio=float(opt["warmup_ratio"]),weight_decay=float(opt["weight_decay"]),
        eval_strategy="steps",eval_steps=int(opt["eval_steps"]),
        save_steps=int(opt["save_steps"]),logging_steps=int(opt["logging_steps"]),
        save_total_limit=3,gradient_checkpointing=bool(opt["gradient_checkpointing"]),
        bf16=dtype==torch.bfloat16,fp16=dtype==torch.float16,
        report_to="none",seed=seed,data_seed=seed)
    if mode=="cpt":
        def tokenize(batch):
            return tok(batch["text"],truncation=True,
                       max_length=int(cfg["objective"]["sequence_length"]))
        tokenized=ds.map(tokenize,batched=True,
                         remove_columns=ds["train"].column_names)
        trainer=Trainer(model=model,args=TrainingArguments(**common),
            train_dataset=tokenized["train"],eval_dataset=tokenized["validation"],
            data_collator=DataCollatorForLanguageModeling(tokenizer=tok,mlm=False))
    else:
        if tok.chat_template is None: raise RuntimeError("chat template missing")
        peft_config=None
        if mode in {"sft-lora","sft-qlora"}:
            if qlora: model=prepare_model_for_kbit_training(model)
            l=cfg["lora"]
            peft_config=LoraConfig(
                task_type=TaskType.CAUSAL_LM,r=int(l["rank"]),
                lora_alpha=int(l["alpha"]),lora_dropout=float(l["dropout"]),
                target_modules=l["targets"],bias="none")
        trainer=SFTTrainer(
            model=model,processing_class=tok,peft_config=peft_config,
            train_dataset=ds["train"],eval_dataset=ds["validation"],
            args=SFTConfig(**common,max_length=int(cfg["objective"]["sequence_length"]),
                           assistant_only_loss=bool(cfg["objective"]["assistant_only_loss"])))
    resume=cfg.get("resume_from_checkpoint")
    trainer.train(resume_from_checkpoint=resume or None)
    out=Path(cfg["output"]); trainer.save_model(out); tok.save_pretrained(out)
    record={"status":"EXECUTED_REQUIRES_REVIEW","mode":mode,"seed":seed,
      "model":model_cfg,"data_manifest_sha256":cfg["data"]["manifest_sha256"],
      "config_sha256":file_hash(a.config),"git_commit":git_commit(),
      "python":platform.python_version(),"torch":torch.__version__,
      "cuda":torch.version.cuda,"hardware":torch.cuda.get_device_name(0)
        if torch.cuda.is_available() else "cpu",
      "resume_parent":resume}
    (out/"run_record.json").write_text(json.dumps(record,indent=2,sort_keys=True),"utf-8")
if __name__=="__main__": main()
```

Before using assistant-only loss, inspect the exact token mask. Current template/API behavior is version-sensitive.

**Listing TRAIN-L03 — extraction path `src/slm_impl/train/inspect_mask.py`.**

```python
import argparse, json
from transformers import AutoTokenizer
p=argparse.ArgumentParser()
p.add_argument("--model",required=True); p.add_argument("--revision",required=True)
p.add_argument("--record",required=True); a=p.parse_args()
tok=AutoTokenizer.from_pretrained(a.model,revision=a.revision,trust_remote_code=False)
messages=json.load(open(a.record,encoding="utf-8"))["messages"]
encoded=tok.apply_chat_template(messages,tokenize=True,add_generation_prompt=False,
    return_dict=True,return_assistant_tokens_mask=True)
mask=encoded.get("assistant_masks")
if mask is None or len(mask)!=len(encoded["input_ids"]) or not any(mask):
    raise RuntimeError("no trustworthy assistant mask")
for token_id,active in zip(encoded["input_ids"],mask):
    print(int(bool(active)),token_id,
          repr(tok.decode([token_id],skip_special_tokens=False)),sep="\t")
```

Manually verify system/user tokens are masked, every intended assistant turn and chosen end token is active, and
truncation preserves prompt plus target.

## Full fine-tuning, LoRA, and QLoRA comparison

Use identical parent revision, rendered records, supervised-token budget, maximum length, effective batch, evaluation,
seed schedule, and checkpoint rule. Give each method an equal search budget over method-appropriate learning rates.
Record:

- exact trainable names/counts and adapter targets;
- peak allocated/reserved GPU and host memory;
- supervised tokens/s, wall time, failed runs, and checkpoint bytes;
- domain, safety, retention, and memorization results;
- adapter-attached, merged, and release-artifact parity.

QLoRA's NormalFloat 4 training representation is not a mobile serving format. Save the adapter and merge it into a fresh
exact high-precision base through a supported path before target-specific quantization.

```bash
# Verification note: not executed here
python -m slm_impl.train.run --config configs/train.yaml --smoke
accelerate launch -m slm_impl.train.run --config configs/train.yaml
```

For distributed full fine-tuning, generate an Accelerate configuration in the frozen environment and archive it. Use
Fully Sharded Data Parallel or DeepSpeed only after a single-device smoke test and explicit resume test. Do not infer
global effective batch from configuration alone; log realized supervised tokens per optimizer step.

## CPT workflow

CPT uses raw `text` and predicts every valid next token. It does not teach chat policy. Required arms:

```text
base → same SFT
base → domain CPT → same SFT
base → domain+general replay CPT → same SFT
optional base → task-adaptive pretraining → same SFT
```

Hold CPT update tokens constant across mixture arms. Save by processed tokens, not just epochs. Log realized token
contribution by source. Select on domain validation plus general retention, instruction behavior after the same SFT,
safety, privacy/memorization, and contamination; lower CPT loss alone cannot promote a checkpoint. “Skip CPT” is a valid
result.

Production CPT needs reviewed packing with document end boundaries and deterministic multi-source sampling; TRAIN-L02
intentionally uses truncation and is only a baseline.

## Preference optimization

Start with chosen-only SFT and Direct Preference Optimization (DPO). Add IPO, KTO, ORPO, SimPO, RLOO, GRPO, or PPO only
for a specific data/reward/resource hypothesis documented in
[CH-07](../02-textbook/part-02-training-and-domain-adaptation/ch-07-preference-optimization-and-post-training.md).

**Listing TRAIN-L04 — extraction path `src/slm_impl/train/preference.py`.**

```python
from __future__ import annotations
import argparse
from pathlib import Path
from datasets import load_dataset
from transformers import AutoModelForCausalLM, AutoTokenizer, set_seed
from trl import DPOConfig, DPOTrainer

p=argparse.ArgumentParser()
p.add_argument("--model",required=True); p.add_argument("--revision",required=True)
p.add_argument("--train",type=Path,required=True)
p.add_argument("--validation",type=Path,required=True)
p.add_argument("--out",type=Path,required=True); p.add_argument("--seed",type=int,default=1729)
a=p.parse_args(); set_seed(a.seed)
tok=AutoTokenizer.from_pretrained(a.model,revision=a.revision,trust_remote_code=False)
model=AutoModelForCausalLM.from_pretrained(a.model,revision=a.revision,
                                           trust_remote_code=False)
data=load_dataset("json",data_files={"train":str(a.train),"validation":str(a.validation)})
config=DPOConfig(output_dir=str(a.out),beta=0.1,learning_rate=5e-7,
    per_device_train_batch_size=1,gradient_accumulation_steps=16,
    max_length=2048,max_prompt_length=1024,loss_type="sigmoid",
    eval_strategy="steps",eval_steps=100,save_steps=100,report_to="none",seed=a.seed)
trainer=DPOTrainer(model=model,ref_model=None,args=config,
    train_dataset=data["train"],eval_dataset=data["validation"],
    processing_class=tok)
trainer.train(); trainer.save_model()
```

The values are experimental. Verify whether the frozen TRL version derives a frozen reference when `ref_model=None`;
inspect formatted pairs, truncation, and loss against a hand calculation. Track chosen and rejected log probabilities
separately. A falling DPO loss can arise from suppressing rejected responses while chosen quality declines.

Online methods require fresh generations and a trustworthy reward. A JSON-schema reward proves syntax, not
automotive/pharmacy correctness. Do not use a weak model judge as the only reward or release evaluator.

## Checkpointing, resumption, and recovery

Every checkpoint directory must contain model/adapter, tokenizer/template, trainer state, optimizer/scheduler/scaler
state where applicable, run record, config, data manifest hash, lock hash, and checksums. Test:

1. uninterrupted four-step control;
2. two steps, process termination, resume to four;
3. compare optimizer step, scheduler, sampled IDs, and resulting metrics;
4. reload final artifact in a clean process;
5. resume from a copy after an interrupted checkpoint write; corrupt copies must fail.

Bitwise identity is not guaranteed across every distributed GPU path. State the expected determinism level and compare
declared tolerances plus task outputs.

## Validation and stop conditions

Preflight:

- exact immutable model/tokenizer/template and license;
- schema, rights, expert review, root splits, duplicate/leakage audit;
- dataset distributions by rows and active target tokens;
- template/mask/EOS/padding/truncation/packing inspection;
- two-batch overfit, save/reload, and resume smoke tests;
- memory pilot at intended sequence length;
- sealed evaluation excluded from training, calibration, teacher, and preference judging.

Stop or reject on non-finite loss/logits, repeated loss spikes, empty masks, exploding gradient/adapter norms, domain or
safety regression, general forgetting beyond margin, memorization/privacy failure, malformed generations, resume
divergence, export incompatibility, or absent lineage.

Common diagnoses:

| Symptom                     | Likely cause                        | Exact first check                         |
| --------------------------- | ----------------------------------- | ----------------------------------------- |
| zero loss                   | all labels `-100`                   | count active labels per batch             |
| NaN                         | precision/overflow/bad data         | replay failing batch in FP32              |
| role markers emitted        | template mismatch                   | decode rendered training IDs              |
| QLoRA OOM                   | activations/length/batch            | measure one full optimizer step           |
| no LoRA learning            | wrong targets/frozen adapters       | list trainable names and gradients        |
| CPT harms chat              | raw continuation overwrote behavior | matched recovery SFT and regression suite |
| great validation            | root/semantic leakage               | ancestry and near-duplicate audit         |
| preference refusal collapse | rubric imbalance                    | benign/unsafe refusal confusion matrix    |

## Completion record

The runner, configuration, dataset contracts, masks, CPT/SFT/full/LoRA/QLoRA/preference procedures, distributed/resume
strategy, commands, metrics, failures, and validation gates are authored. No package API, training run, checkpoint, or
hardware result was executed; all implementation and comparative claims remain unverified.
