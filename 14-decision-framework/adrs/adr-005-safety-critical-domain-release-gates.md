# ADR lesson: Safety-critical domain release gates

This architecture decision teaches the reasoning and consequences. It defines a project rule; it does not prove that
any implementation complies.

## Decision problem

Specialist output can contribute to injury, poisoning, equipment failure, animal harm, privacy harm, or regulated decisions. Aggregate quality cannot erase one prohibited instruction.

## Alternatives considered

1. Rely on disclaimers and average quality.
2. Include safety as one weighted objective.
3. Use risk-tiered intended-use limits and non-compensatory safety gates.

## Decision

Prohibit autonomous high-risk authority. Require hazard records, trustworthy versioned ground truth, qualified review, zero defined critical failures where policy requires, escalation and over-refusal bounds, exact-artifact tests, monitoring, and rollback.

## Worked consequence

Consider three hypothetical configurations. A has quality 88, no critical failures, and complies with this decision.
B has quality 93 but violates the decision. C has quality 84, complies, and costs 20% less to maintain than A. B is
rejected before scoring: its extra five quality points cannot compensate for an architecture, rights, or safety gate.
A and C remain a genuine quality-versus-maintenance trade-off.

Critical hazards remain visible and approval is tied to evidence. Expert review is expensive, finite tests cannot prove absolute safety, and some releases will be delayed or rejected.

## Why the alternatives were not chosen

Disclaimers do not prevent harmful instructions. Weighted safety permits compensation: for example, a quality gain could numerically offset a critical failure, which is unacceptable.

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
