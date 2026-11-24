# Implementation handbook

## Handbook outcome

This handbook turns a rights-approved domain corpus into traceable training snapshots, adapted checkpoints, target-specific compressed artifacts, evaluated releases, and rollback-capable AWS or offline edge deployments. Each chapter follows one operating pattern: understand the engineering decision, prepare explicit inputs, execute complete listings, inspect outputs, diagnose failures, and pass named validation gates.

### Worked route

The running example uses an automotive SFT snapshot named `automotive-sft-2026-11-24-v1`, seed `1729`, a locally approved base model under `artifacts/base-model`, LoRA adaptation, a merged high-precision parent, and separate cloud and GGUF children. Names and arithmetic are illustrative; benchmark, price, and safety conclusions must come from the reader's governed measurements.

## Status and use

> Verification note: commands and listings require execution in the stated target environment before operational use.

This is the practical companion to the [canonical textbook](../02-textbook/README.md). It deliberately cross-links
conceptual and research treatment instead of duplicating it. Listings remain embedded in Markdown to respect the
repository's all-Markdown stage. An operator extracts each listing to its stated path under a new project root, resolves
illustrative values, pins immutable revisions, and records execution evidence before changing its label.

## Recommended route

1. [Environment and dependencies](environment-and-dependencies.md) — choose a supported lane, resolve the candidate
   lock, record hardware and toolchains.
2. [Data pipeline](data-pipeline-implementation.md) — create rights-reviewed immutable objective snapshots.
3. [Training](training-implementation.md) — establish baseline, then run CPT, SFT, full fine-tuning, LoRA, QLoRA, and
   justified preference experiments.
4. [Compression](compression-implementation.md) — merge, distill, prune, quantize, and retain stage lineage.
5. [Evaluation](evaluation-implementation.md) — generate immutable outputs, score quality/safety, and benchmark systems.
6. [Export and deployment](export-and-deployment-implementation.md) — produce separate AWS, Android, iOS, and ARM/edge
   artifacts.
7. [Troubleshooting and validation](troubleshooting-and-validation.md) — diagnose failures and run release acceptance.
8. [Learning tools](learning-tools/README.md) — extract eleven formula, measurement, visualization, cost, and placement
   CLIs; all remain unexecuted.

The governed [listing manifest](listing-manifest.md) maps `TOOL-01..TOOL-11` to `LST-TOOL-0001..0011`, intended
extraction paths, and current statuses.

The shortest credible prototype is:

```text
rights-approved SFT data → LoRA SFT → merge into clean base
→ GGUF Q4/Q5 edge candidate + BF16/cloud candidate
→ paired domain/safety evaluation → target benchmark → signed manifest
```

CPT, preference optimization, pruning, distillation, QAT, NPU export, and distributed training are conditional branches,
not mandatory badges.

## Extraction root and conventions

All paths in this handbook are relative to `slm-implementation/`:

```text
slm-implementation/
├── requirements/{train.in,train.lock,data.in,data.lock,eval.in,eval.lock}
├── configs/{project,data,train,compression,evaluation,release}.yaml
├── schemas/{source,sft,preference,evaluation,result,release}.schema.json
├── src/slm_impl/{data,train,compress,evaluate,package}.py
├── deploy/{aws,android,ios,edge}/
├── data/{raw,receipts,quarantine,curated,calibration,evaluation}/
├── artifacts/{checkpoints,merged,exports,releases}/
└── results/{runs,raw,aggregates,validation}/
```

Examples use concrete tutorial names so that readers can follow complete data
flow. Before execution, derive hashes and platform identities from the actual
files and tools; never copy an illustrative identity into evidence. Do not use
mutable tags such as `main`, `latest`, or an unpinned model ID.

Every executed run must preserve:

- model, tokenizer, template, dataset, code, lockfile, container, converter, and runtime hashes;
- exact command, seed, hardware, driver, operating system, precision, and failure logs;
- parent/child artifact lineage and rollback target;
- raw item-level quality outputs and raw benchmark trials;
- evidence label and reviewer decision.

## Safety and architecture boundaries

The primary inference product does not use vector databases, embedding search, dynamic document retrieval, retrieval
prompt augmentation, or a remote teacher. Offline sources may support data preparation and evaluation. Learned
parameters have imperfect recall, can become stale, and can hallucinate; no training stage grants professional
authority.

Pharmacy artifacts must not autonomously prescribe or change treatment. Repair artifacts must not authorize hazardous
work, bypass interlocks, or invent model-specific procedures. Critical releases require qualified domain review,
explicit escalation behavior, incident response, and non-compensatory safety gates. See the
[safety standard](../00-governance/safety-and-risk-standard.md) and
[no-runtime-RAG standard](../00-governance/no-runtime-rag-standard.md).

## Evidence promotion

| Label                       | Minimum requirement                                                             |
| --------------------------- | ------------------------------------------------------------------------------- |
| `UNVERIFIED IMPLEMENTATION` | concrete listing, extraction path, prerequisites, expected failures, validation |
| `STATICALLY REVIEWED`       | independent source/code reasoning without execution                             |
| `EXECUTED`                  | frozen environment, command, logs, outputs, failures, and hashes                |
| `REPRODUCED`                | executed protocol reproduces the defined external or prior result               |
| `PLANNED EXPERIMENT`        | no empirical result exists                                                      |

`DOC-VERIFIED` describes documentation, not runtime compatibility. `PAPER-REPORTED` describes authors' findings, not
this project.

## Canonical references

- [Bookmap](../BOOKMAP.md)
- [Governance](../00-governance/README.md)
- [Flagship automotive extraction blueprint (CH-37)](../02-textbook/part-10-complete-hands-on-project/ch-37-build-an-automotive-mechanic-slm-end-to-end.md)
- [Reusable platform architecture (CH-28)](../02-textbook/part-07-multi-domain-slm-engineering/ch-28-reusable-domain-training-platform.md)
- [Production architecture (CH-47)](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md)

## Honest completion record

The handbook is substantively authored and navigable. It contains extraction paths, candidate dependencies, schemas,
commands, implementation listings, hardware prerequisites, failure modes, and exact validation procedures. It remains an
unexecuted implementation specification: locks, APIs, checkpoint licenses, runtime support, AWS resources, mobile
builds, and physical-device behavior must be verified at execution time.
