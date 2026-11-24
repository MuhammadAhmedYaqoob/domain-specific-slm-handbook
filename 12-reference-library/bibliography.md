<!-- markdownlint-disable MD034 -->

# Consolidated bibliography

Primary papers and official materials cited in authored chapters and research surveys, consolidated as of
**2026-11-23**. `PAPER-REPORTED` means the cited authors reported a result; `DOC-VERIFIED` means official documentation
was inspected. Neither means this project reproduced execution. “Reproduction” below identifies available
code/artifacts, not a project reproduction.

## How to read and evaluate this bibliography

Read a record from the claim outward. First ask whether the source is an original paper, official documentation, a
model card, or vendor material. Then inspect the contribution and limitations before following the link to the exact
section, table, configuration, or release that supports the claim.

When evaluating a **paper**, check the research question, baselines, dataset and split, model scale, hardware, software,
metric definition, number of trials, uncertainty, ablations, and threats to external validity. A result reported for
one server GPU or benchmark does not automatically transfer to a phone or specialist domain. Code availability helps
inspection, but only a retained project run can support `EXECUTED` or `REPRODUCED`.

When evaluating **official documentation**, pin a version, release, commit, region, and access date. Documentation is
the best source for an API contract or supported matrix, but it does not prove that a particular graph stayed on an
accelerator, met a latency target, or preserved model quality. For **model cards**, separately verify the exact
checkpoint, tokenizer, template, license, revision, and files. For **vendor benchmarks**, preserve the vendor's test
conditions and label the result `VENDOR-REPORTED`.

## Foundations and Transformer architecture

### `vaswani2017attention`

- **Title / authors:** “Attention Is All You Need” — Ashish Vaswani et al.
- **Year / venue:** 2017, NeurIPS.
- **ID / URL:** arXiv:1706.03762; https://arxiv.org/abs/1706.03762
- **Contribution:** Transformer encoder–decoder, scaled dot-product attention, multi-head attention, and sinusoidal
  positions.
- **Limitations:** original translation experiments are not modern decoder-only, edge, or domain-SLM evidence.
- **Reproduction / applicability:** paper and subsequent implementations; foundational for Chapters 1–2. **Evidence:**
  `PAPER-REPORTED`.

### `bengio2003neural-lm`

- **Title / authors:** “A Neural Probabilistic Language Model” — Yoshua Bengio et al.
- **Year / venue:** 2003, _Journal of Machine Learning Research_ 3. <!-- markdownlint-disable-line MD013 MD049 -->
- **ID / URL:** DOI 10.1162/153244303322533223; https://www.jmlr.org/papers/v3/bengio03a.html
- **Contribution:** learned distributed word representations with neural next-word probability estimation.
- **Limitations:** fixed context and historical scale.
- **Reproduction / applicability:** paper materials; historical foundation. **Evidence:** `PAPER-REPORTED`.

### `hochreiter1997lstm`

- **Title / authors:** “Long Short-Term Memory” — Sepp Hochreiter and Jürgen Schmidhuber.
- **Year / venue:** 1997, _Neural Computation_ 9(8). <!-- markdownlint-disable-line MD013 MD049 -->
- **ID / URL:** DOI 10.1162/neco.1997.9.8.1735;
  https://api.crossref.org/works/10.1162/neco.1997.9.8.1735
- **Contribution:** gated recurrent memory and improved long-range error flow.
- **Limitations:** predates modern language-model scale and hardware.
- **Reproduction / applicability:** implementation availability not recorded; historical comparison. **Evidence:**
  `PAPER-REPORTED`.

### `sennrich2016bpe`

- **Title / authors:** “Neural Machine Translation of Rare Words with Subword Units” — Rico Sennrich, Barry Haddow, and
  Alexandra Birch.
- **Year / venue:** 2016, ACL.
- **ID / URL:** DOI 10.18653/v1/P16-1162; https://aclanthology.org/P16-1162/
- **Contribution:** applies BPE-style subword units to neural translation.
- **Limitations:** translation setting; tokenizer implementations differ.
- **Reproduction / applicability:** paper resources; tokenizer foundation. **Evidence:** `PAPER-REPORTED`.

### `kudo2018sentencepiece`

