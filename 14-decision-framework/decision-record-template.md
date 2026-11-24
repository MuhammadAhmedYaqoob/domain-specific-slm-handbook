# Filled decision record and reuse guide

This is a fully filled **hypothetical teaching record**. Names, hashes, measurements, dates, approvals, and outcomes are
invented. It demonstrates completeness; it is not authorization to deploy any real model.

## Record control

- Decision ID: `DEC-LOW-ANDROID-0042`
- Decision state: accepted for a fictional pilot
- Decision date: 2026-04-15
- Evidence cut-off: 2026-04-10
- Reassessment date: 2026-07-15
- Target profile: low-end Android profile revision 3, fictional SHA-256 `8b31-teaching-example-only`
- Accountable roles: Maya Chen, target owner; Luis Ahmed, domain reviewer; Ren Ito, safety reviewer; Priya Shah,
  rights reviewer; Ellen Park, release authority
- Independence: the safety and rights reviewers did not train or export the candidates
- Predecessor: fictional release `REL-LOW-017`
- Supersession: no later record exists in this example

## Context and intended use

The fictional pilot provides offline automotive concept explanations and cautious diagnostic checklists on an ARM64
Android cohort with 4 GiB RAM. It must not authorize repairs, override service documentation, identify emergency-safe
actions without escalation, or provide runtime retrieval. Workloads use at most 2,048 input tokens and 256 output
tokens, one request at a time. The app reserves 1.4 GiB for Android and user-interface processes. A decision is needed
because the predecessor no longer meets the fictional July update-support policy.

## Hard-gate results

| Gate | Frozen rule | Fictional evidence | Outcome |
| --- | --- | --- | --- |
| Identity and lineage | hashes for weights, tokenizer, template, export, runtime, and evaluation | bundle `EV-4201` | pass |
| Commercial rights | use, modification, and app redistribution approved with notices | review `LIC-118` | pass |
| No runtime RAG | network-denial and dependency trace show no retrieval or remote answer path | test `NET-77` | pass |
| Safety | zero critical failures; unsafe-compliance upper bound below 2% | suite `SAFE-51` | pass |
| Domain quality | lower confidence bound at least 78; retention loss no more than 3 points | suite `QUAL-93` | pass |
| Compatibility | exact package loads on all three reference devices without hidden fallback | run `COMP-22` | pass |
| Resources | peak at most 2.50 GiB; p95 first token at most 2.0 s; decode at least 8 token/s | run `SYS-310` | pass |
| Reliability | crash, OOM, and timeout upper bound below 1% across the registered soak | run `REL-64` | pass |
| Operations | signed predecessor, monitored rollout, and rollback drill under 15 minutes | drill `OPS-19` | pass |

## Compared options

The fictional options use the same evaluation snapshot and intended-use policy.

| Option | Quality | Peak GiB | p95 s | Decode token/s | J/token | License | Result |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- |
| Alder-0.8 Q4 CPU | 79 | 1.82 | 1.72 | 10.4 | 0.08 | approved | eligible |
| Birch-1.3 Q4 CPU | 84 | 2.43 | 1.91 | 8.6 | 0.12 | approved | eligible |
| Cypress-1.1 Q4 NPU | 86 | 2.21 | 1.44 | 13.0 | 0.07 | unresolved redistribution | rejected |
| Keep predecessor | 76 | 1.75 | 1.88 | 9.8 | 0.09 | approved | rejected: quality floor |

All values are fabricated for instruction. Cypress is rejected before scoring because unresolved redistribution rights
are non-compensatory. The predecessor fails the quality floor.

## Pareto and preference calculation

Alder and Birch form the eligible frontier: Alder uses fewer resources, while Birch has higher quality. With frozen
bounds quality 75–90, memory 1.5–2.5 GiB, and p95 latency 1–2 s, Alder’s utilities are 0.267, 0.680, and 0.280; Birch’s
are 0.600, 0.070, and 0.090. Weights 0.50, 0.30, and 0.20 yield:

