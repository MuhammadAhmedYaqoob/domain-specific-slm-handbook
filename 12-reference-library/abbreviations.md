# Abbreviations

Expand an abbreviation on first substantive use in each chapter. The **Use** column explains where the abbreviation
belongs; it is not a second definition. Case matters: `B` usually means bytes and `b` means bits.

## Models, architecture, and training

| Short form | Expansion | Use |
| --- | --- | --- |
| AI | Artificial Intelligence | Broad field; prefer a specific method when possible. |
| ALiBi | Attention with Linear Biases | Position-dependent attention bias. |
| BERT | Bidirectional Encoder Representations from Transformers | Encoder-only model family and historical baseline. |
| BPE | Byte Pair Encoding | Subword-tokenization family. |
| CPT | Continued Pretraining | Additional self-supervised training from a checkpoint. |
| DAPT | Domain-Adaptive Pretraining | CPT on a domain distribution. |
| DL | Deep Learning | Neural learning with multilayer models. |
| DPO | Direct Preference Optimization | Preference training from chosen/rejected responses. |
| FFN | Feed-Forward Network | Per-token nonlinear Transformer sublayer. |
| FSDP | Fully Sharded Data Parallel | Distributed sharding of parameters, gradients, and optimizer state. |
| GPT | Generative Pretrained Transformer | Autoregressive Transformer style or named family; identify the exact model. |
| GQA | Grouped-Query Attention | Multiple query heads sharing fewer key/value heads. |
| GRPO | Group Relative Policy Optimization | Policy optimization using group-relative rewards; state the implementation. |
| GRU | Gated Recurrent Unit | Gated recurrent architecture. |
| IA3 | Infused Adapter by Inhibiting and Amplifying Inner Activations | Parameter-efficient activation scaling method. |
| IPO | Identity Preference Optimization | Preference objective; distinguish it from Initial Public Offering. |
| KD | Knowledge Distillation | Teacher-to-student training. |
| KTO | Kahneman–Tversky Optimization | Preference learning from desirable/undesirable examples. |
| LLM | Large Language Model | Deployment-relative size category, never a universal threshold. |
| LM | Language Model | Model of token-sequence probabilities. |
| LoRA | Low-Rank Adaptation | Factorized low-rank weight updates. |
| LSTM | Long Short-Term Memory | Gated recurrent architecture. |
| MHA | Multi-Head Attention | Separate query, key, and value heads. |
| ML | Machine Learning | Broad field of data-driven models. |
| MLP | Multilayer Perceptron | Dense neural subnetwork, often the Transformer FFN. |
| MoE | Mixture of Experts | Sparse expert-routing architecture. |
| MQA | Multi-Query Attention | Query heads sharing one key/value head pair. |
| NLL | Negative Log-Likelihood | Token-prediction loss and basis of perplexity. |
| NoPE | No Positional Encoding | Architecture omitting explicit position encoding. |
| ORPO | Odds Ratio Preference Optimization | Joint supervised/preference objective. |
| PEFT | Parameter-Efficient Fine-Tuning | Adaptation that trains a small parameter subset. |
| PPO | Proximal Policy Optimization | Reinforcement-learning policy update used in some RLHF pipelines. |
| PPL | Perplexity | Predictive-fit metric for a fixed tokenizer and corpus. |
| QKV | Query, Key, Value | The three attention projections or tensors. |
| QLoRA | Quantized Low-Rank Adaptation | LoRA training through a frozen quantized base. |
| RAG | Retrieval-Augmented Generation | Runtime retrieval used to augment generation; excluded from the core product. |
| RLHF | Reinforcement Learning from Human Feedback | Human-preference post-training pipeline. |
| RLOO | REINFORCE Leave-One-Out | Policy-gradient estimator with a leave-one-out baseline. |
| RMSNorm | Root Mean Square Layer Normalization | Normalization by root-mean-square magnitude. |
| RNN | Recurrent Neural Network | Sequential model with recurrent hidden state. |
| RoPE | Rotary Position Embedding | Rotation-based relative-position mechanism. |
| rsLoRA | Rank-Stabilized Low-Rank Adaptation | LoRA scaling using the square root of rank. |
| SFT | Supervised Fine-Tuning | Training on explicit target outputs. |
| SLM | Small Language Model | Model compact relative to declared constraints. |
| SSM | State-Space Model | Sequence model based on learned state transitions. |
| TAPT | Task-Adaptive Pretraining | CPT on text close to a target task. |

## Compression and numerical formats

| Short form | Expansion | Use |
| --- | --- | --- |
| AQLM | Additive Quantization for Language Models | Codebook-based low-bit weight compression. |
| AWQ | Activation-aware Weight Quantization | Calibration-guided weight-only quantization. |
| BF16 | Brain Floating Point, 16-bit | Floating-point dtype with FP32-like exponent width. |
| FLOP | Floating-Point Operation | Arithmetic-count unit; FLOPs do not equal elapsed time. |
| FP4/8/16/32 | 4/8/16/32-bit Floating Point | Precision families; name the exact format where variants exist. |
| GEMM | General Matrix Multiplication | Core dense linear-algebra operation. |
| GGML | Name of the ggml tensor library | Proper project name; do not invent an expansion. |
| GGUF | Name of the current ggml model format | Container/format name, not a quantizer or runtime. |
| GPTQ | Name of the GPTQ method | Layer-wise second-order PTQ; no project-wide expansion is asserted. |
| HQQ | Half-Quadratic Quantization | Weight-only quantization without representative calibration data. |
| INT2/3/4/8 | 2/3/4/8-bit Integer | Integer precision; specify signedness, granularity, and kernel. |
| NF4 | NormalFloat 4-bit | Non-uniform representation used in QLoRA workflows. |
| N:M | N retained values per group of M | Semi-structured sparsity under this repository's convention. |
| PTQ | Post-Training Quantization | Quantization after model training. |
| QAT | Quantization-Aware Training | Training with target quantization effects. |
| SpQR | Sparse-Quantized Representation | Quantization with sparse high-precision outlier handling. |
| TF32 | TensorFloat-32 | NVIDIA compute format; not ordinary IEEE FP32. |
| W4A16/W8A8 | Weight/activation bit-width shorthand | Example: W4A16 means 4-bit weights and 16-bit activations. |

