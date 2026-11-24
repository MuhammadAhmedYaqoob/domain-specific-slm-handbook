# Phase 3: Acquire automotive data lawfully

[Previous phase](phase-02-select-a-baseline-model.md) · [Next phase](phase-04-process-data.md)

## Learning objectives

- Treat public accessibility and training permission as different questions.
- Capture byte-level provenance, terms, attribution, and takedown lineage.
- Plan coverage before collecting volume.

## Theory and purpose

Data acquisition creates a legal and scientific dependency. Each raw object needs an owner, exact source,
acquisition time, content hash, rights decision, intended use, and removal route. Coverage should follow the
Phase 1 taxonomy; otherwise abundant engine text can hide missing electrical, diagnostic-ordering, or safety
material. Synthetic records inherit constraints from their support and teacher.

## Prerequisites

- Approved scope taxonomy and rights-review process.
- Secure raw, receipt, and quarantine storage.
- A documented acquisition client with size, redirect, host, and rate limits.

## Step-by-step method

1. Propose sources by missing taxonomy node rather than by convenience.
2. Snapshot terms and obtain a purpose-specific rights decision for training, redistribution, and commercial use.
3. Download approved HTTPS objects without bypassing access controls.
4. Hash exact bytes and write a receipt before parsing.
5. Quarantine objects with unknown permission, malware, unexpected type, or embedded third-party material.
6. Review coverage and stop collecting when marginal duplication exceeds useful new concepts.

## Worked automotive example

**`ILLUSTRATIVE` acquisition.** A fictional public training handout, `engine-misfire-basics.pdf`, is approved for
training and attribution but not redistribution. Its 842,311 bytes hash to the fictional digest shown below.
A publicly viewable paywalled service-manual page is rejected because access is not permission. An expert writes
24 diagnostic cases under a signed contribution agreement. The accepted set covers ignition, fuel, air,
electrical, and safe diagnostic ordering; the rejected page contributes zero bytes.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.source-receipt/v1",
  "source_id": "src-auto-edu-001",
  "title": "Engine Misfire Basics",
  "canonical_url": "https://example.org/engine-misfire-basics.pdf",
  "acquired_at": "2026-11-20T10:15:00Z",
  "bytes": 842311,
  "sha256": "5d7fd8e87e41b9f2cb413ddf0f5813d9f5b51e89b65f7d680f193fa118e91b22",
  "training_allowed": true,
  "redistribution_allowed": false,
  "attribution": "Example Technical Education Board",
  "coverage": ["engine", "misfire", "diagnostic_ordering"],
  "reviewer_role": "rights_counsel",
  "evidence": "ILLUSTRATIVE"
}
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** The check is read-only and expects the receipt and raw object at fixed paths.
It proves byte/receipt agreement, not that the legal decision is correct. A size or hash mismatch moves the
object to quarantine and stops processing.

```python
import hashlib, json
from pathlib import Path

receipt = json.loads(Path("data/receipts/src-auto-edu-001.json").read_text("utf-8"))
raw = Path("data/raw/src-auto-edu-001.pdf").read_bytes()
assert receipt["training_allowed"] is True
assert receipt["bytes"] == len(raw)
assert receipt["sha256"] == hashlib.sha256(raw).hexdigest()
print("receipt and raw bytes agree")
```

## Expected behavior

Every accepted raw object has matching byte count and SHA-256, an affirmative training-rights decision, attribution, coverage tags, and a removal owner. Unknown or mismatched objects remain quarantined.

## Common failures and recovery

- **Failure:** A robots rule governs automated access, not training rights.
- **Failure:** A source-level license may not cover embedded diagrams or quoted manuals.
- **Failure:** Collecting duplicate DTC lists inflates row counts without improving diagnostic reasoning.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Do not scrape paywalls, bypass controls, collect personal data, or execute downloaded content. Sandbox parsers and retain a takedown owner.

## Interpret the result

Accepted bytes are eligible inputs, not verified facts. Technical correctness and contradictions are handled in Phase 4.

## Transition

[Phase 4](phase-04-process-data.md) converts accepted raw objects into objective-specific snapshots while preserving source lineage.
