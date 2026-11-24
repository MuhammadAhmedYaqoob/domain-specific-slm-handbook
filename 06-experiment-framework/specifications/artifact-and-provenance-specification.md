# Artifact and provenance lesson

## Question answered

Which exact inputs and transformations produced an artifact, and can its identity, rights, and release history be
audited?

## Why controls matter

Names and tags can move; content hashes do not. Provenance prevents a result from being attached to a different model
file, calibration set, or runtime package.

## Common novice mistakes

- Using branch names or registry tags as identity.
- Hashing only the main weight file.
- Deleting failed intermediates.
- Evaluating an abstract model but releasing a different package.

## Complete worked illustrative experiment

An adapter is merged into a 1.20 GB parent and exported as a 0.72 GB package. The package hash differs after one
metadata byte changes, so old evidence cannot be inherited. Hash verification before and after transfer establishes
byte identity, not semantic equivalence; parity tests are still required.

## Fully populated illustrative manifest

```yaml
schema: slm-artifact/v1
artifact_id: artifact-example-quantized-001
kind: quantized-model
sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
bytes: 720000000
media_type: application/vnd.example.gguf
created_at: 2026-11-24T05:10:00Z
created_by: run-example-convert-001
parents: [artifact-example-merged-001]
transformation:
  method: example-groupwise-int4
  code_commit: a1b2c3d4
  config_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  environment_sha256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
inputs: [artifact-example-calibration-001]
identity: {weight_type: int4, group_size: 128, tokenizer_id: tokenizer-example-001}
rights:
  license_ids: [Apache-2.0]
  notices_sha256: "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  permitted_uses: [research, evaluation]
  redistribution: allowed-with-notices
security:
  signature: example-signature-record-001
  malware_safe_load_review: passed-static-format-review
retention:
  class: research
  expires_or_review_on: 2027-11-24
release:
  state: candidate
  targets: [android-example-profile]
  rollback_parent: artifact-example-merged-001
```

Identity fields say what the bytes represent; parents and transformation describe lineage; rights and security govern
use; retention and release describe lifecycle.

## Reusable specification

Use SHA-256 over canonical bytes for every dataset, model, tokenizer/template, adapter, conversion, runtime package,
raw bundle, analysis, and report. Every non-origin node has a parent and records command/config, environment, seed,
logs, and outcome. Keep training, calibration, validation, teacher, and sealed-test roots separate. A correction
creates `supersedes`; revocation changes state without erasing history.