- **Title / authors:** “SentencePiece: A simple and language independent subword tokenizer and detokenizer for Neural
  Text Processing” — Taku Kudo and John Richardson.
- **Year / venue:** 2018, EMNLP System Demonstrations.
- **ID / URL:** DOI 10.18653/v1/D18-2012; https://aclanthology.org/D18-2012/
- **Contribution:** raw-text BPE/Unigram tokenization toolkit.
- **Limitations:** use does not establish a checkpoint’s exact normalization or vocabulary behavior.
- **Reproduction / applicability:** official implementation linked by paper; tokenizer engineering. **Evidence:**
  `PAPER-REPORTED`.

### `su2021roformer`

- **Title / authors:** “RoFormer: Enhanced Transformer with Rotary Position Embedding” — Jianlin Su et al.
- **Year / venue:** 2021, arXiv preprint in this record. Before assigning a later venue, verify the exact title and
  author list against DBLP, Crossref, and the authors' official publication page.
- **ID / URL:** arXiv:2104.09864; https://arxiv.org/abs/2104.09864
- **Contribution:** rotary position embedding.
- **Limitations:** extrapolation depends on training, scaling, and implementation.
- **Reproduction / applicability:** no repository is asserted here; verify a repository through the paper's arXiv
  metadata or author page before linking it. Checkpoint architecture analysis. **Evidence:** `PAPER-REPORTED`.

### `ainslie2023gqa`

- **Title / authors:** “GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints” — Joshua
  Ainslie et al.
- **Year / venue:** 2023, EMNLP.
- **ID / URL:** DOI 10.18653/v1/2023.emnlp-main.298; https://aclanthology.org/2023.emnlp-main.298/
- **Contribution:** grouped-query attention and conversion/training study.
- **Limitations:** reported quality and speed are setting-specific.
- **Reproduction / applicability:** verify artifact availability from the ACL Anthology supplemental-material links
  before claiming code or models; KV-cache design. **Evidence:** `PAPER-REPORTED`.

### `dao2022flashattention`

- **Title / authors:** “FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness” — Tri Dao et al.
- **Year / venue:** 2022, NeurIPS.
- **ID / URL:** arXiv:2205.14135; https://arxiv.org/abs/2205.14135
- **Contribution:** IO-aware exact attention algorithm.
- **Limitations:** benefits depend on hardware, dtype, shape, and kernel.
- **Reproduction / applicability:** official code exists; cloud/mobile kernel decisions require execution. **Evidence:**
  `PAPER-REPORTED`.

### `hoffmann2022chinchilla`

- **Title / authors:** “Training Compute-Optimal Large Language Models” — Jordan Hoffmann et al.
- **Year / venue:** 2022, NeurIPS.
- **ID / URL:** arXiv:2203.15556; https://arxiv.org/abs/2203.15556
- **Contribution:** empirical parameter/data scaling under fixed compute.
- **Limitations:** fitted regimes do not determine domain quality or deployment optimality.
- **Reproduction / applicability:** model/data reproduction is incomplete; planning only. **Evidence:**
  `PAPER-REPORTED`.

## Adaptation and post-training

### `hu2022lora`

- **Title / authors:** “LoRA: Low-Rank Adaptation of Large Language Models” — Edward J. Hu et al.
- **Year / venue:** 2022, ICLR.
- **ID / URL:** arXiv:2106.09685; https://openreview.net/forum?id=nZeVKeeFYf9
- **Contribution:** frozen backbone with trainable low-rank weight updates.
- **Limitations:** does not shrink the merged dense model or KV cache; reported parity is task/model-specific.
- **Reproduction / applicability:** https://github.com/microsoft/LoRA; core PEFT baseline. **Evidence:**
  `PAPER-REPORTED`.

### `dettmers2023qlora`

- **Title / authors:** “QLoRA: Efficient Finetuning of Quantized LLMs” — Tim Dettmers et al.
- **Year / venue:** 2023, NeurIPS.
- **ID / URL:** arXiv:2305.14314; https://arxiv.org/abs/2305.14314
- **Contribution:** adapter training through a frozen four-bit base; NF4, double quantization, and paged optimizers.
- **Limitations:** training-memory findings do not establish final edge format or universal quality parity.
- **Reproduction / applicability:** https://github.com/artidoro/qlora; core adaptation experiment. **Evidence:**
  `PAPER-REPORTED`.

