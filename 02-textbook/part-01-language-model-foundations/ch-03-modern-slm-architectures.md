# CH-03: Modern Small Language Model Architectures

<a id="ch03-status"></a>

## How to use this chapter

This chapter covers text-generative checkpoints from roughly 100 million to 8 billion parameters, emphasizing the
0.5B–4B range. It turns family research into a selection method: first identify an exact artifact, then test legal,
architectural, tokenizer, adaptation, export, and device constraints. A family name is a catalog label; the deployable
unit is a versioned checkpoint-tokenizer-template-runtime combination.

Current facts come from official model cards, repositories, configuration documentation, licenses, and primary papers
assembled in the [model-family survey](../../04-research-comparison-report/model-family-survey.md). Reported benchmark
and compatibility claims remain source claims until reproduced on the intended workload.

<a id="ch03-objectives"></a>

## 3.1 Learning objectives

After completing this chapter, the reader should be able to:

1. distinguish a family, an exact checkpoint, a tokenizer, and a deployable runtime artifact;
2. explain how parameter allocation, depth, width, attention, vocabulary, context, and modality affect adaptation and
   inference;
3. distinguish dense from mixture-of-experts (MoE) models, including total versus active parameters;
4. select exact base and instruction checkpoints for continued pretraining (CPT), supervised fine-tuning (SFT), and
   deployment baselines;
5. evaluate tokenizer, license, provenance, model-card, quantization, and export risks independently; and
6. build a defensible shortlist without inventing scores or importing incomparable vendor benchmarks.

<a id="ch03-selection-problem"></a>

## 3.2 The selection problem

“Choose a small language model” sounds like a parameter-count decision. It is a constrained systems decision. A 1B model
may be unsuitable because it has no clean base checkpoint, fragments domain terms, uses unsupported operators, or
carries incompatible redistribution terms. A 3B model may fit in storage at 4-bit precision but exceed runtime memory
once the key-value (KV) cache, activations, workspace, tokenizer, and operating-system pressure are included. A source
card may advertise 128K context while a mobile package supports 4K or 8K.

The actual object selected is a tuple:

\[ C=(\text{repository},\text{revision},\text{weights},\text{tokenizer}, \text{template},\text{license},\text{runtime
artifact}) \]

Changing any element can change behavior, compatibility, context, or legal obligations.

<a id="ch03-family-is-not-checkpoint"></a>

### 3.2.1 Why family names do not prove checkpoint availability

A family name is a product or research umbrella, not an immutable artifact. “Qwen,” “Gemma,” “Llama,” or “Phi” does not
establish that a desired size has:

- a base checkpoint suitable for controlled CPT;
- an instruction checkpoint and matching chat template;
- a commercially usable license;
- a verified tokenizer;
- an artifact in GGUF, ONNX, LiteRT, ExecuTorch, Core ML, or MLX;
- kernels for the intended precision and device; or
- a maintained conversion route.

Examples:

- Qwen2.5 has explicit base/instruct pairs at 0.5B and 1.5B. The surveyed Qwen3 releases combine pretraining and
  post-training behavior; a clean base for the exact surveyed sizes remains unresolved. `DOC-VERIFIED`
- Phi-4 Mini has an exact instruction checkpoint and official optimized ONNX artifacts, but this pass did not verify an
  equivalent first-party clean base. `DOC-VERIFIED`
- Gemma 3 includes text-only 270M/1B models and a 4B model with a vision path. These are different deployment objects.
  `DOC-VERIFIED`
- Llama 3.2 source cards document 128K context while separately identified quantized variants document 8K. Context
  belongs to the exact artifact. `DOC-VERIFIED`

Repository existence is not execution evidence. A model can load in one framework yet fail export, omit tokenizer
behavior, diverge numerically after conversion, or run too slowly on the target.

<a id="ch03-architecture"></a>

## 3.3 Architecture features that matter

Most candidates are causal decoder-only language models. Their shared objective does not make their deployment behavior
interchangeable.

**FIG-CH03-01 — Architecture-to-deployment path (`ILLUSTRATIVE`)** Alt text: a candidate checkpoint moves through
identity, license, architecture, tokenizer, runtime, export, and target-device checks before selection.

```text
Exact checkpoint
  ├─ tokenizer/vocabulary ──> domain fertility, embedding size
  ├─ block design
  │   ├─ attention type ────> KV-cache size and kernels
  │   ├─ MLP/activation ────> compute and quantization behavior
  │   └─ local/hybrid mix ──> context behavior and exporter risk
  ├─ base/instruct state ───> valid adaptation role
  ├─ license/provenance ────> use, redistribution, auditability
  └─ runtime artifact ──────> actual context, RAM, latency, energy
```

### 3.3.1 Parameters and memory

Parameter count is not a quality score. For \(P\) dense parameters stored at \(b\) bits, idealized weight-only memory
is:

\[ M_\text{weights}=\frac{Pb}{8} \]

A 1.5B model is about 3.0 GB at 16 bits or 0.75 GB at 4 bits before scales, metadata, alignment, tokenizer files, and
runtime overhead. This is `ILLUSTRATIVE`, not a measured artifact size.

\[ M_\text{runtime}\approx M_\text{weights}+M_\text{KV}+M_\text{activations} +M_\text{workspace}+M_\text{runtime
overhead} \]

Model file size, loaded weight memory, peak memory, and sustained interactive performance are distinct. A model can fit
in storage but fail to load; it can load but thermally throttle.

Embeddings matter disproportionately in SLMs. A vocabulary of \(V\) tokens and hidden width \(d\) requires \(Vd\)
input-embedding parameters before considering tied output weights. Phi-4 Mini's documented 200K vocabulary is therefore
both a coverage feature and a capacity/storage choice. `DOC-VERIFIED`

### 3.3.2 Depth, width, and sequence mixing

- Deep-and-thin models use more sequential layers at smaller width. MobileLLM deliberately uses this shape. It may
  improve parameter efficiency but adds sequential execution; target kernels decide latency. `DOC-VERIFIED`,
  `STATICALLY REVIEWED`
