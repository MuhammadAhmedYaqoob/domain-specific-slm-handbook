# Phase 9: Evaluate governed knowledge distillation

[Previous phase](phase-08-optimize-training.md) · [Next phase](phase-10-quantization.md)

## Learning objectives

- Treat teacher output as a review candidate rather than ground truth.
- Compare direct, teacher-only, and mixed student training.
- Recognize when soft-logit distillation is technically valid.

## Theory and purpose

Response distillation trains a student on accepted teacher responses. Logit distillation additionally matches
softened teacher distributions and usually requires aligned token positions and vocabulary. Distillation can
transfer useful behavior, but it can also amplify teacher errors, verbosity, and unsupported confidence. The
decisive control is the same student trained directly on expert data.

## Prerequisites

- Legally usable teacher and output terms.
- Approved support records, prompts, and qualified reviewers.
- Exact student candidates and a same-size direct-training control.

## Step-by-step method

1. Generate teacher candidates only from approved support and record teacher revision, prompt, decoding, and support IDs.
2. Run automated format, duplication, PII, and support-alignment checks.
3. Have experts accept, edit, or reject each candidate with reasons.
4. Train direct-expert, accepted-teacher, and fixed-mixture student arms.
5. Use soft targets only after proving tokenizer alignment, masks, numerical stability, storage bounds, and resumability.
6. Advance a student only when it improves deployment trade-offs over the same-size direct control.

## Worked automotive example

**`ILLUSTRATIVE` review.** A teacher generates 2,000 responses. Experts accept 1,460 unchanged, edit 280, and
reject 260: acceptance is 73%. The direct 360M student scores 0.69 domain; teacher-only scores 0.70 but raises
unsupported claims from 0.05 to 0.09; a 60% expert/40% accepted-teacher mix scores 0.74 with unsupported claims
at 0.04. The mixed student advances. The rejected P0300 response named a coil without evidence, illustrating why
fluency is not supervision quality.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.distillation-decision/v1",
  "evidence": "ILLUSTRATIVE",
  "teacher_candidates": 2000,
  "accepted_unchanged": 1460,
  "edited_then_accepted": 280,
  "rejected": 260,
  "arms": {
    "direct_360m": {"domain": 0.69, "unsupported_rate": 0.05},
    "teacher_360m": {"domain": 0.70, "unsupported_rate": 0.09},
    "mixed_360m": {"domain": 0.74, "unsupported_rate": 0.04}
  },
  "decision": "advance_mixed_360m"
}
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` calculation.** Unchanged acceptance is 0.73; usable-after-review is 0.87. Keep these denominators
separate because editing cost matters. Real records also stratify rejection by factual, safety, support, style,
and duplication reasons.

```python
total, unchanged, edited, rejected = 2000, 1460, 280, 260
assert unchanged + edited + rejected == total
unchanged_acceptance = unchanged / total
usable_after_review = (unchanged + edited) / total
print({"unchanged_acceptance": unchanged_acceptance, "usable_after_review": usable_after_review})
```

## Expected behavior

Teacher candidates retain support and review lineage; accepted, edited, and rejected totals reconcile. A distilled arm advances only when it beats the same-size direct control under deployment constraints.

## Common failures and recovery

- **Failure:** Teacher self-citations or confident wording are mistaken for factual support.
- **Failure:** A stronger teacher can still be worse on the project's safety policy.
- **Failure:** Token-level KL loss is invalid when teacher and student positions do not align.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Experts review every safety-critical synthetic response; rejected rows remain traceable so descendants can be invalidated.

## Interpret the result

Distillation is worthwhile only if the student changes the quality-memory-latency frontier, not merely imitates teacher style.

## Transition

[Phase 10](phase-10-quantization.md) converts the selected dense and student artifacts into target-native low-precision candidates.
