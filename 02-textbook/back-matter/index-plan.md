<a id="technical-index"></a>

# Detailed Technical Index

Entries link to the canonical chapter or reference page. Section anchors are used only where explicitly stable; chapter
files remain the durable fallback.

## How to use this index

Start with the term you encountered, open its conceptual chapter, then follow any equation, implementation, or
measurement link listed beside it. For example, “KV cache” leads first to Transformer mechanics (CH-02), then mobile
optimization (CH-19), then the reusable formula in Appendix A. Use your editor's search for narrower names and aliases;
the glossary supplies short definitions while this index supplies learning destinations.

Compound topics may appear under both the method and the deployment concern: quantization is under Q, while mobile
compatibility is under backends and edge feasibility. Product names are indexed only when a chapter treats them
substantively.

## A

- **Ablation studies:**
  [CH-32](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md);
  [CH-46](../part-13-experimental-research-roadmap/ch-46-benchmark-matrix.md)
- **Adapters / adapter routing:**
  [CH-08](../part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md);
  [CH-09](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md);
  [CH-27](../part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md)
- **Android:** [CH-20](../part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md);
  [CH-36](../part-09-aws-training-optimization-and-deployment/ch-36-aws-to-edge-model-delivery.md)
- **Attention (causal, MHA, MQA, GQA):**
  [CH-02](../part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md);
  [mathematics](appendix-mathematical-reference.md)
- **AWS:**
  [CH-33 infrastructure](../part-09-aws-training-optimization-and-deployment/ch-33-aws-infrastructure-for-slm-development.md);
  [CH-34 training](../part-09-aws-training-optimization-and-deployment/ch-34-training-slms-on-aws.md);
  [CH-35 serving](../part-09-aws-training-optimization-and-deployment/ch-35-aws-inference-serving.md);
  [CH-36 edge delivery](../part-09-aws-training-optimization-and-deployment/ch-36-aws-to-edge-model-delivery.md)

## B

- **Backends and operator fallback:**
  [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md);
  [hardware reference](appendix-hardware-and-memory-reference.md)
- **Base versus instruct models:** [CH-01](../part-01-language-model-foundations/ch-01-evolution-of-language-models.md)
- **Benchmark matrix:** [CH-46](../part-13-experimental-research-roadmap/ch-46-benchmark-matrix.md)
- **Beekeeping:** [CH-40](../part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md)

## C

- **Calibration:**
  quantization—[CH-11](../part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md);
  probabilistic—[CH-29](../part-08-evaluation-benchmarking-and-research-experiments/ch-29-model-quality-evaluation.md)
- **Cloud versus edge:** [CH-16](../part-05-edge-ai-and-mobile-inference/ch-16-edge-ai-fundamentals.md);
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md)
- **Compression:** [Part IV](../part-04-model-compression-and-optimization/README.md);
  measurement—[CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
- **Continued pretraining (CPT):**
  [CH-05](../part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md)
- **Cost:** [CH-44](../part-12-production-engineering-safety-and-maintenance/ch-44-cost-optimization.md)
- **Core ML:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md);
  [CH-21](../part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md)

## D

- **Data acquisition, rights, and provenance:**
  [CH-23](../part-06-data-acquisition-and-dataset-engineering/ch-23-finding-domain-specific-training-data.md)
- **Dataset processing and deduplication:**
  [CH-24](../part-06-data-acquisition-and-dataset-engineering/ch-24-dataset-collection-and-processing.md)
- **Dataset schemas by objective:**
  [CH-25](../part-06-data-acquisition-and-dataset-engineering/ch-25-dataset-design-for-different-training-objectives.md)
- **Decision framework:** [final synthesis](final-decision-framework.md)
- **Distillation:**
  [CH-13](../part-04-model-compression-and-optimization/ch-13-knowledge-distillation-complete-research-and-implementation.md);
  equations—[mathematics](appendix-mathematical-reference.md)
- **Domain onboarding:** [CH-41](../part-11-extending-the-platform-to-other-domains/ch-41-new-domain-onboarding.md)

## E

- **Edge feasibility gates:** [Part V](../part-05-edge-ai-and-mobile-inference/README.md);
  [hardware reference](appendix-hardware-and-memory-reference.md)
- **Energy and thermals:**
  [CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md);
  [hardware reference](appendix-hardware-and-memory-reference.md)
- **Evidence labels:** [reader guide](../front-matter/evidence-labels.md);
  [governance](../../00-governance/evidence-and-citation-standard.md)
- **Experiment reporting:** [checklist](appendix-experiment-reporting-checklist.md);
  [CH-32](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md)
- **ExecuTorch:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)

## F–H

