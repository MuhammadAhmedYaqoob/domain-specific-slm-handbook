# Diagram learning plan

## How diagrams support learning

The 33 briefs are not decorative requirements. Each visual should let a learner reconstruct a mechanism, compare
alternatives, or identify a measurement boundary that prose alone can obscure. Authored schematics remain
`ILLUSTRATIVE`, formula-only FIG-07/18 remain `ESTIMATE`, and FIG-13–15/24 remain `PLANNED EXPERIMENT` until reviewed
measurements exist. Canonical registration does not imply project execution.

Every figure must include a stable ID, descriptive caption, alt text, readable legend, explicit units, source/protocol
link, evidence label, and editable source. Educational schematics are `ILLUSTRATIVE`. Measured charts require immutable
result and analysis hashes. Paper/vendor plots must be redrawn only when licensing permits and must be labelled
`PAPER-REPORTED` or `VENDOR-REPORTED`; they never appear as project measurements.

Use three passes. First, predict the diagram from the caption before viewing it. Second, trace one concrete example
through every arrow and annotate tensor shapes, units, ownership, or trust boundaries. Third, redraw it from memory and
explain which elements are measured, inferred, or illustrative. A figure is learned when the reader can use it to make
a design decision and can name its failure to represent reality.

## Learning objectives by figure group

- **FIG-01–06:** explain the model and adaptation lifecycle from tokens to trainable low-rank updates.
- **FIG-07–15:** distinguish compression mechanisms from measured quality, memory, and latency outcomes.
- **FIG-16–19 and FIG-25:** reason about hardware placement, inference phases, memory growth, and offline application
  behavior.
- **FIG-20, FIG-26–33:** identify cloud, release, security, governance, rollback, and multi-domain boundaries.
- **FIG-21–24:** connect data lineage and evaluation evidence to target-specific model selection.

## Mandatory figure briefs

- **FIG-01 — End-to-end SLM lifecycle.** Owner CH-28. Labels: source/rights, raw bytes, validated objective datasets,
  base revision, adaptation, compression, evaluation gates, registry, AWS/edge release, monitoring, rollback. Data
  class: `ILLUSTRATIVE`.
- **FIG-02 — Decoder-only Transformer.** Owner CH-02. Labels: token IDs, embeddings, position representation, repeated
  causal attention/RMSNorm/MLP/residual blocks, logits, sampler, generated token; tensor shapes required.
  `ILLUSTRATIVE`.
- **FIG-03 — Self-attention and token processing.** Owner CH-02. Labels: Q/K/V projections, causal mask, scale by square
  root of key dimension, softmax, weighted values, heads and output projection; dimensions required. `ILLUSTRATIVE`.
- **FIG-04 — Pretraining versus CPT versus SFT.** Owner CH-05. Labels: initialization, corpus type, objective, updated
  weights/adapters, expected learning role, evaluation branch, forgetting checks. `ILLUSTRATIVE`.
- **FIG-05 — Full FT versus LoRA versus QLoRA.** Owner CH-09. Labels: frozen/trainable tensors, optimizer states,
  quantized storage versus compute dtype, merge/export boundary. No relative-memory numbers without a declared
  model/config. `ILLUSTRATIVE`.
- **FIG-06 — Low-rank update.** Owner CH-09. Labels: frozen weight W, matrices A/B, rank r, alpha scaling, input/output
  dimensions, additive update and merge. `ILLUSTRATIVE`.
- **FIG-07 — Precision memory comparison.** Owner CH-11. Axes: parameter count and theoretical payload bytes; series:
  FP32/FP16/BF16/INT8/INT4/lower-bit; overhead for scales/zero-points/metadata shown separately. Formula-derived values
  labelled `ESTIMATE`.
- **FIG-08 — Quantization calibration and error.** Owner CH-11. Labels: representative calibration inputs,
  activation/weight statistics, scale/group selection, quantize/dequantize, layer sensitivity, error metric and held-out
  quality gate. `ILLUSTRATIVE` unless fed measured traces.
- **FIG-09 — Pruning structures.** Owner CH-12. Labels: original matrix, arbitrary zeros, channel/head/layer removal,
  N:M pattern, stored representation and compatible kernel path. `ILLUSTRATIVE`.
- **FIG-10 — Teacher–student distillation.** Owner CH-13. Labels: frozen teacher, student, shared/translated inputs,
  hard labels, soft targets, temperature, losses, reviewer and deployment child. `ILLUSTRATIVE`.
- **FIG-11 — Response, logit and feature distillation.** Owner CH-13. Three aligned panels with supervision location,
  tokenizer/alignment requirement, retained artifact and failure modes. `ILLUSTRATIVE`.
- **FIG-12 — Combined compression workflow.** Owner CH-15. Labels: source baseline, branch order, calibration/recovery
  data, parity gates after each transform, target export, rejection and rollback. `ILLUSTRATIVE`.
- **FIG-13 — Quality versus parameter count.** Owner CH-32. Axes include metric/unit and effective parameter definition;
  points encode exact model revision and uncertainty. `PLANNED EXPERIMENT`; no points until reviewed results exist.
- **FIG-14 — Quality versus latency.** Owner CH-32. Axes include quality metric and p95 visible TTFT or sustained decode
  with units; facet by exact target/workload. `PLANNED EXPERIMENT`.
- **FIG-15 — Quality versus memory.** Owner CH-32. Axes include quality metric and measured peak resident bytes; facet
  by target/runtime. `PLANNED EXPERIMENT`.
