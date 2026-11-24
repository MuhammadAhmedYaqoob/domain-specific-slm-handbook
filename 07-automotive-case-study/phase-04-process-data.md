# Phase 4: Process, validate, and split the data

[Previous phase](phase-03-acquire-data.md) · [Next phase](phase-05-establish-baseline.md)

## Learning objectives

- Build different snapshots for CPT, SFT, calibration, distillation, and evaluation.
- Prevent exact, near-duplicate, family, and source leakage.
- Preserve original technical meaning while normalizing for comparison.

## Theory and purpose

Training objectives require different statistical units. CPT learns from contiguous prose; SFT learns from
messages and target behavior; quantization calibration estimates activation distributions; evaluation must be
independently authored and sealed. Random row splitting leaks paraphrases, vehicle cases, and document editions.
Split at the case-family or source-family level before materializing rows.

## Prerequisites

- Approved Phase 3 receipts and a taxonomy.
- Schemas for each objective and a secret deterministic split key.
- Qualified reviewers for safety-critical facts and contradiction adjudication.

## Step-by-step method

1. Parse in a sandbox and preserve original bytes, extraction confidence, page, and source IDs.
2. Normalize Unicode and whitespace for matching while retaining original text for training.
3. Validate units, record schemas, PII policy, and source rights.
4. Cluster exact hashes, near duplicates, editions, paraphrases, and synthetic siblings.
5. Assign entire groups to train, validation, or test with a keyed deterministic function.
6. Build separate manifests and prove no ancestry or similarity leakage into sealed evaluation.

## Worked automotive example

**`ILLUSTRATIVE` transformation.** Three records describe the same P0300 case: an expert original, a paraphrase,
and a synthetic harder variant. Their normalized hashes differ, but similarity and shared `case_group` place all
three in training. A separately authored evaluation case about intermittent misfire has no source or case-family
ancestry. From 1,240 fictional candidate rows, 82 exact/near duplicates and 18 unsupported rows are removed,
leaving 1,140 rows: 912 train, 114 validation, and 114 test.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.sft-record/v1",
  "id": "case-misfire-017",
  "source_ids": ["src-auto-expert-004"],
  "case_group": "misfire-family-03",
  "split": "train",
  "risk": "moderate",
  "messages": [
    {"role": "system", "content": "Separate observations, hypotheses, checks, and conclusions."},
    {"role": "user", "content": "P0300 is present and idle is rough. What should be checked first?"},
    {"role": "assistant", "content": "Confirm safe conditions, record related codes and misfire counters, then compare cylinders before selecting a component."}
  ],
  "review": {"status": "accepted", "reviewer_role": "ase_master_technician"},
  "evidence": "ILLUSTRATIVE"
}
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** The HMAC makes group assignment deterministic without exposing a predictable
split rule. Use a real secret stored outside version control; the displayed key is fictional and only makes the
example complete. Determinism does not detect duplicates, so similarity and ancestry audits remain mandatory.

```python
import hmac, hashlib

def assign(case_group: str, key: bytes) -> str:
    value = int.from_bytes(hmac.new(key, case_group.encode(), hashlib.sha256).digest()[:8], "big") % 100
    return "train" if value < 80 else "validation" if value < 90 else "test"

key = bytes.fromhex("7d7f5b4a2f3992ee0199a3ec59b1eb8e3ec4c9f2e4f6a192c148f250a5ca8f31")
group = "misfire-family-03"
assert assign(group, key) == assign(group, key)
print({group: assign(group, key)})
```

## Expected behavior

The processor emits schema-valid objective-specific snapshots, stable group assignments, source lineage, and zero cross-split family leakage. Re-running identical inputs produces identical manifests.

## Common failures and recovery

- **Failure:** Row-level random splitting leaks paraphrases and editions.
- **Failure:** Converting every document into chat format destroys useful CPT structure.
- **Failure:** Unit normalization that rewrites original values can alter technical meaning.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Quarantine low-confidence OCR, contradictions, personal data, and safety-critical claims until reviewed; never majority-vote conflicting procedures.

## Interpret the result

Counts demonstrate pipeline accounting, not quality. Inspect taxonomy coverage, source diversity, review status, and leakage evidence together.

## Transition

[Phase 5](phase-05-establish-baseline.md) evaluates untouched checkpoints against the sealed suite before any adaptation.
