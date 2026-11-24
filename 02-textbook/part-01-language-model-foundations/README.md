<a id="part-01"></a>

# Part I — Language Model Foundations

This part builds the vocabulary and mechanisms needed to reason about every later training, compression, and deployment
choice. It moves from statistical language modeling to the Transformer computation graph and then to the architectural
families from which an SLM baseline is selected.

## Conceptual map

Text becomes tokens; tokens become vectors; repeated attention and feed-forward blocks transform those vectors; an
output projection produces next-token probabilities; decoding turns probabilities into generated text. Architecture
choices determine the model's capacity, KV-cache shape, operator graph, and ultimately which training and deployment
paths are feasible.

## Learning outcomes

Readers should be able to explain next-token modeling, trace a decoder-only Transformer, calculate the dominant
attention and parameter costs, compare architecture choices, and distinguish a model-family claim from a target-specific
deployment result.

## Dependencies

- **Prerequisites:** [Audience and prerequisites](../front-matter/audience-and-prerequisites.md) and
  [notation](../front-matter/notation-and-conventions.md).
- **Read concurrently:** the [mathematical reference](../back-matter/appendix-mathematical-reference.md).
- **Feeds:** Parts II–V and the baseline selection gate in CH-47.

## Chapters

1. [CH-01 — Evolution of Language Models](ch-01-evolution-of-language-models.md) establishes the historical transitions
   and the scaling, data, and deployment pressures behind modern SLMs.
2. [CH-02 — Transformer Architecture in Depth](ch-02-transformer-architecture-in-depth.md) traces tokenization,
   embeddings, attention, position encoding, normalization, MLPs, residuals, and autoregressive decoding.
3. [CH-03 — Modern SLM Architectures](ch-03-modern-slm-architectures.md) compares model families and turns architecture
   descriptions into baseline-selection criteria.

## Decision checkpoint

Before Part II, produce a requirements card with domain, context length, language coverage, licensing constraints,
target memory, latency/energy targets, safety tier, and at least two candidate baselines. Do not select solely by
parameter count or a vendor leaderboard.

### Practice before continuing

Take one candidate model card and identify its tokenizer, parameter count, layer count, attention type, context claim,
license, and instruction status. Mark every missing item “unknown”; do not infer it from the family name.

## Cross-links

[Textbook index](../README.md) · [Part II](../part-02-training-and-domain-adaptation/README.md) ·
[Detailed contents](../../01-planning/detailed-table-of-contents.md)
