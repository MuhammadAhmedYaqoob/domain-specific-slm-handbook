# ADR lesson: Separate target-specific artifacts

This architecture decision teaches the reasoning and consequences. It defines a project rule; it does not prove that
any implementation complies.

## Decision problem

Android, iOS, laptops, ARM boards, and AWS runtimes support different formats, kernels, precisions, context policies, and packaging. Conversion can alter outputs.

## Alternatives considered

1. Release one source checkpoint everywhere.
2. Package one lowest-common-denominator artifact.
3. Identify and qualify a separate exported child for each target.

## Decision

Treat each exported child—bound to tokenizer, template, runtime, target, policy, notices, SBOM, and predecessor—as a separate release unit.

## Worked consequence

Consider three hypothetical configurations. A has quality 88, no critical failures, and complies with this decision.
B has quality 93 but violates the decision. C has quality 84, complies, and costs 20% less to maintain than A. B is
rejected before scoring: its extra five quality points cannot compensate for an architecture, rights, or safety gate.
A and C remain a genuine quality-versus-maintenance trade-off.

Failures and regressions become attributable to the artifact users receive. The project pays for more conversions, storage, target tests, approvals, and rollback records.

## Why the alternatives were not chosen

A universal checkpoint is operationally simpler but makes compatibility an assumption. A lowest-common-denominator package reduces variants but can waste target accelerators and still does not prove identical behavior.

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
