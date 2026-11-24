# Glossary

Each entry moves from intuition to precision: **plain meaning**, **technical definition**, **why it matters**, an
**example**, and **related terms**. Normative distinctions live in the
[definition register](definition-register.md).

## Models and architecture

### Attention

- **Plain meaning:** lets a token weigh which other tokens are useful. **Technical definition:** a differentiable
  weighted sum of value vectors, commonly using softmax-scaled query–key scores. **Why it matters:** attention carries
  context but can dominate prefill time and cache memory. **Example:** “it” can attend strongly to the earlier noun it
  refers to. **Related:** causal mask, MHA, GQA, MQA, KV cache.

### Autoregressive model

- **Plain meaning:** writes one token at a time. **Technical definition:** factorizes a sequence probability into
  conditional next-token probabilities. **Why it matters:** generation is sequential, so decode latency differs from
  prompt processing. **Example:** after “tighten to 40,” the model predicts a likely unit. **Related:** logits,
  teacher forcing, decode.

### Base model

- **Plain meaning:** a pretrained model before instruction coaching. **Technical definition:** an exact checkpoint
  primarily optimized for next-token prediction without a documented instruction post-training stage. **Why it
  matters:** base and instruct checkpoints need different prompts and adaptation plans. **Example:** a base checkpoint
  may continue text instead of answering a question. **Related:** instruct model, checkpoint, SFT.

### Chat template

- **Plain meaning:** the exact formatting recipe for a conversation. **Technical definition:** deterministic rendering
  of roles, messages, separators, and special tokens into model input. **Why it matters:** the wrong template can
  severely degrade behavior without changing weights. **Example:** a template inserts system, user, and end-of-turn
  tokens around messages. **Related:** tokenizer, special token, instruct model.

### Checkpoint

- **Plain meaning:** one exact saved state of a model. **Technical definition:** parameters and configuration identified
  by owner, repository, identifier, and immutable revision. **Why it matters:** family-level claims do not establish a
  checkpoint's license or behavior. **Example:** base and instruct variants at different commits are two checkpoints.
  **Related:** model family, artifact, model card.

### Context window

- **Plain meaning:** how many tokens a model can accept at once. **Technical definition:** the maximum sequence length
  supported by a declared model–runtime configuration. **Why it matters:** a long advertised window does not guarantee
  useful recall or feasible memory. **Example:** a 32k-token model may OOM on a phone at 32k because of its KV cache.
  **Related:** token, KV cache, position encoding.

### Decoder-only Transformer

- **Plain meaning:** the common architecture that predicts the continuation of text. **Technical definition:** a stack
  of causally masked Transformer blocks trained for next-token prediction. **Why it matters:** most generative SLM
  runtimes optimize this pattern. **Example:** GPT-style chat models are decoder-only after chat formatting. **Related:**
  causal mask, autoregressive model, prefill.

### Dense model

- **Plain meaning:** uses its ordinary blocks for every token. **Technical definition:** a model without token-wise
  sparse expert routing in its main learned blocks. **Why it matters:** total and active parameter counts are usually
  similar, simplifying memory estimates. **Example:** a dense 1B model uses its full layer stack for every token.
  **Related:** MoE, active parameters, resident parameters.

### Embedding

- **Plain meaning:** a vector representation of an identifier. **Technical definition:** a learned mapping from token or
  position IDs to continuous vectors. **Why it matters:** embeddings connect discrete text to neural computation and
  can consume substantial parameters. **Example:** token ID 431 maps to a 2,048-dimensional vector. **Related:**
  tokenizer, vocabulary, output projection.

### Feed-forward network

- **Plain meaning:** the per-token nonlinear processing part of a Transformer block. **Technical definition:** a
  position-wise MLP, often gated, applied after attention. **Why it matters:** FFNs commonly hold most model parameters
  and compute. **Example:** a SwiGLU FFN expands hidden width and projects it back. **Related:** MLP, SwiGLU, pruning.

### Grouped-query attention

- **Plain meaning:** several query heads share fewer key/value heads. **Technical definition:** attention with more query
  heads than KV heads. **Why it matters:** it reduces KV-cache size while retaining multiple query heads. **Example:**
  16 query heads share 4 KV heads. **Related:** MHA, MQA, KV cache.

### Instruction-tuned model

- **Plain meaning:** a pretrained model coached to follow requests. **Technical definition:** an exact checkpoint
  post-trained on instruction, conversation, preference, or related data. **Why it matters:** it usually needs its exact
  chat template and is not automatically more factual. **Example:** it answers “Explain a torque wrench” rather than
  merely continuing the sentence. **Related:** SFT, preference optimization, base model.

### Key–value cache

