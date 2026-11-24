# Initial survey of small language-model families and checkpoints

## Status, scope, and evidence discipline

**Survey status:** initial research survey, not a final chapter or final model-selection matrix.

**Verification date:** 2026-11-23.

**Scope:** text-generative checkpoints from roughly 100M to 8B parameters, with emphasis on 0.5B–4B models that could
support domain continued pretraining, supervised fine-tuning, quantization, and offline edge experiments. Multimodal
models are included only when their language component is in range and their extra modality materially affects
deployment.

**Source policy:** this pass uses developer model cards, official repositories, official licenses, and primary papers.
It does not treat a repository tag, a family name, or a vendor statement as proof of successful conversion or acceptable
device performance.

Evidence labels follow [the project standard](../00-governance/evidence-and-citation-standard.md):

- `DOC-VERIFIED`: an official card, repository, configuration, or license was inspected.
- `PAPER-REPORTED`: a primary paper reports the item; this is not independent reproduction.
- `VENDOR-REPORTED`: the developer makes a capability, efficiency, or deployment claim.
- `STATICALLY REVIEWED`: this survey's reasoned assessment from documentation; no runtime execution is implied.
- `PLANNED EXPERIMENT`: requires controlled local evaluation.
- `UNVERIFIED IMPLEMENTATION`: an artifact or route exists, but this project has not executed it.

No benchmark score is reproduced in this initial survey. Cross-family scores in model cards use differing prompts,
harness versions, precision, and post-training states and are not yet a defensible common ranking. No mobile
compatibility statement below means that this project has successfully exported or run a checkpoint.

## Lesson: why model-family selection is an artifact problem

### Problem, intuition, and historical reason

Early language-model comparisons could often treat architecture and parameter count as useful summaries. Modern Small
Language Models (SLMs) add instruction tuning, reasoning modes, custom licenses, long-context extensions, multimodal
components, and runtime-specific quantized releases. Consequently, a family name now hides several materially different
artifacts. The practical intuition is to select a releasable package, not a brand: weights, tokenizer, chat template,
license, revision, adaptation state, and export route must remain together.

### Mechanism and concrete example

A base checkpoint predicts text but has not necessarily learned a chat policy. An instruct checkpoint adds supervised
and sometimes preference post-training. Starting Domain-Adaptive Pretraining (DAPT) from a base checkpoint gives a
cleaner test of domain-language acquisition; starting from an instruct checkpoint may preserve useful behavior but can
also disturb alignment. For example, an automotive experiment can compare `Qwen/Qwen2.5-1.5B` after DAPT and
Supervised Fine-Tuning (SFT) with `Qwen/Qwen2.5-1.5B-Instruct` after the same SFT. The comparison asks whether raw
domain-text training added value, not whether “Qwen is better.”

### Strengths, limitations, and compatibility

Smaller checkpoints lower storage and often memory, but may lack capacity for broad specialist knowledge. Larger
checkpoints may improve capability yet exceed mobile memory or thermal limits. Grouped-Query Attention (GQA) can reduce
Key-Value (KV) cache size, while a large vocabulary can increase embedding storage. A documented GGUF, Open Neural
Network Exchange (ONNX), or LiteRT route lowers conversion uncertainty but proves neither numerical parity nor
interactive speed. Licenses are checkpoint-specific and must be checked independently of technical fit.

### Decision walkthrough

1. Exclude artifacts whose license, modality, language, or safety posture cannot satisfy the product.
2. Separate base, instruct, reasoning, and multimodal candidates so post-training is not mistaken for architecture.
3. Measure tokenizer fertility on actual codes, units, drug names, species names, and target languages.
4. Test source-framework behavior and target-runtime conversion before expensive adaptation.
5. Compare one sub-billion control, several 0.5B–2B candidates, and one 3B–4B upper arm under a common evaluation.
6. Use when the exact tuple passes; avoid selecting from family reputation, advertised context, or incomparable model
   card benchmarks.

## Selection principles for this project

1. Prefer an exact **base/pretrained checkpoint** for continued pretraining and an exact **instruct checkpoint** as a
   separate SFT-only baseline. A family name is not a checkpoint.
2. Treat license, acceptable-use terms, redistribution obligations, and training-data rights as independent gates.
3. Prefer standard dense decoder-only architectures for the first experiments. Hybrid convolution/attention or
   multimodal architectures are valuable comparison arms but add exporter and kernel risk.
