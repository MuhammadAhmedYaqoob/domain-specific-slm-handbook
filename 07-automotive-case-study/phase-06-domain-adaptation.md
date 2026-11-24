# Phase 6: Decide whether continued pretraining is justified

[Previous phase](phase-05-establish-baseline.md) · [Next phase](phase-07-supervised-fine-tuning.md)

## Learning objectives

- Distinguish domain knowledge exposure from answer-format learning.
- Design equal-token CPT controls.
- Balance domain gains against general forgetting and compute.

## Theory and purpose

Continued pretraining applies next-token learning to domain prose before instruction tuning. It can improve
terminology and domain distributions, but it can also overfit duplicated manuals or forget general language.
SFT is often more direct for response behavior. The decision therefore requires equal-token controls, held-out
domain perplexity, downstream task scores, general retention, and safety—not falling training loss alone.

## Prerequisites

- Exact base checkpoint and untouched Phase 5 evidence.
- CPT prose and optional governed general-replay snapshots.
- A fixed token budget, checkpoint schedule, seeds, and development-only selection rule.

## Step-by-step method

1. Create no-CPT, automotive-only CPT, automotive-plus-replay, and equal-token control arms.
2. Hold optimizer family, token budget, context length, and evaluation schedule constant.
3. Monitor validation perplexity, loss spikes, gradient norms, and replay proportions.
4. Evaluate intermediate checkpoints on development domain, safety, and general-retention suites.
5. Repeat required seeds and compute paired deltas with uncertainty.
6. Accept CPT only when practical downstream gain exceeds forgetting, risk, compute, and maintenance costs.

## Worked automotive example

**`ILLUSTRATIVE` decision.** With 20 million tokens per arm, no-CPT scores 0.61 domain and 0.78 general retention.
Automotive-only CPT reaches 0.68 domain but drops retention to 0.69, below the fictional 0.74 floor.
Automotive plus 15% replay reaches 0.67 domain and 0.76 retention with no added safety failures. The replay arm
is accepted as the SFT parent because a 0.06 domain gain survives the retention constraint. These are teaching
values, not executed findings.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```yaml
schema: automotive-slm.cpt-decision/v1
evidence: ILLUSTRATIVE
token_budget: 20000000
arms:
  no_cpt: {domain_score: 0.61, general_retention: 0.78, critical_failures: 2}
  automotive_only: {domain_score: 0.68, general_retention: 0.69, critical_failures: 2}
  automotive_replay: {domain_score: 0.67, general_retention: 0.76, critical_failures: 2}
retention_floor: 0.74
decision: accept_automotive_replay_for_sft_experiment
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` code.** This complete calculation shows a constrained decision: filter by the retention floor,
then maximize domain score. Real selection must read measured multi-seed records and also enforce safety,
confidence, and compute constraints.

```python
arms = {
    "no_cpt": {"domain": 0.61, "retention": 0.78},
    "automotive_only": {"domain": 0.68, "retention": 0.69},
    "automotive_replay": {"domain": 0.67, "retention": 0.76},
}
eligible = {k: v for k, v in arms.items() if v["retention"] >= 0.74}
selected = max(eligible, key=lambda k: eligible[k]["domain"])
assert selected == "automotive_replay"
print(selected)
```

## Expected behavior

Every CPT arm consumes the declared token budget and yields comparable domain, retention, safety, and compute records. The decision can legitimately reject CPT and return SFT to the untouched base.

## Common failures and recovery

- **Failure:** Comparing unequal token budgets confounds domain data with extra training.
- **Failure:** Selecting the lowest perplexity checkpoint may not improve diagnostic behavior.
- **Failure:** Using an instruction checkpoint for CPT without testing behavior loss can damage alignment.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

CPT does not validate facts or guarantee recall. Safety-critical claims still require expert-grounded SFT and evaluation.

## Interpret the result

Rejecting CPT is a valid engineering result. Continue from the original base when gains are small, unstable, or expensive.

## Transition

[Phase 7](phase-07-supervised-fine-tuning.md) teaches the accepted parent the structured diagnostic behavior defined in Phase 1.
