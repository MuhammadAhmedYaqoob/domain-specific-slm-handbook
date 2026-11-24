# Adaptation and parameter-efficient fine-tuning (PEFT) comparison

**Research freeze:** 2026-11-23  
**Status:** authored synthesis; paper and documentation evidence is attributed, and no training run was executed.

## Core distinction

Adaptation methods solve different problems:

- **Pretraining/CPT/DAPT/TAPT** changes next-token modeling over raw or task-adjacent text and is the strongest
  candidate for broad distributional/domain-language acquisition.
- **Supervised fine-tuning (SFT)** teaches response behavior, task mappings, structure, and conversational policy from
  labeled examples; it does not guarantee comprehensive knowledge acquisition.
- **Preference/reinforcement post-training** changes relative behavior according to preferences or rewards; it is not a
  substitute for authoritative domain data.
- **Parameter-efficient fine-tuning (PEFT)** constrains which trainable state changes. It describes resource allocation,
  not the learning objective.

These axes compose: DAPT can be full-parameter or adapter-based; SFT can use full fine-tuning, LoRA, or QLoRA. Treating
“QLoRA versus DAPT” as one axis is a category error.

## Lesson: choose the learning signal before the update method

The problem is often framed as “Which fine-tuning method should I use?” That combines two decisions. The learning
objective determines what evidence the model sees; the update method determines which parameters can respond. This
distinction arose historically because pretraining, instruction tuning, preference alignment, and
Parameter-Efficient Fine-Tuning (PEFT) were developed to solve different cost and behavior problems.

Imagine an automotive model that recognizes technical prose but answers diagnostic questions poorly. If it lacks
terminology and domain relationships, Continued Pretraining (CPT) or Domain-Adaptive Pretraining (DAPT) on validated
manual-style text may help. If it knows the concepts but gives unstructured replies, Supervised Fine-Tuning (SFT) on
diagnostic dialogues is the direct tool. Low-Rank Adaptation (LoRA) can implement either objective with a frozen
backbone; Quantized LoRA (QLoRA) lowers the frozen base's training-memory footprint. Neither LoRA nor QLoRA defines what
the model learns.

Use SFT first when the gap is answer behavior and labeled examples exist. Add DAPT when controlled evidence shows a
domain-language or knowledge-distribution gap worth the compute and forgetting risk. Use LoRA/QLoRA when full optimizer
state is unaffordable or many domain artifacts must share a base. Avoid preference optimization until a stable SFT
baseline exposes a measurable preference, refusal, or formatting problem.

## Training-objective comparison

<!-- markdownlint-disable MD013 -->
| Method                             | Signal and mechanism                                                 | Best-supported use                                              | Principal risks                                                           | Project priority              |
| ---------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------- | ----------------------------- |
| Train from scratch                 | Next-token learning from initialized weights                         | New architecture/tokenizer and sufficient corpus/compute        | Highest data/compute burden; instability and weak coverage                | Experimental for this project |
| Continued pretraining (CPT)        | Next-token learning on additional text                               | Extend an explicit base checkpoint                              | Forgetting, data contamination, altered calibration                       | Advanced practical            |
| Domain-adaptive pretraining (DAPT) | CPT on domain corpus, often with general replay                      | Domain terminology and discourse before SFT                     | Raw text can teach errors; knowledge gains may not become answer behavior | Advanced practical            |
| Task-adaptive pretraining (TAPT)   | CPT on unlabeled task distribution                                   | Narrow input-distribution alignment                             | Small/repetitive corpus overfit                                           | Conditional                   |
| SFT/instruction tuning             | Cross-entropy on desired completions, with explicit loss mask        | Task format, dialogue, procedures, uncertainty/refusal examples | Memorization, answer-style imitation, template dependence                 | Core practical                |
| Preference optimization            | Pairwise or reward-derived objective after SFT                       | Refine helpfulness, format, refusal, style                      | Reward hacking, preference noise, stability/compute                       | Advanced after SFT            |
| Full fine-tuning                   | Update declared backbone parameters under any objective              | Highest adaptation freedom and upper control                    | Optimizer/gradient memory, full artifact per domain, forgetting           | Core control where feasible   |
| PEFT                               | Freeze backbone; train prompts/adapters/scales/bias/low-rank updates | Reduce trainable state and per-domain artifacts                 | Capacity/targets may underfit; backbone inference size remains            | Core practical                |
<!-- markdownlint-restore -->