4. GQA, tied embeddings, vocabulary size, context design, and attention pattern affect memory and export behavior;
   parameter count alone is insufficient.
5. Official GGUF, ONNX, or LiteRT artifacts reduce conversion uncertainty, but do not prove acceptable latency, memory,
   power, or quality on target devices.
6. Long advertised context is not the initial mobile target. KV-cache cost and sustained memory must be measured at the
   project's actual prompt and output lengths.

## Priority checkpoint survey

### Qwen

#### Exact checkpoints in the primary 0.5B–4B band

- `Qwen/Qwen3-0.6B` — 0.6B total, 0.44B non-embedding; dense causal LM; 28 layers; GQA 16 query/8 KV heads; tied
  embeddings; native 32,768-token context. `DOC-VERIFIED`
- `Qwen/Qwen3-1.7B` — 1.7B total, 1.4B non-embedding; dense causal LM; 28 layers; GQA 16/8; tied embeddings;
  32,768-token context. `DOC-VERIFIED`
- `Qwen/Qwen3-4B` — 4.0B total, 3.6B non-embedding; dense causal LM; 36 layers; GQA 32/8; tied embeddings; native 32,768
  context; the card documents 131,072 with YaRN. `DOC-VERIFIED`
- `Qwen/Qwen2.5-0.5B` and `Qwen/Qwen2.5-0.5B-Instruct` — 0.49B total/0.36B non-embedding; base and instruct pair; RoPE,
  SwiGLU, RMSNorm, QKV bias, tied embeddings, GQA 14/2; 32,768 context. `DOC-VERIFIED`
- `Qwen/Qwen2.5-1.5B` and `Qwen/Qwen2.5-1.5B-Instruct` — base and instruct pair in the core band. `DOC-VERIFIED`
- `Qwen/Qwen2.5-3B` and its instruct variant exist, but the exact 3B license is the Qwen Research License rather than
  Apache-2.0. `DOC-VERIFIED`

**Architecture and adaptation:** Qwen3's dense GQA models and Qwen2.5's conventional RoPE/SwiGLU/RMSNorm stack are
plausible PEFT and quantization candidates. Qwen3's published checkpoint combines pretraining and post-training and
supports switchable thinking/non-thinking behavior; it is not automatically interchangeable with a clean base checkpoint
for controlled DAPT. Qwen2.5 supplies explicit base checkpoints, making `Qwen/Qwen2.5-0.5B` and `Qwen/Qwen2.5-1.5B`
stronger first candidates for a base-versus-domain-adapted experiment. `STATICALLY REVIEWED`

**License/source caveat:** Qwen3 open-weight checkpoints are Apache-2.0. Qwen2.5 licensing is checkpoint-specific:
official 0.5B and 1.5B repositories are Apache-2.0, while the official 3B license restricts use to non-commercial
research/evaluation unless a commercial license is obtained. Do not infer one Qwen license across the family.
`DOC-VERIFIED`

**Model-card quality:** high for exact architecture, parameter split, context, usage, and generation controls; medium
for training-data provenance at document level. The Qwen3 technical report strengthens family-level evidence.
`STATICALLY REVIEWED`

**Edge/export:** official Qwen3 GGUF repositories exist for 1.7B and 4B; Google's LiteRT-LM source declares Qwen3 and
Qwen2.5 model types, and its published ready-model list includes qwen2.5-1.5b at a reduced 4,096 runtime context. These
are format/runtime-specific facts, not blanket support for every Qwen checkpoint. `DOC-VERIFIED`,
`UNVERIFIED IMPLEMENTATION`

**Initial disposition:** highest-priority family. Compare Qwen3-0.6B/1.7B/4B for post-trained quality and
Qwen2.5-0.5B/1.5B base checkpoints for controlled DAPT/SFT. `PLANNED EXPERIMENT`

Primary sources:

- <https://huggingface.co/Qwen/Qwen3-0.6B>
- <https://huggingface.co/Qwen/Qwen3-1.7B>
- <https://huggingface.co/Qwen/Qwen3-4B>
- <https://github.com/QwenLM/Qwen3>
- <https://raw.githubusercontent.com/QwenLM/Qwen3/main/Qwen3_Technical_Report.pdf>
- <https://huggingface.co/Qwen/Qwen2.5-0.5B>
- <https://huggingface.co/Qwen/Qwen2.5-0.5B/blob/main/LICENSE>
- <https://huggingface.co/Qwen/Qwen2.5-1.5B/blob/main/LICENSE>
- <https://huggingface.co/Qwen/Qwen2.5-3B/raw/main/LICENSE>
- <https://huggingface.co/Qwen/Qwen3-1.7B-GGUF>
- <https://huggingface.co/Qwen/Qwen3-4B-GGUF>
- <https://github.com/google-ai-edge/LiteRT-LM>