### `liu2024dora`

- **Title / authors:** “DoRA: Weight-Decomposed Low-Rank Adaptation” — Shih-Yang Liu et al.
- **Year / venue:** 2024, ICML.
- **ID / URL:** PMLR 235 record; https://proceedings.mlr.press/v235/liu24bn.html. The PMLR page is the canonical
  proceedings record; query Crossref by exact title before adding a DOI.
- **Contribution:** separates trainable weight magnitude and low-rank directional adaptation.
- **Limitations:** extra unmerged state/compute and implementation constraints.
- **Reproduction / applicability:** https://github.com/NVlabs/DoRA; advanced LoRA ablation. **Evidence:**
  `PAPER-REPORTED`.

### `ouyang2022instructgpt`

- **Title / authors:** “Training Language Models to Follow Instructions with Human Feedback” — Long Ouyang et al.
- **Year / venue:** 2022, NeurIPS.
- **ID / URL:** arXiv:2203.02155;
  https://papers.nips.cc/paper_files/paper/2022/hash/b1efde53be364a73914f58805a001731-Abstract-Conference.html
- **Contribution:** supervised demonstrations, reward modeling, and RLHF.
- **Limitations:** proprietary model/data details and scale limit direct reproduction; helpfulness does not imply domain
  factuality.
- **Reproduction / applicability:** no complete official reproduction; post-training foundation. **Evidence:**
  `PAPER-REPORTED`.

## Quantization

### `dettmers2022llmint8`

- **Title / authors:** “LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale” — Tim Dettmers et al.
- **Year / venue:** 2022, NeurIPS.
- **ID / URL:** arXiv:2208.07339; https://arxiv.org/abs/2208.07339
- **Contribution:** vector-wise INT8 matmul with higher-precision outlier paths.
- **Limitations:** hardware/backend dependent and not a native mobile format.
- **Reproduction / applicability:** bitsandbytes implementation; cloud memory/compute baseline. **Evidence:**
  `PAPER-REPORTED`.

### `frantar2023gptq`

- **Title / authors:** “GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers” — Elias
  Frantar et al.
- **Year / venue:** 2023, ICLR.
- **ID / URL:** arXiv:2210.17323; https://arxiv.org/abs/2210.17323
- **Contribution:** layer-wise approximate second-order low-bit weight reconstruction.
- **Limitations:** calibration and implementation-specific layouts/kernels; “GPTQ” files are not universally
  interchangeable.
- **Reproduction / applicability:** https://github.com/IST-DASLab/gptq; cloud quantization baseline. **Evidence:**
  `PAPER-REPORTED`.

### `lin2024awq`

- **Title / authors:** “AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration” — Ji Lin et al.
- **Year / venue:** 2024, MLSys; reported Best Paper.
- **ID / URL:** arXiv:2306.00978; https://arxiv.org/abs/2306.00978
- **Contribution:** activation-aware salience and scaling for low-bit weight-only quantization.
- **Limitations:** calibration and runtime-specific kernels; edge GPU results do not generalize to arbitrary phones.
- **Reproduction / applicability:** https://github.com/mit-han-lab/llm-awq; cloud/edge candidate. **Evidence:**
  `PAPER-REPORTED`.

### `xiao2023smoothquant`

- **Title / authors:** “SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models” —
  Guangxuan Xiao et al.
- **Year / venue:** 2023, ICML.
- **ID / URL:** PMLR 202 record; https://proceedings.mlr.press/v202/xiao23c.html. Query Crossref by exact title before
  adding a DOI.
- **Contribution:** channel-wise rescaling enabling W8A8 quantization.
- **Limitations:** calibration and tested server kernels; not mobile evidence.
- **Reproduction / applicability:** https://github.com/mit-han-lab/smoothquant; server INT8 experiment. **Evidence:**
  `PAPER-REPORTED`.

### `egiazarian2024aqlm`