- OpenELM uses layer-wise parameter scaling rather than one uniform repeated width. `DOC-VERIFIED`
- Gemma 3 alternates local sliding-window and global attention. `DOC-VERIFIED`
- LFM2 replaces many attention blocks with double-gated short-convolution blocks. This is relevant to edge efficiency
  but creates backend-specific operator risk. `DOC-VERIFIED`
- SmolLM3 documents GQA, NoPE, 64K trained context, and extension to 128K with YaRN. `DOC-VERIFIED`

No shape is inherently fastest across a server GPU, laptop CPU, mobile GPU, and NPU.

### 3.3.3 MHA, MQA, and GQA

Multi-head attention (MHA) gives query heads corresponding key/value heads. Multi-query attention (MQA) shares one KV
pair. Grouped-query attention (GQA) uses several KV groups. A simplified autoregressive cache estimate is:

\[ M_\text{KV}\approx2LTH_\text{KV}d_hs \]

where \(L\) is layers, \(T\) cached tokens, \(H_\text{KV}\) KV heads, \(d_h\) head dimension, \(s\) bytes per scalar,
and 2 represents keys and values. Fewer KV heads can materially reduce cache memory. Qwen2.5-0.5B documents 14 query
heads and 2 KV heads; Llama 3.2 and many candidates use GQA. `DOC-VERIFIED` This does not guarantee overall speed.

### 3.3.4 Dense versus MoE

Dense models activate their block parameters for every token. MoE models route each token through a subset of expert
feed-forward networks. Every MoE description must report:

1. total parameters, affecting storage/loading;
2. active parameters per token, approximating routed compute;
3. experts and selected experts per token; and
4. router, communication, and kernel requirements.

“3B active” can conceal a much larger weight footprint. Routing also creates exporter and scheduler complexity. The
principal shortlist here is dense or hybrid-dense. Qwen's broader family includes MoE releases, but surveyed
`Qwen3-0.6B`, `Qwen3-1.7B`, and `Qwen3-4B` are dense. `DOC-VERIFIED` A family containing MoE models does not make every
checkpoint MoE.

### 3.3.5 Tokenizers

Tokenizer behavior affects domain strings, Unicode, multilingual text, codes, units, sequence length, and runtime
portability. Vocabulary size alone does not prove efficiency. Measure token fertility:

\[ F=\frac{\text{tokens produced}}{\text{defined text units}} \]

Use fixed strata: prose, `P0420`-style codes, part numbers, `μg/mL`, drug names, species names, and target languages.
`PLANNED EXPERIMENT`

Replacing a tokenizer changes embedding/output rows, breaks original template compatibility, and complicates export.
Test the original first; modify it only when measured fragmentation and downstream gains justify the cost.

### 3.3.6 Base, instruct, and reasoning checkpoints

- **Base/pretrained/PT:** cleanest for controlled CPT/domain-adaptive pretraining (DAPT).
- **Instruct/chat:** post-trained for response behavior; useful as product and SFT baselines, but CPT may damage
  alignment.
- **Reasoning:** specially post-trained, often for mathematics; not automatically superior for broad specialist
  factuality.

Always use the matching chat template. Raw base chat behavior and official instruct behavior are different experimental
strata.

<a id="ch03-families"></a>

## 3.4 Required families and exact checkpoints

### 3.4.1 Qwen

Verified exact repositories include:

- `Qwen/Qwen3-0.6B`: 0.6B total/0.44B non-embedding, 28 layers, GQA 16 query/8 KV heads, tied embeddings, 32,768 native
  context.
- `Qwen/Qwen3-1.7B`: 1.7B total/1.4B non-embedding, 28 layers, GQA 16/8, tied embeddings, 32,768 context.
- `Qwen/Qwen3-4B`: 4.0B total/3.6B non-embedding, 36 layers, GQA 32/8, tied embeddings, 32,768 native context; 131,072
  with documented YaRN configuration.
- `Qwen/Qwen2.5-0.5B` and `-Instruct`: 0.49B total/0.36B non-embedding, RoPE, SwiGLU, RMSNorm, QKV bias, tied
  embeddings, GQA 14/2, 32,768 context.
- `Qwen/Qwen2.5-1.5B` and its instruct variant.
- `Qwen/Qwen2.5-3B` and its instruct variant.

All bullets are `DOC-VERIFIED`.

Qwen2.5's explicit base/instruct pairs make 0.5B and 1.5B strong controlled DAPT candidates. Qwen3's switchable
thinking/non-thinking post-training makes it a useful packaged baseline, not an automatically clean DAPT control.
`STATICALLY REVIEWED`

Surveyed Qwen3 and Qwen2.5 0.5B/1.5B repositories state Apache-2.0. Qwen2.5 3B uses a checkpoint-specific Qwen Research
License restricting commercial use absent separate licensing. `DOC-VERIFIED`

Official Qwen3 GGUF repositories exist for 1.7B/4B. LiteRT-LM declares Qwen3/Qwen2.5 model types and lists Qwen2.5-1.5B
packaged at 4,096 context. `DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

Sources: [Qwen3-0.6B](https://huggingface.co/Qwen/Qwen3-0.6B), [1.7B](https://huggingface.co/Qwen/Qwen3-1.7B),
[4B](https://huggingface.co/Qwen/Qwen3-4B), [repository](https://github.com/QwenLM/Qwen3),
[technical report](https://raw.githubusercontent.com/QwenLM/Qwen3/main/Qwen3_Technical_Report.pdf),
[Qwen2.5-0.5B](https://huggingface.co/Qwen/Qwen2.5-0.5B),
[Qwen2.5-1.5B license](https://huggingface.co/Qwen/Qwen2.5-1.5B/blob/main/LICENSE),
[Qwen2.5-3B license](https://huggingface.co/Qwen/Qwen2.5-3B/raw/main/LICENSE),
[Qwen3-1.7B-GGUF](https://huggingface.co/Qwen/Qwen3-1.7B-GGUF), and
[LiteRT-LM](https://github.com/google-ai-edge/LiteRT-LM).

### 3.4.2 Gemma

- `google/gemma-3-270m` / `-it`: approximately 270M, text-only, 32K context.
- `google/gemma-3-1b-pt` / `-it`: approximately 1B, text-only, 32K.
- `google/gemma-3-4b-pt` / `-it`: approximately 4B-class, image-and-text input/text output, documented 128K input
  context.

`DOC-VERIFIED`

Gemma 3 alternates local and global attention. Explicit `-pt` models support controlled DAPT. The 270M model is a
pipeline/extreme-edge control; 1B is the cleaner text production baseline; 4B belongs in the main comparison only if
vision is required. `STATICALLY REVIEWED`

Weights require acceptance of Gemma terms and a prohibited-use policy; `license: gemma` is not an OSI license. Google
publishes QAT GGUF such as `google/gemma-3-1b-pt-qat-q4_0-gguf`; LiteRT-LM lists a 4-bit 1B package at 4,096 context.
`DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