### Gemma

#### Exact Gemma checkpoints

- `google/gemma-3-270m` and `google/gemma-3-270m-it` — approximately 270M; text-only at this size; 32K context.
  `DOC-VERIFIED`
- `google/gemma-3-1b-pt` and `google/gemma-3-1b-it` — approximately 1B; text-only; 32K context. `DOC-VERIFIED`
- `google/gemma-3-4b-pt` and `google/gemma-3-4b-it` — approximately 4B class; image-and-text input with text output;
  128K input context is documented for this size. `DOC-VERIFIED`

**Architecture and adaptation:** Gemma 3 uses a decoder language model with alternating local sliding-window and global
attention; the 4B checkpoint also includes a vision path. The explicit `-pt` checkpoints make controlled CPT/DAPT
possible. The 270M model is useful for pipeline validation, tokenizer/data ablations, and aggressive edge tests, but its
capacity should not be assumed sufficient for broad specialist knowledge. The 1B and 4B checkpoints are more relevant to
the production-oriented comparison. `DOC-VERIFIED`, `STATICALLY REVIEWED`

**License/source caveat:** weights are gated behind acceptance of Google's Gemma terms and prohibited-use policy. The
repository declares `license: gemma`, not an OSI open-source license. Commercial and redistribution review must use the
exact current terms rather than the phrase “open model.” `DOC-VERIFIED`

**Model-card quality:** high for intended use, modalities, context, training-token totals, safety, and vendor benchmark
protocols; medium for detailed data provenance. `STATICALLY REVIEWED`