- **Title / authors:** “Extreme Compression of Large Language Models via Additive Quantization” — Vage Egiazarian et al.
- **Year / venue:** 2024, ICML.
- **ID / URL:** arXiv:2401.06118; https://arxiv.org/abs/2401.06118
- **Contribution:** learned additive codebooks for extreme weight compression.
- **Limitations:** expensive optimization and narrower kernel/platform support.
- **Reproduction / applicability:** https://github.com/Vahe1994/AQLM; advanced low-bit research. **Evidence:**
  `PAPER-REPORTED`.

### `wang2023bitnet`

- **Title / authors:** “BitNet: Scaling 1-bit Transformers for Large Language Models” — Hongyu Wang et al.
- **Year / venue:** 2023, arXiv preprint.
- **ID / URL:** arXiv:2310.11453; https://arxiv.org/abs/2310.11453
- **Contribution:** native low-bit Transformer training.
- **Limitations:** not drop-in PTQ; checkpoint, training, and kernel maturity must be verified.
- **Reproduction / applicability:** Microsoft BitNet ecosystem; separate-model research branch. **Evidence:**
  `PAPER-REPORTED`.

## Pruning and distillation

### `frantar2023sparsegpt`

- **Title / authors:** “SparseGPT: Massive Language Models Can Be Accurately Pruned in One-Shot” — Elias Frantar and Dan
  Alistarh.
- **Year / venue:** 2023, ICML.
- **ID / URL:** PMLR 202 record; https://proceedings.mlr.press/v202/frantar23a.html. Query Crossref by exact title
  before adding a DOI.
- **Contribution:** one-shot layer-wise approximate second-order pruning.
- **Limitations:** sparse zeros do not imply runtime acceleration; calibration and architecture support matter.
- **Reproduction / applicability:** https://github.com/IST-DASLab/sparsegpt; pruning baseline. **Evidence:**
  `PAPER-REPORTED`.

### `sun2024wanda`

- **Title / authors:** “A Simple and Effective Pruning Approach for Large Language Models” — Mingjie Sun et al.
- **Year / venue:** 2024, ICLR.
- **ID / URL:** OpenReview ID PxoFut3dWW; https://openreview.net/forum?id=PxoFut3dWW
- **Contribution:** weight-magnitude times activation-norm pruning score.
- **Limitations:** calibration-dependent and produces sparsity requiring matching kernels.
- **Reproduction / applicability:** https://github.com/locuslab/wanda; simple sparse baseline. **Evidence:**
  `PAPER-REPORTED`.

### `ashkboos2024slicegpt`

- **Title / authors:** “SliceGPT: Compress Large Language Models by Deleting Rows and Columns” — Saleh Ashkboos et al.
- **Year / venue:** 2024, ICLR.
- **ID / URL:** arXiv:2401.15024; https://arxiv.org/abs/2401.15024
- **Contribution:** orthogonal transformations followed by dense dimension removal.
- **Limitations:** model assumptions, shape alignment, export, and recovery are checkpoint/runtime specific.
- **Reproduction / applicability:** https://github.com/microsoft/TransformerCompression; portable structured pruning
  candidate. **Evidence:** `PAPER-REPORTED`.

### `hinton2015distillation`

- **Title / authors:** “Distilling the Knowledge in a Neural Network” — Geoffrey Hinton, Oriol Vinyals, and Jeff Dean.
- **Year / venue:** 2015, NIPS Deep Learning Workshop / arXiv.
- **ID / URL:** arXiv:1503.02531; https://arxiv.org/abs/1503.02531
- **Contribution:** temperature-softened teacher targets mixed with hard labels.
- **Limitations:** classification framing requires careful autoregressive adaptation.
- **Reproduction / applicability:** no official code is asserted in this record; check the arXiv metadata and author
  pages before linking an implementation. Foundational KD. **Evidence:** `PAPER-REPORTED`.

### `kim2016sequence-kd`

- **Title / authors:** “Sequence-Level Knowledge Distillation” — Yoon Kim and Alexander M. Rush.
- **Year / venue:** 2016, EMNLP.
- **ID / URL:** DOI 10.18653/v1/D16-1139; https://aclanthology.org/D16-1139/
- **Contribution:** trains students on teacher-decoded sequences.
- **Limitations:** loses alternative distributions; original task is machine translation.
- **Reproduction / applicability:** inspect the ACL Anthology paper and supplemental links before asserting artifact
  availability; teacher-response SFT design. **Evidence:** `PAPER-REPORTED`.