\[
U(\text{Alder})=0.50(0.267)+0.30(0.680)+0.20(0.280)=0.394,
\]

\[
U(\text{Birch})=0.50(0.600)+0.30(0.070)+0.20(0.090)=0.339.
\]

Alder is preferred for the balanced role. When quality weight is raised to 0.70, Birch becomes preferred, so the
decision is preference-sensitive. The pilot owner explicitly values memory headroom and interaction stability over the
illustrative five-point quality gain.

## Decision

Release fictional `Alder-0.8 Q4 CPU` tuple `REL-LOW-042` for the balanced low-end Android pilot. This does not establish
that Alder is a real model, universally best, or safe outside the stated fictional profile.

## Consequences and alternatives

The pilot gains 0.61 GiB memory headroom relative to Birch and simpler CPU-only placement. It accepts five fewer
quality points and may need earlier retraining. Birch remains an alternative for a segmented higher-memory cohort.
Cypress is reconsidered only after rights are resolved and evidence is refreshed. No-deployment was rejected because
the fictional intended use is non-autonomous and all gates passed, but it remains the fallback if monitoring invalidates
the evidence.

## Safety, security, rights, and maintenance

The pilot displays escalation language for injury, fuel, high voltage, lifting, and braking hazards. The package and
SBOM are signed; prompts remain local. Required notices ship in the app. The runtime team owns quarterly dependency
updates, a two-device backward-compatibility window, and security response. Residual risk includes plausible but wrong
diagnostic ordering and knowledge staleness; users must verify against current service information.

## Release, monitoring, and rollback

Release proceeds to internal users, then 5%, 25%, and 100% cohorts only while safety, crash/OOM, first-token latency,
and refusal metrics remain within gates. Any critical unsafe response, integrity failure, rights revocation, crash/OOM
rate above 1%, or p95 first-token latency above 2.0 s for two daily windows pauses rollout.

Rollback atomically activates complete signed predecessor `REL-LOW-017`; weights, tokenizer, runtime, and policy are
never mixed across releases. The operations owner has a 15-minute objective. Verification checks signature, load,
five domain probes, five safety probes, telemetry, and incident linkage. A forward fix creates a new candidate and
decision.

## Approval and dissent

All five fictional owners approved record hash `dec42-teaching-hash` on 2026-04-15. The domain reviewer recorded the
condition that Birch be retested if the cohort memory reserve rises by 0.5 GiB. No other dissent was recorded.

## Line-by-line reuse guidance

1. Replace the decision ID with a unique immutable identifier; never recycle an ID.
2. State the actual decision state in words and distinguish approval from deployment completion.
3. Use UTC decision, evidence-cut-off, and reassessment dates so stale evidence is visible.
4. Link and hash the exact target profile; a family label such as “low-end phone” is insufficient.
5. Name accountable people and roles, including reviewer independence and conflicts.
6. Link predecessor and supersession records so rollback and history remain navigable.
7. Rewrite intended and prohibited uses, workload, cohort, privacy boundary, and reason for deciding now.
8. Replace every gate rule before confirmation, then link immutable evidence and record pass, fail, or incomplete.
9. List every exact configuration, the no-deployment option, failures, missing data, rights, and maintenance ownership.
10. Recalculate the Pareto set in original units; show normalization bounds, weights, arithmetic, and sensitivity.
11. State one target-specific role, not a universal winner, and explain why frontier alternatives were not selected.
12. Describe beneficial and adverse consequences, residual uncertainty, and what evidence does not establish.
13. Record notices, restrictions, SBOM, support horizon, custom dependencies, incident ownership, and recency triggers.
14. Specify staged release, monitored denominators and windows, stop conditions, and evidence retention.
15. Name the complete predecessor tuple, atomic rollback method, recovery objective, and post-rollback probes.
16. Bind approvals, conditions, and dissent to the same immutable record hash; never erase disagreement.

## References

- [Decision framework](README.md)
- [Criteria lesson](criteria-and-measurement.md)
- [Pareto lesson](pareto-selection-process.md)