- **FIG-16 — CPU versus GPU versus NPU.** Owner CH-17. Labels: memory placement/copies, supported operators, fallback,
  compiler/runtime and measurement boundary. Educational diagram first; measured overlay only per exact device.
  `ILLUSTRATIVE`.
- **FIG-17 — Prefill versus decode.** Owner CH-19. Labels: prompt tokens, parallel prefill, first-token boundary,
  autoregressive decode loop, KV reads/writes, TTFT and inter-token metrics. `ILLUSTRATIVE`.
- **FIG-18 — KV-cache growth.** Owner CH-19. Axes: cached tokens and bytes; parameters: layers, KV heads, head
  dimension, dtype bytes, batch. Formula-derived curves are `ESTIMATE`; measured allocator curves remain separate.
- **FIG-19 — Mobile inference pipeline.** Owner CH-20, with CH-21 variant. Labels: signed package, integrity check,
  storage, memory map/load, tokenizer/template, runtime/backend, streaming UI, cancellation, telemetry and offline
  boundary. `ILLUSTRATIVE`.
- **FIG-20 — AWS training and deployment.** Owner CH-33. Labels: account/Region/VPC trust boundaries, S3/ECR, training,
  registry, endpoint, API/auth, logs/metrics, CI/CD, rollback and cleanup. Services shown only when selected.
  `ILLUSTRATIVE`.
- **FIG-21 — Multi-domain models/adapters.** Owner CH-27. Panels: independent models, shared frozen base with adapters,
  unified model, router; labels include storage, load/switch and isolation boundaries. `ILLUSTRATIVE`.
- **FIG-22 — Dataset engineering pipeline.** Owner CH-24. Labels: acquisition receipt, rights gate, extraction/OCR,
  normalize, deduplicate roots, quality/PII/safety quarantine, objective transforms, split, registry and takedown
  lineage. `ILLUSTRATIVE`.
- **FIG-23 — Benchmark and evaluation workflow.** Owner CH-31. Labels: approved plan, immutable tuple, scorer fixtures,
  raw generation once, quality/system scoring, failures, paired statistics, review and release decision. `ILLUSTRATIVE`.
- **FIG-24 — Target-specific Pareto analysis.** Owner CH-32. Labels: hard-gate exclusion, objective directions/units,
  uncertainty/MID, dominance, non-dominated set and preference-sensitive selection. A worked synthetic example is
  `ILLUSTRATIVE`; a project front requires `EXECUTED` evidence.
- **FIG-25 — Offline installation and inference.** Owner CH-20 with CH-21 companion. Labels: bundled/downloaded
  artifact, resumable transfer, signature/hash, compatibility/storage gate, atomic activation, local-only generation,
  prior-version rollback. `ILLUSTRATIVE`.
- **FIG-26 — Model update lifecycle.** Owner CH-36. Labels: train/evaluate/sign/publish, manifest channel, client
  compatibility, staged rollout, health/quality feedback, revoke and rollback; cloud independence after install
  explicit. `ILLUSTRATIVE`.

## Additional production architecture figures

- **FIG-27 — Research and training infrastructure.** Owner CH-47. Show orchestration, isolated workers, datasets,
  tracking, registries, secrets, evidence and trust boundaries.
- **FIG-28 — AWS MLOps platform.** Owner CH-47. Show IaC, accounts/environments, artifact promotion, endpoint release,
  observability, approval and teardown.
- **FIG-29 — Offline Android application.** Owner CH-47. Show app/runtime/JNI if used, local model/tokenizer, integrity,
  streaming, lifecycle and no-network inference boundary.
- **FIG-30 — Offline iOS application.** Owner CH-47. Show SwiftUI/service/runtime, local artifacts, compute backend,
  streaming, lifecycle and no-network inference boundary.
- **FIG-31 — Multi-domain management.** Owner CH-47. Show domain config, source/data/model/adapters, evaluation suites,
  access policy, registry and independent rollback.
- **FIG-32 — Cloud-to-edge distribution.** Owner CH-47. Show signed manifest/package, object storage/CDN if selected,
  resumable download, compatibility, atomic install, revocation and rollback.
- **FIG-33 — Evaluation and release approval.** Owner CH-47. Show sealed tests, expert review,
  safety/rights/compatibility/system gates, Pareto decision, signatures and rejected branches.

FIG-27–33 inherit the same evidence rules and are `ILLUSTRATIVE` until populated with an approved implementation. CH-47
now uses these canonical IDs directly while retaining readable “architecture 1” through “architecture 7” role text; the
former `FIG-CH47-01`–`07` identities are retired. They satisfy CH-47’s seven-architecture requirement without
renumbering FIG-01–26.

## Production gate

Before merge, validate Mermaid or vector syntax, text contrast, grayscale legibility, minimum print size, alt text,
caption cross-reference, unique ID, source license, and link integrity. Before publication, the figure/table ledger must
name the final asset path and status. Measured figures additionally require raw-data, protocol, analysis-code and review
links.

## Worked illustrative exercise

For FIG-14, create four synthetic candidates with explicitly invented quality and p95 TTFT values, label the figure
`ILLUSTRATIVE`, and identify dominated points. Then replace the synthetic rows with reviewed result records from one
exact device/workload, add uncertainty intervals, and change the evidence label only after protocol and analysis
review. This exercise teaches why plausible-looking points are not measurements and why fronts cannot be transferred
between targets.

## Cross-links

- [Figure/table standard](../00-governance/figure-table-standard.md)
- [Figure/table ledger](../15-audit-ledgers/figure-table-ledger.md)
- [Experiment framework](../06-experiment-framework/README.md)
