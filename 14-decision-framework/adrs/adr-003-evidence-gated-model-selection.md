# ADR lesson: Evidence-gated model selection

This architecture decision teaches the reasoning and consequences. It defines a project rule; it does not prove that
any implementation complies.

## Decision problem

Family names, leaderboards, parameter counts, nominal bit width, and vendor claims do not prove fitness for an exact domain and target.

## Alternatives considered

1. Select by reputation or leaderboard.
2. Rank every option with one weighted score.
3. Apply exact-tuple gates, then uncertainty-aware Pareto analysis.

## Decision

Require immutable identity, current rights review, controlled target measurements, uncertainty, complete failure denominators, and hard-gate success before preference scoring.

## Worked consequence

Consider three hypothetical configurations. A has quality 88, no critical failures, and complies with this decision.
B has quality 93 but violates the decision. C has quality 84, complies, and costs 20% less to maintain than A. B is
rejected before scoring: its extra five quality points cannot compensate for an architecture, rights, or safety gate.
A and C remain a genuine quality-versus-maintenance trade-off.

Recommendations become auditable and bounded. Decisions take longer, may remain inconclusive, and can legitimately end with no qualified candidate.

## Why the alternatives were not chosen

Reputation is useful for experiment admission, not release. A single early score permits strength in one metric to conceal safety, rights, compatibility, or reliability failure.

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