**Edge/export:** Google publishes QAT GGUF artifacts such as `google/gemma-3-1b-pt-qat-q4_0-gguf`. LiteRT-LM lists
Gemma3-1B as a chat-ready 4-bit model with a 4,096 packaged context. This is strong official evidence for a defined edge
route, but the project has not reproduced conversion or device execution. The 4B vision encoder creates a materially
different memory/export problem from the text-only 1B checkpoint. `DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

**Initial disposition:** high priority at 1B; include 270M as an edge floor and 4B as a higher-capacity multimodal
comparison only if the vision path serves the domain requirement. `PLANNED EXPERIMENT`

Primary sources:

- <https://huggingface.co/google/gemma-3-270m>
- <https://huggingface.co/google/gemma-3-270m-it>
- <https://huggingface.co/google/gemma-3-1b-pt>
- <https://huggingface.co/google/gemma-3-1b-it>
- <https://huggingface.co/google/gemma-3-4b-pt>
- <https://huggingface.co/google/gemma-3-4b-it>
- <https://huggingface.co/google/gemma-3-1b-pt-qat-q4_0-gguf>
- <https://arxiv.org/abs/2503.19786>
- <https://github.com/google-ai-edge/LiteRT-LM>

### Llama

#### Exact Llama checkpoints

- `meta-llama/Llama-3.2-1B` and `meta-llama/Llama-3.2-1B-Instruct` — 1.23B parameters; text-only; GQA; shared
  embeddings; 128K context in the standard card. `DOC-VERIFIED`
- `meta-llama/Llama-3.2-3B` and `meta-llama/Llama-3.2-3B-Instruct` — 3.21B parameters; text-only; GQA; shared
  embeddings; 128K context. `DOC-VERIFIED`

**Architecture and adaptation:** these are dense autoregressive transformer checkpoints with explicit base and
instruction-tuned variants. Their conventional Llama lineage and broad ecosystem make them useful reference baselines
for LoRA, quantization, and format conversion. The card separately identifies vendor-quantized variants with an 8K
context; do not transfer the 128K claim to a quantized artifact without checking that artifact. `DOC-VERIFIED`,
`STATICALLY REVIEWED`

**License/source caveat:** Llama 3.2 uses Meta's custom Llama 3.2 Community License and Acceptable Use Policy, not
Apache-2.0. The official card lists eight supported languages and permits additional-language fine-tuning subject to the
terms; legal and language validation remain required. `DOC-VERIFIED`

**Model-card quality:** high for exact parameter counts, variants, context, supported languages, training-token upper
bound, intended use, and safety; medium for underlying data provenance. `STATICALLY REVIEWED`

**Edge/export:** the official card explicitly positions quantized variants for on-device use, but that is a vendor
statement. This pass did not verify a current first-party ExecuTorch checkpoint recipe for these exact repositories.
llama.cpp/GGUF and other community routes must be checked separately and executed before compatibility is claimed. This
point is currently unresolved. `VENDOR-REPORTED`

**Initial disposition:** high-priority ecosystem baseline, especially 1B and 3B base checkpoints; license and exact
export route are gating checks. `PLANNED EXPERIMENT`

Primary sources:

- <https://huggingface.co/meta-llama/Llama-3.2-1B>
- <https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct>
- <https://huggingface.co/meta-llama/Llama-3.2-3B>
- <https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct>
- <https://huggingface.co/meta-llama/Llama-3.2-3B/blob/main/LICENSE.txt>

### Phi

#### Exact Phi checkpoints

- `microsoft/Phi-3.5-mini-instruct` — 3.8B; dense decoder-only transformer; 128K context; 3.4T training tokens reported;
  multilingual instruction model. `DOC-VERIFIED`
- `microsoft/Phi-4-mini-instruct` — 3.8B; dense decoder-only transformer; 128K context; 5T training tokens; 200K
  vocabulary, GQA, and shared input/output embeddings distinguish it from Phi-3.5 Mini. `DOC-VERIFIED`
- `microsoft/Phi-4-mini-reasoning` — same 3.8B architecture, post-trained specifically for mathematical reasoning; not a
  general-purpose domain baseline. `DOC-VERIFIED`
- `microsoft/Phi-4-mini-instruct-onnx` — official optimized ONNX conversion with CPU/mobile INT4 RTN and GPU artifacts.
  `DOC-VERIFIED`

**Architecture and adaptation:** Phi-4 Mini's large 200K vocabulary increases embedding-related storage relative to a
smaller-vocabulary design, while GQA reduces KV-head count. The official release is an instruct model; this survey has
not verified an equivalent first-party Phi-4 Mini base checkpoint for clean continued-pretraining experiments. Use Phi
primarily as an SFT/PEFT and deployment comparison unless a base checkpoint is verified; base-checkpoint availability is
currently unresolved. `STATICALLY REVIEWED`

**License/source caveat:** the exact Phi-4 Mini instruct model and official ONNX conversion state MIT. Training relies
heavily on synthetic and filtered public data; provenance is described by category rather than as a fully
reconstructable corpus. `DOC-VERIFIED`

**Model-card quality:** very high for architecture deltas, training budget/dates, languages, limitations, safety, and
official optimized artifacts. `STATICALLY REVIEWED`

**Edge/export:** this family has one of the strongest first-party ONNX paths in the survey. Microsoft says the optimized
files target ONNX Runtime across desktop and mobile CPUs and warns that optimized output may differ from the base model.
LiteRT-LM also lists phi-4-mini, but its packaged model uses a 4,096 context. Actual Android/iOS packaging, peak memory,
and thermal behavior remain untested. `DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

**Initial disposition:** high-priority 3.8B deployment baseline and reasoning-focused comparison; lower priority for
controlled CPT until an exact base checkpoint is verified. `PLANNED EXPERIMENT`

Primary sources:

- <https://huggingface.co/microsoft/Phi-3.5-mini-instruct>
- <https://huggingface.co/microsoft/Phi-4-mini-instruct>
- <https://huggingface.co/microsoft/Phi-4-mini-reasoning>
- <https://huggingface.co/microsoft/Phi-4-mini-instruct-onnx>
- <https://huggingface.co/microsoft/Phi-4-mini-instruct-onnx/blob/main/LICENSE>
- <https://github.com/google-ai-edge/LiteRT-LM>

### SmolLM

#### Exact SmolLM checkpoints

- `HuggingFaceTB/SmolLM2-135M`, `HuggingFaceTB/SmolLM2-360M`, and `HuggingFaceTB/SmolLM2-1.7B`, plus exact `-Instruct`
  variants. `DOC-VERIFIED`
- `HuggingFaceTB/SmolLM3-3B-Base` and `HuggingFaceTB/SmolLM3-3B` (instruct/hybrid-reasoning post-trained model).
  `DOC-VERIFIED`

