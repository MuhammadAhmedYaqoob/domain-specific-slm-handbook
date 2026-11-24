<a id="notation"></a>

# Notation and Conventions

## Mathematical notation

Read an expression in three passes: identify each object's meaning, check its shape or unit, then ask what operation
changes it. For example, $X\in\mathbb{R}^{T\times d}$ means $T$ token representations with $d$ features each.
Multiplying by $W_Q\in\mathbb{R}^{d\times d_h}$ produces
$Q=XW_Q\in\mathbb{R}^{T\times d_h}$; the shared inner dimension $d$ must agree.

| Symbol                     | Meaning                                    |
| -------------------------- | ------------------------------------------ |
| $x_{1:T}$               | Token sequence of length $T$             |
| $p_\theta$              | Model parameterized by $\theta$          |
| $V$                      | Vocabulary size                            |
| $d$, $d_h$             | Model width and per-head width             |
| $L$, $H$               | Number of layers and attention heads       |
| $W$, $A$, $B$        | Base matrix and low-rank factors           |
| $r$, $\alpha$          | Adapter rank and scaling coefficient       |
| $b_w$, $b_{kv}$        | Weight and KV-cache bit widths             |
| $B$, $S$               | Batch size and sequence length             |
| $N$                      | Context or token count, as locally defined |
| $\mathbb{E}$             | Expectation                                |
| $\lVert\cdot\rVert$      | Norm; subscript identifies type            |
| $\operatorname{softmax}$ | Normalized exponential                     |

Bold uppercase denotes matrices, bold lowercase vectors, plain lowercase scalars, and calligraphic letters datasets or
sets unless a chapter declares otherwise. Shapes use `[batch, sequence, feature]` order unless stated. Logarithms are
natural logs. Probabilities are dimensionless.

### Tutorial: reading a loss

$$

\mathcal{L}=-\frac{1}{N}\sum_{i=1}^{N}\log p_\theta(x_i\mid x_{<i})

$$

For each of $N$ predicted tokens, take the probability assigned to the correct token, apply the natural log, negate
it, and average. If the correct-token probabilities in an `ILLUSTRATIVE` two-token example are 0.5 and 0.25, then

$$

\mathcal{L}=-\frac{\log(0.5)+\log(0.25)}{2}\approx1.040,
\qquad \operatorname{PPL}=e^{1.040}\approx2.83.

$$

This teaches the calculation; it is not a measured model result.

### Tutorial: checking a LoRA shape

For $W\in\mathbb{R}^{d_{\text{out}}\times d_{\text{in}}}$, choose
$A\in\mathbb{R}^{r\times d_{\text{in}}}$ and
$B\in\mathbb{R}^{d_{\text{out}}\times r}$. Then
$BA\in\mathbb{R}^{d_{\text{out}}\times d_{\text{in}}}$, so it can be added to $W$.

## Units

Use SI units for time, energy, power, and frequency. Distinguish decimal storage (`MB`, `GB`) from binary memory (`MiB`,
`GiB`). Report tokens/s with batch size and decode conditions. Latency reports distinguish time to first token (TTFT),
inter-token latency, and end-to-end latency. Memory reports distinguish artifact storage, steady loaded memory, KV-cache
growth, workspace allocations, and observed peak resident memory.

## Model and training terms

- **Base model:** pretrained next-token model before instruction alignment.
- **Instruct model:** model aligned to follow instructions.
- **Continued pretraining (CPT):** next-token training on additional domain corpus.
- **Supervised fine-tuning (SFT):** training on input/target demonstrations.
- **Full fine-tuning:** updates all selected model parameters.
- **PEFT:** updates a restricted trainable parameterization.
- **PTQ/QAT:** post-training quantization / quantization-aware training.

See the [terminology standard](../../00-governance/terminology-standard.md) for controlling definitions.

## Systems conventions

“CPU,” “GPU,” and “NPU” name execution classes, not guaranteed operator support. Backend claims require runtime,
version, model format, target SoC/device, OS, and fallback behavior. “On-device” means required inference executes
locally after artifact installation; optional telemetry does not convert a local inference path into a cloud path.

## Evidence and identifiers

Evidence labels appear in code formatting, for example `EXECUTED`. Stable IDs use `CH-*`, `FIG-*`, `TBL-*`, `LST-*`,
`CLM-*`, and `SRC-*`. Required project-wide figures use `FIG-01`–`FIG-26`; chapter-local figures may use `FIG-CHxx-yy`.

## Architectural convention

“No runtime RAG” excludes vector search, dynamic corpus retrieval, retrieved prompt injection, and remote teacher
fallback from primary inference. Offline source acquisition, training-data construction, evaluation references, and
human review are not runtime RAG.

## Uncertainty

Use “may” for possibility, “should” for a recommendation, and “must” for a binding requirement. Avoid “supports,”
“works,” “efficient,” “real-time,” and “production-ready” without a declared scope and evidence trace.

When a symbol is reused locally, the chapter must redefine it. In particular, $B$ can mean batch size while bold
$\mathbf{B}$ can denote a LoRA matrix; typography and local definitions prevent ambiguity.