### `gu2024minillm`

- **Title / authors:** “MiniLLM: Knowledge Distillation of Large Language Models” — Yuxian Gu et al.
- **Year / venue:** 2024, ICLR.
- **ID / URL:** OpenReview ID 5h0qf7IBZZ; https://openreview.net/forum?id=5h0qf7IBZZ
- **Contribution:** on-policy reverse-KL generative distillation.
- **Limitations:** requires teacher logits on student samples and greater training complexity than response SFT.
- **Reproduction / applicability:** https://github.com/microsoft/LMOps/tree/main/minillm; research KD baseline.
  **Evidence:** `PAPER-REPORTED`.

## Small-model families and model cards

The following are official, mutable model repositories cited by the
[model-family survey](../04-research-comparison-report/model-family-survey.md). Authorship and publication metadata
belong to each linked card/report and are not inferred here.

### `qwen3-official`

- **Title / author / status:** Qwen3 model cards and technical report — Qwen Team, 2025 technical release/report.
- **URL:** https://huggingface.co/Qwen/Qwen3-1.7B and https://github.com/QwenLM/Qwen3
- **Contribution:** exact checkpoint architecture, usage, context, license references, and family report.
- **Limitations:** mutable default revisions; vendor benchmarks are not common project benchmarks.
- **Reproduction / applicability:** official code and weights; candidate checkpoint discovery. **Evidence:**
  `DOC-VERIFIED`.

### `gemma3-official`

- **Title / author / status:** Gemma 3 model cards and technical report — Google, 2025 technical release.
- **ID / URL:** arXiv:2503.19786; https://arxiv.org/abs/2503.19786 and https://huggingface.co/google/gemma-3-1b-pt
- **Contribution:** checkpoint architecture, training, safety, license terms, and QAT artifact route.
- **Limitations:** gated terms and vendor-reported evaluations; exact revision must be frozen.
- **Reproduction / applicability:** official weights/cards; 1B edge candidate. **Evidence:** `DOC-VERIFIED`, paper
  findings `PAPER-REPORTED`.

### `llama32-official`

- **Title / author / status:** Llama 3.2 model cards and license — Meta, 2024 release.
- **URL:** https://huggingface.co/meta-llama/Llama-3.2-1B and
  https://huggingface.co/meta-llama/Llama-3.2-3B/blob/main/LICENSE.txt
- **Contribution:** exact model variants, context, supported languages, intended use, and custom license.
- **Limitations:** gated/custom terms; on-device positioning is not execution evidence.
- **Reproduction / applicability:** official weights subject to terms; ecosystem baseline. **Evidence:** `DOC-VERIFIED`.

### `smollm2-official`

- **Title / authors:** “SmolLM2: When Smol Goes Big — Data-Centric Training of a Small Language Model” — use the ordered
  author list from arXiv:2502.02737. Before approval, export the arXiv citation and compare it with the repository's
  citation metadata; this record deliberately avoids guessing an unverified list.
- **Year / status:** 2025, arXiv.
- **ID / URL:** arXiv:2502.02737; https://arxiv.org/abs/2502.02737 and https://github.com/huggingface/smollm
- **Contribution:** small checkpoints, data-centric training, recipes, and open research artifacts.
- **Limitations:** edge execution and domain capacity remain target-specific.
- **Reproduction / applicability:** official repository/checkpoints; pipeline and reproducibility candidate.
  **Evidence:** `PAPER-REPORTED`, `DOC-VERIFIED`.

### `mobilellm2024`

- **Title / authors:** “MobileLLM: Optimizing Sub-billion Parameter Language Models for On-Device Use Cases” — Zechun
  Liu et al.
- **Year / venue:** 2024, ICML.
- **ID / URL:** arXiv:2402.14905; https://arxiv.org/abs/2402.14905
- **Contribution:** deep-and-thin architecture choices for sub-billion models.
- **Limitations:** paper/model results and “on-device” framing do not establish project runtime/device performance;
  current weight license is restrictive.
- **Reproduction / applicability:** https://github.com/facebookresearch/MobileLLM; architecture ablation only.
  **Evidence:** `PAPER-REPORTED`.

## Edge runtimes and formats — official documentation