**Architecture and adaptation:** SmolLM2 is Llama-2-like with RoPE, SwiGLU, tied embeddings, and an 8K final context;
the 135M and 360M variants use GQA, while the primary paper's 1.7B architecture table lists 24 layers, 2,048 model
width, and 32 attention heads. SmolLM3-3B changes the comparison materially: it uses GQA and NoPE, reports a 64K trained
context and up to 128K with YaRN, and has explicit base and post-trained checkpoints. `PAPER-REPORTED`, `DOC-VERIFIED`

**Domain adaptation:** SmolLM2 offers unusually small checkpoints for validating end-to-end data, training, evaluation,
and export pipelines. SmolLM2-1.7B and SmolLM3-3B-Base are more plausible domain-capacity candidates. The open training
recipes, data tooling, and intermediate checkpoints make this family especially valuable for research on CPT and
forgetting. `STATICALLY REVIEWED`

**License/source caveat:** the surveyed checkpoints state Apache-2.0. SmolLM2 cards identify public dataset mixtures and
link training/post-training recipes; SmolLM3 states that some mid/post-training datasets would be uploaded later, so
“fully open” should be rechecked against the actual available artifacts at experiment freeze time. `DOC-VERIFIED`

**Model-card quality:** high to very high. SmolLM2 provides per-size training details and limitations; SmolLM3 supplies
architecture, data curriculum, framework, checkpoints, and explicit limitations. `STATICALLY REVIEWED`