Sources: [270M](https://huggingface.co/google/gemma-3-270m), [270M IT](https://huggingface.co/google/gemma-3-270m-it),
[1B PT](https://huggingface.co/google/gemma-3-1b-pt), [1B IT](https://huggingface.co/google/gemma-3-1b-it),
[4B PT](https://huggingface.co/google/gemma-3-4b-pt), [4B IT](https://huggingface.co/google/gemma-3-4b-it),
[QAT GGUF](https://huggingface.co/google/gemma-3-1b-pt-qat-q4_0-gguf), and [paper](https://arxiv.org/abs/2503.19786).

### 3.4.3 Llama

`meta-llama/Llama-3.2-1B` / `-Instruct` are 1.23B; `Llama-3.2-3B` / `-Instruct` are 3.21B. They are text-only dense
autoregressive transformers with GQA, shared embeddings, and 128K source-card context. `DOC-VERIFIED`

Explicit base/instruct pairs and the broad Llama ecosystem make them valuable LoRA, quantization, and conversion
references. Ecosystem popularity is not exact-version support. `STATICALLY REVIEWED`

Llama 3.2 uses Meta's custom Community License/AUP, not Apache-2.0. Vendor-quantized variants separately document 8K
context; do not transfer 128K to them. A current first-party ExecuTorch procedure for the exact selected checkpoint is
`UNRESOLVED COMPATIBILITY`.

Sources: [1B](https://huggingface.co/meta-llama/Llama-3.2-1B),
[1B Instruct](https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct),
[3B](https://huggingface.co/meta-llama/Llama-3.2-3B),
[3B Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct), and
[license](https://huggingface.co/meta-llama/Llama-3.2-3B/blob/main/LICENSE.txt).

### 3.4.4 Phi

- `microsoft/Phi-3.5-mini-instruct`: 3.8B, dense decoder, 128K, 3.4T vendor-reported training tokens.
- `microsoft/Phi-4-mini-instruct`: 3.8B, dense decoder, 128K, 5T reported tokens, 200K vocabulary, GQA, tied embeddings.
- `microsoft/Phi-4-mini-reasoning`: same architecture with math-reasoning post-training.
- `microsoft/Phi-4-mini-instruct-onnx`: official CPU/mobile INT4 RTN and GPU artifacts.

`DOC-VERIFIED`

Phi-4 Mini is strongest as an instruction/PEFT/deployment arm; without a verified clean base, it is weaker for
controlled DAPT attribution. The exact instruction and ONNX repositories state MIT. Training relies heavily on synthetic
and filtered public data described by category, not as a reconstructable corpus. Microsoft warns optimized outputs can
differ from the source. LiteRT-LM lists Phi-4 Mini at 4,096 packaged context. `DOC-VERIFIED`,
`UNVERIFIED IMPLEMENTATION`

Sources: [Phi-3.5 Mini](https://huggingface.co/microsoft/Phi-3.5-mini-instruct),
[Phi-4 Mini](https://huggingface.co/microsoft/Phi-4-mini-instruct),
[Reasoning](https://huggingface.co/microsoft/Phi-4-mini-reasoning),
[ONNX](https://huggingface.co/microsoft/Phi-4-mini-instruct-onnx), and
[ONNX license](https://huggingface.co/microsoft/Phi-4-mini-instruct-onnx/blob/main/LICENSE).

### 3.4.5 SmolLM

Exact checkpoints include `HuggingFaceTB/SmolLM2-135M`, `-360M`, `-1.7B` and `-Instruct` variants; plus
`HuggingFaceTB/SmolLM3-3B-Base` and post-trained `SmolLM3-3B`. `DOC-VERIFIED`

SmolLM2 is Llama-2-like with RoPE, SwiGLU, tied embeddings, and 8K final context; 135M/360M use GQA. The paper lists the
1.7B model as 24 layers, 2,048 width, and 32 heads. `PAPER-REPORTED` SmolLM3 uses GQA/NoPE and documents 64K trained
context, extendable to 128K with YaRN. `DOC-VERIFIED`

Open recipes, data tooling, and intermediate checkpoints make this family valuable for CPT and forgetting research.
Surveyed weights state Apache-2.0. SmolLM3 noted that some mid/post-training data would be uploaded later, so
completeness must be rechecked at freeze time. Official documentation references llama.cpp, MLX, and Transformers.js; a
SmolLM3 ONNX repository exists. `DOC-VERIFIED`, `UNVERIFIED IMPLEMENTATION`

Sources: [135M](https://huggingface.co/HuggingFaceTB/SmolLM2-135M),
[360M](https://huggingface.co/HuggingFaceTB/SmolLM2-360M), [1.7B](https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B),
[paper](https://arxiv.org/abs/2502.02737), [repository](https://github.com/huggingface/smollm),
[SmolLM3 Base](https://huggingface.co/HuggingFaceTB/SmolLM3-3B-Base),
[SmolLM3](https://huggingface.co/HuggingFaceTB/SmolLM3-3B), and
[ONNX](https://huggingface.co/HuggingFaceTB/SmolLM3-3B-ONNX).

### 3.4.6 TinyLlama

`TinyLlama/TinyLlama-1.1B-intermediate-step-1431k-3T` is a base checkpoint after 3T tokens; `TinyLlama-1.1B-Chat-v1.0`
uses Zephyr-style SFT/DPO; `TinyLlama_v1.1` describes later general, math/code, and Chinese branches. `DOC-VERIFIED`

Architecture: 22-layer Llama-style dense decoder, 2,048 width, 5,632 SwiGLU width, 32 query heads/four KV groups,
RoPE/RMSNorm, Llama-2 tokenizer, and 2,048 context. `PAPER-REPORTED`

The standard shape and base model make it a useful legacy control, but 2K context and an archived repository reduce
production appeal. Code is Apache-2.0; synthetic post-training dataset terms need separate review. No first-party mobile
guarantee was verified: `UNRESOLVED COMPATIBILITY`.

Sources: [repository](https://github.com/jzhang38/TinyLlama), [paper](https://arxiv.org/abs/2401.02385),
[3T base](https://huggingface.co/TinyLlama/TinyLlama-1.1B-intermediate-step-1431k-3T),
[Chat](https://huggingface.co/TinyLlama/TinyLlama-1.1B-Chat-v1.0), and
[v1.1](https://huggingface.co/TinyLlama/TinyLlama_v1.1).

### 3.4.7 MobileLLM

Official cards document:

- `facebook/MobileLLM-125M`: 124.6M, 30 layers, width 576, 9 query/3 KV heads;
- `MobileLLM-350M`: 345.3M, 32 layers, width 960, 15/5;
- `MobileLLM-600M`: 603.1M, 40 layers, width 1,152, 18/6;
- `MobileLLM-1B`: 1.01B, 54 layers, width 1,280, 20/5;
- `MobileLLM-1.5B`: 1.51B, 54 layers, width 1,600, 25/5.

`DOC-VERIFIED`

The family combines deep/thin blocks, SwiGLU, tied embeddings, and GQA; cards report 2K context and 1T training tokens.
It is valuable architecture research, but depth can add sequential overhead and short context constrains document CPT.
Current pages state the FAIR Noncommercial Research License (updated 2025-04-17), not a commercial production license.
Historical loading used `trust_remote_code=True`; exact exporter support remains `UNRESOLVED COMPATIBILITY`. “Mobile” is
a design objective, not a device result. `VENDOR-REPORTED`

Sources: [repository](https://github.com/facebookresearch/MobileLLM),
[125M](https://huggingface.co/facebook/MobileLLM-125M), [350M](https://huggingface.co/facebook/MobileLLM-350M),
[600M](https://huggingface.co/facebook/MobileLLM-600M), and [1B](https://huggingface.co/facebook/MobileLLM-1B).

<a id="ch03-alternatives"></a>

## 3.5 Credible alternatives and controls

### OLMo 2 1B

`allenai/OLMo-2-0425-1B` is a 1B-class, 16-layer, 2,048-wide English transformer with 16 attention heads and 4,096
context. Exact SFT and instruct stages exist. Ai2 publishes code, mixtures, checkpoints, logs, evaluation, and recipes
under Apache-2.0. `DOC-VERIFIED` It is the strongest reproducibility control here, though English focus, short context,
and unverified mobile export may limit product fit.

Sources: [base](https://huggingface.co/allenai/OLMo-2-0425-1B),
[SFT](https://huggingface.co/allenai/OLMo-2-0425-1B-SFT),
[Instruct](https://huggingface.co/allenai/OLMo-2-0425-1B-Instruct), [repository](https://github.com/allenai/OLMo).

### LFM2

`LiquidAI/LFM2-350M`, `-700M`, `-1.2B`, and `-2.6B` are exact post-trained repositories. Cards for 350M–1.2B describe 16
layers: ten double-gated short-convolution and six GQA blocks, with 32,768 context. Cards recommend narrow fine-tuning
and caution against knowledge-intensive/programming tasks at these sizes. `DOC-VERIFIED`

This is the primary hybrid edge arm but requires backend-specific kernels. LFM Open License v1.0 permits commercial use
only below a stated $10M annual-revenue threshold. Sources: [350M](https://huggingface.co/LiquidAI/LFM2-350M),
[700M](https://huggingface.co/LiquidAI/LFM2-700M), [1.2B](https://huggingface.co/LiquidAI/LFM2-1.2B),
[license](https://huggingface.co/LiquidAI/LFM2-1.2B/raw/main/LICENSE).

### Ministral 3 3B

`mistralai/Ministral-3-3B-Base-2512`, `-Instruct-2512-BF16`, and `-Reasoning-2512` combine a 3.4B language model with a
0.4B vision encoder and document 256K context. Official BF16, FP8, and GGUF repositories exist; cards state Apache-2.0.
`DOC-VERIFIED` The vision encoder confounds text-only size, quality, and edge comparisons. Long-context and edge claims
need common-context testing.

Sources: [Base](https://huggingface.co/mistralai/Ministral-3-3B-Base-2512),
[Instruct](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512-BF16),
[Reasoning](https://huggingface.co/mistralai/Ministral-3-3B-Reasoning-2512),
[GGUF](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512-GGUF).

### OpenELM, Stable LM 2, and Pythia

OpenELM provides 270M, 450M, 1.1B, and 3B checkpoints, layer-wise scaling, code, procedures, and logs. Exact weights use
Apple's noncommercial research license; some examples require custom code; Core ML support is unresolved. `DOC-VERIFIED`
Sources: [collection](https://huggingface.co/apple/OpenELM), [1.1B](https://huggingface.co/apple/OpenELM-1_1B),
[3B/license](https://huggingface.co/apple/OpenELM-3B).

`stabilityai/stablelm-2-1_6b` is documented as 1,644,417,024 parameters, 24 layers, width 2,048, 32 heads, and 4,096
context. It is a conventional base comparison under Stability's Community License; export is unresolved.
[Official card](https://huggingface.co/stabilityai/stablelm-2-1_6b). `DOC-VERIFIED`

Pythia spans 70M–6.9B with deduplicated/non-deduplicated pairs and 154 intermediate checkpoints per size under
Apache-2.0. It is unusually useful for training-dynamics and contamination controls but explicitly not designed as a
deployment finalist. [Pythia 1B](https://huggingface.co/EleutherAI/pythia-1b). `DOC-VERIFIED`

<a id="ch03-comparison-table"></a>

## 3.6 Checkpoint comparison

<a id="tbl-11"></a> **TBL-11 — Model families and sizes: verified landscape. (`DOC-VERIFIED`)** “Route” means documented
artifact evidence, not executed support.

<!-- markdownlint-disable MD013 -->
| Exact checkpoint(s) |        Class | Key design/context       | Training role           | License gate              | Route or unresolved issue             |
| ------------------- | -----------: | ------------------------ | ----------------------- | ------------------------- | ------------------------------------- |
| SmolLM2 135M/360M   |  0.135/0.36B | Dense, GQA, 8K           | Base/instruct controls  | Apache-2.0                | Documented routes; targets unexecuted |
| MobileLLM 125M–600M | 0.125–0.603B | Deep/thin GQA, 2K        | Architecture research   | Noncommercial             | Custom export unresolved              |
| Gemma 3 270M        |        0.27B | Text-only, 32K           | PT/instruct controls    | Gemma terms               | Device execution unverified           |
| Qwen2.5 0.5B        |        0.49B | GQA 14/2, 32K            | Base/instruct           | Apache-2.0                | Exact export test needed              |
| Qwen3 0.6B          |         0.6B | GQA, 32K                 | Post-trained            | Apache-2.0                | Clean base unresolved                 |
| LFM2 700M           |         0.7B | Hybrid conv/GQA, 32K     | Narrow adaptation arm   | Revenue condition         | Kernel/export unresolved              |
| OLMo 2 1B           |          ~1B | Dense, 4K                | Reproducibility control | Apache-2.0                | Mobile route unverified               |
| Gemma 3 1B          |          ~1B | Local/global, 32K        | PT/instruct             | Gemma terms               | Official QAT/LiteRT, unexecuted       |
| Llama 3.2 1B        |        1.23B | GQA, source 128K         | Base/instruct           | Llama custom              | Exact first-party route unresolved    |
| TinyLlama 1.1B      |         1.1B | Llama-like GQA, 2K       | Legacy control          | Apache code/data separate | Mobile unresolved                     |
| Qwen2.5 1.5B        |         1.5B | Dense, 32K               | Base/instruct           | Apache-2.0                | LiteRT package at 4K                  |
| SmolLM2 1.7B        |         1.7B | Dense, 8K                | Base/instruct           | Apache-2.0                | Targets unexecuted                    |
| Qwen3 1.7B          |         1.7B | GQA, 32K                 | Post-trained            | Apache-2.0                | Official GGUF; base unresolved        |
| Llama 3.2 3B        |        3.21B | GQA, source 128K         | Base/instruct           | Llama custom              | Artifact context must be checked      |
| SmolLM3 3B          |           3B | GQA/NoPE, trained 64K    | Base/post-trained       | Apache-2.0                | Official ONNX, unexecuted             |
| Phi-4 Mini          |         3.8B | GQA, 200K vocab, 128K    | Instruct/deployment     | MIT                       | Official ONNX; base unresolved        |
| Qwen3 4B            |           4B | GQA, native 32K          | Post-trained            | Apache-2.0                | Official GGUF; base unresolved        |
| Gemma 3 4B          |          ~4B | Multimodal, local/global | PT/instruct             | Gemma terms               | Vision export burden                  |
| Ministral 3 3B      |  ~3.8B total | 3.4B LM + 0.4B vision    | Base/instruct/reasoning | Apache-2.0                | Text comparison confounded            |
| Pythia 6.9B         |         6.9B | Dense research control   | Scaling control         | Apache-2.0                | Not a deployment finalist             |
<!-- markdownlint-restore -->

Repository/card fields are `DOC-VERIFIED`; roles are `STATICALLY REVIEWED`; routes remain `UNVERIFIED IMPLEMENTATION`.

<a id="ch03-inference-export"></a>

## 3.7 Inference, quantization, and export

### Prefill and decode

Prefill processes prompt tokens and builds KV cache; decode generates one token at a time. GQA can reduce cache traffic;
local attention can bound some work; more layers add sequential work; large vocabularies enlarge output projection;
vision adds preprocessing and memory. These are expectations, not rankings. `STATICALLY REVIEWED`

### Quantization is artifact-specific

“Supports INT4” is incomplete. Record:

1. source checkpoint/revision;
2. PTQ versus quantization-aware training (QAT);
3. format/method (for example GGUF, ONNX INT4 RTN, AWQ, GPTQ);
4. weight-only versus weight/activation precision;
5. group size and calibration snapshot;
6. runtime/kernel/device; and
7. quality, memory, latency, and context results.

Official Gemma QAT GGUF and Phi ONNX INT4 reduce provenance uncertainty but do not prove domain-quality retention.
`DOC-VERIFIED`, `PLANNED EXPERIMENT` A smaller file may not run faster without matching kernels. Hybrid blocks require
special scrutiny.

### Loading is not export support

Framework loading does not prove static operator support, tokenizer parity, cache semantics, shapes, or device kernels.
Use these escalating statuses:

1. documented source loading;
2. documented converted artifact;
3. project conversion executed;
4. numerical/behavioral parity checked;
5. named physical device executed;
6. deployment qualified for quality, RAM, latency, energy, thermals, and stability.

This chapter verifies only selected instances of the first two.

### Context is artifact-specific

Record native/trained context, extension method, packaged runtime maximum, tested prompt/output lengths, cache
precision, OOM threshold, and long-position quality. Compare at one common practical context before testing each model's
maximum. Long context does not prove reliable recall or supply missing domain knowledge.

<a id="ch03-domain-adaptation"></a>

## 3.8 Domain adaptation and instruction behavior

For controlled DAPT, prioritize exact base checkpoints: Qwen2.5 0.5B/1.5B, Gemma `-pt`, Llama base, SmolLM base, OLMo
base, TinyLlama base, and Ministral Base only if multimodality matters. `STATICALLY REVIEWED`

Use Qwen3, Phi-4 Mini, and exact instruct/reasoning variants as separate PEFT/product arms. Continued pretraining of an
aligned model may alter safety and chat behavior; measure before/after.

Capacity bands suggest experimental roles, not guaranteed ability:

- **100M–400M:** pipeline checks, ablations, distillation students, extreme-edge research.
- **0.5B–2B:** core adaptation/edge band.
- **3B–4B:** higher-capacity candidates with greater memory/bandwidth demand.
- **4B–8B:** upper controls and likely cloud/laptop arms.

Domain tests must include:

- automotive/industrial codes, units, ordered diagnosis, and unsafe-procedure refusal;
- pharmacy names, dosage units, contraindication recognition, uncertainty, and escalation;
- beekeeping/animal-care scientific names, regional variation, and sparse-source uncertainty;
- general-capability retention and cross-domain interference.

The no-runtime-RAG constraint makes stale knowledge, imperfect recall, conflicts, and hallucination especially
important. Training does not guarantee exact fact storage. Safety-critical outputs require authoritative evaluation,
qualified review, and escalation; no general model card licenses autonomous prescribing or repair authority.

Instruction capability combines pretraining, SFT, preference training, template, and decoding. To compare it:

- compare base models after one common SFT;
- compare official instruct models separately;
- apply official templates;
- fix decoding and output budgets;
- score format compliance separately from factuality;
- evaluate refusal/calibration separately from helpfulness; and
- isolate Qwen3 thinking modes and reasoning variants into separate protocol cells.

<a id="ch03-license-reproducibility"></a>

## 3.9 Licensing, provenance, and reproducibility

**TBL-CH03-02 — Checkpoint-specific license gates (`DOC-VERIFIED`)**

<!-- markdownlint-disable MD013 -->
| Group                                                                     | Verified status                   | Consequence                                           |
| ------------------------------------------------------------------------- | --------------------------------- | ----------------------------------------------------- |
| Qwen3 surveyed sizes; Qwen2.5 0.5B/1.5B; SmolLM; OLMo; surveyed Ministral | Apache-2.0 stated                 | Permissive starting point; notices/data rights remain |
| Phi-4 Mini Instruct/ONNX                                                  | MIT stated                        | Permissive weights; provenance remains separate       |
| Qwen2.5 3B                                                                | Qwen Research License             | Do not inherit neighboring Qwen licenses              |
| Gemma 3                                                                   | Gemma terms/prohibited-use policy | Gated and custom-term review                          |
| Llama 3.2                                                                 | Llama Community License/AUP       | Custom obligations                                    |
| MobileLLM                                                                 | FAIR Noncommercial Research       | Research-only under surveyed terms                    |
| OpenELM weights                                                           | Apple research license            | Noncommercial/derivative obligations                  |
| LFM2                                                                      | Revenue-conditioned LFM license   | Entity eligibility check                              |
| Stable LM 2                                                               | Stability Community License       | Commercial-term review                                |
<!-- markdownlint-restore -->

`DOC-VERIFIED`; this is issue-spotting, not legal advice. Weights, code, tokenizer, training data, post-training data,
and converted artifacts may have different terms.

Reproducibility has four layers:

- **inference:** weights, tokenizer, template, runtime, precision, seed, decoding;
- **adaptation:** base revision, data, optimizer, packing, seeds, hardware;
- **pretraining:** architecture, corpus, filtering, order, code, logs;
- **benchmark:** prompts, few-shot examples, harness, metrics, contamination policy, raw outputs.

OLMo and SmolLM provide especially useful research trails, but openness does not imply they will win.
`STATICALLY REVIEWED` Freeze commit hashes and artifact checksums; default-branch URLs are not immutable experiment IDs.

A strong model card identifies architecture, tokenizer, lineage, context, modalities, training categories, excluded
uses, protocol, license, limitations, and artifacts. The survey found strong configuration/usage cards across Phi,
Llama, Gemma, Qwen, and SmolLM, but incomplete document-level data provenance across most families. OLMo offers the
strongest overall reproducibility trail. `STATICALLY REVIEWED`

<a id="ch03-benchmark-comparability"></a>

## 3.10 Benchmark comparability

Vendor scores differ in benchmark versions, prompts, few-shot counts, base/instruct state, reasoning budget, precision,
harness, contamination controls, and answer extraction. Merging them into one ranking creates false precision.

Rules:

1. vendor tables prove only that the vendor evaluated a checkpoint (`VENDOR-REPORTED`);
2. cross-family claims require one frozen harness and retained raw outputs (`PLANNED EXPERIMENT`);
3. generic benchmarks are secondary to domain factuality, safety, calibration, and deployment measurements.

Report uncertainty, not only point estimates. Human review needs blinded rubrics and agreement measures. Device
benchmarks need named hardware, OS, runtime, threads, thermal/charging state, prompt/output lengths, warm-up, and
repeated trials.

<a id="ch03-selection-process"></a>

## 3.11 Evidence-based model-selection process

**FIG-CH03-02 — Gated workflow (`ILLUSTRATIVE`)** Alt text: a candidate passes license, artifact, tokenizer, conversion,
quality, and target compatibility gates; failures stop promotion.

```text
Workload/devices
  -> legal + exact-artifact gate
  -> base/instruct role gate
  -> tokenizer + architecture audit
  -> source baseline
  -> common adaptation protocol
  -> export + parity check
  -> physical-device benchmark
  -> domain/safety regression
  -> Pareto selection
```

### Phase A: hard gates

Define intended domain/languages, checkpoint role, commercial/redistribution requirements, storage/RAM/context ceilings,
target hardware, approved runtimes, safety category, and compute budget before downloading models. A failed license is
not offset by benchmark quality.

### Phase B: non-scored matrix

**TBL-CH03-03 — Initial role matrix. (`STATICALLY REVIEWED`)** Labels describe evidence fit, not quality scores.

<!-- markdownlint-disable MD013 -->
| Candidate          | Clean base        | Instruct comparator | Reproducibility  | First-party edge evidence | Main unresolved gate     | Role                  |
| ------------------ | ----------------- | ------------------- | ---------------- | ------------------------- | ------------------------ | --------------------- |
| Qwen2.5 0.5B/1.5B  | Yes               | Yes                 | Moderate         | LiteRT evidence           | Exact target/domain test | Core DAPT             |
| Qwen3 0.6B/1.7B/4B | Unresolved        | Post-trained        | Moderate         | GGUF at 1.7B/4B           | Clean base/mode control  | Product baseline      |
| Gemma 3 1B         | Yes               | Yes                 | Moderate         | QAT GGUF/LiteRT           | Terms/device test        | Core edge             |
| Llama 3.2 1B/3B    | Yes               | Yes                 | Moderate         | Vendor quantized          | Exact route/license      | Ecosystem reference   |
| Phi-4 Mini         | Not verified      | Yes                 | Strong artifacts | ONNX INT4                 | Base/parity/resources    | 3.8B deployment       |
| SmolLM2 1.7B       | Yes               | Yes                 | Strong           | Documented routes         | Device qualification     | Reproducible core     |
| SmolLM3 3B         | Yes               | Yes                 | Strong           | ONNX                      | Context/runtime parity   | Open 3B               |
| OLMo 2 1B          | Yes               | Yes                 | Very strong      | Not verified              | Mobile/4K context        | Scientific control    |
| TinyLlama 1.1B     | Yes               | Yes                 | Historical       | Not verified              | Archived/2K              | Legacy control        |
| MobileLLM          | Yes               | Limited role        | Moderate         | Not verified              | License/custom export    | Architecture research |
| LFM2               | Not verified here | Post-trained        | Moderate         | Backend-dependent         | License/hybrid kernels   | Hybrid arm            |
| Gemma 4B/Ministral | Yes               | Yes                 | Moderate         | Artifacts vary            | Vision/memory confound   | Multimodal only       |
<!-- markdownlint-restore -->

Facts are `DOC-VERIFIED`; roles are `STATICALLY REVIEWED`; gates are `PLANNED EXPERIMENT`.

### Phase C: controlled adaptation

Use one immutable corpus, common token budget, context, optimizer family, schedule, seed policy, stopping rule, SFT set,
and evaluation set. Evaluate domain factuality, procedural order, uncertainty, hallucination, safety, and general
retention. Retain per-item outputs and repeat enough trials to quantify uncertainty. Tokenizers produce different token
counts, so control tokens processed, not only records or epochs.

### Phase D: deployment qualification

For each survivor:

1. freeze revision and hashes;
2. merge adapters if required;
3. export or obtain the exact official artifact;
4. verify tokenizer/template round trips;
5. compare source/export logits where feasible;
6. run deterministic generation tests;
7. measure file size, load peak, steady memory, and KV growth;
8. measure startup, time to first token, prefill, decode, and end-to-end latency;
9. measure battery, energy, and thermals on named devices;
10. rerun domain/safety tests after quantization; and
11. test OOM/crash behavior across context lengths.

All are `PLANNED EXPERIMENT`.

### Phase E: Pareto selection

Compare domain quality/safety, calibration, general retention, RAM, artifact size, latency, energy/thermals, cost,
license fit, and maintenance burden. A model is Pareto-dominated when another is no worse on all required objectives and
better on at least one. Prefer named outcomes—“lowest-memory qualified,” “highest-domain-quality qualified,” and
“balanced edge”—over one invented aggregate score.

### Worked shortlist example

Suppose a phone budget allows 1.5 GiB for model weights and 256 MiB for KV cache at a 4,096-token context. Idealized
4-bit weights for 1B, 1.7B, and 3B models are about 0.47, 0.79, and 1.40 GiB respectively
(\(P\times4/8\), converted to GiB). The 3B candidate is already too close to the weight ceiling once quantization
scales, metadata, runtime workspaces, and the application are included. It should not advance merely because its quality
score is highest.

The remaining 1B and 1.7B candidates then face tokenizer, license, export-parity, domain-quality, safety, and sustained
device tests. If the 1.7B model improves domain accuracy but exceeds the thermal or cache budget, neither model
universally “wins”: the 1B model is the qualified phone choice and the 1.7B model may remain the cloud or laptop choice.
This example illustrates constrained selection; it is not a benchmark of any named checkpoint.

<a id="ch03-shortlist"></a>

## 3.12 Practical baseline shortlist

This is a candidate set, not a ranking.

**Pipeline controls:** SmolLM2-135M/360M and Gemma 3 270M.

**Core 0.5B–2B adaptation sweep:** Qwen2.5-0.5B/1.5B, Gemma 3 1B PT, Llama 3.2 1B, SmolLM2 1.7B, and OLMo 2 1B as the
reproducibility control. Add exact instruct variants in a separate stratum.

**Higher-capacity/post-trained sweep:** Qwen3 1.7B, SmolLM3 3B Base, Llama 3.2 3B, Phi-4 Mini Instruct, and Qwen3 4B.

**Optional hypotheses:** LFM2-700M for hybrid mixing; MobileLLM 350M/600M for noncommercial architecture research; Gemma
3 4B or Ministral only for multimodality; TinyLlama/Pythia for controls.

<a id="ch03-manifest"></a>

## 3.13 Minimum reproducibility manifest

Record one machine-readable manifest per run. It must contain:

- experiment ID and lifecycle state;
- source repository, immutable revision, weight/tokenizer/license hashes, and checkpoint role;
- parameter count, attention type, modality, and native/package context limits;
- dataset snapshot, processed tokens, seed, environment lock, and training hardware;
- export format, converter revision, quantization method, bit width, group size, and calibration snapshot;
- evaluation-harness revision and domain-suite snapshot; and
- exact inference device, OS, RAM, runtime build, and artifact hashes.

Do not populate fields with “latest,” family names, or blank template values. The run should fail validation when a
required identity is unknown.

<a id="ch03-checklist"></a>

## 3.14 Verification checklist

### Identity and architecture

- [ ] Exact repository and immutable revision recorded; weights/tokenizer hashed.
- [ ] Parameter count reconciled with the frozen configuration.
- [ ] Base/instruct/reasoning lineage identified.
- [ ] Dense/hybrid/MoE status verified; MoE total/active parameters separated.
- [ ] Layers, widths, Q/KV heads, position method, tying, modality, and contexts recorded.

### Tokenizer and adaptation

- [ ] Special tokens and official template frozen.
- [ ] Source/target tokenizer round trip tested.
- [ ] Fertility measured across domain strata, codes, units, Unicode, and languages.
- [ ] A real base checkpoint exists for claimed CPT/DAPT.
- [ ] Data snapshot, tokens processed, replay, SFT masking, and forgetting tests recorded.
- [ ] Safety-critical ground truth and expert review defined.

### License and provenance

- [ ] Weight, code, tokenizer, and dataset licenses archived.
- [ ] Gated terms, acceptable use, commercial use, redistribution, notices, and derivatives reviewed.
- [ ] Training-data provenance gaps documented.

### Export and runtime

- [ ] Exact runtime version supports architecture and tokenizer.
- [ ] Conversion executed rather than inferred from family similarity.
- [ ] Quantizer, precision, group, and calibration snapshot recorded.
- [ ] Source/export parity and packaged context tested.
- [ ] Custom/fallback operators identified.
- [ ] Load peak, steady RAM, KV growth, latency, energy, and thermals measured.
- [ ] Domain and safety regression rerun after quantization.

### Comparability

- [ ] Prompt set, template policy, decoding, and output budget frozen.
- [ ] Base/instruct/reasoning/multimodal strata kept separate.
- [ ] Raw outputs, scores, trials, seeds, uncertainty, failures, and exclusions retained.
- [ ] No vendor score represented as reproduced.

<a id="ch03-unresolved"></a>

## 3.15 Unresolved compatibility and evidence gaps

1. Default repository URLs are not immutable experiment revisions.
2. Counts/configuration fields were not independently recomputed for every model.
3. llama.cpp, ExecuTorch, LiteRT-LM, Core ML, MLX, ONNX Runtime, and MLC were not executed for every exact checkpoint.
4. No Android, iOS, ARM Linux, Raspberry Pi, Jetson, laptop, or AWS benchmark was reproduced.
5. No vendor benchmark was normalized or independently reproduced.
6. Quantized artifacts may change output or supported context.
7. “Mobile,” “edge,” “on-device,” and “fits in RAM” do not prove sustained interactive speed.
8. Document-level training provenance is incomplete for most families.
9. Custom licenses require qualified legal review.
10. Clean Qwen3 and Phi-4 Mini base availability for surveyed sizes remains unresolved.
11. An exact first-party Llama 3.2 ExecuTorch path remains unresolved.
12. OpenELM and MobileLLM are research-only/noncommercial under reviewed terms.
13. LFM2 hybrid-kernel/export behavior is unresolved on each target.
14. Vision encoders confound text-only size, quality, memory, and latency.

<a id="ch03-summary"></a>

<a id="ch03-chapter-status-and-evidence-boundary"></a>

## 3.16 Chapter summary

- Select exact checkpoint/runtime tuples, not family names.
- The core shortlist is dense; do not mislabel it MoE because a broader family contains MoE releases.
- GQA, tying, vocabulary, depth/width, local or hybrid mixing, context, and modality affect adaptation and deployment.
- Use explicit base checkpoints for controlled CPT/DAPT and evaluate official instruct/reasoning models separately.
- Official GGUF, ONNX, or LiteRT artifacts reduce uncertainty but do not prove quality, context, latency, memory,
  energy, or thermal viability.
- License, provenance, model-card quality, reproducibility, safety, and runtime support are independent gates.
- Vendor benchmarks are not a common ranking. Final choices require controlled domain and device experiments and should
  form a Pareto set rather than one universal winner.

<a id="ch03-questions"></a>

## 3.17 Review questions

1. Why does a family name not establish a clean base checkpoint?
2. Distinguish total, non-embedding, and active MoE parameters.
3. How does GQA affect KV cache?
4. How can a large vocabulary both help and hurt an SLM?
5. Why separate base-after-common-SFT from official instruct evaluation?
6. What does an official INT4/GGUF artifact prove, and not prove?
7. Why can source and mobile context limits differ?
8. Why is “MobileLLM” not phone-latency evidence?
9. Which exact checkpoints are best suited to controlled DAPT, and why?
10. Why do multimodal models complicate text-only comparisons?
11. Why is OLMo useful even if another model scores higher?
12. Name five confounders in vendor benchmark comparison.
13. Why can a 4-bit file that fits in storage still fail on a phone?
14. What evidence is needed to claim runtime support?
15. Why prefer Pareto analysis to one weighted winner?

<a id="ch03-exercises"></a>

## 3.18 Engineering exercises

1. **Artifact audit:** freeze base/instruct repositories, revisions, licenses, and hashes for three families. Report
   missing pairs. `PLANNED EXPERIMENT`
2. **Tokenizer fertility:** compare 10,000 stratified records covering prose, codes, units, drug names, species, and two
   languages. Report distributions and failures. `PLANNED EXPERIMENT`
3. **KV cache:** estimate and measure cache at 512/2,048/4,096 tokens for two GQA models and one MHA control. Explain
   discrepancies. `PLANNED EXPERIMENT`
4. **Base versus instruct:** design a protocol separating pretraining, common SFT, and official post-training. Identify
   five confounders before execution.
5. **Export qualification:** compare one official and one local conversion for tokenizer parity, logits where
   accessible, deterministic generation, context, peak RAM, and domain regression. `PLANNED EXPERIMENT`
6. **Domain matrices:** apply TBL-CH03-03 independently to automotive and pharmacy education. Explain differing Pareto
   sets.
7. **MoE accounting:** inspect one MoE candidate and report total/active parameters, experts, footprint, router support,
   and kernels. Compare with a dense model. `PLANNED EXPERIMENT`

<a id="ch03-references"></a>

## 3.19 References and cross-links

Direct official and primary URLs are embedded in each family section. The consolidated evidence record is the
[model-family survey](../../04-research-comparison-report/model-family-survey.md). See also:

- [Part I index](README.md)
- [CH-02: Transformer Architecture in Depth](ch-02-transformer-architecture-in-depth.md)
- [CH-03 assessment](../../13-assessments/ch-03-assessment.md)
- [Evidence standard](../../00-governance/evidence-and-citation-standard.md)
- [Reproducibility standard](../../00-governance/reproducibility-standard.md)
- [Safety and risk standard](../../00-governance/safety-and-risk-standard.md)
- [No-runtime-RAG standard](../../00-governance/no-runtime-rag-standard.md)