- **Plain meaning:** saved attention state for tokens already processed. **Technical definition:** per-layer key and
  value tensors retained to avoid recomputing prior tokens during decode. **Why it matters:** cache memory grows with
  context and can determine mobile feasibility. **Example:** GQA stores fewer KV heads than MHA. **Related:** prefill,
  decode, context window.

### Logit

- **Plain meaning:** a raw score for a possible next token. **Technical definition:** an unnormalized output value before
  softmax. **Why it matters:** logits are not probabilities or calibrated truth confidence. **Example:** logits 3, 2,
  and 0 become probabilities only after softmax. **Related:** softmax, sampling, distillation.

### Mixture of Experts

- **Plain meaning:** routes a token through selected specialist sub-networks. **Technical definition:** an architecture
  whose router activates a subset of expert modules per token. **Why it matters:** active compute may be low while all
  expert weights still require storage or memory. **Example:** two of eight experts process each token. **Related:**
  router, active parameters, resident parameters.

### Perplexity

- **Plain meaning:** how surprised a model is by held-out text. **Technical definition:** the exponential of mean token
  negative log-likelihood for a fixed corpus and tokenizer. **Why it matters:** it measures predictive fit, not
  factuality or safety. **Example:** compare adaptation checkpoints on the same automotive validation split. **Related:**
  NLL, tokenizer, evaluation.

### Small language model

- **Plain meaning:** a language model intentionally sized for constrained use. **Technical definition:** a model chosen
  for explicit memory, latency, energy, cost, or update limits; no universal parameter boundary exists. **Why it
  matters:** “small” must be tied to a target. **Example:** a 1B four-bit model may be small for a phone but large for a
  microcontroller. **Related:** LLM, model fits, quantization.

### Token and tokenizer

- **Plain meaning:** a token is a model's text unit; a tokenizer converts text to and from those units. **Technical
  definition:** the tokenizer is a versioned normalization, segmentation, vocabulary, special-token, and decoding
  contract mapping text and IDs. **Why it matters:** counts, templates, and model compatibility depend on it.
  **Example:** “OBD-II” may split into several tokens. **Related:** BPE, vocabulary, token fertility.

## Training and adaptation

### Catastrophic forgetting

- **Plain meaning:** learning a new domain damages older abilities. **Technical definition:** degradation of previously
  useful behavior after optimization on new data or objectives. **Why it matters:** specialization can reduce general
  language and safety performance. **Example:** automotive tuning lowers performance on general instructions.
  **Related:** DAPT, replay, regression evaluation.

### Continued, domain-adaptive, and task-adaptive pretraining

- **Plain meaning:** continue next-token learning broadly, on a domain, or near a task. **Technical definition:** CPT is
  any additional self-supervised pretraining; DAPT uses a domain distribution; TAPT uses text close to a downstream
  task. **Why it matters:** these objectives teach text distributions, not answer formatting. **Example:** train on
  licensed repair manuals before instruction tuning. **Related:** pretraining, SFT, domain shift.

### Full fine-tuning

- **Plain meaning:** update the whole model. **Technical definition:** gradient optimization of all declared trainable
  model parameters. **Why it matters:** it offers flexibility but requires more training memory and stores a full
  variant. **Example:** update every layer of a 1B checkpoint on domain data. **Related:** PEFT, LoRA, optimizer state.

### Supervised fine-tuning

- **Plain meaning:** teach with examples of desired inputs and outputs. **Technical definition:** gradient training on
  labeled target sequences, often with prompt tokens masked. **Why it matters:** it teaches response behavior but does
  not guarantee comprehensive domain knowledge. **Example:** train on question–answer repair conversations. **Related:**
  instruction tuning, completion-only loss, DAPT.

### Preference optimization

- **Plain meaning:** train a model to favor better responses over worse ones. **Technical definition:** optimization
  from rankings, chosen/rejected pairs, or reward signals. **Why it matters:** it can improve behavior but may amplify
  annotation bias or reward hacking. **Example:** prefer a cautious diagnostic answer over an unsafe confident one.
  **Related:** DPO, RLHF, reward model.

### Adapter and PEFT

- **Plain meaning:** train a small attachment instead of changing the whole model. **Technical definition:** PEFT updates
  a small parameter subset or module while freezing most backbone weights. **Why it matters:** it lowers training and
  per-domain storage cost. **Example:** keep one base model and separate automotive and beekeeping adapters. **Related:**
  LoRA, frozen backbone, adapter merging.

### LoRA

- **Plain meaning:** represent a weight change with two small matrices. **Technical definition:** Low-Rank Adaptation
  learns a factorized additive update to selected frozen weights. **Why it matters:** it greatly reduces trainable
  parameters but does not shrink a merged dense model. **Example:** rank-16 factors adapt attention projections.
  **Related:** rank, alpha, PEFT, DoRA.