Treat the entries in this section as changing software records. Follow both the documentation and release history,
then pin the runtime, converter, backend, SDK, and model support matrix. An example app proves that a pathway exists; it
does not prove your checkpoint, quantization, device placement, quality, or performance.

### `llamacpp-official`

- **Title / owner / status:** llama.cpp repository, Android/build guides, releases, and GGUF specification — ggml-org,
  active project.
- **Year / venue / DOI:** continuously maintained; not applicable.
- **URL:** https://github.com/ggml-org/llama.cpp and https://github.com/ggml-org/ggml/blob/master/docs/gguf.md
- **Contribution:** GGUF conversion/quantization, inference backends, cache, server, Android example, and format
  definition.
- **Limitations:** rapid releases; architecture/tokenizer/backend behavior must be pinned and executed.
- **Reproduction / applicability:** source and examples available; portable CPU/GGUF baseline. **Evidence:**
  `DOC-VERIFIED`.

### `executorch-official`

- **Title / owner / status:** ExecuTorch documentation and repository — PyTorch/Meta, active 1.x project at cutoff.
- **URL:** https://docs.pytorch.org/executorch/stable/backends-overview.html and https://github.com/pytorch/executorch
- **Contribution:** `torch.export` to backend-specialized `.pte`, mobile bindings, LLM runners, and multiple hardware
  backends.
- **Limitations:** model/backend artifacts are target-specific; high-level LLM APIs and delegates have differing
  maturity.
- **Reproduction / applicability:** official source/examples; PyTorch-owned edge candidate. **Evidence:**
  `DOC-VERIFIED`.

### `litertlm-official`

- **Title / owner / status:** LiteRT and LiteRT-LM documentation — Google, active project; Swift maturity was preview at
  cutoff.
- **URL:** https://developers.google.com/edge/litert-lm/overview and https://github.com/google-ai-edge/LiteRT-LM
- **Contribution:** LiteRT successor path plus LLM conversation, tokenizer, KV-cache, packaging, and streaming
  orchestration.
- **Limitations:** supported architecture/export/backend matrix is narrower than generic framework support.
- **Reproduction / applicability:** official source/examples; Android-first graph/runtime candidate. **Evidence:**
  `DOC-VERIFIED`.

### `onnxruntime-genai-official`

- **Title / owner / status:** ONNX Runtime Mobile and ONNX Runtime GenAI documentation — Microsoft, active; GenAI API
  marked preview at cutoff.
- **URL:** https://onnxruntime.ai/docs/tutorials/mobile/ and https://onnxruntime.ai/docs/genai/
- **Contribution:** mobile execution providers and autoregressive tokenizer, sampling, cache, and generation APIs.
- **Limitations:** core ORT support is not GenAI model support; provider partitioning and iOS status require
  exact-version checks.
- **Reproduction / applicability:** official source/packages; ONNX portability candidate. **Evidence:** `DOC-VERIFIED`.

### `coreml-official`

- **Title / owner / status:** Core ML documentation and Core ML Tools — Apple, platform documentation and open-source
  converter.
- **URL:** https://developer.apple.com/documentation/coreml and https://github.com/apple/coremltools
- **Contribution:** Apple CPU/GPU/Neural Engine execution, conversion, model packages, compression, and stateful models.
- **Limitations:** not a complete tokenizer/chat stack; graph placement and numerical parity require profiling.
- **Reproduction / applicability:** official tools/examples; native iOS path. **Evidence:** `DOC-VERIFIED`.

### `mlxswiftlm-official`

- **Title / owner / status:** MLX and MLX Swift LM repositories — Apple ML research ecosystem, active.
- **URL:** https://github.com/ml-explore/mlx and https://github.com/ml-explore/mlx-swift-lm
- **Contribution:** Apple unified-memory array framework and Swift LLM generation/tokenizer/cache integration.
- **Limitations:** CPU/GPU rather than ANE path; architecture support and memory behavior are checkpoint/device
  specific.
- **Reproduction / applicability:** official source/examples; Apple prototype and deployment candidate. **Evidence:**
  `DOC-VERIFIED`.

### `mlcllm-official`

