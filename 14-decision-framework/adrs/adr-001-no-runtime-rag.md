# ADR lesson: No runtime RAG

This architecture decision teaches the reasoning and consequences. It defines a project rule; it does not prove that
any implementation complies.

## Decision problem

The primary product must demonstrate knowledge learned into weights or trained adapters and must work without a document service.

## Alternatives considered

1. Use runtime retrieval and prompt augmentation.
2. Delegate answers to a remote teacher model.
3. Use parameterized specialist inference with deterministic non-retrieval tools.
4. Abandon the offline specialist objective.

## Decision

Use parameterized specialist inference. Offline sources may prepare training data and evaluation ground truth, but primary answer generation cannot search, select document chunks, augment prompts with retrieved text, or delegate answers remotely.

## Worked consequence

Consider three hypothetical configurations. A has quality 88, no critical failures, and complies with this decision.
B has quality 93 but violates the decision. C has quality 84, complies, and costs 20% less to maintain than A. B is
rejected before scoring: its extra five quality points cannot compensate for an architecture, rights, or safety gate.
A and C remain a genuine quality-versus-maintenance trade-off.

Inference remains independently testable and can work offline. The cost is weaker factual freshness: updates require governed data, training, evaluation, and a new release. Learned parameters may omit, distort, or stale facts.

## Why the alternatives were not chosen

A retrieval design would improve factual update speed but would violate the stated product boundary and change privacy, availability, evaluation, and threat models. A remote teacher would hide rather than solve the specialist-model capability gap.

## Operational consequences

The release process must identify exact artifacts, retain failed evidence, test the decision on the deployed target,
monitor for violations, and keep a complete signed predecessor. A violation pauses release and triggers rollback.
Changing the rule requires a superseding ADR that explains migration, compatibility, affected decisions, approvers,
and effective date; history remains intact.

## Verification questions

1. What observable test would falsify compliance?
2. Which package, runtime, network, safety, rights, and rollback evidence is required?
3. Which costs move from development into recurring maintenance?
4. Which rejected alternative should be reconsidered if the product boundary changes?
5. What residual risk remains even after every test passes?

## Related material

- [Decision framework](../README.md)
- [Filled decision record](../decision-record-template.md)
- [Governance](../../00-governance/README.md)
