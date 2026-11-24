# Model-family comparison

**Research freeze:** 2026-11-23  
**Status:** authored synthesis of the [source survey](model-family-survey.md); no checkpoint was downloaded, adapted,
converted, or benchmarked.

## Decision frame

The comparison object is not a family name. It is:

```text
repository + immutable revision + weights + tokenizer + chat template
+ license + adaptation state + export/quantization + runtime/backend/device
```

`DOC-VERIFIED` below means an official card, repository, license, configuration, or artifact listing was inspected.
“Route” means documented evidence, not executed compatibility. No vendor benchmark is normalized into a cross-family
score.

## Comparative lesson

The matrix below answers a constrained question: which exact checkpoints deserve an experiment role? It does not rank
general intelligence. Parameter count is only the first approximation to deployment cost; tokenizer size, attention
layout, modality, context, quantized kernels, and license can reverse an apparently simple size comparison.

Consider a learner choosing between Gemma 3 1B PT and Llama 3.2 1B. Gemma offers an explicit
pretraining/instruction pair and documented Quantization-Aware Training (QAT) and LiteRT routes; Llama offers a broad
tooling ecosystem and a different license. Neither fact establishes domain accuracy or phone speed. If controlled
Domain-Adaptive Pretraining (DAPT) is the goal, both base artifacts are eligible. If a specific Android LiteRT route is
mandatory, Gemma may enter feasibility first. If ecosystem portability is more important, Llama remains a useful
comparator. The decision changes only after identical tokenizer, quality, safety, conversion, memory, and latency tests.

Use a checkpoint when its exact role and release obligations are clear. Avoid it when the experiment would confound
base versus instruct training, text versus multimodal architecture, or source context versus packaged context. The
historical reason for this discipline is that model families increasingly publish multiple post-training and deployment
variants under one recognizable name.

## Exact-checkpoint matrix

