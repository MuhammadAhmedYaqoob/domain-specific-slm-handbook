# Phase 14: Deploy a qualified model to AWS

[Previous phase](phase-13-evaluate.md) · [Next phase](phase-15-mobile-deployment.md)

## Learning objectives

- Bind model, image, runtime, and infrastructure identities.
- Design authentication, privacy, scaling, observability, and rollback.
- Measure allocated cost rather than quoting an undated price example.

## Theory and purpose

Deployment is a new experimental condition. Container libraries, templates, batching, hardware, and concurrency
can change behavior and latency. A release should reference an immutable model hash and container digest, run
with least privilege, avoid prompt logging by default, expose bounded health behavior, and retain a tested prior
release. Production cost depends on region, date, utilization, and successful requests.

## Prerequisites

- Phase 13-qualified cloud artifact.
- AWS account, chosen region, IAM/VPC design, budget, and cleanup owner.
- Pinned inference image and a load-test plan with no sensitive prompts.

## Step-by-step method

1. Build a non-root, read-only container that loads only the embedded approved artifact.
2. Run local readiness, bounded-request, template, timeout, and failure tests.
3. Push to private ECR and deploy by image digest, not a mutable tag.
4. Configure TLS, authentication, request/token/rate limits, network controls, and secret handling.
5. Add queue, TTFT, end-to-end latency, errors, utilization, alarms, and privacy-safe logs.
6. Canary the release, test rollback, reconcile dated billing, and delete unused resources.

## Worked automotive example

**`ILLUSTRATIVE` canary.** `auto-sft-int8-aws-r1` receives 10% traffic for 30 minutes at a fictional offered load of
2 requests/s. It produces p95 TTFT 720 ms, p95 end-to-end latency 3.8 s, 0.4% errors, and 0.0062 allocated cost
units per successful request. The prior release is 810 ms, 4.1 s, 0.3%, and 0.0068 units. Since the error budget
is 0.5% and all quality probes match, traffic advances gradually. “Cost units” deliberately avoid implying an
actual AWS price.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```json
{
  "schema": "automotive-slm.aws-release/v1",
  "evidence": "ILLUSTRATIVE",
  "release_id": "aws-auto-r1",
  "model_id": "auto-sft-int8-aws-r1",
  "image_digest": "sha256:75da9a3e72fbc7f6097d47e8cb10559a29f9ed146672344d2527319ac95e37de",
  "region": "us-east-2",
  "canary_percent": 10,
  "ttft_p95_ms": 720,
  "error_rate": 0.004,
  "allocated_cost_units_per_success": 0.0062,
  "previous_release": "aws-auto-r0"
}
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** This startup check fails closed before model loading. The digest is fictional and
complete for teaching; replace it only from the signed real release manifest. Container-image and model digests
are distinct and both must be verified.

```python
import hashlib
from pathlib import Path

artifact = Path("artifacts/releases/aws/model.bin")
expected = "75da9a3e72fbc7f6097d47e8cb10559a29f9ed146672344d2527319ac95e37de"
actual = hashlib.sha256(artifact.read_bytes()).hexdigest()
if actual != expected:
    raise SystemExit(f"model digest mismatch: {actual}")
print("model digest verified before server startup")
```

## Expected behavior

The service starts only when model and image identities match, exposes bounded readiness and inference behavior, records privacy-safe tail metrics, and can shift traffic back to the prior digest.

## Common failures and recovery

- **Failure:** Deploying a mutable image tag makes rollback identity ambiguous.
- **Failure:** Average latency hides queueing and tail failures.
- **Failure:** Logging raw prompts creates privacy and retention risk.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Cloud release does not expand model authority. Preserve input limits, hazard responses, incident withdrawal, and human verification requirements.

## Interpret the result

Canary success is scoped to measured load and conditions. Rebenchmark after engine, hardware, concurrency, or model changes.

## Transition

[Phase 15](phase-15-mobile-deployment.md) applies the same identity and rollback principles to signed offline packages.