**Edge/export:** the official repository describes quantized use through llama.cpp, MLX, and Transformers.js, and an
official `HuggingFaceTB/SmolLM3-3B-ONNX` repository exists. Those references do not establish support for every
precision, operator, or target. `DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

**Initial disposition:** highest priority for reproducible research; use 135M/360M for pipeline tests, 1.7B for the core
efficient baseline, and SmolLM3-3B-Base for the stronger open 3B arm. `PLANNED EXPERIMENT`

Primary sources:

- <https://huggingface.co/HuggingFaceTB/SmolLM2-135M>
- <https://huggingface.co/HuggingFaceTB/SmolLM2-360M>
- <https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B>
- <https://arxiv.org/abs/2502.02737>
- <https://github.com/huggingface/smollm>
- <https://huggingface.co/HuggingFaceTB/SmolLM3-3B-Base>
- <https://huggingface.co/HuggingFaceTB/SmolLM3-3B>
- <https://huggingface.co/HuggingFaceTB/SmolLM3-3B-ONNX>

### TinyLlama

#### Exact TinyLlama checkpoints

- `TinyLlama/TinyLlama-1.1B-intermediate-step-1431k-3T` — 1.1B base checkpoint after 3T tokens. `DOC-VERIFIED`
- `TinyLlama/TinyLlama-1.1B-Chat-v1.0` — chat model fine-tuned from the 3T checkpoint with a Zephyr-style SFT/DPO
  recipe. `DOC-VERIFIED`
- `TinyLlama/TinyLlama_v1.1` — later staged-pretraining release with general, Math&Code, and Chinese branches described
  by the card. `DOC-VERIFIED`

**Architecture and adaptation:** 22-layer Llama-style dense decoder, 2,048 hidden size, 5,632 SwiGLU intermediate size,
32 query heads and four KV groups, RoPE/RMSNorm, Llama-2 tokenizer, and 2,048 context. The short context and older
architecture are disadvantages for document-scale domain training, but the small standard Llama shape and available base
checkpoint make it a useful legacy/control model. `PAPER-REPORTED`, `STATICALLY REVIEWED`

**License/source caveat:** official code is Apache-2.0. The chat card documents synthetic UltraChat and GPT-4-ranked
UltraFeedback post-training. Dataset and generated-data terms require separate review before redistribution of
derivatives. The GitHub repository is archived, so maintenance risk is higher than for active families. `DOC-VERIFIED`

**Model-card quality:** medium. Architecture and training are well documented, but variant naming, historical bug notes,
intermediate checkpoints, and retrained v1.1 branches require care. `STATICALLY REVIEWED`

**Edge/export:** Llama-like structure suggests broad community runtime coverage, but this survey found no current
first-party mobile export guarantee. Treat all target-runtime support as currently unresolved until exact conversion
tests pass. `STATICALLY REVIEWED`

**Initial disposition:** retain as a reproducible historical/control baseline, not the leading production candidate.
`PLANNED EXPERIMENT`

Primary sources:

- <https://github.com/jzhang38/TinyLlama>
- <https://arxiv.org/abs/2401.02385>
- <https://huggingface.co/TinyLlama/TinyLlama-1.1B-intermediate-step-1431k-3T>
- <https://huggingface.co/TinyLlama/TinyLlama-1.1B-Chat-v1.0>
- <https://huggingface.co/TinyLlama/TinyLlama_v1.1>

### MobileLLM

#### Exact MobileLLM checkpoints

- `facebook/MobileLLM-125M` — 124.6M, 30 layers, width 576, 9 query/3 KV heads.
- `facebook/MobileLLM-350M` — 345.3M, 32 layers, width 960, 15/5 heads.
- `facebook/MobileLLM-600M` — 603.1M, 40 layers, width 1,152, 18/6 heads.
- `facebook/MobileLLM-1B` — 1.01B, 54 layers, width 1,280, 20/5 heads.
- `facebook/MobileLLM-1.5B` — 1.51B, 54 layers, width 1,600, 25/5 heads.

All figures above are from Meta's official family card. `DOC-VERIFIED`

**Architecture and adaptation:** MobileLLM intentionally combines deep-and-thin blocks, SwiGLU, embedding sharing, and
GQA. The cards report 2K context and 1T training tokens. This is a valuable architecture-efficiency ablation, especially
below 1B, but short context may constrain document-oriented CPT and each additional layer adds sequential execution
overhead that must be measured on the target backend. `DOC-VERIFIED`, `STATICALLY REVIEWED`

**License/source caveat:** current official model pages state the FAIR Noncommercial Research License, updated
2025-04-17. Older pull-request/README revisions used CC-BY-NC-4.0. The current FAIR terms, not stale metadata, govern
current use; commercial deployment is not permitted by the surveyed license. `DOC-VERIFIED`

**Model-card quality:** high for architectural dimensions, context, training budget, and paper linkage; medium for data
detail and deployment artifacts. `STATICALLY REVIEWED`

**Edge/export:** “on-device” is the design objective and paper framing, not proof of compatibility with this project's
runtimes. The official loading example historically required `trust_remote_code=True`; exporter support for the exact
custom architecture is currently unresolved and must be tested. `VENDOR-REPORTED`

**Initial disposition:** strong research comparison for architecture scaling and sub-billion experiments; not a
commercial production candidate under the current license. `PLANNED EXPERIMENT`

Primary sources:

- <https://github.com/facebookresearch/MobileLLM>
- <https://huggingface.co/facebook/MobileLLM-125M>
- <https://huggingface.co/facebook/MobileLLM-350M>
- <https://huggingface.co/facebook/MobileLLM-600M>
- <https://huggingface.co/facebook/MobileLLM-1B>

## Credible current alternatives

### OLMo 2 1B

`allenai/OLMo-2-0425-1B` is a 1B-class, 16-layer, 2,048-wide English autoregressive transformer with 16 attention heads
and 4,096 context. Exact SFT, DPO, and final instruct stages are published, including `allenai/OLMo-2-0425-1B-SFT` and
`allenai/OLMo-2-0425-1B-Instruct`. Ai2 publishes code, data mixtures, intermediate checkpoints, logs, evaluation code,
and fine-tuning recipes under Apache-2.0. This is the strongest surveyed reproducibility/control candidate, though its
short context and English focus reduce its direct fit for some deployments. Export/mobile support remains unverified.
`DOC-VERIFIED`, `STATICALLY REVIEWED`

Sources:

- <https://huggingface.co/allenai/OLMo-2-0425-1B>
- <https://huggingface.co/allenai/OLMo-2-0425-1B-SFT>
- <https://huggingface.co/allenai/OLMo-2-0425-1B-Instruct>
- <https://github.com/allenai/OLMo>

### Liquid AI LFM2

Exact post-trained checkpoints include `LiquidAI/LFM2-350M`, `LiquidAI/LFM2-700M`, `LiquidAI/LFM2-1.2B`, and
`LiquidAI/LFM2-2.6B`. The 350M–1.2B models use 16 layers comprising ten double-gated short-convolution blocks and six
GQA blocks; the cards state 32,768 context. This hybrid architecture is highly relevant to edge efficiency but requires
backend-specific kernels and export validation. The cards explicitly recommend narrow fine-tuning and advise against
knowledge-intensive/programming tasks at these sizes. The LFM Open License v1.0 permits commercial use only below a \$10M
annual-revenue threshold; larger entities need a separate license. `DOC-VERIFIED`

Sources:

- <https://huggingface.co/LiquidAI/LFM2-350M>
- <https://huggingface.co/LiquidAI/LFM2-700M>
- <https://huggingface.co/LiquidAI/LFM2-1.2B>
- <https://huggingface.co/LiquidAI/LFM2-1.2B/raw/main/LICENSE>

### Ministral 3 3B

`mistralai/Ministral-3-3B-Base-2512`, `mistralai/Ministral-3-3B-Instruct-2512-BF16`, and
`mistralai/Ministral-3-3B-Reasoning-2512` are exact base, instruct, and reasoning variants. The family combines a 3.4B
language model with a 0.4B vision encoder and documents a 256K context. Official BF16, FP8, and GGUF repositories exist
and the card states Apache-2.0. It is a credible modern 4B-total multimodal candidate, but the vision encoder
complicates apples-to-apples text-model comparisons and edge memory. The unusually long context and vendor edge claims
require independent evaluation at controlled context lengths. `DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

