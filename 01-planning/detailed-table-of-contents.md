# Detailed learner syllabus

## Purpose and use

This syllabus maps every stable chapter identifier to its canonical file, learning outcome, prerequisite, and
demonstration artifact. Read each contract before its chapter, test the prerequisites, and produce the artifact before
advancing. The canonical path is the listed chapter file; companion implementation belongs in the linked deliverable
rather than being duplicated.

The default order is Parts 1, 6, 8 foundations, 2–3, 4, 5, the remainder of 8, 9, and then 10–14. This differs from
simple page order because data rights and evaluation design must begin before adaptation. Parts 6 and 8 may be read in
parallel with Parts 1–3, but their practical gates precede training. Use the
[dependency map](learning-dependency-map.md) for shorter role-specific tracks.

## Canonical contents

### Part 1: Language Model Foundations

- [CH-01: Evolution of Language Models](../02-textbook/part-01-language-model-foundations/ch-01-evolution-of-language-models.md)
- [CH-02: Transformer Architecture in Depth](../02-textbook/part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md)
- [CH-03: Modern SLM Architectures](../02-textbook/part-01-language-model-foundations/ch-03-modern-slm-architectures.md)

### Part 2: Training and Domain Adaptation

- [CH-04: Pretraining Fundamentals](../02-textbook/part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md)
- [CH-05: Continued Pretraining and Domain Adaptation](../02-textbook/part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md)
- [CH-06: Supervised Fine-Tuning](../02-textbook/part-02-training-and-domain-adaptation/ch-06-supervised-fine-tuning.md)
- [CH-07: Preference Optimization and Post-Training](../02-textbook/part-02-training-and-domain-adaptation/ch-07-preference-optimization-and-post-training.md)

### Part 3: Parameter-Efficient Fine-Tuning

- [CH-08: Full Fine-Tuning Versus PEFT](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md)
- [CH-09: LoRA and Its Evolution](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md)

### Part 4: Model Compression and Optimization

- [CH-10: The Fundamentals of Model Compression](../02-textbook/part-04-model-compression-and-optimization/ch-10-the-fundamentals-of-model-compression.md)
- [CH-11: Quantization — Complete Research and Implementation](../02-textbook/part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md)
- [CH-12: Pruning — Complete Research and Implementation](../02-textbook/part-04-model-compression-and-optimization/ch-12-pruning-complete-research-and-implementation.md)
- [CH-13: Knowledge Distillation — Complete Research and Implementation](../02-textbook/part-04-model-compression-and-optimization/ch-13-knowledge-distillation-complete-research-and-implementation.md)
- [CH-14: Additional Model Compression Techniques](../02-textbook/part-04-model-compression-and-optimization/ch-14-additional-model-compression-techniques.md)
- [CH-15: Combining Optimization Techniques](../02-textbook/part-04-model-compression-and-optimization/ch-15-combining-optimization-techniques.md)

### Part 5: Edge AI and Mobile Inference

