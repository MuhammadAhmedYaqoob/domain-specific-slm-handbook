# Research comparison report

**Research freeze:** 2026-11-23  
**Status:** authored research synthesis; documentation and primary sources reviewed; no project experiment, model
conversion, cloud deployment, or physical-device benchmark executed.

## Purpose and evidence boundary

This report compares exact Small Language Model (SLM) checkpoints, adaptation methods, compression techniques, runtime
stacks, and Amazon Web Services (AWS) capabilities for a no-runtime-Retrieval-Augmented Generation (RAG)
specialist-model platform. It is the decision layer between the detailed surveys and the reproducible experiments.

Evidence labels follow the [project standard](../00-governance/evidence-and-citation-standard.md). `DOC-VERIFIED` and
`PAPER-REPORTED` identify inspected documentation and attributed paper findings; neither means independent execution.
`STATICALLY REVIEWED` is engineering synthesis. Every proposed run remains `PLANNED EXPERIMENT`. The report contains no
fabricated benchmark, price, conversion result, or compatibility claim.

## How to study these comparative lessons

The engineering problem is not “Which technique has the best headline?” It is “Which exact model, training method,
compression path, runtime, and infrastructure tuple satisfies a declared quality and safety threshold within a real
memory, latency, energy, licensing, and maintenance budget?” The intuition is to treat each technology as a tool that
removes one bottleneck while possibly creating another. Quantization can reduce weight traffic but add numerical error;
Parameter-Efficient Fine-Tuning (PEFT) can reduce training memory while leaving inference size unchanged; and a Neural
Processing Unit (NPU) can save energy only if the exported graph is actually delegated to it.

Each lesson therefore follows the same reasoning sequence:

1. identify the constraint and the historical reason the technique exists;
2. explain the mechanism in ordinary engineering terms;
3. work through a concrete specialist-model example;
4. compare strengths, limitations, and compatibility boundaries;
5. decide when to use or avoid the technique; and
6. state what evidence would change the decision.

Read every table as a compact index, not as a verdict. The prose before a table defines what its columns mean; the prose
after it explains the trade-off. Evidence labels preserve uncertainty as an actionable caveat. For example,
`DOC-VERIFIED` support means that an official source documents a route. It does not mean this project has converted the
model, preserved its tokenizer, achieved full accelerator placement, or measured acceptable performance.

### Worked illustrative comparison

Suppose an automotive assistant must run offline on a phone with a fixed memory budget. Candidate A is a 1B-parameter
model with a documented 4-bit mobile route. Candidate B is a 3B-parameter model whose model card reports stronger
general results but whose mobile export is unresolved. No benchmark result may be invented to bridge that evidence
gap. The rational first decision is to qualify A as the feasibility baseline and retain B as a higher-capacity
experiment. If B later converts and passes the same domain, safety, memory, and sustained-latency protocol, compare the
measured Pareto trade-off. If it does not, the model-card advantage is irrelevant to this deployment.

## Report map

<!-- markdownlint-disable MD013 -->
| Question                                             | Synthesis                                                             | Detailed evidence                                                                                               |
| ---------------------------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Which checkpoint roles should enter the funnel?      | [Model-family comparison](model-family-comparison.md)                 | [Model-family survey](model-family-survey.md)                                                                   |
| Which adaptation method should be tried first?       | [Adaptation and PEFT comparison](adaptation-and-peft-comparison.md)   | [PEFT methods](peft-methods.md)                                                                                 |
| Which compression route can create deployment value? | [Compression comparison](compression-methods-comparison.md)           | [Quantization](quantization-methods.md), [pruning](pruning-methods.md), [distillation](distillation-methods.md) |
| In what order should methods be combined?            | [Combined pipelines](combined-optimization-pipelines.md)              | [Experiment protocols](../06-experiment-framework/README.md)                                                    |
| Which runtime and deployment route fits each target? | [Runtime/deployment comparison](runtime-and-deployment-comparison.md) | [Edge survey](edge-runtime-survey.md), [AWS survey](aws-capability-survey.md)                                   |
| What is mature enough to prioritize?                 | [Evidence and maturity ranking](evidence-and-maturity-ranking.md)     | All surveys above                                                                                               |
<!-- markdownlint-restore -->

The map separates synthesis from detailed evidence so that a learner can first understand the decision and then audit
its sources. Start with the synthesis in the middle column. Use the evidence file to inspect mechanisms, exact
compatibility limits, dates, and primary sources. Finish with the experiment framework, because no documentation-only
comparison can select a production artifact.

## Executive synthesis

1. **Select exact artifacts, not family names.** The unit of evaluation is checkpoint revision + tokenizer + chat
   template + license + adaptation state + quantized/exported artifact + runtime/backend/device. Model-card context and
   source-framework loading do not establish packaged context or device viability.