Sources:

- <https://huggingface.co/mistralai/Ministral-3-3B-Base-2512>
- <https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512-BF16>
- <https://huggingface.co/mistralai/Ministral-3-3B-Reasoning-2512>
- <https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512-GGUF>

### OpenELM

Apple publishes `apple/OpenELM-270M`, `apple/OpenELM-450M`, `apple/OpenELM-1_1B`, and `apple/OpenELM-3B`, plus instruct
variants. OpenELM uses layer-wise parameter scaling and publishes training/inference code, preparation procedures,
checkpoints, and logs. It is relevant to architecture-efficiency research, but exact model repositories now use the
Apple Machine Learning Research Model License: non-commercial scientific research and academic development only, with
derivative obligations. The parent code repository's license must not be confused with the model-weight license. Some
examples require `trust_remote_code=True`; current mobile/Core ML support is unresolved. `DOC-VERIFIED`

Sources:

- <https://huggingface.co/apple/OpenELM>
- <https://huggingface.co/apple/OpenELM-1_1B>
- <https://huggingface.co/apple/OpenELM-3B>
- <https://huggingface.co/apple/OpenELM-3B/blob/main/LICENSE>

### Stable LM 2 1.6B

`stabilityai/stablelm-2-1_6b` is a 1,644,417,024-parameter, 24-layer decoder with 2,048 hidden size, 32 heads, and 4,096
sequence length, trained on a multilingual/code mixture. It provides a conventional base-model comparison but uses the
Stability AI Community License with separate commercial terms. It is less compelling than current Apache-2.0
alternatives unless its data/language profile is specifically valuable. Export status is unresolved. `DOC-VERIFIED`,
`STATICALLY REVIEWED`

Source:

- <https://huggingface.co/stabilityai/stablelm-2-1_6b>

### Pythia as a scientific control

The EleutherAI Pythia suite spans 70M, 160M, 410M, 1B, 1.4B, 2.8B, and 6.9B within this survey's range, with
deduplicated/non-deduplicated pairs and 154 intermediate checkpoints per size. It is Apache-2.0 and unusually strong for
training-dynamics and contamination studies. The card explicitly says deployment was not its design goal and
human-facing deployment is out of scope. Use Pythia as a research control, not a current production finalist.
`DOC-VERIFIED`

Source:

- <https://huggingface.co/EleutherAI/pythia-1b>

## Initial shortlist by experimental role

### Sub-billion pipeline and extreme-edge checks

- `HuggingFaceTB/SmolLM2-135M`
- `HuggingFaceTB/SmolLM2-360M`
- `google/gemma-3-270m`
- `Qwen/Qwen3-0.6B`
- `LiquidAI/LFM2-700M` as a hybrid-architecture arm
- MobileLLM 350M/600M only for non-commercial research

### Core 0.5B–2B domain-adaptation candidates

- `Qwen/Qwen2.5-0.5B`
- `Qwen/Qwen2.5-1.5B`
- `Qwen/Qwen3-1.7B` for post-trained/PEFT comparison
- `google/gemma-3-1b-pt`
- `meta-llama/Llama-3.2-1B`
- `HuggingFaceTB/SmolLM2-1.7B`
- `allenai/OLMo-2-0425-1B` as the reproducibility control

