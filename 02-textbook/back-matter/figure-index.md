<a id="figure-index"></a>

# Figure Index

Status reflects the 2026-11-23 repository audit, not intended final state. “Authored” means concept/diagram text exists;
it does not imply accessibility review, rendering validation, or project execution. The
[figure/table ledger](../../15-audit-ledgers/figure-table-ledger.md) is the controlling closure record.

Use this index as a visual study route. Start with FIG-01 for the lifecycle, FIG-02–03 for model mechanics, FIG-04–12
for training and compression, FIG-16–19 for device execution, and FIG-20–33 for delivery and production architecture.
Figures labeled `ILLUSTRATIVE` explain relationships; those labeled `ESTIMATE` derive values from assumptions; a
`PLANNED EXPERIMENT` entry is a chart specification, not missing or implied data.

- **FIG-01 — End-to-end SLM lifecycle.**
  [CH-28 canonical figure](../part-07-multi-domain-slm-engineering/ch-28-reusable-domain-training-platform.md#fig-01).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-02 — Decoder-only Transformer.**
  [CH-02 canonical figure](../part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md#fig-02).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-03 — Self-attention and token processing.**
  [CH-02 canonical figure](../part-01-language-model-foundations/ch-02-transformer-architecture-in-depth.md#fig-03).
  `ILLUSTRATIVE`; authored with symbolic dimensions and alt text.
- **FIG-04 — Pretraining versus CPT versus SFT.**
  [CH-05 canonical figure](../part-02-training-and-domain-adaptation/ch-05-continued-pretraining-and-domain-adaptation.md#fig-04).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-05 — Full FT versus LoRA versus QLoRA.**
  [CH-09 canonical figure](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#fig-05).
  `ILLUSTRATIVE`; authored.
- **FIG-06 — Low-rank update.**
  [CH-09 canonical figure](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#fig-06).
  `ILLUSTRATIVE`; authored.
- **FIG-07 — Precision memory comparison.**
  [CH-11 canonical figure](../part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md#fig-07).
  `ESTIMATE`; formula-derived with overhead excluded explicitly.
- **FIG-08 — Quantization calibration and error.**
  [CH-11 canonical figure](../part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md#fig-08).
  `ILLUSTRATIVE`; authored.
- **FIG-09 — Pruning structures.**
  [CH-12 canonical figure](../part-04-model-compression-and-optimization/ch-12-pruning-complete-research-and-implementation.md#fig-09).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-10 — Teacher–student distillation.**
  [CH-13 canonical figure](../part-04-model-compression-and-optimization/ch-13-knowledge-distillation-complete-research-and-implementation.md#fig-10).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-11 — Response, logit, and feature distillation.**
  [CH-13 canonical figure](../part-04-model-compression-and-optimization/ch-13-knowledge-distillation-complete-research-and-implementation.md#fig-11).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-12 — Combined compression workflow.**
  [CH-15 canonical figure](../part-04-model-compression-and-optimization/ch-15-combining-optimization-techniques.md#fig-12).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-13 — Quality versus parameter count.**
  [CH-32 canonical specification](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md#fig-13).
  `PLANNED EXPERIMENT`; construct only from artifact-linked quality results using a fixed suite.
- **FIG-14 — Quality versus latency.**
  [CH-32 canonical specification](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md#fig-14).
  `PLANNED EXPERIMENT`; plot paired exact-target latency distributions and quality results.
- **FIG-15 — Quality versus memory.**
  [CH-32 canonical specification](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md#fig-15).
  `PLANNED EXPERIMENT`; use observed peak memory under one frozen workload, not ideal payload.
- **FIG-16 — CPU versus GPU versus NPU.**
  [CH-17 canonical figure](../part-05-edge-ai-and-mobile-inference/ch-17-mobile-hardware-architecture.md#fig-16).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-17 — Prefill versus decode.**
  [CH-19 canonical figure](../part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#fig-17).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-18 — KV-cache growth.**
  [CH-19 canonical figure](../part-05-edge-ai-and-mobile-inference/ch-19-mobile-inference-optimization.md#fig-18).
  `ESTIMATE`; formula-derived, with measured allocator behavior explicitly unresolved.
- **FIG-19 — Mobile inference pipeline.**
  [CH-20 canonical figure](../part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#fig-19). `ILLUSTRATIVE`;
  authored with alt text; CH-18 retains a CH-scoped companion.
- **FIG-20 — AWS training and deployment.**
  [CH-33 canonical figure](../part-09-aws-training-optimization-and-deployment/ch-33-aws-infrastructure-for-slm-development.md#fig-20).
  `ILLUSTRATIVE`; authored with alt text; CH-34/35 alternatives are CH-scoped.
- **FIG-21 — Multi-domain models/adapters.**
  [CH-27 canonical figure](../part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#fig-21).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-22 — Dataset engineering pipeline.**
  [CH-24 canonical figure](../part-06-data-acquisition-and-dataset-engineering/ch-24-dataset-collection-and-processing.md#fig-22).
  `ILLUSTRATIVE`; authored with alt text; CH-23 source discovery is CH-scoped.
- **FIG-23 — Benchmark and evaluation workflow.**
  [CH-31 canonical figure](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md#fig-23).
  `ILLUSTRATIVE`; authored with alt text.
- **FIG-24 — Target-specific Pareto analysis.**
  [CH-32 canonical specification](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md#fig-24).
  `PLANNED EXPERIMENT`; remove hard-gate failures, preserve uncertainty, then compute non-dominated candidates.
- **FIG-25 — Offline installation and inference.**
  [CH-20 canonical figure](../part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md#fig-25), with CH-21
  companion treatment. `ILLUSTRATIVE`; authored with alt text.
- **FIG-26 — Model update lifecycle.**
  [CH-36 canonical figure](../part-09-aws-training-optimization-and-deployment/ch-36-aws-to-edge-model-delivery.md#fig-26).
  `ILLUSTRATIVE`; authored with alt text; CH-35 cloud-only lifecycle is CH-scoped.

## Additional architecture figures

CH-47's seven local architecture roles are canonically registered as
[FIG-27 through FIG-33](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-27). All are
`ILLUSTRATIVE`, retain readable local-role captions, and do not claim implementation.

- **FIG-27 — Research and training infrastructure.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-27).
- **FIG-28 — AWS MLOps platform.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-28).
- **FIG-29 — Offline Android application.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-29).
- **FIG-30 — Offline iOS application.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-30).
- **FIG-31 — Multi-domain management.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-31).
- **FIG-32 — Cloud-to-edge distribution.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-32).
- **FIG-33 — Evaluation and release approval.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#fig-33).

No measured figure in this index is represented as executed. FIG-13–FIG-15 are validation recipes: generate them only
from immutable protocols, raw data, analysis, uncertainty, and reviewed artifacts.

To interpret any figure, read its caption, evidence label, axes and units, workload, and source before drawing a
conclusion. A conceptual arrow shows flow, not measured speed; a formula-derived curve shows assumptions, not observed
hardware behavior.
