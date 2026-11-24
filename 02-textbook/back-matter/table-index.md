<a id="table-index"></a>

# Table Index

Status reflects the 2026-11-23 audit. “Authored” does not imply measured, executed, or release-reviewed. The
[figure/table ledger](../../15-audit-ledgers/figure-table-ledger.md) controls closure.

Use TBL-01–20 to compare concepts and methods before implementation. Use TBL-21–25 as decision instruments: fill them
only from a frozen protocol and linked evidence. A qualitative table organizes trade-offs; it does not rank methods
unless its rows share a valid comparison basis.

- **TBL-01 — LLM versus SLM.**
  [CH-01](../part-01-language-model-foundations/ch-01-evolution-of-language-models.md#tbl-01). Authored qualitative
  comparison.
- **TBL-02 — Base versus instruction-tuned models.**
  [CH-01](../part-01-language-model-foundations/ch-01-evolution-of-language-models.md#tbl-02). Authored qualitative
  comparison.
- **TBL-03 — Training from scratch versus CPT versus SFT.**
  [CH-04](../part-02-training-and-domain-adaptation/ch-04-pretraining-fundamentals.md#tbl-03). Authored qualitative
  comparison.
- **TBL-04 — Full fine-tuning versus LoRA versus QLoRA.**
  [CH-09](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#tbl-04). `PLANNED EXPERIMENT`; no
  measured results.
- **TBL-05 — LoRA variants.**
  [CH-09](../part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md#tbl-05). Evidence/maturity
  comparison, not a ranking.
- **TBL-06 — Quantization formats.**
  [CH-11](../part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md#tbl-06).
  Ideal payload and engineering comparison; target performance unresolved.
- **TBL-07 — Quantization algorithms.**
  [CH-11](../part-04-model-compression-and-optimization/ch-11-quantization-complete-research-and-implementation.md#tbl-07).
  Research/documentation synthesis.
- **TBL-08 — Pruning methods.**
  [CH-12](../part-04-model-compression-and-optimization/ch-12-pruning-complete-research-and-implementation.md#tbl-08).
  Mechanism synthesis; empirical outcomes `TEST-REQUIRED`.
- **TBL-09 — Distillation methods.**
  [CH-13](../part-04-model-compression-and-optimization/ch-13-knowledge-distillation-complete-research-and-implementation.md#tbl-09).
  Research synthesis.
- **TBL-10 — Combined compression pipelines.**
  [CH-15](../part-04-model-compression-and-optimization/ch-15-combining-optimization-techniques.md#tbl-10). Engineering
  comparison; outcomes require experiments.
- **TBL-11 — Model families and sizes.**
  [CH-03](../part-01-language-model-foundations/ch-03-modern-slm-architectures.md#tbl-11). Verified landscape; not a
  quality ranking.
- **TBL-12 — GPU memory requirements.**
  [CH-10](../part-04-model-compression-and-optimization/ch-10-the-fundamentals-of-model-compression.md#tbl-12), with
  CH-34 application guidance. Formula/component treatment; no capacity benchmark.
- **TBL-13 — Training costs.**
  [CH-44](../part-12-production-engineering-safety-and-maintenance/ch-44-cost-optimization.md#tbl-13). `ESTIMATE`
  template with blank rates; no fabricated prices.
- **TBL-14 — Edge inference runtimes.**
  [CH-18](../part-05-edge-ai-and-mobile-inference/ch-18-edge-inference-frameworks.md#tbl-14). Documentation-based
  capabilities; not a measured ranking.
- **TBL-15 — Android versus iOS deployment.**
  [CH-21](../part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md#tbl-15), with CH-20 companion treatment.
  Documentation-based capabilities and recommendations.
- **TBL-16 — CPU versus GPU versus NPU.**
  [CH-17](../part-05-edge-ai-and-mobile-inference/ch-17-mobile-hardware-architecture.md#tbl-16). Qualitative guidance;
  no performance ranking.
- **TBL-17 — Cloud versus edge inference.**
  [CH-16](../part-05-edge-ai-and-mobile-inference/ch-16-edge-ai-fundamentals.md#tbl-17). Qualitative engineering
  comparison.
- **TBL-18 — Independent domain models versus adapters.**
  [CH-27](../part-07-multi-domain-slm-engineering/ch-27-single-domain-versus-multi-domain-strategies.md#tbl-18).
  Illustrative engineering priors.
- **TBL-19 — Domain-specific datasets.**
  [CH-23](../part-06-data-acquisition-and-dataset-engineering/ch-23-finding-domain-specific-training-data.md#tbl-19),
  with CH-25 objective-design detail. Static decision aid, not a license opinion.
- **TBL-20 — Evaluation benchmarks.**
  [CH-29](../part-08-evaluation-benchmarking-and-research-experiments/ch-29-model-quality-evaluation.md#tbl-20), with
  CH-30 domain-suite detail. Applicability and limitations, not a score ranking.
- **TBL-21 — Inference performance.**
  [CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md#tbl-21).
  `PLANNED EXPERIMENT`; populate from exact-target repeated trials with workload and uncertainty.
- **TBL-22 — Energy consumption.**
  [CH-31](../part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md#tbl-22).
  `PLANNED EXPERIMENT`; populate from a synchronized power boundary, idle baseline, and repeated workload.
- **TBL-23 — Optimization quality retention.**
  [CH-32](../part-08-evaluation-benchmarking-and-research-experiments/ch-32-optimization-experiments-and-ablation-studies.md#tbl-23).
  `PLANNED EXPERIMENT`; populate with paired baseline/candidate results on the same sealed suite.
- **TBL-24 — AWS service selection.**
  [CH-33](../part-09-aws-training-optimization-and-deployment/ch-33-aws-infrastructure-for-slm-development.md#tbl-24).
  `DOC-VERIFIED` capabilities plus explicit recommendations.
- **TBL-25 — Production architecture choices.**
  [CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md#tbl-25). Engineering hypotheses
  with stated evidence requirements.

For TBL-21–TBL-23, preserve protocol, raw measurements, analysis configuration, uncertainty, failures, and review trace.
Until those exist, the useful content is the method for producing the table—not invented cells.