### QLoRA

- **Plain meaning:** train LoRA while the frozen base is stored in four-bit form. **Technical definition:** adapter
  optimization through a quantized backbone, commonly using NF4 and higher-precision computation. **Why it matters:** it
  reduces training memory; it is not itself a mobile deployment format. **Example:** train adapters on one GPU, then
  merge and separately export. **Related:** LoRA, NF4, deployment quantization.

### Knowledge distillation

- **Plain meaning:** train a smaller student using a stronger teacher. **Technical definition:** optimization from
  teacher responses, probability distributions, features, or relations. **Why it matters:** it may transfer capability
  into a smaller model, along with teacher errors. **Example:** train a 500M student on validated teacher explanations.
  **Related:** sequence distillation, logit distillation, student model.

## Compression

### Calibration dataset

- **Plain meaning:** representative examples used to configure compression. **Technical definition:** non-test data used
  to estimate ranges, salience, reconstruction statistics, or pruning scores. **Why it matters:** poor calibration can
  damage rare domain behavior and test reuse leaks evaluation data. **Example:** 512 varied repair prompts tune an AWQ
  conversion. **Related:** PTQ, leakage, representative sampling.

### Effective bits per weight

- **Plain meaning:** the real average storage cost of each weight. **Technical definition:** total encoded bytes,
  including scales, metadata, outliers, and padding, multiplied by eight and divided by weight count. **Why it matters:**
  nominal four-bit files often exceed exactly 0.5 byte per weight. **Example:** a 600 MB encoding of 1B weights uses
  4.8 effective bits/weight. **Related:** artifact size, quantization format.

### GGUF

- **Plain meaning:** a container for model tensors and metadata. **Technical definition:** a typed model file format used
  by llama.cpp and GGML tooling. **Why it matters:** GGUF is neither one quantization method nor an execution engine.
  **Example:** two GGUF files can use different quantization encodings. **Related:** llama.cpp, quantizer, runtime.

### Post-training and quantization-aware training

- **Plain meaning:** quantize after training, or train while modeling quantization error. **Technical definition:** PTQ
  transforms a trained model; QAT includes target numerical effects during optimization. **Why it matters:** QAT can
  recover quality but costs training effort. **Example:** compare four-bit PTQ with a vendor-provided QAT checkpoint.
  **Related:** calibration, fake quantization, deployment kernel.

### Quantization

- **Plain meaning:** store or compute numbers with fewer representable values. **Technical definition:** map values to a
  lower-precision encoding with declared granularity, scales, zero-points, and execution semantics. **Why it matters:**
  it can reduce storage and memory, but speed requires matching kernels. **Example:** group-wise four-bit weights with
  FP16 accumulation. **Related:** PTQ, QAT, weight-only quantization.

### Pruning

- **Plain meaning:** remove model weights or structures. **Technical definition:** eliminate scalars, patterns, channels,
  neurons, heads, or layers under a declared criterion. **Why it matters:** zeros do not improve latency unless the
  runtime exploits them; structural removal may create portable smaller shapes. **Example:** remove entire FFN channels
  and recovery-tune. **Related:** unstructured, N:M, structured pruning.

### Structured, semi-structured, and unstructured pruning

- **Plain meaning:** remove whole structures, fixed patterns, or arbitrary weights. **Technical definition:** structured
  pruning changes complete dimensions; semi-structured follows patterns such as N:M; unstructured removes scalars
  without changing dense shapes. **Why it matters:** hardware speedups differ radically. **Example:** 2:4 sparsity needs
  a compatible sparse kernel. **Related:** pruning, sparse kernel, dense shape.

### Weight-only quantization

- **Plain meaning:** compress weights but keep other tensors at another precision. **Technical definition:** reduced
  precision for parameter tensors while activations and accumulators use declared dtypes. **Why it matters:** it often
  saves memory with simpler kernels than full W8A8 quantization. **Example:** INT4 weights with FP16 activations.
  **Related:** activation quantization, W4A16, mixed precision.

## Evidence, evaluation, and deployment

### Artifact

- **Plain meaning:** a specific file or package that can be tested or released. **Technical definition:** an immutable
  model, adapter, tokenizer, or runtime package identified by digest and provenance. **Why it matters:** tests apply to
  exact bytes, not a product name. **Example:** a quantized GGUF file with SHA-256 and conversion manifest. **Related:**
  checkpoint, format, integrity.

### Benchmark contamination and leakage

- **Plain meaning:** evaluation answers influence training or selection. **Technical definition:** exposure of test
  content, derivatives, answers, or metadata to training, calibration, tuning, or prompts. **Why it matters:** results
  become optimistically biased. **Example:** using benchmark questions as quantization calibration data. **Related:**
  split, deduplication, held-out test.