All expected-use judgments are `STATICALLY REVIEWED`; effectiveness is `PLANNED EXPERIMENT`.

### What the objective table means

The table progresses from broad, expensive distribution learning to narrow behavior shaping. It should not be read as a
quality ladder. Training from scratch offers maximum freedom but demands the most data and compute. DAPT can acquire
domain regularities but can forget general skills. SFT efficiently teaches observable behavior but may imitate answers
without building broad knowledge. Preference methods refine relative behavior and depend on trustworthy judgments.
Full fine-tuning versus PEFT is then a separate capacity-and-resource choice applied to one of those objectives.

## PEFT mechanism matrix

<!-- markdownlint-disable MD013 -->
| Method                 | Trainable mechanism                                | Inference/deployment effect                        | Main compatibility caveat                         | Maturity/priority                 |
| ---------------------- | -------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------- | --------------------------------- |
| Bottleneck adapters    | Down/nonlinearity/up modules inside blocks         | Extra sequential operations unless fused           | Decoder evidence and runtime support vary         | Established; conditional edge use |
| Prompt tuning          | Learned input embeddings                           | Consumes context; tiny artifact                    | Scale-dependent evidence may not transfer to SLMs | Established; low priority         |
| Prefix tuning          | Learned layer-wise key/value-like prefixes         | Adds effective KV/sequence work                    | Export and cache handling                         | Established; conditional          |
| P-Tuning / v2          | Learned prompt encoder / deep layer prompts        | Context/runtime overhead                           | v1 and v2 differ; implementation-specific         | Established research              |
| IA3                    | Learned activation-rescaling vectors               | May merge into compatible weights                  | Exact target names and quantized merge            | Advanced practical                |
| BitFit                 | Existing biases only                               | No new modules                                     | Many modern decoders have few relevant biases     | Cheap control                     |
| LoRA                   | Additive low-rank $sBA$ updates                  | Mergeable; dense model size unchanged              | Targets, rank, scale, fused modules               | Core practical                    |
| QLoRA                  | LoRA through frozen 4-bit training base            | Training-memory method; deployment format separate | bitsandbytes/hardware and merge path              | Core practical                    |
| AdaLoRA                | Importance-driven dynamic rank allocation          | LoRA-like after finalization                       | Schedules/state add complexity                    | Advanced                          |
| DoRA                   | Train magnitude plus low-rank direction            | Merge preferred; more unmerged overhead            | Quantization/distributed support varies           | Advanced                          |
| rsLoRA                 | $\alpha/\sqrt r$ scaling                         | Same adapter shape                                 | Scaling metadata must be preserved                | Advanced ablation                 |
| LoRA+                  | Different learning rates for A and B               | No inference change                                | Optimizer-group integration                       | Advanced ablation                 |
| PiSSA                  | Principal weight-SVD initialization/residual       | Mergeable after correct conversion                 | SVD cost and residual provenance                  | Advanced                          |
| OLoRA                  | QR/orthonormal initialization and base translation | Mergeable after conversion                         | Base mutation/provenance                          | Experimental                      |
| EVA                    | Activation-SVD initialization/rank allocation      | Standard low-rank shape                            | Representative initialization data required       | Advanced/experimental             |
| LoftQ                  | Quantization-aware low-rank initialization         | Not a generic edge quantizer                       | Narrow safetensors/bitsandbytes workflows         | Advanced QLoRA                    |
| CorDA                  | Context-oriented decomposition                     | Conversion/merge metadata required                 | Covariance/calibration provenance                 | Advanced                          |
| LoRA-FA                | Freeze A, train B                                  | Additive merge                                     | Different capacity at same rank                   | Advanced                          |
| QA-LoRA                | Quantization-aware grouped LoRA integration        | Potential quantized merged artifact                | Implementation/backend specificity                | Experimental here                 |
| aLoRA                  | Invocation-token-gated adapter                     | Not globally mergeable; enables cache reuse        | Runtime and invocation-token coupling             | Specialized server method         |
| Astra, MiCA, NoRA-init | Recent task/weight-informed initializations        | LoRA-like after correct conversion                 | Recent evidence and metadata burden               | Emerging experiments              |
<!-- markdownlint-restore -->