- **Title / owner / status:** MLC LLM documentation and repository — MLC project, active.
- **URL:** https://github.com/mlc-ai/mlc-llm/ and https://llm.mlc.ai/docs/compilation/compile_models.html
- **Contribution:** compiler-driven Android, iOS, desktop, and web deployment with compiled model libraries and
  converted weights.
- **Limitations:** target-specific compiler/package coupling and operational complexity.
- **Reproduction / applicability:** official source/SDK examples; cross-platform GPU candidate. **Evidence:**
  `DOC-VERIFIED`.

## AWS and serving — official documentation

AWS features, quotas, regions, instance availability, and prices change independently. Read the service guide together
with the region table, quota page, pricing page, and dated release notes. Freeze these facts at experiment time and
record account-specific constraints rather than treating this bibliography as a live service catalog.

### `aws-ec2-accelerated`

- **Title / owner / status:** EC2 accelerated-computing instance documentation — Amazon Web Services, mutable service
  documentation.
- **URL:** https://docs.aws.amazon.com/ec2/latest/instancetypes/ac.html
- **Contribution:** authoritative instance-family and accelerator specifications.
- **Limitations:** does not establish Region/AZ capacity, quota, price, model compatibility, or project performance.
- **Reproduction / applicability:** not applicable; check at run freeze. **Evidence:** `DOC-VERIFIED`.

### `aws-sagemaker-deployment`

- **Title / owner / status:** SageMaker AI deployment and inference options — Amazon Web Services, mutable service
  documentation.
- **URL:** https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model-options.html
- **Contribution:** real-time, asynchronous, serverless, and batch deployment choices.
- **Limitations:** feature matrices, Regions, quotas, and pricing change; no exact model/runtime guarantee.
- **Reproduction / applicability:** not applicable; managed baseline design. **Evidence:** `DOC-VERIFIED`.

### `aws-bedrock-custom-models`

- **Title / owner / status:** Amazon Bedrock custom models and Custom Model Import documentation — Amazon Web Services,
  mutable.
- **URL:** https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html and
  https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-import-model.html
- **Contribution:** supported customization/import pathways and constraints.
- **Limitations:** architecture, Region, feature, quota, and serving limits; not arbitrary model/runtime hosting.
- **Reproduction / applicability:** not applicable; conditional managed option. **Evidence:** `DOC-VERIFIED`.

### `vllm-official`

- **Title / owner / status:** vLLM documentation and repository — vLLM project, active.
- **URL:** https://docs.vllm.ai/en/stable/ and https://github.com/vllm-project/vllm
- **Contribution:** OpenAI-compatible GPU serving, batching, model and quantization support matrices.
- **Limitations:** exact architecture, quantization, feature, accelerator, and plugin support vary by release.
- **Reproduction / applicability:** official source; primary AWS GPU serving baseline. **Evidence:** `DOC-VERIFIED`.

### `tgi-status`

- **Title / owner / status:** Text Generation Inference engine documentation — Hugging Face; maintenance mode stated as
  of 2025-11-12 in cited docs.
- **URL:** https://huggingface.co/docs/inference-endpoints/engines/tgi
- **Contribution:** authoritative maintenance status and migration guidance.
- **Limitations:** mutable status; maintenance mode does not mean broken.
- **Reproduction / applicability:** source at https://github.com/huggingface/text-generation-inference; existing-system
  compatibility only. **Evidence:** `DOC-VERIFIED`.

## Governance and openness

### `osi-osaid-1`

- **Title / author / year:** “Open Source AI Definition 1.0” — Open Source Initiative, 2024.
- **Venue / URL:** authoritative definition; https://opensource.org/ai/open-source-ai-definition
- **DOI/arXiv:** not applicable.
- **Contribution:** required freedoms and forms of access for open-source AI.
- **Limitations:** does not replace checkpoint-specific license or legal review.
- **Reproduction / applicability:** not applicable; terminology governance. **Evidence:** `DOC-VERIFIED`.

## Maintenance notes

- Mutable documentation entries require an access date and immutable version/commit in an experiment’s
  [source record](source-record-template.md).
- The authored chapters contain additional primary references, especially tokenizer, normalization, LoRA-variant,
  model-family, and runtime sources. Add them here when a cross-chapter claim depends on them; do not duplicate records
  merely for page-local citations.
- Unresolved author lists, DOI values, and venue details are intentionally marked rather than inferred.
