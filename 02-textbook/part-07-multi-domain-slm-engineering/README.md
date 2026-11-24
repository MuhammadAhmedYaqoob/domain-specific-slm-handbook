<a id="part-07"></a>

# Part VII — Multi-Domain SLM Engineering

This part compares ten ways to organize specialist capability: independent models, one mixed model, shared base with
adapters, sequential specialization, distillation, routing, and hybrid portfolios. It then turns the choice into a
reusable, governed platform.

## Conceptual map

Domain capability can live in separate checkpoints, one mixed checkpoint, or adapters attached to a shared base.
Isolation reduces interference but increases storage and lifecycle work; sharing can reduce duplication but introduces
routing, compatibility, and cross-domain risk. The platform makes data, training, evaluation, packaging, and release
contracts reusable while leaving domain authority and gates domain-specific.

## Learning outcomes and dependencies

After Parts II, III, and VI, readers should be able to quantify interference, isolation, storage, update, routing, and
serving trade-offs; choose a strategy per risk tier; and define portable data/train/evaluate/package contracts. This
part feeds domain extension and the final architecture.

## Chapters

1. [CH-27 — Single-Domain Versus Multi-Domain Strategies](ch-27-single-domain-versus-multi-domain-strategies.md)
   compares capability organization, negative transfer, adapter selection, routing, and lifecycle cost.
2. [CH-28 — Reusable Domain Training Platform](ch-28-reusable-domain-training-platform.md) defines manifests,
   interfaces, orchestration backends, artifact lineage, evaluation gates, and portability.

## Decision checkpoint

Choose per-domain isolation deliberately. Record routing authority, base/adapter compatibility, cross-domain and
wrong-adapter tests, rollback unit, and deployment overhead. Routing may select a trained artifact; it may not retrieve
corpus chunks for primary inference.

### Practice before continuing

Compare two domains with different risk tiers. Decide whether they may share a base, adapters, evaluation infrastructure,
and release cadence. For every shared component, name the cross-domain failure test and rollback unit.

## Cross-links

[Part III](../part-03-parameter-efficient-fine-tuning/README.md) ·
[Part XI](../part-11-extending-the-platform-to-other-domains/README.md) ·
[CH-47](../part-14-final-engineering-architecture/ch-47-proposed-production-system.md)
