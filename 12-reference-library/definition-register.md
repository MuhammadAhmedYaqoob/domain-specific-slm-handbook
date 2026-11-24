# Normative definition register

This register resolves terms whose ambiguity can invalidate comparisons. These definitions are normative for the
repository. A chapter may introduce a more specific local term, but must not silently redefine one below.

Read each distinction as a reporting rule, not merely vocabulary. The worked examples at the end show how an imprecise
claim becomes a testable statement.

## Model identity and openness

### Model family, checkpoint, and artifact

- **Model family:** related architectures/releases under one name. It is not a loadable or licensable unit.
- **Checkpoint:** exact parameter and configuration state identified by owner, repository, identifier, and immutable
  revision.
- **Artifact:** serialized file or package derived from a checkpoint, identified by format, transformation history, and
  digest.
- **Rule:** never transfer a license, context limit, benchmark, tokenizer, or runtime result from a family to an
  unchecked checkpoint or artifact.

### Base, pretrained, instruct, chat, and reasoning

- **Base/pretrained:** primarily next-token-pretrained checkpoint without a documented instruction/chat post-training
  stage.
- **Instruct/chat:** checkpoint post-trained for instruction or conversation behavior and requiring its exact template.
- **Reasoning:** developer label for a post-training behavior or evaluation emphasis; it does not prove reliable
  reasoning.
- **Rule:** these labels describe exact checkpoints. An instruct checkpoint is not automatically a suitable clean base
  for controlled DAPT.

### Open-weight, open-source, and reproducible

- **Open-weight:** weights are downloadable under stated terms.
- **Open-source:** satisfies the applicable open-source definition and license for use, study, modification, and
  redistribution of the relevant preferred form.
- **Reproducible model:** sufficient data provenance, code, configuration, environment, and training evidence exist to
  recreate the declared result.
- **Rule:** none implies either of the other two. Commercial permission is a separate exact-license question.

### LLM and SLM

- **LLM:** model large for the declared engineering context.
- **SLM:** model intentionally compact for declared memory, latency, energy, cost, or update constraints.
- **Rule:** do not claim a universal parameter boundary. Always provide parameter count, precision, architecture,
  context, runtime, and target.

## Knowledge and adaptation

### Pretraining, CPT, DAPT, TAPT, SFT, and post-training

- **Pretraining:** initial broad self-supervised optimization.
- **CPT:** any additional self-supervised pretraining from a checkpoint.
- **DAPT:** CPT on a domain distribution.
- **TAPT:** CPT on text close to a target task.
- **SFT:** supervised optimization on explicit target outputs.
- **Post-training:** umbrella for SFT, preference optimization, RL, safety, and related stages after pretraining.
- **Rule:** DAPT and SFT datasets/objectives are not interchangeable. SFT does not automatically provide comprehensive
  domain knowledge.

### Domain knowledge in parameters

Means behavior learned into model weights or trained adapters. It does not mean exact storage, current truth,
provenance-aware lookup, or guaranteed recall. Without runtime retrieval, stale and conflicting facts remain release
risks.

### Fine-tuning versus adaptation

**Adaptation** is the broad process. **Fine-tuning** means gradient updates to declared trainable parameters. Prompting,
runtime configuration, and model selection are adaptation choices but not fine-tuning.

### LoRA, QLoRA, and deployment quantization

- **LoRA:** low-rank additive parameter update.
- **QLoRA:** LoRA training through a frozen quantized base.
- **Deployment quantization:** final artifact encoding and executable kernels.
- **Rule:** QLoRA’s NF4 training representation is not evidence of a mobile four-bit artifact. Merge/convert/quantize
  and test the final target format.

### Distillation

Use **response/sequence distillation** for training on teacher text, **logit distillation** for matching teacher
distributions, and **feature distillation** for internal representations. Do not call black-box response SFT
“soft-target distillation.”

## Memory, compute, and performance

### Storage, artifact size, load memory, and peak memory

- **Storage capacity:** available persistent device space.
- **Artifact size:** bytes of the serialized model/package.
- **Loaded-model memory:** memory after loading under a declared runtime state.
- **Peak runtime memory:** maximum measured memory during a declared workload.
- **Rule:** these are never interchangeable. Report host, accelerator, and application scope separately.

### Parameter count, active parameters, and resident parameters

- **Parameter count:** total declared learned scalars.
- **Active parameters:** scalars involved for a token, relevant to sparse MoE.
- **Resident parameters:** weights that must be present in memory.
- **Rule:** active MoE count does not estimate download or resident memory.

### Latency and throughput

- **Latency:** elapsed time for one declared event or request.
- **Throughput:** completed work per unit time under declared batching and concurrency.
- **Rule:** never infer one from the other. State queueing, prompt/output length, warm-up, and percentile/statistic.

### TTFT and first-token latency

Synonyms only when measurement boundaries match. State whether tokenization, queueing, model load, prefill, network
transport, and client rendering are included.

### Prefill and decode

