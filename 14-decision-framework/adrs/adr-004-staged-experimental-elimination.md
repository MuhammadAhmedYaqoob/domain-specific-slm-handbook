# ADR lesson: Staged experimental elimination

This architecture decision teaches the reasoning and consequences. It defines a project rule; it does not prove that
any implementation complies.

## Decision problem

The Cartesian product of models, adaptation methods, quantizers, runtimes, targets, workloads, and seeds is unaffordable and increases repeated-test selection bias.

## Alternatives considered

1. Run every combination.
2. Immediately commit to one favored pipeline.
3. Use baseline-first stages with frozen stop/go rules and one bounded rescue branch.

## Decision

Progress from rights and compatibility screens to untouched baselines, adaptation, isolated compression, qualified combinations, target execution, and fresh finalist confirmation. Preserve all failures and immediate parents.

## Worked consequence

Consider three hypothetical configurations. A has quality 88, no critical failures, and complies with this decision.
B has quality 93 but violates the decision. C has quality 84, complies, and costs 20% less to maintain than A. B is
rejected before scoring: its extra five quality points cannot compensate for an architecture, rights, or safety gate.
A and C remain a genuine quality-versus-maintenance trade-off.

Resources focus on informative candidates while controls remain available. Early gates can miss favorable interactions, introduce survival bias, and overfit the staged process.

## Why the alternatives were not chosen

An exhaustive matrix offers coverage but is usually infeasible. Immediate commitment is cheap but cannot distinguish model, training, compression, and runtime effects.

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