<!-- markdownlint-disable MD013 -->
| Exact checkpoint/role                                 | Size class and architecture                        | Adaptation value                                     | Checkpoint/license caveat                                                 | Documented deployment evidence                                        | Initial role                       |
| ----------------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------- | ---------------------------------- |
| `HuggingFaceTB/SmolLM2-135M`, `-360M`                 | 0.135B/0.36B dense, GQA, 8K                        | Explicit base/instruct; open recipes                 | Apache-2.0 stated; capacity is not presumed sufficient                    | llama.cpp/MLX/Transformers.js routes documented at family level       | Pipeline and extreme-edge controls |
| `google/gemma-3-270m`, `-it`                          | ~0.27B text-only, 32K                              | PT/instruct pair                                     | Gemma custom terms and prohibited-use policy                              | Official ecosystem route; exact target unexecuted                     | Pipeline control                   |
| `Qwen/Qwen2.5-0.5B`, `-Instruct`                      | 0.49B total, GQA 14/2, 32K                         | Strong explicit base/instruct pair for DAPT/SFT      | Exact 0.5B license states Apache-2.0                                      | LiteRT-LM declares family support; exact conversion required          | Core low-memory adaptation         |
| `Qwen/Qwen3-0.6B`                                     | 0.6B dense GQA, 32K                                | Useful post-trained baseline                         | Clean base for this exact release unresolved; Apache-2.0 stated           | Runtime route must be qualified                                       | Post-trained sub-billion arm       |
| `LiquidAI/LFM2-700M`                                  | 0.7B hybrid short-convolution/GQA, 32K             | Narrow fine-tuning arm                               | Revenue-conditioned LFM license; card cautions on knowledge-intensive use | Hybrid kernels/export unresolved per target                           | Architecture-efficiency hypothesis |
| `allenai/OLMo-2-0425-1B`                              | ~1B dense, 4K                                      | Base/SFT/DPO/instruct lineage, data/logs/recipes     | Apache-2.0 stated; English/short-context limits                           | Mobile export unverified                                              | Reproducibility control            |
| `google/gemma-3-1b-pt`, `-it`                         | ~1B text-only, local/global attention, 32K         | Explicit PT/instruct pair                            | Gemma terms                                                               | Official QAT GGUF and LiteRT-LM listing                               | Core edge candidate                |
| `meta-llama/Llama-3.2-1B`, `-Instruct`                | 1.23B dense GQA, source card 128K                  | Explicit base/instruct; broad tooling ecosystem      | Llama Community License/AUP                                               | Vendor quantized variants have separate context; exact route required | Ecosystem baseline                 |
| `TinyLlama/TinyLlama-1.1B-intermediate-step-1431k-3T` | 1.1B Llama-like GQA, 2K                            | Explicit base and historical chat comparator         | Repository archived; data terms separate from code                        | No current first-party mobile guarantee verified                      | Legacy/control arm                 |
| `Qwen/Qwen2.5-1.5B`, `-Instruct`                      | 1.5B dense, 32K                                    | Strong DAPT/SFT pair                                 | Exact license states Apache-2.0                                           | LiteRT-LM packaged route documents 4K runtime context                 | Core adaptation candidate          |
| `HuggingFaceTB/SmolLM2-1.7B`, `-Instruct`             | 1.7B dense, 8K                                     | Open recipes and base/instruct pair                  | Apache-2.0 stated                                                         | Documented community/official routes; unexecuted                      | Reproducible core candidate        |
| `Qwen/Qwen3-1.7B`                                     | 1.7B dense GQA, 32K                                | Post-trained quality arm                             | Clean base unresolved; Apache-2.0 stated                                  | Official GGUF repository exists                                       | Product/edge comparator            |
| `meta-llama/Llama-3.2-3B`, `-Instruct`                | 3.21B dense GQA, source card 128K                  | Explicit base/instruct                               | Custom license; quantized context is artifact-specific                    | Exact first-party path unresolved in survey                           | 3B ecosystem comparator            |
| `HuggingFaceTB/SmolLM3-3B-Base`, `SmolLM3-3B`         | 3B, GQA/NoPE, trained 64K                          | Explicit base/post-trained pair, open research trail | Apache-2.0 stated; data-release completeness must be rechecked            | Official ONNX repository exists                                       | Open 3B arm                        |
| `microsoft/Phi-4-mini-instruct`                       | 3.8B dense GQA, 200K vocabulary, 128K              | Strong instruct/PEFT baseline                        | Equivalent clean base not verified; exact repository states MIT           | Official ONNX CPU/mobile INT4 RTN and GPU artifacts                   | Deployment upper arm               |
| `Qwen/Qwen3-4B`                                       | 4B dense GQA, native 32K                           | Higher-capacity post-trained arm                     | Clean base unresolved; Apache-2.0 stated                                  | Official GGUF exists                                                  | Higher-capacity comparator         |
| `google/gemma-3-4b-pt`, `-it`                         | ~4B multimodal, local/global attention, 128K input | Explicit PT/instruct                                 | Gemma terms; vision changes memory/export problem                         | Official ecosystem artifacts                                          | Include only if vision is required |
| `mistralai/Ministral-3-3B-Base-2512` and instruct     | ~3.4B LM + 0.4B vision, 256K documented            | Explicit base/instruct/reasoning                     | Apache-2.0 stated; multimodal confound                                    | Official BF16/FP8/GGUF repositories                                   | Multimodal comparison only         |
| `EleutherAI/pythia-*`                                 | 70M–6.9B dense scientific suite                    | Intermediate checkpoints and dedup controls          | Apache-2.0 stated; deployment not intended                                | No product route implied                                              | Scaling/training-dynamics control  |
<!-- markdownlint-restore -->

All architecture, repository, artifact, and license statements are `DOC-VERIFIED`; role assignments are
`STATICALLY REVIEWED`. Direct official URLs and fuller caveats are in the [model-family survey](model-family-survey.md).

### Interpreting the matrix

The “initial role” column is an experiment assignment, not an award. Pipeline controls expose data and export defects
cheaply; core candidates test realistic product capacity; upper arms reveal whether extra capacity justifies resource
cost. Conditional arms enter only when their distinguishing feature—such as multimodality or hybrid sequence
processing—matches a requirement. This staged design prevents a large, expensive checkpoint from becoming the default
before the deployment route is known.

## Required family-specific cautions

- **Qwen:** license is checkpoint-specific. Qwen2.5 3B uses a research license unlike surveyed 0.5B/1.5B. Qwen3
  thinking/non-thinking behavior must be isolated in evaluation.
- **Gemma:** gated terms are not Apache-2.0. The 4B vision path must not be compared as if it were a text-only 4B
  artifact.
