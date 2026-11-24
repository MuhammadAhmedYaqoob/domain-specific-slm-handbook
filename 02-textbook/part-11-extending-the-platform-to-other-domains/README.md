<a id="part-11"></a>

# Part XI — Extending the Platform to Other Domains

Reusing a platform does not mean copying an automotive dataset or scorecard. This part shows how source authority,
taxonomy, language, hazards, expert ownership, and release criteria change across domains while infrastructure contracts
remain reusable.

## Conceptual map

Reuse the platform contract, not the domain assumptions. Each new domain supplies its own intended uses, hazards,
source hierarchy, vocabulary, task taxonomy, experts, evaluation cases, update cadence, and release authority. Shared
mechanics such as ingestion or training orchestration are acceptable only when domain-specific controls remain visible.

## Learning outcomes and dependencies

After Parts VI–X, readers should be able to create a domain charter, source and safety register, objective-specific
datasets, domain evaluation suite, adapter/model isolation decision, and onboarding acceptance record. Part XII supplies
the production controls that follow onboarding.

## Chapters

1. [CH-38 — Pharmaceutical Knowledge SLM](ch-38-pharmaceutical-knowledge-slm.md): authoritative sources, terminology,
   interactions, contraindications, abstention, and R3/R4 boundaries.
2. [CH-39 — Motorcycle and Truck Mechanic SLMs](ch-39-motorcycle-and-truck-mechanic-slms.md): shared mechanical
   concepts, vehicle-specific variation, heavy-equipment hazards, and isolation choices.
3. [CH-40 — Beekeeping and Zookeeping SLMs](ch-40-beekeeping-and-zookeeping-slms.md): husbandry, species/context
   variation, biosecurity, welfare, and veterinary escalation.
4. [CH-41 — New Domain Onboarding](ch-41-new-domain-onboarding.md): repeatable intake, risk, data, evaluation, training,
   release, and no-runtime-RAG attestation.

## Decision checkpoint

A new domain needs named expert ownership, intended/prohibited uses, highest credible risk tier, lawful sources,
domain/safety tests, isolation/routing decision, update cadence, and rollback. R4 autonomous behavior is prohibited.

### Practice before continuing

Create a one-page onboarding delta between automotive and pharmacy: list what can be reused unchanged, what requires
new configuration, and what demands different expert approval. Any safety taxonomy copied unchanged should be treated
as suspect.

## Cross-links

[Part VII](../part-07-multi-domain-slm-engineering/README.md) · [Safety](../front-matter/safety-and-responsible-use.md)
· [Part XII](../part-12-production-engineering-safety-and-maintenance/README.md)
