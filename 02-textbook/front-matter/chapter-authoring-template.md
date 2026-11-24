<a id="chapter-template"></a>

# Worked Chapter Model

This is a complete miniature lesson, not a page of blanks. It demonstrates the structure used by the book while
teaching a real first calculation: ideal model-weight storage. Authors may reuse the pattern for major revisions;
learners can use it to practice separating estimates from measurements.

## Example metadata

- **Lesson:** EX-00 — Can the weights fit?
- **Prerequisites:** bits, bytes, parameter count, decimal GB, and binary GiB
- **Scope:** ideal FP16 and INT4 weight payload
- **Exclusions:** metadata, activations, KV cache, workspaces, application memory, and measured speed
- **Outcome:** calculate payload and explain why it is not a device-memory guarantee
- **Evidence:** formulas are derived; example values are `ILLUSTRATIVE`
- **Safety:** R0 arithmetic; any deployment inherits its application's higher risk tier

## 1. Problem and motivation

A team asks whether quantizing a 1.5-billion-parameter model from FP16 to INT4 makes it fit a phone. Arithmetic can
estimate the ideal parameter payload. It cannot prove that the converted artifact loads, runs, or retains quality.

## 2. Intuition

Imagine each parameter as an item in a warehouse. FP16 gives each item a 16-bit box; ideal INT4 gives it a 4-bit box.
Smaller boxes reduce payload, but the warehouse still needs labels, shelves, aisles, and working space. A runtime
likewise needs quantization metadata, KV cache, activations, operator workspaces, and application memory.

## 3. Definitions and terminology

- $P$: number of parameters.
- $b$: stored bits per parameter in the idealized calculation.
- **Weight payload:** parameter-value bytes alone.
- **Peak memory:** maximum observed memory within a declared workload boundary.

## 4. Mechanism

Quantization maps higher-precision values into a smaller representable set and stores values plus scales and, in some
schemes, zero points. At inference, a kernel may dequantize blocks or compute through a native low-precision path.
Reduced storage therefore does not automatically imply reduced peak memory or latency.

## 5. Mathematical treatment

The ideal payload in bytes is

$$

M_W=\frac{P b}{8}.

$$

Division by eight converts bits to bytes. Decimal GB uses $10^9$ bytes; binary GiB uses $2^{30}$ bytes. This formula
deliberately excludes all non-weight allocations.

## 6. Worked example

`ILLUSTRATIVE`: with $P=1.5\times10^9$:

- FP16: $1.5\times10^9\times16/8=3.0\times10^9$ bytes = 3.00 GB, about 2.79 GiB.
- ideal INT4: $1.5\times10^9\times4/8=0.75\times10^9$ bytes = 0.75 GB, about 0.70 GiB.
- ideal payload reduction: $3.0/0.75=4$ times.

The valid conclusion is “the ideal payload is four times smaller.” “The model uses 0.70 GiB on a phone” is not yet
supported because no runtime measurement exists.

## 7. Implementation

`UNVERIFIED IMPLEMENTATION` — LST-EX00-01:

```python
def ideal_weight_bytes(parameters: int, bits_per_parameter: float) -> float:
    if parameters < 0 or bits_per_parameter <= 0:
        raise ValueError("parameters must be nonnegative and bits must be positive")
    return parameters * bits_per_parameter / 8

for bits in (16, 4):
    value = ideal_weight_bytes(1_500_000_000, bits)
    print(bits, value / 1e9, "GB", value / 2**30, "GiB")
```

Promotion to `EXECUTED` requires the command, interpreter version, output, date, and source revision.

## 8. Resource and systems analysis

For deployment, add tokenizer files, scales and zero points, alignment, graph buffers, prompt activations, KV cache,
operator workspaces, UI memory, and OS pressure. Then measure cold load, peak memory, TTFT, sustained decode, energy,
and thermals on the exact runtime and device.

## 9. Strengths, weaknesses, and alternatives

The formula is fast and useful for rejecting obviously oversized candidates. It is insufficient for acceptance because
grouped quantizers add metadata and runtimes allocate differently. A converter output measures artifact size; an
exact-target profiler measures peak memory.

## 10. Domain and safety application

Automotive and pharmacy models use the same arithmetic, but quantization regressions have different consequences. A
pharmacy candidate requires R3 safety evaluation and qualified review. Passing memory limits never compensates for an
unsafe output.

## 11. Edge and cloud application

On edge, RAM pressure, operator fallback, battery, and thermals may dominate. On AWS, concurrency, accelerator memory,
batching, and cost may dominate. Measure both; a server-GPU result is not mobile-NPU evidence.

## 12. Evaluation and failure analysis

Compare FP16 and INT4 with identical prompts and decoding. Record artifact size, loaded and peak memory, quality,
critical safety cases, TTFT, decode, and failures. If size falls but latency rises, inspect kernel fallback. If quality
falls, test calibration data, grouping, sensitive layers, or a less aggressive format.

## 13. Summary and assessment

You can now estimate ideal weight payload and state its limits. Check yourself:

1. Why is a 4× payload reduction not a 4× speed claim?
2. Which allocations are absent from $M_W$?
3. For 800 million parameters at 8 bits, what is the ideal payload? Answer: 0.8 GB.
4. What evidence proves the candidate fits a specific phone?

Proceed when answer 4 names an exact artifact, runtime, workload, observed peak, and safety margin.

## 14. References and status

The formula is also derived in the [mathematical reference](../back-matter/appendix-mathematical-reference.md); the full
memory ledger is in the [hardware reference](../back-matter/appendix-hardware-and-memory-reference.md). This lesson has
no empirical performance claim: values are `ILLUSTRATIVE`, and the listing is `UNVERIFIED IMPLEMENTATION`.

## Figure and table requirements

Every figure/table needs a stable ID, descriptive title, evidence class, source or derivation, scope, and in-text
interpretation. Follow the [figure/table standard](../../00-governance/figure-table-standard.md). Diagrams must remain
understandable in source form and rendered output.

## Author and learner completion check

- All objectives are addressed and prerequisites are correct.
- Links and anchors resolve; IDs are unique.
- Equations define symbols and units.
- Claims have citations and honest evidence labels.
- Listings state execution status and do not expose secrets.
- Safety and architectural boundaries are explicit.
- Required figures/tables are present and indexed.
- Markdown, code fences, and Mermaid blocks validate.
- No unfinished marker or implied execution remains.