- **Llama:** broad ecosystem support is not exact artifact support. Source-card and vendor-quantized context limits
  differ.
- **Phi:** official ONNX artifacts reduce conversion uncertainty, but optimized output may differ and the large
  vocabulary contributes substantial embedding/output state.
- **SmolLM:** unusually useful for reproducibility, but open research artifacts do not establish deployment superiority.
- **TinyLlama:** useful as a stable historical control; short context and archived maintenance lower production
  priority.
- **MobileLLM/OpenELM:** architecture research only under the reviewed noncommercial/research weight licenses.
- **LFM2:** hybrid layers can be efficient only where target kernels exist; entity revenue conditions require legal
  review.
- **MoE:** none of the principal Qwen3 0.6B/1.7B/4B candidates above is MoE. Any MoE candidate must report total and
  active parameters, expert routing, storage, and kernel support separately.

## Tokenizer, context, and memory comparison rules

Tokenizer quality is measured, not inferred from vocabulary size. Freeze strata for diagnostic codes, part numbers,
units, drug and species names, Unicode, and target languages, then report tokens per character/word/record and outliers.
Replacing a tokenizer changes embeddings, output rows, and export compatibility.

Context is artifact-specific: record trained/native context, extension method, packaged runtime maximum, tested length,
cache precision, and quality near limits. Long advertised context is not evidence of reliable long-context recall.

For dense weights, \(Pb/8\) is only an `ESTIMATE`. Runtime qualification adds quantization metadata, embeddings,
Key-Value cache, activations, workspaces, tokenizer, runtime, and application memory. “Fits in storage,” “loads,” and
“runs interactively without thermal collapse” are separate gates.

## Controlled shortlist

### Tier 0 — pipeline controls

- SmolLM2-135M/360M
- Gemma 3 270M

### Tier 1 — core 0.5B–2B base-model sweep

- Qwen2.5-0.5B and 1.5B
- Gemma 3 1B PT
- Llama 3.2 1B
- SmolLM2 1.7B
- OLMo 2 1B reproducibility control

### Tier 2 — higher-capacity/post-trained sweep

- Qwen3 1.7B
- SmolLM3 3B Base
- Llama 3.2 3B
- Phi-4 Mini Instruct
- Qwen3 4B

### Conditional arms

LFM2-700M tests hybrid sequence mixing; MobileLLM 350M/600M tests deep/thin scaling for noncommercial research; Gemma 3
4B and Ministral enter only if multimodality is a requirement; Pythia/TinyLlama remain scientific controls.

This is a candidate set, not a quality ranking. `PLANNED EXPERIMENT`

## Fair selection protocol

1. Archive exact revision, weight/tokenizer/template hashes, and every applicable license.
2. Keep base, official instruct, reasoning, and multimodal strata separate.
3. Run tokenizer fertility and untouched domain/general/safety baselines.
4. Apply one common SFT to base models with fixed accepted tokens, masking, optimizer budget, seed policy, and
   evaluation.
5. Test conversion before expensive CPT/DAPT where deployment is a hard requirement.
6. Compare source and target tokenizer IDs, template bytes, logits where feasible, greedy tokens, and task quality.
7. Measure artifact size, load/peak/steady memory, KV growth, startup, time to first token, prefill, decode, energy,
   thermals, and failures on named targets.
8. Rerun quality and safety after each quantization/export.
9. Select target-specific Pareto sets; do not average safety into a compensating score.

## Evidence limits

No candidate is declared best. No cross-family benchmark, tokenizer study, CPT/SFT run, quantization, conversion,
mobile/AWS execution, power measurement, or legal determination was produced here. Model cards usually provide
incomplete document-level pretraining provenance. Runtime support changes quickly and must be pinned at experiment
freeze.

## Direct references and cross-links

- [Detailed model-family survey and primary URLs](model-family-survey.md)
- [CH-03: Modern SLM architectures](../02-textbook/part-01-language-model-foundations/ch-03-modern-slm-architectures.md)
- [Adaptation comparison](adaptation-and-peft-comparison.md)
- [Runtime/deployment comparison](runtime-and-deployment-comparison.md)
- [Evidence/maturity ranking](evidence-and-maturity-ranking.md)
- [Decision criteria](../14-decision-framework/criteria-and-measurement.md)
