<a id="part-14"></a>

# Part XIV — Final Engineering Architecture

The final part integrates requirements, data, learning, compression, cloud, edge, evaluation, security, lifecycle, and
cost into a proposed production system. “Proposed” is material: the architecture is conditional until its gates are
executed and retained.

## Conceptual map

The data plane turns governed sources into versioned datasets. The learning plane turns datasets and configurations
into candidate artifacts. The evidence plane tests candidates and records decisions. The delivery plane serves AWS
artifacts or installs signed edge artifacts. The operations plane monitors, rolls back, updates, and retires releases.
Primary inference consumes the selected model or adapter—not retrieved domain documents.

## Learning outcomes and dependencies

Readers should be able to trace every component to a requirement and evidence artifact, identify trust boundaries,
distinguish control plane from inference path, choose domain isolation, and issue an accept/revise/reject decision. All
prior parts are inputs; Parts VIII and XIII determine which alternatives survive.

## Chapter

[CH-47 — Proposed Production System](ch-47-proposed-production-system.md) defines the reference components, data and
artifact flows, AWS and edge deployment paths, security and monitoring controls, no-runtime-RAG compliance, and staged
implementation plan.

## Non-negotiable boundary

Primary inference derives domain capability from model parameters or explicitly selected trained adapters. Runtime
vector stores, corpus retrieval, dynamic chunk augmentation, and remote-teacher fallback are excluded. Artifact
registries, policy/config distribution, telemetry, and update checks are control-plane functions and must not smuggle
retrieved domain content into prompts.

## Final gate

Use the [final decision framework](../back-matter/final-decision-framework.md). Architecture approval requires evidence
for quality, critical safety, exact-target compatibility, peak memory, performance, energy/thermal behavior, security,
licensing, operability, cost, and rollback. Any mandatory failure yields revise or reject.

### Practice before finishing

Trace one user request backward from generated token to runtime, artifact manifest, release decision, evaluation report,
training run, dataset snapshot, and source rights. Then trace an incident forward through detection, containment,
rollback, correction, reevaluation, and signed redistribution. Missing links are architecture work, not documentation
cosmetics.

## Cross-links

[Part XIII](../part-13-experimental-research-roadmap/README.md) ·
[Decision framework](../back-matter/final-decision-framework.md) ·
[Errata and limitations](../back-matter/errata-and-limitations.md)