- **Fine-tuning, full:** [CH-08](../part-03-parameter-efficient-fine-tuning/ch-08-full-fine-tuning-versus-peft.md)
- **Figures:** [figure index](figure-index.md)
- **GGUF / llama.cpp:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- **GPU/CPU/NPU:** [CH-17](../part-05-edge-ai-and-mobile-inference/ch-17-mobile-hardware-architecture.md);
  [hardware reference](appendix-hardware-and-memory-reference.md)
- **Hallucination/factuality:**
  [CH-29](../part-08-evaluation-benchmarking-and-research-experiments/ch-29-model-quality-evaluation.md)
- **Hardware and memory:** [appendix](appendix-hardware-and-memory-reference.md)

## I–L

- **Inference serving:** [CH-35](../part-09-aws-training-optimization-and-deployment/ch-35-aws-inference-serving.md)
- **iOS:** [CH-21](../part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md)
- **KV cache:** [CH-02](../part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md);
  [CH-19](../part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md);
  [mathematics](appendix-mathematical-reference.md)
- **Latency (TTFT/decode):** [CH-19](../part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md);
  [CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md)
- **LiteRT/LiteRT-LM:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- **LoRA family:** [CH-09](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md)

## M–P

- **Mathematics:** [Appendix A](appendix-mathematical-reference.md)
- **Memory ledger:**
  [CH-10](../part-04-model-compression-and-optimization/ch-10-the-fundamentals-of-model-compression.md);
  [hardware reference](appendix-hardware-and-memory-reference.md)
- **MLC LLM, MNN, ncnn:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- **Mobile optimization:** [CH-19](../part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md)
- **Model families:** [CH-03](../part-01-language-model-foundations/ch-03-modern-slm-architectures.md)
- **Monitoring and lifecycle:**
  [CH-43](../part-12-production-engineering-safety-and-maintenance/ch-43-monitoring-and-model-lifecycle.md)
- **Multi-domain strategies:**
  [CH-27](../part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md)
- **No runtime RAG:** [governance](../../00-governance/no-runtime-rag-standard.md);
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md)
- **ONNX Runtime Mobile/GenAI:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- **Pareto selection:**
  [CH-32](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md);
  [decision framework](final-decision-framework.md)
- **Pharmacy:** [CH-38](../part-11-extending-the-platform-to-other-domains/ch-38-pharmaceutical-knowledge-slm.md)
- **Preference optimization:**
  [CH-07](../part-02-training-and-domain-adaptation/ch-07-preference-optimization-and-post-training.md)
- **Pretraining:** [CH-04](../part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md)
- **Pruning:**
  [CH-12](../part-04-model-compression-and-optimization/ch-12-pruning-complete-research-and-implementation.md)

## Q–S

- **QAIRT/QNN/Genie:** [CH-17](../part-05-edge-ai-and-mobile-inference/ch-17-mobile-hardware-architecture.md);
  [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- **QLoRA:** [CH-09](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md)
- **Quantization:**
  [CH-11](../part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md)
- **Research questions:** [CH-45](../part-13-experimental-research-roadmap/ch-45-research-questions-and-hypotheses.md)
- **Risk tiers R0–R4:** [safety guide](../front-matter/safety-and-responsible-use.md)
- **Runtimes:** [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md)
- **Security and reliability:**
  [CH-42](../part-12-production-engineering-safety-and-maintenance/ch-42-model-security-and-reliability.md)
- **SFT:** [CH-06](../part-02-training-and-domain-adaptation/ch-06-supervised-fine-tuning.md)
- **Synthetic data:**
  [CH-26](../part-06-data-acquisition-and-dataset-engineering/ch-26-synthetic-data-and-knowledge-generation.md)

## T–Z

- **Tables:** [table index](table-index.md)
- **Tokenizer/chat template:**
  [CH-02](../part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md);
  [CH-06](../part-02-training-and-domain-adaptation/ch-06-supervised-fine-tuning.md)
- **Transformer:** [CH-02](../part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md)
- **Updates and signed delivery:**
  [CH-36](../part-09-aws-training-optimization-and-deployment/ch-36-aws-to-edge-model-delivery.md)
- **Veterinary/animal-care escalation:**
  [CH-40](../part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md)
- **Zookeeping:** [CH-40](../part-11-extending-the-platform-to-other-domains/ch-40-beekeeping-and-zookeeping-slms.md)

For exact chapter ordering, use the [textbook index](../README.md). For unresolved terminology or broken identities, use
[errata and limitations](errata-and-limitations.md).

If a useful term is absent, add the preferred term, common aliases, and links to the page that defines, implements, and
evaluates it. Do not add an empty entry that promises future content.