- **Prefill:** prompt processing and initial cache creation.
- **Decode:** sequential generation after prefill.
- **Rule:** report separate tokens/s or latency. A runtime can be fast at one and slow at the other.

### Model fits

- **Fits storage:** artifact can be retained.
- **Fits memory:** declared workload completes without memory failure.
- **Interactive:** meets explicit latency and streaming thresholds.
- **Sustainable:** remains within power and thermal thresholds over the declared duration.
- **Rule:** satisfying an earlier gate does not establish a later gate.

### FLOPs, utilization, and wall time

FLOP estimates describe arithmetic, utilization describes achieved hardware activity, and wall time measures elapsed
duration. None may substitute for the others in cost or performance conclusions.

## Quantization and pruning

### Precision, dtype, quantization algorithm, format, and kernel

- **Precision/dtype:** numerical representation semantics.
- **Quantization algorithm:** procedure choosing codes/scales or transforming weights/activations.
- **Format:** serialized tensor/container layout.
- **Kernel:** executable operation consuming the representation.
- **Rule:** “INT4,” “GPTQ,” or “GGUF” alone is not a complete deployment specification. Record all four plus granularity
  and accumulation.

### Nominal versus effective bits per weight

Nominal width counts low-bit codes. Effective width includes scales, zero-points, outliers, codebooks, padding, and
metadata. Use effective width for storage comparisons.

### Static, dynamic, PTQ, and QAT

- **Static quantization:** ranges/calibration parameters fixed before inference.
- **Dynamic quantization:** some ranges chosen during inference.
- **PTQ:** quantization after training.
- **QAT:** training with target quantization effects.
- **Rule:** “static” is not synonymous with PTQ and “dynamic” is not QAT.

### Structured, semi-structured, and unstructured pruning

- **Unstructured:** arbitrary scalar removals, nominal shapes unchanged.
- **Semi-structured:** constrained patterns such as N:M.
- **Structured:** removal of complete dimensions, channels, heads, neurons, or layers, potentially creating smaller
  dense tensors.
- **Rule:** zero count or checkpoint compression is not latency evidence.

### N:M convention

This repository states N:M as **N retained values per contiguous group of M**, unless a cited source uses another
convention, in which case the source convention must be quoted explicitly.

### Compression ratio

Must name numerator and denominator: serialized bytes, dense parameter count, nonzero count, resident memory, or
arithmetic. A bare “4× compression” is invalid.

## Runtime and platform support

### Framework, runtime, backend, and orchestrator

- **Framework:** model construction/training or general execution system.
- **Runtime:** loads and executes a deployment artifact.
- **Backend/execution provider/delegate:** hardware-targeted implementation.
- **LLM orchestrator:** tokenization, generation, cache, sampling, streaming, and conversation layer over a runtime.
- **Rule:** core runtime support does not imply complete chat-generation support.

### Supported, convertible, loadable, executable, and production-ready

- **Supported:** official documentation names the exact combination.
- **Convertible:** conversion tool produces an artifact without error.
- **Loadable:** runtime accepts the artifact.
- **Executable:** declared test produces numerically and functionally valid output.
- **Production-ready:** quality, performance, lifecycle, security, licensing, observability, and rollback gates pass.
- **Rule:** each is a separate evidence state; earlier states do not imply later ones.

### CPU/GPU/NPU support

Means the exact backend executed the intended graph on named hardware. A selectable backend or published package is not
proof of full graph placement; record partitions and fallbacks.

### Offline, on-device, edge, cloud, and hybrid

- **Offline:** no network needed for inference after artifact installation.
- **On-device:** execution occurs on the user device.
- **Edge:** execution near source/user, possibly on a gateway.
- **Cloud:** execution on remote shared infrastructure.
- **Hybrid:** deliberately combines local and remote execution.
- **Rule:** this project’s primary product is no-runtime-RAG; “hybrid” must not quietly introduce document retrieval.

## Evidence and claims

### Reported, verified, executed, and reproduced

- **PAPER-REPORTED:** original paper states the result.
- **VENDOR-REPORTED:** vendor states the result.
- **DOC-VERIFIED:** official documentation was inspected.
- **EXECUTED:** project ran the declared procedure and retained evidence.
- **REPRODUCED:** project independently recreated a referenced result under a declared protocol.
- **Rule:** these labels describe evidence origin, not a monotonic confidence ladder. Documentation verification is not
  runtime verification.

### Static review and unverified implementation

- **STATICALLY REVIEWED:** inspected without execution.
- **UNVERIFIED IMPLEMENTATION:** plausible code/procedure not run on the declared target.
- **Rule:** neither supports a successful-run claim.

### Estimate, illustration, and planned experiment

- **ESTIMATE:** calculated planning value with assumptions.
- **ILLUSTRATIVE:** educational example or hypothetical visual.
- **PLANNED EXPERIMENT:** protocol awaiting execution.
- **Rule:** none is empirical project evidence.

### Primary, official, secondary, and authoritative

