# Implementation listing manifest

## Purpose and reusable procedure

The manifest is an inventory and verification ledger, not a list of promises. Each row identifies a teaching tool, its canonical listing, extraction path, expected input/output, and validation command. To add a tool, assign the next stable IDs, document units and assumptions, include a hand calculation and self-test, extract the exact fenced bytes, record SHA-256 and Python identity, and archive both successful and malformed-input runs.

### Fully populated example

For `TOOL-01`, extract `LST-TOOL-0001` to `tools/parameter_model_size.py`, run its self-test, then run the ten-parameter worked example. The expected JSON contains `parameter_count: 10`, `payload_bytes: 20`, and `total_bytes: 28`. A duplicate tensor name must exit nonzero. Record the extracted file hash and command output before promotion.

## Learning-tool registry

Verification note: on 2026-11-24 the eleven listings were extracted to a
temporary directory, parsed, byte-compiled, and exercised with their documented
self-test or arithmetic fixture. That check validates listing syntax and fixture
behavior only; it does not validate hardware fit, measured quality, AWS prices,
or target performance.

<!-- markdownlint-disable MD013 -->
| Tool | Listing and path | Input → output | Exact fixture check |
| --- | --- | --- | --- |
| TOOL-01 | [Parameter/model size](learning-tools/tool-01-parameter-model-size.md), `tools/parameter_model_size.py` | tensor shapes and bits → parameter and byte totals | 10 parameters, 20-byte payload, 28-byte total |
| TOOL-02 | [Quantization memory](learning-tools/tool-02-quantization-memory.md), `tools/quantization_memory.py` | quantizer layout → code and metadata bytes | 3 groups, 5 code bytes, 19 total bytes |
| TOOL-03 | [LoRA parameters](learning-tools/tool-03-lora-parameters.md), `tools/lora_parameters.py` | module dimensions and ranks → trainable counts | two modules produce 40 parameters |
| TOOL-04 | [GPU training memory](learning-tools/tool-04-gpu-training-memory.md), `tools/gpu_training_memory.py` | memory ledger → total and headroom | 560 bytes total, 440 bytes headroom |
| TOOL-05 | [Mobile feasibility](learning-tools/tool-05-mobile-feasibility.md), `tools/mobile_feasibility.py` | storage/RAM components → arithmetic gates | 750 installed and 500 active bytes pass |
| TOOL-06 | [Inference latency](learning-tools/tool-06-inference-latency.md), `tools/inference_latency.py` | monotonic events and token counts → latency/rates | 130 ms TTFT, 500 prefill, 20 decode tok/s |
| TOOL-07 | [KV cache](learning-tools/tool-07-kv-cache.md), `tools/kv_cache.py` | architecture and context → cache payload | GQA, 128 bytes/token, 1,280 bytes total |
| TOOL-08 | [Teacher/student](learning-tools/tool-08-teacher-student.md), `tools/teacher_student.py` | paired records → compression and quality delta | 4× ratios and −0.05 quality delta |
| TOOL-09 | [Quality/compression SVG](learning-tools/tool-09-quality-compression-visualization.md), `tools/quality_compression_svg.py` | measured JSONL → SVG and summary | two points map to opposite plot corners |
| TOOL-10 | [AWS training cost](learning-tools/tool-10-aws-training-cost.md), `tools/aws_training_cost.py` | dated meter lines → subtotals and total | arithmetic-only fixture totals 23 currency units |
| TOOL-11 | [Cloud/mobile comparison](learning-tools/tool-11-cloud-mobile-comparison.md), `tools/cloud_mobile_comparison.py` | paired deployment records → deltas and failure rates | +40 ms TTFT and −12 decode tok/s |
<!-- markdownlint-restore -->

## Promotion boundary

For each listing, extract exact fenced bytes, calculate SHA-256, retain Python/environment identity, run syntax/static
checks and documented fixtures, add malformed-input tests, execute against governed real inputs where applicable, and
archive command, stdout, stderr, exit code, outputs, failures, reviewer, and UTC date. Hardware-, price-, quality-, and
performance-related tools additionally require independent source/measurement review. Update the
[implementation verification ledger](../15-audit-ledgers/implementation-verification-ledger.md) only after evidence
exists.
