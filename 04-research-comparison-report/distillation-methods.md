# Knowledge-distillation methods

## Scope and evidence policy

This survey covers every distillation topic and named method in `prompt.txt`, researched through 2026-11-23. Historical
encoder results establish mechanisms but do not prove decoder-only SLM outcomes. `PAPER-REPORTED` claims have not been
reproduced in this project; proprietary-teacher behavior, costs, licenses, and current API terms require a fresh
release-time check.

Knowledge distillation (KD) trains a student using information from one or more teachers. It may create a genuinely
smaller dense architecture, which is often more portable than sparsity. It also transfers teacher errors, biases, stale
facts, and unsafe behavior; it is not factual verification.

## Lesson: transfer behavior into a deliberately smaller student

### Problem, intuition, and historical reason

Small models may lack the behavior learned by a stronger model, while the stronger model may be too expensive to deploy.
Knowledge Distillation (KD) was developed to transfer useful information from a teacher into a student. The intuition is
an apprenticeship: the student learns from examples, probability judgments, or internal representations, but remains
limited by its own capacity and by the teacher's mistakes.

### Mechanisms and worked example

Response distillation trains on validated teacher text. Logit distillation matches softened next-token distributions.
Feature and attention distillation match internal representations. Online methods query the teacher during student
training; offline methods cache signals. For an automotive assistant, compare a native 600M student trained on
expert-validated examples, the same student trained on validated teacher responses, a mixed dataset, and aligned-logit
KD where both models expose compatible vocabularies. Quantize only the best high-precision student afterward. This is an
experimental design, not a prediction that the teacher-trained variant wins.

### Strengths, limitations, compatibility, and decision

Response KD works across tokenizers and black-box teachers but exposes only sampled answers. Logits carry uncertainty
but require internal access, large storage or online compute, and token alignment. Feature losses are most plausible
between related architectures. Use response KD as the practical baseline when outputs can legally be retained and
authoritatively validated. Add logit or on-policy KD only when the richer signal addresses a measured gap. Avoid
unfiltered synthetic facts, unverifiable reasoning traces, or teacher consensus as ground truth. Always compare with a
natively pretrained student of similar deployment cost.

## Core objectives

For teacher logits \(z_t\), student logits \(z_s\), and temperature \(T\):

\[ p_t^{(T)}=\operatorname{softmax}(z_t/T),\qquad p_s^{(T)}=\operatorname{softmax}(z_s/T) \]

A common logit loss is \(T^2 D_{KL}(p_t^{(T)}\|p_s^{(T)})\), mixed with hard-label cross-entropy. The direction of KL,
sampled context, vocabulary alignment, and reduction all matter. The \(T^2\) factor is a conventional gradient-scale
correction, not a universal requirement.

<!-- markdownlint-disable MD013 -->
| Family                  | Mechanism                                                                                                   | Key limitation and domain-SLM relevance                                                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Response/sequence-level | Train on complete teacher-generated responses, usually ordinary next-token cross-entropy.                   | Works with black-box teachers but exposes only selected sequences, not the full distribution. Quality filtering and rights to retain outputs are essential. |
| Token/logit-based       | Match teacher next-token distributions, often with temperature/KL.                                          | Rich signal but requires teacher logits and aligned token positions/vocabularies; storing full-vocabulary logits is expensive.                              |
| Feature-based           | Match hidden states through projections/layer mappings.                                                     | Requires internal access and compatible sequence/width geometry. Can overconstrain a differently designed student.                                          |
| Attention transfer      | Match attention maps, relations, or query/key/value products.                                               | Head counts, attention variants, masks, GQA/MQA, and implementation semantics may differ. Attention equality is not a guarantee of behavioral equality.     |
| Offline                 | Precompute teacher outputs/logits/features.                                                                 | Reusable and auditable but storage-heavy and fixed to one teacher revision.                                                                                 |
| Online                  | Teacher runs during student training.                                                                       | Avoids giant caches and supports on-policy data, but increases compute, coupling, and failure surface.                                                      |
| Self-distillation       | A model/checkpoint teaches another version of itself, or a model trains on self-generated/filtered outputs. | Can reinforce errors or collapse diversity; “no external teacher” does not mean data-free or self-validating.                                               |
| Multi-teacher           | Combines outputs from multiple teachers.                                                                    | Requires conflict resolution, calibration, licenses, and provenance for each teacher. Averaging incompatible logits can be meaningless.                     |
| Task/domain-specific    | Distills on target-domain prompts/tasks.                                                                    | Efficient specialization, but narrow prompts can erase general capabilities or produce answer-format imitation without broad knowledge.                     |
| Data-free/synthetic     | Generates transfer inputs or responses without the original training set.                                   | Usually still needs seed prompts, priors, or a generator. Synthetic facts require authoritative validation; “data-free” is not evidence-free.               |
<!-- markdownlint-restore -->

