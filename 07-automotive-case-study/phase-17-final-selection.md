# Phase 17: Make final selections with Pareto reasoning

[Previous phase](phase-16-benchmark.md) · [Course index](README.md)

## Learning objectives

- Apply hard eligibility gates before optimization.
- Compute and explain a multi-objective Pareto front.
- Choose different artifacts for accuracy, mobile, balance, low memory, and AWS cost.

## Theory and purpose

One model rarely minimizes memory, latency, energy, and cost while maximizing quality. Candidate A dominates B
when A is no worse on every required objective and strictly better on at least one under comparable conditions.
Non-dominated candidates form the Pareto front. Hard gates—rights, safety, compatibility, quality floors, and
reliability—are applied first; stakeholder utility chooses among the front afterward.

## Prerequisites

- Phase 13 gate records and Phase 16 comparable measurements for exact artifacts.
- Target-specific objective directions and hard limits.
- Decision owners, release pointers, and rollback targets.

## Step-by-step method

1. Partition records by target and comparable protocol.
2. Remove candidates with missing metrics, failed rights or safety, incompatible runtimes, or violated floors.
3. Normalize objective direction: maximize quality and throughput; minimize memory, TTFT, energy, and cost.
4. Compare every eligible pair and retain candidates not dominated by another.
5. Select among Pareto members using predeclared target utility or a documented stakeholder decision.
6. Publish evidence hashes, limitations, intended use, known failures, and rollback for each release.

## Worked automotive example

**`ILLUSTRATIVE` complete dataset.** All values below are fictional and exist only to teach the calculation.
Quality is maximized; peak memory, TTFT, energy, and AWS cost are minimized. `M3` fails the 0.78 quality floor.
`M5` fails the zero-critical-failure gate. Mobile energy is not used for AWS-only candidates, and AWS cost is not
used for mobile-only candidates; those candidates belong to separate strata.

| ID | Target | Quality | Peak GiB | TTFT ms | Energy J/token | Cost units/success | Critical failures | Eligible |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| M1 LoRA-Q4 | mobile | 0.80 | 2.35 | 1900 | 0.42 | n/a | 0 | yes |
| M2 Student-Q4 | mobile | 0.79 | 1.72 | 1450 | 0.31 | n/a | 0 | yes |
| M3 Tiny-Q3 | mobile | 0.74 | 1.31 | 1250 | 0.25 | n/a | 0 | no: quality |
| M4 LoRA-INT8 | mobile | 0.81 | 3.10 | 1750 | 0.55 | n/a | 0 | yes |
| M5 Fast-Q4 | mobile | 0.82 | 2.20 | 1350 | 0.38 | n/a | 1 | no: safety |
| C1 INT8 GPU | AWS | 0.82 | 6.80 | 720 | n/a | 0.0062 | 0 | yes |
| C2 BF16 GPU | AWS | 0.83 | 9.90 | 680 | n/a | 0.0091 | 0 | yes |
| C3 Q4 CPU | AWS | 0.79 | 3.20 | 1450 | n/a | 0.0048 | 0 | yes |

For mobile, M2 dominates M1: M2 has lower memory, TTFT, and energy, while its quality 0.79 is lower, so it
actually does **not** dominate M1 under raw quality. Both are Pareto members. M1 may look preferable to M4 on
memory and energy, but it has lower quality and slower TTFT; strict Pareto dominance permits neither trade-off,
so M4 also remains non-dominated. If the stakeholder declares quality within 0.01
equivalent *before* comparison, epsilon-dominance may remove M4. Under strict dominance, the mobile front is
M1, M2, and M4. For AWS, C1, C2, and C3 each trade quality against TTFT, memory, or cost, so all three are on the
strict front.

Target decisions are then explicit: C2 is best accuracy; M2 is best mobile and low-memory; M1 is the balanced
mobile choice when quality must be at least 0.80; C3 is lowest AWS cost; C1 is the balanced AWS choice when TTFT
must remain below 1,000 ms.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.final-selection/v1",
  "evidence": "ILLUSTRATIVE",
  "strict_pareto_sets": {
    "mobile": ["M1", "M2", "M4"],
    "aws": ["C1", "C2", "C3"]
  },
  "selections": {
    "best_accuracy": "C2",
    "best_mobile": "M2",
    "best_balanced_mobile_quality_at_least_0_80": "M1",
    "best_low_memory": "M2",
    "best_cost_efficient_aws": "C3",
    "best_balanced_aws_ttft_below_1000_ms": "C1"
  },
  "real_project_status": "awaiting_executed_measurements"
}
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` Pareto calculation.** The function preserves every raw trade-off and produces the strict mobile
front. A weighted score is not Pareto analysis: weights add stakeholder preferences and should be applied only
after the front is known. Real code must reject missing values, compare uncertainty, and process each target
stratum separately.

```python
rows = [
    {"id": "M1", "quality": 0.80, "memory": 2.35, "ttft": 1900, "energy": 0.42},
    {"id": "M2", "quality": 0.79, "memory": 1.72, "ttft": 1450, "energy": 0.31},
    {"id": "M4", "quality": 0.81, "memory": 3.10, "ttft": 1750, "energy": 0.55},
]

def dominates(a, b):
    no_worse = (a["quality"] >= b["quality"] and a["memory"] <= b["memory"]
                and a["ttft"] <= b["ttft"] and a["energy"] <= b["energy"])
    strictly_better = (a["quality"] > b["quality"] or a["memory"] < b["memory"]
                       or a["ttft"] < b["ttft"] or a["energy"] < b["energy"])
    return no_worse and strictly_better

front = [a["id"] for a in rows if not any(dominates(b, a) for b in rows if b is not a)]
assert front == ["M1", "M2", "M4"]
print(front)
```

## Expected behavior

The calculation removes ineligible candidates before dominance checks, returns the stated strict Pareto sets, and applies target utility only afterward. Without executed evidence, no real release pointer changes.

## Common failures and recovery

- **Failure:** Including a safety failure on the front treats safety as negotiable.
- **Failure:** A weighted sum can hide that arbitrary weights changed the winner.
- **Failure:** Mixing phones, tokenizers, workloads, or cloud offered loads creates false dominance.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Any rights, critical-safety, compatibility, or reliability failure makes a candidate ineligible before Pareto analysis.

## Interpret the result

Replace every illustrative row with an `EXECUTED` Phase 13/16 record for the exact artifact, runtime, target, and
workload. Preserve the same columns, add confidence intervals and evidence hashes, rerun hard gates, recompute
fronts, and then apply the predeclared utility rules. Until those measurements exist, this chapter teaches the
decision method but selects no real project model.

## Transition

Return to the [course index](README.md), publish target-specific model cards, and schedule regression measurement whenever data, model, runtime, OS, hardware, or scope changes.
