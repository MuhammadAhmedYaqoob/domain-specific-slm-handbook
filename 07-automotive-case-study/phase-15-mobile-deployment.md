# Phase 15: Deploy offline on Android and iOS

[Previous phase](phase-14-cloud-deployment.md) · [Next phase](phase-16-benchmark.md)

## Learning objectives

- Package model, tokenizer, template, runtime, and compatibility metadata together.
- Validate physical-device offline behavior and lifecycle handling.
- Design integrity-checked updates and atomic rollback.

## Theory and purpose

A model that fits storage may still exceed runtime memory or throttle under sustained generation. Mobile release
therefore tests installation, loading, KV-cache growth, streaming, cancellation, process death, low storage,
thermals, and updates. Android and iOS backends are separate execution targets. Simulator success and declared
accelerator support do not prove physical placement or performance.

## Prerequisites

- Phase 13-qualified target-native artifact and pinned runtime.
- Android arm64 and iOS physical devices representing declared cohorts.
- Signing keys, app-private storage, update server, and rollback package.

## Step-by-step method

1. Create a signed manifest with artifact hash, bytes, runtime commit, context, source revision, and minimum app version.
2. Verify token IDs, template bytes, stop tokens, and golden greedy outputs against the reference.
3. Install or download into staging after free-space and compatibility checks.
4. Verify detached signature and SHA-256, then atomically activate the package.
5. Test airplane mode, streaming Unicode, cancellation, reset, backgrounding, process death, and bad updates.
6. Measure physical-device memory, latency, energy, and early-to-late thermal behavior; test rollback.

## Worked automotive example

**`ILLUSTRATIVE` device test.** On a fictional mid-range Android cohort, `auto-sft-lora-q4-r1` occupies 0.92 GB,
peaks at 2.35 GiB, has 1.9 s p95 TTFT, decodes at 6.1 tok/s, and slows 12% over 20 minutes without crashes.
On a fictional modern iPhone control path it peaks at 2.28 GiB, reaches 1.6 s TTFT and 7.0 tok/s, and slows 8%.
Both pass a bad-signature update test by retaining `mobile-auto-r0`. These values are not device claims.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "offline-slm-artifact/v1",
  "evidence": "ILLUSTRATIVE",
  "id": "automotive-specialist",
  "version": "1.0.0-course",
  "file": "model.gguf",
  "sha256": "869a88cc443d6eac51b90ad8e035877fdd650a474276281f8e655bd6779a9594",
  "bytes": 920000000,
  "runtime_commit": "course-runtime-7f2c91a",
  "context_tokens": 2048,
  "minimum_app_version": "1.3.0",
  "previous_version": "0.9.0-course"
}
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** This host-side check validates size and digest but intentionally does not pretend
to verify a detached signature. The app must verify a trusted signature before activation and keep the previous
package until the new model passes a startup health check.

```python
import hashlib, json
from pathlib import Path

root = Path("mobile-package")
manifest = json.loads((root / "model-manifest.json").read_text("utf-8"))
model = root / manifest["file"]
assert model.stat().st_size == manifest["bytes"]
assert hashlib.sha256(model.read_bytes()).hexdigest() == manifest["sha256"]
print("package bytes and digest verified; signature verification is still required")
```

## Expected behavior

The app rejects bad signatures, hashes, versions, and insufficient storage; valid packages stream offline and survive lifecycle tests. Failed activation leaves the previous signed package usable.

## Common failures and recovery

- **Failure:** Bundling a large model without staging and rollback can exceed storage limits.
- **Failure:** A simulator omits physical memory pressure, thermals, and power behavior.
- **Failure:** Hash checking without signature verification cannot establish publisher authenticity.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

No hidden cloud fallback is allowed. Keep prompts out of crash logs and fail closed on integrity, compatibility, or unsafe thermal states.

## Interpret the result

Android and iOS measurements form separate strata. Do not infer NPU or ANE use from an API's requested compute unit.

## Transition

[Phase 16](phase-16-benchmark.md) standardizes cloud and physical-device measurements for fair comparison.