**Primary** means original research; **official** means maintained by the responsible project/vendor/body; **secondary**
interprets other sources; **authoritative** is domain-specific and must be justified. Official vendor material can be
authoritative for API syntax but not independent performance.

### Current and latest

Use **current as of YYYY-MM-DD** with a source. Avoid “latest” unless a time-stamped release query establishes it;
immutable records should use exact versions rather than mutable superlatives.

## Worked examples of confusing distinctions

### Identity and training

- **Family versus checkpoint versus artifact:** “Qwen runs on this phone” is too broad. A valid statement names a Qwen
  checkpoint at a commit, the converted file and hash, the runtime version, and the device.
- **Base versus instruct:** a base checkpoint continuing “How do I…” is not necessarily broken; it was not post-trained
  for conversation. An instruct checkpoint must be tested with its own template.
- **Open-weight versus open-source:** downloadable weights under custom terms are open-weight. Call the system
  open-source only after checking the applicable definition, preferred forms, and licenses.
- **DAPT versus SFT:** next-token training on repair manuals is DAPT; training on mechanic question–answer pairs is SFT.
  The first targets domain text distribution, while the second targets response behavior.
- **Fine-tuning versus adaptation:** LoRA is fine-tuning because gradients update adapter parameters. Selecting a better
  prompt adapts system behavior but does not fine-tune parameters.
- **QLoRA versus deployment quantization:** a four-bit NF4 base used during adapter training does not prove that the
  merged model can execute as a four-bit Android artifact.
- **Distillation types:** training on teacher-written answers is sequence distillation. Matching the teacher's full
  next-token distribution is logit distillation; do not call the first “soft-target” training.

### Memory and performance

- **File size versus runtime memory:** a 700 MB model file may require 1.8 GB at peak after weights, KV cache, workspace,
  and app overhead are included.
- **Total versus active parameters:** an MoE can activate 2B of 8B parameters per token yet still need all 8B resident.
- **Latency versus throughput:** batching may raise tokens per second while making one user's response take longer.
- **TTFT boundaries:** 300 ms measured after tokenization cannot be compared directly with 450 ms measured from a
  user's tap through first rendered text.
- **Prefill versus decode:** 500 prompt tokens/s and 20 generated tokens/s describe different stages; neither should be
  reported simply as “520 tokens/s.”
- **Fits versus interactive:** a model that completes without OOM “fits memory.” It is interactive only after meeting a
  stated TTFT and decode target, and sustainable only after a thermal-duration test.
- **FLOPs versus wall time:** a theoretically lower-operation kernel can still be slower because of memory movement,
  launch overhead, or poor utilization.

### Compression and runtime support

- **Precision, algorithm, format, and kernel:** “INT4 GGUF” omits how values were chosen and which executable kernel
  consumes them. A complete statement names quantizer, exact encoding, granularity, accumulation dtype, and runtime.
- **Nominal versus effective width:** 1B four-bit codes equal 500 MB in theory, but scales and metadata can make the
  artifact 600 MB, or 4.8 effective bits per weight.
- **Static versus PTQ:** ranges fixed before inference are static; applying quantization after training is PTQ. A method
  can be both, but the words answer different questions.
- **Pruning types:** setting arbitrary weights to zero is unstructured; retaining two of every four is semi-structured;
  deleting whole FFN channels is structured.
- **Compression ratio:** “4× smaller” is incomplete. Say “serialized artifact bytes fell from 2.0 GB to 0.5 GB.”
- **Runtime versus orchestrator:** a tensor runtime may execute the graph but not provide tokenization, sampling,
  streaming, stop handling, or chat history.
- **Supported versus production-ready:** documentation can establish support; conversion establishes an artifact;
  execution establishes a run; production readiness additionally requires quality, safety, lifecycle, security, and
  rollback evidence.
- **Accelerator selectable versus used:** choosing “NPU” in an API is not proof of placement. A trace may reveal that an
  unsupported operator fell back to CPU.
- **Offline versus on-device:** inference can execute on the phone but still require a network check. It is on-device,
  but not fully offline under this repository's definition.

### Evidence language

- **Documentation versus execution:** reading an official Android guide supports `DOC-VERIFIED`; retaining a successful
  device run supports `EXECUTED`.
- **Executed versus reproduced:** running a new procedure is executed. Recreating a published result under its declared
  protocol is reproduced.
- **Illustration versus estimate:** a made-up chart that teaches a relationship is illustrative. A calculated memory
  budget with explicit assumptions is an estimate.
- **Official versus authoritative:** a vendor page is authoritative for its API syntax, but its speed claim remains a
  vendor claim rather than independent performance evidence.
- **Current versus latest:** “v2.4.1 was current on 2026-11-24 according to the release page” is auditable; “use the
  latest runtime” is not.

Cross-links: [glossary](glossary.md) · [abbreviations](abbreviations.md) ·
[evidence standard](../00-governance/evidence-and-citation-standard.md)