2. **Begin with controlled base/instruct strata.** Explicit base checkpoints support causal attribution for continued
   pretraining (CPT)/domain-adaptive pretraining (DAPT); official instruct checkpoints are separate product baselines.
   Qwen2.5, Gemma 3, Llama 3.2, SmolLM, and OLMo provide useful base arms; Phi-4 Mini is stronger as an
   instruct/deployment arm. See the [checkpoint matrix](model-family-comparison.md).
3. **Use LoRA and QLoRA as adaptation baselines, not compression claims.** They reduce trainable state. A merged adapter
   normally retains dense inference dimensions and needs fresh runtime-specific quantization for edge release.
4. **Quantization is the first deployment compression experiment.** Compare high precision, INT8, and a mature
   runtime-native 4-bit path. Storage reduction is not latency evidence; measure resident memory, prefill, decode,
   quality, energy, and thermals.
5. **Prefer dense architectural reduction over unsupported zeros.** Distillation and validated structured pruning can
   reduce ordinary dense work. Unstructured and N:M sparsity need matching kernels; otherwise a sparse checkpoint may
   have unchanged runtime work.
6. **Choose a runtime portfolio.** llama.cpp/GGUF is the portable CPU baseline; ExecuTorch and LiteRT-LM are leading
   graph/export candidates; Core ML/MLX deserve Apple-specific comparison; vendor NPU paths require device-cohort
   artifacts. On AWS, compare vLLM and SGLang first, with llama.cpp for GGUF/CPU and TensorRT-LLM only after a measured
   NVIDIA-specific need.
7. **Stop at the first qualified Pareto set.** More transformations increase quality, conversion, maintenance, and
   lineage risk. A simpler candidate that meets safety, quality, memory, and latency gates should not be displaced by
   nominal compression.

## Project experiment shortlist

The first funnel is deliberately smaller than the full research landscape:

1. **Feasibility:** untouched exact checkpoints in source framework and intended runtimes; tokenizer/template goldens;
   license and export gate.
2. **Model sweep:** one sub-billion control, several 0.5B–2B candidates, and one 3B–4B upper arm using one frozen domain
   evaluation.
3. **Adaptation:** common SFT with LoRA and QLoRA; feasible full fine-tuning control; CPT/DAPT only on explicit base
   checkpoints.
4. **Deployment quantization:** merged high-precision parent, INT8, and two supported 4-bit recipes; no requantization.
5. **Architecture reduction if needed:** native smaller model versus validated response distillation and one
   structured-pruning candidate.
6. **Runtime bake-off:** Android CPU baseline plus documented accelerator path; Apple Metal/Core ML path; AWS
   vLLM/SGLang; final physical-device and offered-load tests.

Full protocols and record fields are in the [experiment framework](../06-experiment-framework/README.md),
[prioritized roadmap](../11-research-roadmap/prioritized-experiment-roadmap.md), and
[combined pipeline report](combined-optimization-pipelines.md).

## Hard gates

- Exact license, acceptable-use, redistribution, and derivative obligations are archived; family-level license inference
  is prohibited.
- Model/tokenizer/template revisions and artifact hashes are immutable.
- Calibration, recovery, training, validation, and sealed test sets are separate and traceable.
- Domain correctness, general retention, uncertainty, refusal, and risk-stratified safety pass before systems
  optimization.
- Conversion preserves tokenizer/template behavior and passes source-versus-target parity checks.
- Accelerator placement is demonstrated; silent CPU fallback is not an NPU/GPU result.
- Peak memory includes weights, Key-Value cache, activations, buffers, runtime, and app/service overhead.
- Runtime claims name hardware, operating system, backend, precision, context, workload, and trial conditions.
- No vector database, runtime document retrieval, embedding search, dynamic chunking, or remote-teacher answer
  substitution enters the primary inference path.

## Known evidence limits

No report in this directory establishes a universal model winner, paper-result reproduction, successful conversion,
acceptable mobile speed, cloud capacity, energy use, thermal stability, or current dollar cost. Repository-default URLs
are discovery references, not immutable experiment revisions. Custom model licenses require qualified legal review, and
pharmaceutical or hazardous-repair releases require qualified domain review. Learned parameters can be stale,
conflicting, incomplete, or hallucinatory; training does not guarantee exact factual storage.

## Cross-links

- [Bookmap](../BOOKMAP.md)
- [Governance](../00-governance/README.md)
- [No-runtime-RAG standard](../00-governance/no-runtime-rag-standard.md)
- [Safety standard](../00-governance/safety-and-risk-standard.md)
- [Decision framework](../14-decision-framework/README.md)
- [Research roadmap](../11-research-roadmap/README.md)
