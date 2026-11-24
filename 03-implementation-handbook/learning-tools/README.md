# Learning tools

## Teaching workflow

For every tool, read the formula and define each symbol before touching the CLI. Reproduce the hand-worked fixture, predict the JSON result, run the self-test, run the fixture command, and compare the output field by field. Then replace one illustrative input at a time with a traced real value. A tool is useful only when its units, provenance, and limitations accompany the result.

A complete tool exercise produces: the hand calculation, extracted script hash, Python version, exact command, input or measurement provenance, JSON output, one rejected malformed case, and an interpretation that explicitly separates arithmetic from empirical evidence.

These eleven standalone, standard-library Python CLIs implement prompt §10's
learning tools. Verification note: their extracted syntax and documented
arithmetic fixtures passed on 2026-11-24. Real artifact, hardware, quality,
performance, and price validation remains input-specific.

## Tool index

1. [TOOL-01 — Parameter/model-size calculator](tool-01-parameter-model-size.md)
2. [TOOL-02 — Quantization-memory calculator](tool-02-quantization-memory.md)
3. [TOOL-03 — LoRA trainable-parameter calculator](tool-03-lora-parameters.md)
4. [TOOL-04 — GPU training-memory estimator](tool-04-gpu-training-memory.md)
5. [TOOL-05 — Mobile model feasibility estimator](tool-05-mobile-feasibility.md)
6. [TOOL-06 — Inference-latency calculator](tool-06-inference-latency.md)
7. [TOOL-07 — KV-cache memory estimator](tool-07-kv-cache.md)
8. [TOOL-08 — Teacher–student compression comparison](tool-08-teacher-student.md)
9. [TOOL-09 — Quality-versus-compression visualization](tool-09-quality-compression-visualization.md)
10. [TOOL-10 — AWS training-cost estimator](tool-10-aws-training-cost.md)
11. [TOOL-11 — Cloud-versus-mobile inference comparison](tool-11-cloud-mobile-comparison.md)

## Shared contract

- Extract only bytes inside each `extract` marker's single primary `python` fence to its declared intended path under
  `slm-implementation/`; preserve UTF-8, LF endings, and one final newline.
- Python 3.11+ standard library is the only dependency. No listing performs network access or supplies hidden defaults.
- Inputs are explicit CLI arguments or provenance-bearing JSON/JSONL. JSON is emitted to standard output; errors are
  nonzero exits. Decimal units use bytes/MB/GB; binary units use MiB/GiB and are never interchanged.
- Hand-calculated fixtures are educational assertions, not measured model, GPU, phone, latency, quality, or price
  results. Run the shown `--self-test`, then independent tests against real artifacts and measurements.
- See the [listing manifest](../listing-manifest.md), [code-listing standard](../../00-governance/code-listing-standard.md),
  [CH-31](../../02-textbook/part-08-evaluation-benchmarking-and-research-experiments/ch-31-compression-and-edge-benchmarking.md),
  and [CH-44](../../02-textbook/part-12-production-engineering-safety-and-maintenance/ch-44-cost-optimization.md).

Do not promote a tool to `EXECUTED` from syntax checking or self-tests alone. Retain extracted hash, environment,
commands, inputs, outputs, failures, reviewer, and execution date in the verification ledger.
