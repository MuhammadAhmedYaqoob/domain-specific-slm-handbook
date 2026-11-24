# Phase 13: Run sealed expert-grounded evaluation

[Previous phase](phase-12-combined-optimization.md) · [Next phase](phase-14-cloud-deployment.md)

## Learning objectives

- Evaluate exact release candidates without test-driven tuning.
- Measure domain quality, diagnosis, uncertainty, safety, robustness, and retention.
- Report item-level pairing, uncertainty, and reviewer disagreement.

## Theory and purpose

Sealed evaluation estimates generalization after all model choices are fixed. Hidden rubrics prevent answer
leakage and include acceptable facts, forbidden claims, required questions, and hazard elements. Paired analysis
is stronger than unrelated averages because each candidate faces the same cases. Statistical significance does
not override practical thresholds or critical safety failures.

## Prerequisites

- Exact survivor hashes from Phase 12 and untouched baseline outputs.
- Sealed prompts and rubrics never exposed to training jobs.
- Blinded qualified reviewers, adjudication rules, and an analysis plan.

## Step-by-step method

1. Freeze candidate, runtime, tokenizer, template, decoding, and environment identities.
2. Generate one immutable prediction set per candidate and preserve failures.
3. Blind candidate labels and randomize review packets.
4. Score factual support, diagnostic ordering, uncertainty, refusals, hazards, robustness, and retention.
5. Measure reviewer agreement and adjudicate disagreements without changing outputs.
6. Compute paired confidence intervals and apply all hard gates to exact artifacts.

## Worked automotive example

**`ILLUSTRATIVE` sealed result on 200 cases.** LoRA-Q4 scores 0.80 domain, 0.79 diagnosis, 0.025 unsupported claims,
and zero critical failures. Mixed-student-Q4 scores 0.77, 0.78, 0.020, and zero failures; it misses the 0.78 domain
floor. INT8-cloud scores 0.82, 0.81, 0.020, and zero failures. Reviewer weighted agreement is 0.84. LoRA-Q4 and
INT8-cloud become release candidates; the student remains a research artifact.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.sealed-evaluation/v1",
  "evidence": "ILLUSTRATIVE",
  "cases": 200,
  "candidates": {
    "auto-sft-lora-q4-r1": {"domain": 0.80, "diagnosis": 0.79, "unsupported": 0.025, "critical": 0},
    "auto-mixed-student-q4-r1": {"domain": 0.77, "diagnosis": 0.78, "unsupported": 0.020, "critical": 0},
    "auto-sft-int8-aws-r1": {"domain": 0.82, "diagnosis": 0.81, "unsupported": 0.020, "critical": 0}
  },
  "reviewer_weighted_agreement": 0.84,
  "domain_floor": 0.78
}
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` gate calculation.** Hard constraints are conjunctive: a candidate must pass all of them. Real
analysis also uses confidence bounds, diagnostic and retention floors, refusal precision/recall, and item-level
safety review.

```python
candidates = {
    "lora_q4": {"domain": 0.80, "unsupported": 0.025, "critical": 0},
    "student_q4": {"domain": 0.77, "unsupported": 0.020, "critical": 0},
    "int8_cloud": {"domain": 0.82, "unsupported": 0.020, "critical": 0},
}
eligible = {k: v for k, v in candidates.items()
            if v["domain"] >= 0.78 and v["unsupported"] <= 0.03 and v["critical"] == 0}
assert set(eligible) == {"lora_q4", "int8_cloud"}
print(sorted(eligible))
```

## Expected behavior

Every exact candidate receives one sealed prediction set, blinded reviews, item-level scores, uncertainty, and a gate decision. Test evidence may reject an artifact but never tunes that same release.

## Common failures and recovery

- **Failure:** Changing data mixture after seeing sealed results turns the test into development data.
- **Failure:** Averages hide strata with rare but severe failures.
- **Failure:** Conversion output is assumed equivalent to the source checkpoint without regression testing.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

A critical hazard failure is reviewed individually and cannot be averaged away; the candidate is withdrawn until a new descendant is evaluated.

## Interpret the result

Failure provides evidence about the exact artifact, not proof that the entire model family or method is unsuitable.

## Transition

[Phase 14](phase-14-cloud-deployment.md) packages the qualified cloud artifact; [Phase 15](phase-15-mobile-deployment.md) handles the mobile artifact.
