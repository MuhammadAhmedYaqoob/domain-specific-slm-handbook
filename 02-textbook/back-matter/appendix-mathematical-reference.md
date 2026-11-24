<a id="mathematical-reference"></a>

# Appendix A — Mathematical Reference

This appendix collects the equations repeatedly used across the book. It defines mechanisms; it does not supply measured
project results.

Use each section in the same way: identify symbols and units, substitute a small example, check dimensions, then state
what the result does **not** prove.

## Autoregressive language modeling

For tokens \(x_{1:T}\), the chain rule factorization is

\[ p_\theta(x_{1:T})=\prod_{t=1}^{T}p_\theta(x_t\mid x_{<t}). \]

The mean negative log-likelihood over \(N\) predicted tokens is

\[ \mathcal{L}_{\mathrm{NLL}}(\theta)=-\frac{1}{N}\sum_{i=1}^{N}\log p_\theta(x_i\mid x_{<i}). \] <!-- markdownlint-disable-line MD013 MD049 -->

For a one-hot target \(y\) and predicted distribution \(p\), cross-entropy is

\[ H(y,p)=-\sum_{v=1}^{V} y_v\log p_v. \]

Perplexity under the same tokenization and masking convention is

\[ \operatorname{PPL}=\exp(\mathcal{L}_{\mathrm{NLL}}). \]

Perplexities are not directly comparable when tokenizer, corpus, masking, or aggregation differs.

### Worked loss example

`ILLUSTRATIVE`: a model assigns probabilities 0.8, 0.5, and 0.25 to three correct next tokens. Then

\[
\mathcal{L}_{\mathrm{NLL}}
=-\frac{\log 0.8+\log 0.5+\log 0.25}{3}
\approx0.768,
\qquad
\operatorname{PPL}\approx e^{0.768}\approx2.16.
\]

This result applies only to this fixed tokenization and example; it is not a model benchmark.

## Attention

For input \(X\in\mathbb{R}^{T\times d}\),

\[ Q=XW_Q,\quad K=XW_K,\quad V=XW_V. \]

Scaled dot-product attention is

\[ \operatorname{Attn}(Q,K,V)=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_h}}+M\right)V, \]

where \(d_h\) is head width and causal mask \(M_{ij}=-\infty\) when token \(i\) must not attend to token \(j\).
Multi-head attention concatenates head outputs and applies \(W_O\).

RoPE rotates query/key coordinate pairs at position \(m\):

\[ \begin{bmatrix}x'_{2i}\\x'_{2i+1}\end{bmatrix} = \begin{bmatrix}\cos(m\omega_i)&-\sin(m\omega_i)\\ <!-- markdownlint-disable-line MD013 MD049 -->
\sin(m\omega_i)&\cos(m\omega_i)\end{bmatrix} \begin{bmatrix}x_{2i}\\x_{2i+1}\end{bmatrix}. \]

## Optimization and regularization

A generic parameter update is

\[ \theta_{t+1}=\theta_t-\eta_t\,\widehat{g}_t, \]

where \(\eta_t\) is learning rate and \(\widehat{g}_t\) an optimizer-transformed gradient estimate. For a mixed
objective,

\[ \mathcal{L}=\sum_k \lambda_k\mathcal{L}_k, \]

and the \(\lambda_k\) values, normalization, and sampling process must be reported.

## LoRA and adapters

For frozen \(W_0\in\mathbb{R}^{d_{\mathrm{out}}\times d_{\mathrm{in}}}\),