### Data provenance and lineage

- **Plain meaning:** where data came from and what happened to it. **Technical definition:** provenance records identity,
  rights, and origin; lineage traces transformations and releases. **Why it matters:** both are needed for quality,
  licensing, corrections, and reproducibility. **Example:** retain source URL, license, extraction hash, cleaning
  version, and final dataset record. **Related:** dataset registry, custody, deduplication.

### Hallucination

- **Plain meaning:** a model presents unsupported content as if supported. **Technical definition:** generated claims
  lack support under the input, accepted ground truth, or declared use contract. **Why it matters:** fluent specialist
  errors can be dangerous. **Example:** inventing a torque specification not present in validated knowledge. **Related:**
  unsupported claim rate, uncertainty, human escalation.

### Pareto frontier

- **Plain meaning:** candidates that are not beaten on every goal. **Technical definition:** a set where no other point
  is at least as good on all objectives and better on one. **Why it matters:** one model rarely maximizes quality,
  memory, latency, and energy simultaneously. **Example:** a slower accurate model and a faster smaller model may both
  be Pareto-optimal. **Related:** trade-off, multi-objective selection.

### Reproducibility

- **Plain meaning:** another qualified person can recreate the result. **Technical definition:** retained code, data,
  configuration, environment, seeds, hardware, commands, and raw outputs permit repetition. **Why it matters:** a
  headline number without its conditions cannot guide engineering. **Example:** rerun a quantization benchmark from a
  pinned manifest. **Related:** run record, evidence label, immutable revision.

### Prefill and decode

- **Plain meaning:** prefill reads the prompt; decode emits later tokens. **Technical definition:** prefill computes
  prompt states and initial logits, while decode performs sequential cached generation. **Why it matters:** they stress
  hardware differently and need separate measurements. **Example:** a GPU may prefill quickly but decode only modestly
  for batch one. **Related:** TTFT, KV cache, throughput.

### Runtime, backend, and fallback

- **Plain meaning:** the runtime coordinates execution, the backend targets hardware, and fallback handles unsupported
  work elsewhere. **Technical definition:** a runtime loads and executes artifacts; a backend implements operators for
  a hardware class; fallback partitions unsupported operations to another backend. **Why it matters:** selectable GPU
  support does not prove full GPU execution. **Example:** an NPU graph silently runs one operation on CPU. **Related:**
  execution provider, graph placement, trace.

### Time to first token

- **Plain meaning:** how long a user waits before output begins. **Technical definition:** elapsed time from a declared
  request boundary to the first usable generated token. **Why it matters:** comparisons are invalid if one includes
  queueing or loading and another does not. **Example:** measure from tapping Send through rendering the first token.
  **Related:** cold start, prefill, latency.

### On-device, offline, edge, cloud, and hybrid

- **Plain meaning:** these terms say where execution occurs and whether a network is needed. **Technical definition:**
  on-device means the user device; offline means no inference network dependency after install; edge means near the
  source; cloud means remote infrastructure; hybrid combines locations. **Why it matters:** they imply different
  privacy, latency, update, and failure properties. **Example:** a phone can run on-device yet require a network license
  check, so it is not fully offline. **Related:** no-runtime-RAG, gateway, deployment.

### Peak runtime memory

- **Plain meaning:** the highest memory use during a declared run. **Technical definition:** maximum resident or
  allocator memory in scope, including weights, cache, workspaces, and app overhead. **Why it matters:** file size does
  not determine whether inference fits. **Example:** a 700 MB file peaks at 1.8 GB during a long prompt. **Related:**
  loaded memory, artifact size, OOM.

### Open weight and open source

- **Plain meaning:** downloadable weights are not automatically an open-source system. **Technical definition:**
  open-weight means parameters are available under stated terms; open source requires the applicable freedoms and
  preferred forms under a qualifying license. **Why it matters:** training data, code, modification, and commercial
  rights may differ. **Example:** a checkpoint can allow download under a custom restricted license. **Related:**
  license, reproducibility, model card.

### No-runtime-RAG architecture

- **Plain meaning:** the product does not fetch documents to answer each request. **Technical definition:** domain
  answers are generated from installed model parameters or adapters without runtime retrieval augmentation. **Why it
  matters:** this project must confront stale knowledge and imperfect recall rather than hiding retrieval in the
  inference path. **Example:** manuals may prepare training data but are not searched during a phone response.
  **Related:** domain adaptation, offline inference, hallucination.

Cross-links: [abbreviations](abbreviations.md) · [definition register](definition-register.md) ·
[library index](README.md)