- [CH-16: Edge AI Fundamentals](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-16-edge-ai-fundamentals.md)
- [CH-17: Mobile Hardware Architecture](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-17-mobile-hardware-architecture.md)
- [CH-18: Edge Inference Frameworks](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- [CH-19: Mobile Inference Optimization](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md)
- [CH-20: Android Deployment](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md)
- [CH-21: iOS Deployment](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md)
- [CH-22: Other Edge Devices](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-22-other-edge-devices.md)

### Part 6: Data Acquisition and Dataset Engineering

- [CH-23: Finding Domain-Specific Training Data](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-23-finding-domain-specific-training-data.md)
- [CH-24: Dataset Collection and Processing](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-24-dataset-collection-and-processing.md)
- [CH-25: Dataset Design for Different Training Objectives](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md)
- [CH-26: Synthetic Data and Knowledge Generation](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-26-synthetic-data-and-knowledge-generation.md)

### Part 7: Multi-Domain SLM Engineering

- [CH-27: Single-Domain Versus Multi-Domain Strategies](../02-textbook/part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md)
- [CH-28: Reusable Domain Training Platform](../02-textbook/part-07-multi-domain-slm-engineering/ch-28-reusable-domain-training-platform.md)

### Part 8: Evaluation, Benchmarking, and Research Experiments

- [CH-29: Model Quality Evaluation](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-29-model-quality-evaluation.md)
- [CH-30: Domain-Specific Evaluation Suites](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-30-domain-specific-evaluation-suites.md)
- [CH-31: Compression and Edge Benchmarking](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
- [CH-32: Optimization Experiments and Ablation Studies](../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md)

### Part 9: AWS Training, Optimization, and Deployment

- [CH-33: AWS Infrastructure for SLM Development](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-33-aws-infrastructure-for-slm-development.md)
- [CH-34: Training SLMs on AWS](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-34-training-slms-on-aws.md)
- [CH-35: AWS Inference Serving](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-35-aws-inference-serving.md)
- [CH-36: AWS-to-Edge Model Delivery](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-36-aws-to-edge-model-delivery.md)

### Part 10: Complete Hands-On Project

- [CH-37: Build an Automotive Mechanic SLM End to End](../02-textbook/part-10-complete-hands-on-project/ch-37-build-an-automotive-mechanic-slm-end-to-end.md)

### Part 11: Extending the Platform to Other Domains

- [CH-38: Pharmaceutical Knowledge SLM](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-38-pharmaceutical-knowledge-slm.md)
- [CH-39: Motorcycle and Truck Mechanic SLMs](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-39-motorcycle-and-truck-mechanic-slms.md)
- [CH-40: Beekeeping and Zookeeping SLMs](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md)
- [CH-41: New Domain Onboarding](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-41-new-domain-onboarding.md)

### Part 12: Production Engineering, Safety, and Maintenance

- [CH-42: Model Security and Reliability](../02-textbook/part-12-production-engineering-safety-and-maintenance/ch-42-model-security-and-reliability.md)
- [CH-43: Monitoring and Model Lifecycle](../02-textbook/part-12-production-engineering-safety-and-maintenance/ch-43-monitoring-and-model-lifecycle.md)
- [CH-44: Cost Optimization](../02-textbook/part-12-production-engineering-safety-and-maintenance/ch-44-cost-optimization.md)

### Part 13: Experimental Research Roadmap

- [CH-45: Research Questions and Hypotheses](../02-textbook/part-13-experimental-research-roadmap/ch-45-research-questions-and-hypotheses.md)
- [CH-46: Benchmark Matrix](../02-textbook/part-13-experimental-research-roadmap/ch-46-benchmark-matrix.md)

### Part 14: Final Engineering Architecture

- [CH-47: Proposed Production System](../02-textbook/part-14-final-engineering-architecture/ch-47-proposed-production-system.md)

## Chapter contracts

Each contract states `outcome; direct prerequisite(s); minimum authored artifact`.

### Part 1 contracts

- **CH-01:** explain the statistical-to-foundation-model progression and the engineering case for SLMs; none beyond
  reader prerequisites; historical timeline plus TBL-01 and sourced scaling/generalization discussion.
- **CH-02:** trace tokenization, causal Transformer computation, decoding, context, KV cache, dense/sparse/MoE and
  sequence alternatives; CH-01; annotated FIG-02, FIG-03, formulas, and numerical memory example.
- **CH-03:** select rather than merely list current 100M–8B checkpoints; CH-01–02; versioned model-card matrix with
  license, tokenizer, architecture, reproducibility, and export evidence.

### Part 2 contracts

- **CH-04:** design and diagnose pretraining from objective through distributed checkpointing; CH-02; executable-intent
  training configuration, resource equations, and failure catalogue.
- **CH-05:** decide whether CPT/DAPT/TAPT is justified and protect general capability; CH-03–04 and CH-23–25 concepts;
  FIG-04, mixture design, forgetting controls, and CPT decision protocol.
- **CH-06:** build chat/SFT datasets and loss masks without mistaking format learning for knowledge acquisition; CH-02,
  CH-04–05; valid JSON/JSONL schemas and SFT procedure.
- **CH-07:** compare reward modeling, RLHF/PPO, DPO-family and verified modern post-training; CH-06 and CH-29 concepts;
  decision tree, data contracts, safety/calibration gates, and compute comparison.

### Part 3 contracts

- **CH-08:** choose full tuning, adapters, prompt/prefix methods, IA3 or BitFit under resource and deployment
  constraints; CH-04 and CH-06; TBL-04 and deployment implications.
- **CH-09:** derive LoRA and compare QLoRA, AdaLoRA, DoRA, rsLoRA, LoRA+, PiSSA, OLoRA, EVA and LoftQ only where
  verified; CH-08 and CH-11 concepts; FIG-05–06, implementation, compatibility matrix, and controlled baseline.

### Part 4 contracts

- **CH-10:** account for weights, gradients, optimizer states, activations, KV cache, buffers, bandwidth and latency;
  CH-02; memory worksheet and hardware-aware bottleneck analysis.
- **CH-11:** explain and test quantization formats, granularity, PTQ/QAT, calibration and named algorithms; CH-10 and
  CH-29–31 measurement concepts; FIG-07–08, TBL-06–07, reproducible protocol, and result schemas populated only by
  executed measurements.
- **CH-12:** distinguish parameter sparsity from portable speedup and test pruning/recovery; CH-10 and CH-31 concepts;
  FIG-09, TBL-08, dense-dimension and sparse-kernel checks.
- **CH-13:** implement response, sequence, logit, feature, attention and domain distillation; CH-02, CH-06 and CH-10;
  FIG-10–11, TBL-09, tokenizer-mismatch controls, and five-arm experiment.
- **CH-14:** classify factorization, decomposition, layer dropping, early exit, weight sharing, vocabulary/embedding
  compression, entropy coding and hardware-aware search by maturity; CH-10–13; applicability register with rejection
  reasons.
- **CH-15:** order compatible fine-tuning, distillation, pruning, recovery, QAT and quantization operations; CH-09–14;
  FIG-12, TBL-10 and branch-specific rollback protocols.

### Part 5 contracts

- **CH-16:** distinguish storage fit, runtime-memory fit, interactive performance and sustained operation; CH-10;
  cloud/edge trade-off model and explicit offline/privacy constraints.
- **CH-17:** connect ARM CPU, GPU, NPU, memory bandwidth and thermal management to runtime choice; CH-16; tiered device
  characterization procedure without assumed specifications.
- **CH-18:** verify llama.cpp/GGUF, ExecuTorch, LiteRT-LM, ONNX Runtime Mobile, Core ML, MLX Swift, MLC LLM, MNN, NCNN
  and vendor paths; CH-02, CH-11 and CH-17; versioned runtime decision matrix with unsupported cells.
- **CH-19:** separate useful single-user mobile optimizations from server techniques; CH-02 and CH-16–18; FIG-17–18 and
  controlled optimization protocol.
- **CH-20:** build an offline Android path from conversion through lifecycle, integrity, streaming and updates;
  CH-18–19; release-build listing, FIG-19/25, compatibility manifest and physical-device validation instructions.
- **CH-21:** build the corresponding Swift/iOS path and explain platform differences; CH-18–19; signed-release listing,
  compatibility manifest and physical-device validation instructions.
- **CH-22:** assess ARM64 Linux, Raspberry Pi-class, Jetson-class, industrial, laptop and WebGPU targets honestly;
  CH-16–19; target-specific feasibility and benchmark plans.

### Part 6 contracts

- **CH-23:** discover legally usable, authoritative data for every required domain; governance plus CH-01 concepts;
  source register with rights, provenance, update frequency and acquisition method.
- **CH-24:** implement ingestion, extraction/OCR, normalization, deduplication, contradiction/PII handling, lineage and
  leakage-safe splitting; CH-23; reusable pipeline specification and failure quarantine.
- **CH-25:** produce non-interchangeable CPT, SFT, conversation, reasoning, preference, distillation, calibration and
  evaluation datasets; CH-24; schemas, coverage model and rare-critical-case controls.
- **CH-26:** generate and validate synthetic examples without laundering unsupported claims; CH-23–25 and CH-29
  concepts; teacher provenance, filtering, expert review and collapse/contamination controls.

### Parts 7–8 contracts

- **CH-27:** compare independent models, adapters, unified models, continual training, composition, merging, routing and
  MoE; CH-05–09 and CH-13; interference/storage/update/safety decision matrix.
- **CH-28:** specify configuration-driven ingestion, training, evaluation, compression, export, registry, monitoring and
  rollback; CH-24–27; repository structure, FIG-01/21 and platform contracts.
- **CH-29:** define quality, factuality, uncertainty, hallucination, robustness, retention, safety and statistical
  evaluation; CH-06 and CH-25; metric registry, rubric, confidence procedure and contamination controls.
- **CH-30:** design trustworthy automotive, pharmacy, beekeeping, zookeeping and other professional suites; CH-23–26 and
  CH-29; domain schemas, reviewer qualifications, risk-stratified gates and escalation.
- **CH-31:** measure artifact size, memory, startup, TTFT, prefill/decode, energy, thermal, failures and cost under
  controlled conditions; CH-10–19 and CH-29; FIG-23, standardized records and device/cloud protocols.
- **CH-32:** build staged ablations and target-specific Pareto fronts without universal-winner claims; CH-11–15 and
  CH-29–31; FIG-13–15/24, registered matrix and uncertainty analysis.

### Parts 9–10 contracts

- **CH-33:** choose only necessary EC2, SageMaker AI, S3, ECR, orchestration, IAM, network, observability, security and
  IaC services; CH-28 and CH-31; FIG-20 and TBL-24 with dated capability evidence.
- **CH-34:** train CPT, SFT/LoRA/QLoRA, distillation and compression on AWS with recovery and cleanup; CH-04–15 and
  CH-33; pinned container/IaC workflow and cost telemetry.
- **CH-35:** compare current serving runtimes and implement an authenticated, observable, versioned endpoint; CH-31 and
  CH-33–34; serving package, load protocol, canary/rollback and cost controls.
- **CH-36:** sign, publish, resume-download, verify and roll back edge artifacts while preserving offline inference;
  CH-20–21 and CH-33–35; FIG-26, manifest/signature contract and release drill.
- **CH-37:** execute the 17-phase automotive case study from scope to five target-specific selections; all prior
  chapters as phase dependencies; exact companion path, immutable evidence chain and explicit unexecuted cells.

### Parts 11–14 contracts

- **CH-38:** adapt the platform to pharmaceutical education with regulatory, recency and clinical-escalation controls;
  CH-23–30 and CH-37; domain delta, qualified-review protocol and prohibited-use statement.
- **CH-39:** test shared mechanical foundations versus separate motorcycle/truck specialization; CH-27 and CH-37;
  transfer/interference ablation.
- **CH-40:** adapt to lower-resource beekeeping and zookeeping while controlling regional/species uncertainty; CH-23–30
  and CH-37; source/reviewer/uncertainty plan.
- **CH-41:** onboard a new domain primarily through configuration, data and evaluation; CH-28 and CH-37–40; template,
  validation gates and rollback.
- **CH-42:** secure model/data/dependency supply chains, mobile artifacts and inference APIs; CH-28, CH-33–36; threat
  model, controls and incident evidence.
- **CH-43:** version, monitor, shadow/canary, review, retrain, deprecate and roll back cloud/edge releases; CH-29–36 and
  CH-42; lifecycle state machine and privacy-safe telemetry.
- **CH-44:** compare lifecycle costs with dated prices and editable equations; CH-31 and CH-33–43; assumptions sheet and
  sensitivity analysis, never fabricated prices.
- **CH-45:** preregister the 15 required research questions with variables, controls, metrics, hardware, repetitions,
  limitations and criteria; CH-29–32; hypothesis registry tied to protocol IDs.
- **CH-46:** stage model/training/compression/runtime/target experiments under small, medium and research budgets;
  CH-45; elimination matrix and resource caps, not a full Cartesian product.
- **CH-47:** integrate data-to-release architecture for research, AWS, Android, iOS, multi-domain management and
  approvals; CH-28, CH-33–46; seven architecture diagrams, evolution path and governance boundaries.

## Global artifact ownership

The canonical chapter owns conceptual prose, equations, chapter figures/tables, and chapter references.
Executable-intent procedures live in `03-implementation-handbook/`, formal trials in `06-experiment-framework/`,
automotive execution contracts in `07-automotive-case-study/`, deployment companions in `08-aws-guide/` and
`09-edge-guide/`, and decisions in `14-decision-framework/`. Cross-link instead of copying.

## Study checkpoints

After CH-03, explain and inspect a complete decoder-only inference path. After CH-09, produce a reproducible adaptation
and compare its trainable state with full tuning. After CH-15, defend one compression sequence from mechanism and target
support. After CH-26, produce rights-aware objective datasets with leakage evidence. After CH-32, preregister an
ablation and interpret a target-specific Pareto front. After CH-36, deploy and roll back one cloud artifact and one
signed edge package. CH-37 integrates these demonstrations; CH-38–47 test transfer, production reasoning, and research
judgment.

Preserve CH identifiers because assessments, figures, protocols, and cross-references depend on them. Chapter prose,
implementation, experiment execution, and independent review retain separate evidence states under the
[definition of done](../00-governance/definition-of-done.md).