The objective table orders methods by teacher access. Moving rightward from text to logits to hidden features can expose
more information, but coupling increases. Different tokenizers are usually compatible with response KD because the
student retokenizes text; they are not directly compatible with per-token distribution matching without a justified
mapping.

## Influential named approaches

<!-- markdownlint-disable MD013 MD049 -->
| Approach          | Identity, publication, and source                                                                                                                                                                                                                                 | Mechanism and claimed contribution                                                                                                                                                          | Limitations, maturity, and relevance                                                                                                                                                                                                     |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Classical KD      | Hinton, Vinyals, and Dean, _Distilling the Knowledge in a Neural Network_, NIPS workshop 2015 ([paper](https://arxiv.org/abs/1503.02531)).                                                                                                                        | Popularized temperature-softened teacher targets mixed with hard labels.                                                                                                                    | **Foundational.** Classification framing must be adapted carefully to autoregressive per-token distributions and long sequences.                                                                                                         |
| Sequence-level KD | Kim and Rush, _Sequence-Level Knowledge Distillation_, EMNLP 2016 ([paper](https://aclanthology.org/D16-1139/)).                                                                                                                                                  | Approximates the teacher sequence distribution with decoded sequences and trains the student on them. Paper reports NMT speed/quality results and a pruning combination (`PAPER-REPORTED`). | **Established concept.** In modern generative LMs this often resembles supervised fine-tuning on teacher completions. It loses alternatives and uncertainty, and teacher decoding policy changes the dataset.                            |
| DistilBERT        | Sanh et al., _DistilBERT_, arXiv 2019 ([paper](https://arxiv.org/abs/1910.01108), [archived official model/code context](https://github.com/huggingface/transformers/tree/33eef992503689ba1af98090e26d3e98865b2a9b/examples/research_projects/distillation)).                                 | Distills during pretraining with language-model, soft-target, and cosine hidden-state losses into a shallower BERT. Paper reports size/speed/quality results (`PAPER-REPORTED`).            | **Mature historical encoder reference.** Not a decoder-generation recipe. Useful for illustrating combined losses and layer initialization; direct applicability to domain chat SLMs is limited.                                         |
| TinyBERT          | Jiao et al., _TinyBERT: Distilling BERT for Natural Language Understanding_, Findings of EMNLP 2020 ([paper](https://aclanthology.org/2020.findings-emnlp.372/), [official code](https://github.com/huawei-noah/Pretrained-Language-Model/tree/master/TinyBERT)). | Two-stage general and task-specific distillation over embeddings, hidden states, attention, and prediction layer, with augmentation.                                                        | **Mature historical encoder reference.** High pipeline complexity and architecture alignment. Its paper-reported compression/latency cannot be transferred to causal SLMs.                                                               |
| MiniLM            | Wang et al., _MiniLM: Deep Self-Attention Distillation_, NeurIPS 2020 ([paper](https://arxiv.org/abs/2002.10957), [official repo](https://github.com/microsoft/unilm/tree/master/minilm)).                                                                        | Distills last-layer self-attention distributions and value-relation products, allowing flexible student depth.                                                                              | **Established encoder-oriented technique.** Relation matching can transfer internal structure with fewer layer mappings, but causal masks/GQA and decoder generation need a new experiment. Do not confuse MiniLM with MiniLLM.          |
| MiniLLM           | Gu et al., _MiniLLM: Knowledge Distillation of Large Language Models_, ICLR 2024 ([paper](https://openreview.net/forum?id=5h0qf7IBZZ), [official code](https://github.com/microsoft/LMOps/tree/main/minillm)).                                                    | Uses on-policy student samples and reverse KL to reduce exposure mismatch/mode averaging in generative distillation.                                                                        | **Modern research baseline.** Requires teacher logits on student-generated sequences and careful optimization; substantially more complex/costly than response SFT. Official code/model coverage must be adapted to current checkpoints. |
<!-- markdownlint-restore -->

The historical table explains lineage rather than offering drop-in recipes. DistilBERT, TinyBERT, and MiniLM established
important encoder techniques. Sequence-level KD supplies the practical text-generation foundation. MiniLLM is a modern
generative research direction using student-generated contexts and reverse Kullback-Leibler divergence, but its teacher
compute and optimization burden are substantially higher than response SFT.

## Modern generative-model distillation patterns

### Teacher-generated supervised fine-tuning

Generate domain questions, explanations, diagnostic dialogues, counterexamples, and refusals; validate against licensed
authoritative sources and experts; then train the student on accepted outputs. This is the most practical black-box
route, but should be called **sequence/response distillation**, not logit distillation. Preserve teacher identifier,
prompt, decoding settings, raw output, validators, rejection reason, and source grounding in the dataset lineage.

**Limitations:** the student learns one sampled behavior; teacher verbal fluency can hide factual errors; proprietary
terms may restrict output use; API updates make datasets non-reproducible unless snapshots are retained.
Chain-of-thought collection can be unavailable, unreliable, or legally/operationally undesirable. Prefer concise
verified rationales or process supervision designed for release rather than assuming hidden reasoning is necessary.

### Logit and on-policy distillation

When both models are locally accessible, compare forward-KL teacher-forced KD, reverse-KL/on-policy KD, and a
hard-target baseline. On-policy methods expose the teacher to contexts the student actually visits, but multiply teacher
inference cost. For top-k logit storage, retain omitted probability mass or document the approximation.

### Feature and relation distillation

Map teacher layers to student layers and project hidden widths, or match normalized relations instead of raw vectors.
This is feasible when architecture/tokenization are controlled. It becomes fragile across different tokenizers,
RoPE/context designs, attention types, or modalities.

## Compatibility caveats

- **Tokenizer mismatch:** teacher and student text can support response KD with ordinary student tokenization. Direct
  token-logit KD generally cannot align distributions when vocabularies differ without a mapping or text-level
  alternative.
- **Architecture mismatch:** feature dimensions and layer counts need mappings; attention transfer needs compatible
  masks/head semantics.
- **Teacher access:** black-box APIs expose responses, not logits/features. Do not describe response SFT as soft-target
  KD.
- **Licensing and privacy:** confirm teacher model/API terms permit synthetic-data retention and downstream training.
  Never send restricted manuals, customer data, or personally identifiable information to an unauthorized teacher.
- **Domain conflict:** general teachers can confidently contradict specialist references. Validation should favor
  authoritative domain evidence, not teacher consensus.
- **Safety:** pharmacy and safety-critical repair outputs need expert-defined escalation/refusal behavior and
  source-based review. Distillation cannot make an unverified teacher safe.

## Student selection

A smaller student should be a deployable architecture before training begins: supported tokenizer, operators, context
length, GQA/MQA layout, quantization/export path, license, and device memory budget. Compare against a natively
pretrained model of the same approximate size; a compressed student is not automatically better. Distillation changes
quality, while architecture determines much of actual edge speed.

## Required project experiment

Use one immutable domain split and compare:

1. student trained only on expert/reference-validated data;
2. student trained only on validated teacher-generated examples;
3. mixed expert and teacher data with documented sampling weights;
4. soft-target/logit KD where teacher and student access/alignment permit;
5. best student quantized after distillation.

Add an untouched student baseline, direct SFT of the teacher-sized model, and a natively small model. Measure domain
factuality, diagnostic process, unsupported-claim rate, safety/refusal correctness, general retention, calibration,
artifact size, peak memory, TTFT, prefill/decode rates, and physical-device energy/thermal behavior. Use paired prompts,
repeated trials, confidence intervals, and expert adjudication for safety-critical subsets.

## Initial decision

- **Core practical:** validated response/sequence distillation plus mixed expert data.
- **Advanced practical:** offline top-k logit KD with an open teacher and aligned tokenizer; feature loss only if
  architectures align.
- **Research:** MiniLLM-style on-policy reverse-KL, multi-teacher conflict-aware KD, and pruning-aware distillation.
- **Avoid initially:** unfiltered proprietary-teacher dumps, unverifiable chain-of-thought imitation, and “data-free”
  pipelines that generate their own facts and judge themselves.

All costs and outcomes are `PLANNED EXPERIMENT`. Recency checks are required for teacher versions, API terms, open-model
licenses, official repositories, and training-framework APIs immediately before execution.