Original papers, official repositories, current PEFT support, and method-specific limitations are cited in
[PEFT methods](peft-methods.md) and
[CH-09](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md).

### What the PEFT table means

Prompt and prefix methods create tiny artifacts but consume context or add cache work. Bottleneck adapters add
sequential modules. LoRA-family methods modify selected linear transformations and can usually be merged, which is why
they are the practical baseline for edge-bound work. Advanced variants should enter only when a baseline exposes the
specific problem they address—rank allocation, scaling, convergence, magnitude adaptation, or quantization-aware
initialization. Compatibility must be demonstrated for exact module names, quantized layers, merge behavior, and
exporter.

## Preference and reinforcement post-training coverage

These methods refine behavior after a competent SFT baseline; they do not replace domain pretraining or factual
validation.

<!-- markdownlint-disable MD013 -->
| Method                     | Core distinction                                                                                            | Project disposition                                                            |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Reward modeling + RLHF/PPO | Learn a reward model from preferences, then optimize policy with clipped on-policy reinforcement learning   | Foundational, expensive; use only when simpler objectives miss a measured need |
| DPO                        | Optimize chosen over rejected responses directly against a reference policy                                 | Core preference baseline if governed pairs exist                               |
| IPO                        | Uses an identity-preference objective intended to regularize DPO-style overfitting                          | Advanced comparison                                                            |
| ORPO                       | Combines supervised likelihood and odds-ratio preference terms without a separate reference model           | Advanced practical                                                             |
| KTO                        | Learns from desirable/undesirable examples using a prospect-theoretic objective rather than requiring pairs | Conditional for unpaired feedback                                              |
| SimPO                      | Reference-free preference optimization using sequence-normalized reward and a target margin                 | Advanced comparison                                                            |
| RLOO                       | Reinforce Leave-One-Out uses sampled completions and leave-one-out reward baselines                         | Advanced online reinforcement experiment                                       |
| GRPO                       | Group Relative Policy Optimization normalizes rewards among a sampled response group                        | Experimental/advanced; group sampling and reward quality are major costs       |
<!-- markdownlint-restore -->

Exact formulations, publication evidence, trainer APIs, reward/length biases, and smaller-model stability are treated in
[CH-07](../02-textbook/part-02-training-and-domain-adaptation/ch-07-preference-optimization-and-post-training.md). For
every method, compare with SFT-only under a common accepted-token/compute budget and measure factuality, calibration,
safety/refusal, verbosity, and reward hacking. `PLANNED EXPERIMENT`

## Knowledge, formatting, and forgetting

<!-- markdownlint-disable MD013 -->
| Goal                                                   | Primary candidate                              | Required counter-test                                                         |
| ------------------------------------------------------ | ---------------------------------------------- | ----------------------------------------------------------------------------- |
| Learn domain vocabulary/distribution from documents    | DAPT/CPT on explicit base, with general replay | SFT-only at equal accepted-token/compute budget; general perplexity/retention |
| Teach question-answer behavior and diagnostic sequence | SFT                                            | Untouched and DAPT-only baselines; factuality separated from format           |
| Improve refusal/uncertainty/style                      | Preference optimization after stable SFT       | SFT-only and safety-stratified evaluation                                     |
| Support many domains with one base                     | Separate LoRA/IA3 adapters                     | Independent models and one joint multi-domain model                           |
| Fit adaptation on limited GPU memory                   | QLoRA                                          | LoRA/full-FT controls under equal data and trial budget                       |
<!-- markdownlint-restore -->

Raw documents, instruction examples, preference pairs, distillation records, quantization calibration, and sealed
evaluation data are not interchangeable. Reusing evaluation or safety cases for adaptation creates leakage.

