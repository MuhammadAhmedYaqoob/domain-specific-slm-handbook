# Initial research survey and reading guide

## Scope, method, and learner use

**Research pass:** 2026-11-23. Cited pages below were inspected (`PAPER-REPORTED` or `DOC-VERIFIED`), but no project
implementation or benchmark was reproduced. Treat the survey as a map of claims to verify, not as a substitute for
reading primary sources or executing target-specific tests.

The survey answers four startup questions: what is established, what is practically central, what is target-dependent,
and what must be re-verified immediately before use. It prefers original papers, official model cards, official
framework/vendor documentation, and publication records. Vendor or author results are evidence about what that source
reports, not transferable performance guarantees.

Study the sections in this order: foundations and adaptation; data/evaluation/safety; model candidates; quantization
and pruning; distillation; edge runtimes; AWS. This places problem definition and evidence discipline before product
selection. For each section, read the cited primary source, summarize the mechanism in your own words, identify one
claim that transfers to this project and one that does not, then complete the linked practical decision.

## Verified synthesis

### Foundations and adaptation

- The Transformer paper defines scaled dot-product and multi-head attention and supplies the architectural foundation
  for the decoder-only systems in CH-02. Source: Vaswani et al., _Attention Is All You Need_, 2017, arXiv:1706.03762, <!-- markdownlint-disable-line MD013 MD049 -->
  [paper](https://arxiv.org/abs/1706.03762). `PAPER-REPORTED`.
- LoRA freezes pretrained weights and trains low-rank update matrices; QLoRA backpropagates into adapters through a
  frozen 4-bit base and introduced NF4, double quantization, and paged optimizers. Sources: Hu et al., 2021,
  arXiv:2106.09685, [LoRA](https://arxiv.org/abs/2106.09685); Dettmers et al., 2023, arXiv:2305.14314,
  [QLoRA](https://arxiv.org/abs/2305.14314). `PAPER-REPORTED`.
- Current Hugging Face documentation exposes PEFT adapters through Transformers and TRL and documents 4-bit adapter
  training. This verifies an API/documentation path, not architecture-wide success or local dependency resolution.
  Sources: [Transformers PEFT integration](https://huggingface.co/docs/transformers/en/peft),
  [TRL PEFT integration](https://huggingface.co/docs/trl/main/peft_integration). `DOC-VERIFIED`.
- Project implication: SFT with LoRA is the core practical first adaptation arm; untouched, full-tune where feasible,
  and QLoRA are controlled comparators. CPT and preference optimization are hypothesis-driven branches, not automatic
  steps.

### Model candidates

- Exact checkpoint identity matters. The official `Qwen/Qwen2.5-0.5B` card describes a 0.49B causal base model and
  explicitly directs conversational use toward post-training. Source:
  [model card](https://huggingface.co/Qwen/Qwen2.5-0.5B). `DOC-VERIFIED`.
- Official or publisher-controlled cards exist for small Qwen, Gemma, Llama, Phi, SmolLM and TinyLlama families, but
  family existence does not establish a usable candidate tuple. License terms, base versus instruct status,
  tokenizer/template, immutable revision, architecture support, weight format, and target export must be recorded per
  checkpoint.
- Project implication: keep size bands near 0.5B, 1–2B, 3–4B, and an optional larger reference; select exact candidates
  only through the procedure in the [automotive baseline plan](automotive-baseline-plan.md). No model is selected by
  this survey.

### Quantization and pruning

- GPTQ is a one-shot, approximate second-order weight-quantization method; SmoothQuant migrates activation quantization
  difficulty into weights for W8A8; AWQ uses activation statistics to protect salient weight channels. Sources: Frantar
  et al., 2022/ICLR 2023, arXiv:2210.17323, [GPTQ](https://arxiv.org/abs/2210.17323); Xiao et al., 2022/ICML 2023,
  arXiv:2211.10438, [SmoothQuant](https://arxiv.org/abs/2211.10438); Lin et al., MLSys 2024, arXiv:2306.00978,
  [AWQ](https://arxiv.org/abs/2306.00978). `PAPER-REPORTED`.
- SparseGPT and Wanda establish credible one-shot pruning research directions, but sparsity alone does not prove lower
  latency. Runtime representation and hardware kernels determine whether nominal sparsity creates a deployable benefit.
  Sources: Frantar and Alistarh, 2023, arXiv:2301.00774, [SparseGPT](https://arxiv.org/abs/2301.00774); Sun et al.,
  2023, arXiv:2306.11695, [Wanda](https://arxiv.org/abs/2306.11695). `PAPER-REPORTED`.
- Project implication: screen target-native quantizers after a high-precision source baseline; treat structured
  dimension reduction and supported sparse kernels separately; rerun semantic and safety gates after every conversion.

### Distillation

- DistilBERT, MiniLM, and TinyBERT provide established response/hidden/attention transfer precedents for encoder models.
  Their reported compression and speed results are not direct evidence for an automotive generative student. Sources:
  Sanh et al., 2019, arXiv:1910.01108, [DistilBERT](https://arxiv.org/abs/1910.01108); Wang et al., 2020,
  arXiv:2002.10957, [MiniLM](https://arxiv.org/abs/2002.10957); Jiao et al., 2019/Findings EMNLP 2020, arXiv:1909.10351,
  [TinyBERT](https://arxiv.org/abs/1909.10351). `PAPER-REPORTED`.
- Project implication: compare direct expert data, reviewed teacher-generated data, mixed data, soft-target distillation
  where technically feasible, and distilled-then-quantized artifacts. Preserve teacher license, prompt, outputs, review,
  tokenizer mismatch, and cost.

### Edge runtimes

- ExecuTorch documents an LLM path that exports `.pte` artifacts and runs through C++, Swift, or Java bindings. Source:
  [Deploying LLMs to ExecuTorch](https://docs.pytorch.org/executorch/stable/llm/getting-started.html). `DOC-VERIFIED`.
- LiteRT-LM documentation describes a cross-platform orchestration layer with stable CLI/Python/Kotlin/C++ APIs and
  platform/backend support that is explicitly version-sensitive. Source:
  [LiteRT-LM overview](https://developers.google.com/edge/litert-lm/overview). `DOC-VERIFIED`.
- ONNX Runtime documents mobile packages and CPU, XNNPACK, NNAPI and Core ML execution-provider paths, while warning
  that partitioning and model/operator support are device-specific. Source:
  [ONNX Runtime mobile](https://onnxruntime.ai/docs/tutorials/mobile/). `DOC-VERIFIED`.
- MLX Swift provides Swift APIs and LLM examples for Apple silicon. Source:
  [MLX Swift repository](https://github.com/ml-explore/mlx-swift). `DOC-VERIFIED`.
- Project implication: llama.cpp/GGUF, ExecuTorch, LiteRT-LM, ONNX Runtime Mobile, Core ML/MLX Swift, MLC LLM, MNN,
  NCNN, and vendor runtimes remain candidate paths. A documentation claim is insufficient: each exact
  model–format–runtime–backend–device tuple needs export, load, parity, memory, sustained-performance, lifecycle, and
  failure evidence.

### AWS

- AWS Deep Learning Containers documentation lists current serving paths for vLLM and SGLang across EC2, EKS, and
  SageMaker AI, and SageMaker documentation describes bring-your-own-container and managed inference options. Sources:
  [AWS Deep Learning Containers](https://aws.github.io/deep-learning-containers/),
  [SageMaker model deployment](https://aws.amazon.com/sagemaker/ai/deploy/). `DOC-VERIFIED`.
- SageMaker GPU drivers, CUDA compatibility, images, instance families, Regions, quota, and prices change. Source:
  [SageMaker GPU driver documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/inference-gpu-drivers.html).
  `DOC-VERIFIED`.
- Project implication: start with one isolated, tagged, low-concurrency EC2 or SageMaker candidate selected from current
  requirements; add a throughput profile only after a measured load model exists. Capture AWS Price List or billing
  evidence on execution day. This survey makes no price or instance-performance claim.

### Data, evaluation, and safety

- The repository’s strongest authored component is the [experiment framework](../06-experiment-framework/README.md): it
  requires immutable plans, sealed-test discipline, failed-attempt retention, target-specific metrics, uncertainty, and
  hard-gate-first Pareto analysis. `STATICALLY REVIEWED`.
- The [automotive companion](../07-automotive-case-study/README.md) specifies a 17-phase lineage and prohibits
  unverified vehicle-specific specifications and autonomous direction for hazardous systems. `STATICALLY REVIEWED`.
- Project implication: rights, provenance, grouped-root leakage controls, expert review, critical-safety strata,
  semantic export parity, and physical-target evidence are non-compensatory. Learned parameters do not guarantee exact
  or current factual recall.

## Priority classification

- **Foundational:** Transformer mechanics; language-model objective; tokenization; evaluation statistics; provenance;
  safety; memory/latency accounting.
- **Core practical:** immutable checkpoint selection; rights-approved data; SFT; LoRA; target-native 8/4-bit
  quantization; llama.cpp or another verified runtime; baseline/evaluation protocols; one AWS endpoint; one Android and
  one iOS path.
- **Advanced practical:** CPT, QLoRA, structured pruning, reviewed synthetic data, generative distillation,
  multi-adapter deployment, NPU-specific acceleration, canary/rollback automation.
- **Experimental until reproduced:** sub-4-bit methods, aggressive sparse execution, feature/logit distillation across
  incompatible tokenizers, joint compression searches, broad NPU portability, speculative decoding on constrained
  devices.
- **Historical context:** n-grams, RNN/LSTM/GRU, early seq2seq and encoder distillation; retain because they explain
  present design choices, not because they are default implementation choices.

## Known evidence gaps

1. No complete checkpoint/license/export matrix has been recorded in the reference library.
2. No automotive data source has passed rights and qualified-content review.
3. No repository code environment has been installed or executed.
4. No model/runtime tuple has been converted or loaded.
5. No AWS Region, quota, instance, image, price, or endpoint has been selected.
6. No physical Android, iPhone, ARM, Jetson, laptop, battery, or thermal trial exists.
7. No project quality, latency, memory, energy, cost, or Pareto result exists.
8. Later-than-access changes remain possible; every fast-moving claim follows the
   [watchlist](fast-moving-topics-watchlist.md).

These gaps define the learner’s investigation queue. Close them in dependency order: source rights and exact checkpoint
identity; clean environment and untouched baseline; export/runtime probe; adaptation; cloud and physical-device
measurement. A later measurement cannot repair missing rights, leakage, or identity evidence.

## Source-record completion gate

Before a source supports chapter prose, create its record under `12-reference-library/` with title, authors/owner, year,
venue/status, DOI or identifier, canonical URL, access date, contribution, applicable finding, limitations, reproduction
availability, license where relevant, affected claims, and expiry trigger. Consequential claims require primary
authority plus independent or project evidence where feasible.

## Research-reading exit test

Select one foundational paper, one adaptation paper, one compression paper, and one runtime document. For each, produce
a source record, a mechanism diagram, the exact reported evidence, two limitations, and a project-specific falsifiable
hypothesis. Explain why `PAPER-REPORTED`, `DOC-VERIFIED`, and `EXECUTED` are not interchangeable. Recheck the runtime
document immediately before implementation using the [verification guide](fast-moving-topics-watchlist.md).

## Cross-links

- [Reference library](../12-reference-library/README.md)
- [Evidence standard](../00-governance/evidence-and-citation-standard.md)
- [Fast-moving watchlist](fast-moving-topics-watchlist.md)
