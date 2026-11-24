<a id="audience"></a>

# Audience and Prerequisites

## Intended readers

The primary reader can already build or operate software systems and wants to make defensible SLM decisions:

- ML and AI engineers selecting, adapting, compressing, and evaluating models;
- researchers designing controlled domain-adaptation or edge-inference studies;
- mobile, embedded, and cloud engineers integrating inference runtimes;
- data engineers building rights-aware corpora and evaluation sets;
- MLOps, security, and platform engineers governing releases;
- domain experts reviewing automotive, pharmacy, animal-care, industrial, or related content;
- technical leads evaluating feasibility, cost, risk, and architecture.

The book is not a first programming course, a complete linear-algebra course, or a substitute for professional domain
training.

## Knowledge prerequisites

### Required

- Python: environments, packages, functions, classes, files, exceptions, and command-line execution.
- ML basics: training/validation/test splits, overfitting, loss functions, optimization, and classification metrics.
- Neural networks: tensors, matrix multiplication, embeddings, gradients, backpropagation, and minibatches.
- Mathematics: algebra, logarithms, vectors and matrices, probability notation, expectations, and basic derivatives.
- Systems: processes, files, APIs, memory, latency, throughput, serialization, and version control.

### Helpful but taught in context

- Transformer internals and autoregressive decoding;
- PyTorch and Hugging Face workflows;
- CUDA/distributed training concepts;
- quantization, pruning, distillation, LoRA, and QLoRA;
- Linux, containers, AWS identity/networking/storage/compute;
- Android/iOS build systems and native runtime integration;
- experimental design, confidence intervals, and statistical testing.

## Readiness self-check

Try these before choosing a route:

1. If a tensor has shape `[2, 128, 512]`, can you identify batch, sequence, and feature axes?
2. Can you explain why validation examples must not be copied into training data?
3. Can you convert 4 GiB to bytes and distinguish it from 4 GB?
4. Can you create an isolated Python environment and record exact package versions?
5. Can you explain the difference between latency and throughput?

If 1–3 are unfamiliar, review tensors, data splits, and binary units before Part I. If 4 is unfamiliar, practice
environment and version-control basics before running listings. If 5 is unfamiliar, the systems tutorials in Parts IV
and V teach it, but do not begin benchmarking until you can state both metrics with units and workload.

## Practical environments

Conceptual reading requires no hardware. Reproducing the proposed work requires different environments:

- **development:** Git, Python, an isolated environment, storage for versioned artifacts, and a supported deep-learning
  stack;
- **training:** a declared GPU/accelerator environment appropriate to model size and method;
- **AWS:** an authorized account, least-privilege IAM, budgets, encryption, and region/service availability checks;
- **edge:** exact target device or representative device class, runtime/toolchain, profiling access, and thermal/power
  controls;
- **domain review:** reviewers qualified for the applicable safety and regulatory context.

No chapter should be interpreted as guaranteeing that a package version, operator, accelerator, or model artifact works
on a specific target. Compatibility must be verified against the pinned release matrix.

## Entry routes

- New to language models: begin with [Part I](../part-01-language-model-foundations/README.md).
- Comfortable with Transformers: begin with [Part II](../part-02-training-and-domain-adaptation/README.md), then revisit
  CH-02 as needed.
- Focused on mobile deployment: read CH-10, then [Part V](../part-05-edge-ai-and-mobile-inference/README.md), followed
  by Parts VIII and XIV.
- Focused on data and domain learning: read Parts II, VI, VII, and VIII.
- Leading a production program: read [How to use this book](how-to-use-this-book.md), Parts VIII, XII–XIV, and the
  [decision framework](../back-matter/final-decision-framework.md).

Readers without GPU access can still complete the conceptual calculations, inspect datasets and configurations, design
evaluation suites, and run tiny CPU-scale exercises. Treat those exercises as learning evidence, not as performance
evidence for the production-sized artifact.

## Safety prerequisite

Anyone implementing R2–R4 workflows must understand the
[safety tiers and release constraints](safety-and-responsible-use.md). ML expertise alone is insufficient for domain
sign-off.