### Higher-capacity 3B–4B candidates

- `Qwen/Qwen3-4B`
- `meta-llama/Llama-3.2-3B`
- `microsoft/Phi-4-mini-instruct`
- `HuggingFaceTB/SmolLM3-3B-Base`
- `mistralai/Ministral-3-3B-Base-2512` if multimodality is required
- `google/gemma-3-4b-pt` if multimodality is required

This is a **candidate set**, not a quality ranking. `STATICALLY REVIEWED`

## Domain adaptation considerations

- Start CPT/DAPT from explicit base checkpoints where available: Qwen2.5 base, Gemma `-pt`, Llama base, SmolLM base,
  OLMo base, TinyLlama base, and Ministral base. `STATICALLY REVIEWED`
- Keep instruct-only Phi and Qwen3 checkpoints as separate PEFT/SFT baselines unless an exact clean base artifact is
  verified. Continued pretraining an aligned checkpoint may alter chat behavior and safety; the effect requires
  before/after evaluation. `PLANNED EXPERIMENT`
- Test tokenizer fertility on automotive, pharmaceutical, mechanical, species, units, diagnostic codes, and multilingual
  terminology before selecting a family. Vocabulary size alone does not establish domain token efficiency.
  `PLANNED EXPERIMENT`
- For safety-critical domains, no checkpoint's general model card substitutes for domain factuality, uncertainty,
  refusal, and escalation evaluation. `STATICALLY REVIEWED`
- Use the same frozen domain corpus, token budget, optimizer budget, context length, and evaluation set when comparing
  CPT candidates. Otherwise model-size and training-method effects are confounded. `PLANNED EXPERIMENT`

## Edge and export verification plan

For each finalist, freeze the exact model revision and test:

1. tokenizer and chat-template round trip;
2. adapter training and merge, if used;
3. conversion to the selected GGUF, ONNX, LiteRT, ExecuTorch, Core ML, or MLX route;
4. operator coverage and numerical smoke tests against the source checkpoint;
5. supported context after conversion, rather than the source card's maximum;
6. on-disk size, model-load peak memory, steady-state memory, and KV-cache growth;
7. prefill, time to first token, decode rate, battery, and thermal behavior on named devices;
8. domain-quality regression after quantization;
9. license/notice packaging for redistributed model artifacts.

All nine are `PLANNED EXPERIMENT`; none has been executed by this survey.

## Unresolved or currently unverified points

- Exact parameter counts and architecture fields have not yet been independently recomputed from every `config.json`.
- Model repository revisions/commit hashes were not frozen in this initial pass; URLs currently point to the default
  revision unless a source URL itself is immutable.
- Current llama.cpp, ExecuTorch, LiteRT-LM, Core ML, MLX, ONNX Runtime GenAI, and MLC support has not been tested for
  every exact checkpoint.
- No Android, iOS, ARM Linux, Raspberry Pi, Jetson, laptop, or AWS runtime benchmark has been reproduced.
- No vendor benchmark value has been normalized or independently reproduced, so no cross-family performance ranking is
  made.
- Quantized artifacts may reduce supported context or change output; each artifact requires its own card, license, and
  quality check.
- “On-device,” “edge-optimized,” “mobile,” and “fits in memory” do not establish interactive speed or sustained thermal
  viability.
- Training-data provenance and downstream dataset rights are incomplete for most families even when model weights use a
  permissive license.
- The legal effect of custom model licenses must be reviewed by qualified counsel for the intended product and
  jurisdiction.
- Qwen3 clean base-checkpoint availability for the exact surveyed sizes remains unresolved in this pass.
- A current first-party Llama 3.2 ExecuTorch procedure for the exact selected checkpoint remains unresolved in this
  pass.
- OpenELM and MobileLLM are research-only/non-commercial under the current exact weight licenses reviewed here and
  therefore are not production finalists without new licensing.

## Next research step

Freeze repository revisions, inspect every finalist's configuration and tokenizer, then construct a controlled
model-selection matrix with one common evaluation harness. The first practical sweep should include Qwen2.5-0.5B/1.5B,
Qwen3-0.6B/1.7B, Gemma 3 1B, Llama 3.2 1B, SmolLM2 1.7B, OLMo 2 1B, and one 3B–4B candidate. Benchmark scores, runtime
support, and hardware claims belong in that subsequent reproducible phase, not in this initial source survey.