Forgetting must be measured after every stage with general and cross-domain sets. General-data replay, smaller learning
rate, fewer tokens, layer/target restriction, and checkpoint selection are mitigations, not guarantees. Continuing
pretraining from an instruction-tuned checkpoint can alter chat behavior and safety; explicit base checkpoints are
preferable for causal attribution.

## Compute and artifact implications

- Full fine-tuning stores gradients and optimizer state for all selected weights and produces a full domain checkpoint.
- LoRA reduces trainable state to $r(d_{in}+d_{out})$ per targeted matrix, but target count can dominate rank.
- QLoRA reduces frozen-base training storage; the final mobile/cloud artifact still requires merge/reconstruction and
  fresh target quantization.
- Prompt/prefix methods make tiny domain artifacts but can consume context or add cache/runtime operations.
- Multi-adapter cloud serving can share a base only where the serving runtime supports safe adapter loading, batching,
  and isolation.
- Edge release is operationally simplest as a merged, validated, target-quantized artifact per domain unless adapter
  execution is explicitly supported and measured.

No memory or speed outcome is claimed; all are architecture-dependent measurements.

## Initial experiment funnel

### Experiment A — objective choice

On one explicit base checkpoint and immutable corpus:

1. untouched base;
2. common SFT only;
3. DAPT only;
4. DAPT then common SFT.

Control accepted tokens, context, optimizer family, evaluation cadence, and seed policy. Measure domain perplexity where
meaningful, domain factuality/procedure, format compliance, unsupported claims, safety, and general retention.
`PLANNED EXPERIMENT`

### Experiment B — update choice

Using identical SFT text/mask and evaluation:

1. feasible full fine-tuning;
2. LoRA attention-only;
3. LoRA all-linear;
4. QLoRA all-linear.

Report trainable count, peak allocated/reserved accelerator and host memory, tokens/s, wall time, checkpoint bytes,
domain/general/safety results, and adapter/merged parity. Keep equal data and search budgets before a separate
fit-to-hardware comparison. `PLANNED EXPERIMENT`

### Experiment C — focused LoRA ablations

Only after stable baselines: rank/alpha/dropout, rsLoRA, DoRA, and LoRA+. Add PiSSA/EVA/LoftQ only when a measured
initialization, allocation, or quantization problem justifies them. Do not enable multiple variants simultaneously
without a single-factor control.

### Experiment D — multi-domain deployment

Compare independent merged artifacts, shared base + separate adapters, and one joint multi-domain adapter/model. Measure
interference, forgetting, storage, switch latency, runtime support, domain isolation, and release complexity.

## Release caveats

- Archive exact base, tokenizer, template, license, adapter configuration, target modules, rank/alpha pattern,
  initialization metadata, and hashes.
- Verify that intended and only intended parameters were trainable.
- Compare adapter-attached and merged logits/greedy generations before quantization.
- Merge against the exact clean base in supported precision; avoid repeated low-bit merge/unmerge.
- Quantize the validated merged parent into each target runtime format separately.
- Rerun domain/general/safety tests after merge, quantization, and export.
- PEFT cannot make stale, conflicting, or unsafe learned knowledge current. No-runtime-RAG increases the importance of
  uncertainty and escalation.

## Evidence limits and direct references

No method is declared to match full fine-tuning, acquire comprehensive domain knowledge, reduce inference memory, or
improve edge latency in this project. Paper claims are model/task-specific and remain `PAPER-REPORTED`.

- [Detailed PEFT source survey](peft-methods.md)
- [CH-05: Continued pretraining](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md)
- [CH-06: SFT](../02-textbook/part-02-training-and-domain-adaptation/ch-06-supervised-fine-tuning.md)
- [CH-07: Preference optimization](../02-textbook/part-02-training-and-domain-adaptation/ch-07-preference-optimization-and-post-training.md)
- [CH-08: Full FT versus PEFT](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md)
- [Combined pipelines](combined-optimization-pipelines.md)
- [Training comparison protocol](../06-experiment-framework/protocols/training-comparison.md)