## Hardware, runtimes, and edge

| Short form | Expansion | Use |
| --- | --- | --- |
| ABI | Application Binary Interface | Binary calling and packaging compatibility. |
| ANE | Apple Neural Engine | Apple on-device neural accelerator. |
| API | Application Programming Interface | Software contract exposed to callers. |
| ARM64 | 64-bit ARM Architecture | Mobile and edge CPU architecture; also called AArch64 in many tools. |
| CPU | Central Processing Unit | General-purpose processor. |
| CUDA | NVIDIA parallel-computing platform and API name | NVIDIA GPU software stack; treat as a proper name. |
| EP | Execution Provider | ONNX Runtime hardware/backend implementation. |
| GPU | Graphics Processing Unit | Parallel processor for graphics and compute. |
| HTP | Hexagon Tensor Processor | Qualcomm accelerator component. |
| JNI | Java Native Interface | Java/Kotlin bridge to Android native code. |
| KV | Key–Value | Attention cache tensors; define the cache on first use. |
| NPU | Neural Processing Unit | Dedicated neural accelerator; capabilities vary by SoC. |
| ONNX | Open Neural Network Exchange | Graph interchange format and ecosystem. |
| ORT | ONNX Runtime | Runtime that executes ONNX graphs through providers. |
| QAIRT | Qualcomm AI Runtime | Qualcomm deployment SDK/runtime family. |
| QNN | Qualcomm Neural Network | Qualcomm backend/API name used in acceleration tooling. |
| RAM | Random-Access Memory | Main volatile memory; distinguish host and accelerator memory. |
| SIMD | Single Instruction, Multiple Data | Vector execution across multiple values. |
| SoC | System on Chip | Integrated CPU, GPU, NPU, memory controllers, and other components. |
| TTFT | Time to First Token | Latency to first usable output under stated boundaries. |
| VRAM | Video Random-Access Memory | Accelerator memory; unified-memory devices may not expose a separate pool. |

Names such as Metal, MLX, Vulkan, WebGPU, and XNNPACK are product or project names rather than abbreviations. Explain
their role on first use instead of inventing expansions.

## Data, evaluation, infrastructure, and security

| Short form | Expansion | Use |
| --- | --- | --- |
| CI | Confidence Interval | Statistical uncertainty; spell out Continuous Integration in automation contexts. |
| EM | Exact Match | Metric requiring an exact normalized answer match. |
| EOS | End of Sequence | Token marking generation termination. |
| F1 | Harmonic mean of precision and recall | Classification/extraction balance metric. |
| FPR/FNR | False-Positive/False-Negative Rate | Error rates important when costs differ. |
| JSON/JSONL | JavaScript Object Notation / JSON Lines | Structured object format / one JSON value per line. |
| OOD | Out of Distribution | Inputs outside the training or evaluation distribution. |
| PII | Personally Identifiable Information | Data requiring privacy handling. |
| QA | Question Answering | Task name; spell out Quality Assurance when that meaning is intended. |
| RNG | Random Number Generator | Source of pseudorandomness controlled by seeds and state. |
| SHA-256 | Secure Hash Algorithm, 256-bit | Digest for artifact identity and integrity checking. |
| AMI | Amazon Machine Image | Reusable EC2 machine image. |
| ARN | Amazon Resource Name | Unique AWS resource identifier. |
| AWS | Amazon Web Services | Cloud platform. |
| CDK | Cloud Development Kit | Infrastructure-as-code framework; say AWS CDK in this book. |
| EBS | Elastic Block Store | Block storage for EC2. |
| EC2 | Elastic Compute Cloud | AWS virtual compute service. |
| ECR | Elastic Container Registry | AWS container-image registry. |
| ECS/EKS | Elastic Container Service / Elastic Kubernetes Service | AWS container orchestration services. |
| IaC | Infrastructure as Code | Versioned infrastructure definitions. |
| IAM | Identity and Access Management | AWS identity and permission controls. |
| KMS | Key Management Service | Managed key service; qualify the provider. |
| S3 | Simple Storage Service | AWS object storage. |
| SLA/SLO | Service-Level Agreement / Service-Level Objective | Contractual commitment / internal reliability target. |
| VPC | Virtual Private Cloud | Isolated AWS network boundary. |
| AAB/APK | Android App Bundle / Android Package Kit | Android publishing bundle / installable package. |
| CVE | Common Vulnerabilities and Exposures | Public vulnerability identifier. |
| RBAC | Role-Based Access Control | Permissions assigned through roles. |
| SBOM | Software Bill of Materials | Inventory of software components and versions. |
| TLS | Transport Layer Security | Network encryption and peer authentication protocol. |
| TOS | Terms of Service | Contractual service-use rules; do not treat them as a software license. |

Cross-links: [glossary](glossary.md) · [definition register](definition-register.md) · [library index](README.md)