\[ W'=W_0+\Delta W,\qquad \Delta W=\frac{\alpha}{r}BA, \]

with \(A\in\mathbb{R}^{r\times d_{\mathrm{in}}}\), \(B\in\mathbb{R}^{d_{\mathrm{out}}\times r}\), rank \(r\), and scale
\(\alpha\). The adapter has \(r(d_{\mathrm{in}}+d_{\mathrm{out}})\) trainable matrix elements for that projection. QLoRA
stores the frozen base quantized while optimizing adapter parameters in a declared compute precision; it is not
“quantized LoRA weights” by definition.

`ILLUSTRATIVE`: a \(4096\times4096\) projection with rank 16 gains
\(16(4096+4096)=131{,}072\) trainable elements, about 0.78% of that projection's \(16{,}777{,}216\) base elements. Count
every targeted module before estimating a whole model.

## Uniform affine quantization

Given integer range \([q_{\min},q_{\max}]\), range endpoints \(x_{\min},x_{\max}\), scale \(s\), and zero point \(z\),

\[ s=\frac{x_{\max}-x_{\min}}{q_{\max}-q_{\min}},\qquad
z=\operatorname{clip}\left(\operatorname{round}\left(q_{\min}-\frac{x_{\min}}{s}\right)\right), \]

\[ q=\operatorname{clip}\left(\operatorname{round}\left(\frac{x}{s}\right)+z\right),\qquad \hat{x}=s(q-z). \]

Symmetric quantization commonly sets \(z=0\) and chooses \(s\) from the maximum absolute value. Per-channel/group
methods use distinct scales over declared dimensions. Metadata and alignment make real storage larger than \(P b/8\).

## Sparsity and compression

For \(P\) weights and \(Z\) zeros, unstructured sparsity is \(s_p=Z/P\). This ratio does not imply smaller dense tensors
or faster execution. A simple payload compression ratio is

\[ C=\frac{S_{\mathrm{baseline}}}{S_{\mathrm{candidate}}}, \]

where both sizes use the same boundary and include required metadata.

## Distillation

With teacher logits \(z^T\), student logits \(z^S\), and temperature \(\tau\),

\[ p^T_\tau=\operatorname{softmax}(z^T/\tau),\qquad p^S_\tau=\operatorname{softmax}(z^S/\tau). \]

Forward KL distillation is

\[ \mathcal{L}_{KD}=\tau^2\,D_{\mathrm{KL}}(p^T_\tau\Vert p^S_\tau) =\tau^2\sum_v <!-- markdownlint-disable-line MD013 MD049 -->
p^T_{\tau,v}\log\frac{p^T_{\tau,v}}{p^S_{\tau,v}}. \]

A combined objective may be

\[ \mathcal{L}=\lambda\mathcal{L}_{\mathrm{hard}}+(1-\lambda)\mathcal{L}_{KD}+\beta\mathcal{L}_{\mathrm{feature}}. \] <!-- markdownlint-disable-line MD013 MD049 -->

Tokenizer/vocabulary mismatch requires an explicit alignment or response-level strategy.

## Parameter and memory estimates

Ideal weight payload for \(P\) parameters at \(b_w\) bits is

\[ M_W=\frac{P b_w}{8}. \]

For \(L\) layers, batch \(B\), cached sequence \(S\), \(H_{kv}\) key/value heads, head width \(d_h\), and \(b_{kv}\)
bytes per element,

\[ M_{KV}=2LBSH_{kv}d_h b_{kv}. \]

The factor 2 is for keys and values. Allocator, alignment, metadata, runtime workspaces, activations, tokenizer, and
OS/app memory are additional.

### Worked memory example

`ILLUSTRATIVE`: one billion parameters at 4 bits have ideal payload
\(10^9\times4/8=500{,}000{,}000\) bytes, about 0.466 GiB. For \(L=24\), \(B=1\), \(S=2048\),
\(H_{kv}=8\), \(d_h=128\), and two bytes per KV element,

\[
M_{KV}=2(24)(1)(2048)(8)(128)(2)=201{,}326{,}592\text{ bytes}\approx192\text{ MiB}.
\]

Their sum is still not peak memory; profile the runtime.

For Adam-like full training, a planning estimate is

\[ M_{\mathrm{train}}\approx M_{\mathrm{weights}}+M_{\mathrm{gradients}}+
M_{\mathrm{optimizer}}+M_{\mathrm{activations}}+M_{\mathrm{workspace}}, \]

with each term derived from actual precision, sharding, offload, checkpointing, and trainable parameters.

## Compute and performance

A dense matrix multiplication \((m\times k)(k\times n)\) requires approximately \(2mkn\) floating-point operations under
the multiply-plus-add convention. Transformer costs depend on sequence length, width, layers, vocabulary projection,
attention implementation, and decode phase.

Arithmetic intensity is

\[ I=\frac{\text{operations}}{\text{bytes transferred}}. \]

A roofline upper bound is

\[ \operatorname{Throughput}\le \min(P_{\mathrm{peak}},\,B_{\mathrm{mem}}I), \]

where peak compute \(P_{\mathrm{peak}}\) and memory bandwidth \(B_{\mathrm{mem}}\) are hardware bounds, not expected
application performance.

## Statistics and selection

For observations \(x_1,\ldots,x_n\),

\[ \bar{x}=\frac{1}{n}\sum_i x_i,\qquad s=\sqrt{\frac{1}{n-1}\sum_i(x_i-\bar{x})^2}. \]

Report distribution, sample size, aggregation, and interval method. Candidate \(a\) Pareto-dominates \(b\) when it is no
worse on every declared objective and strictly better on at least one, after hard-gate failures are removed. Pareto
membership does not choose a release; policy and risk ownership still decide.

`ILLUSTRATIVE`: A scores 82 at 900 ms, B scores 82 at 700 ms, and C scores 85 at 1,000 ms. B dominates A. Neither B
nor C dominates the other because B is faster while C scores higher.

## Cost and energy

\[ E=\int P(t)\,dt,\qquad C_{\mathrm{run}}=\sum_j u_j r_j+C_{\mathrm{fixed}}, \]

where \(u_j\) is measured or estimated usage and \(r_j\) a dated rate. Keep estimates separate from billed actuals and
include engineering, data, transfer, storage, monitoring, device, and lifecycle costs.
