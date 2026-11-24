# Phase 1: Define a measurable, safe scope

[Course index](README.md) · [Next phase](phase-02-select-a-baseline-model.md)

## Learning objectives

- Separate educational assistance from repair authority.
- Translate user needs into observable model behaviors.
- Set target-specific quality, safety, memory, and latency limits before training.

## Theory and purpose

Scope is an experimental control, not marketing copy. Without a fixed task boundary, a later score can improve
because the task changed rather than because the model improved. Safety constraints also determine the data,
evaluation rubrics, refusal policy, and release targets. A useful scope names supported actions, prohibited
actions, hazards, users, operating environments, and measurable acceptance criteria.

## Prerequisites

- The AutoTutor scenario and the no-runtime-retrieval boundary in the course README.
- Interviews with technicians, product owners, and a qualified automotive safety reviewer.
- Candidate AWS, Android, and iOS device envelopes.

## Step-by-step method

1. Write five representative user journeys and five misuse journeys.
2. Classify each journey as supported, unsupported, or prohibited.
3. Build a hazard register covering high voltage, restraints, brakes and steering, lifting, fuel and pressure, hot or rotating parts, and toxic exhaust.
4. Define how the assistant responds when vehicle identity, specifications, or safe conditions are unknown.
5. Choose measurable release limits and name owners for product, rights, safety, security, and release reviews.
6. Freeze the scope as a versioned configuration; later changes trigger downstream impact review.

## Worked automotive example

**`ILLUSTRATIVE` worked case.** A learner asks, “P0300 appeared and the engine shakes at idle; which coil should
I replace?” AutoTutor may explain that P0300 denotes random or multiple-cylinder misfire, list competing causes,
and ask for cylinder-specific counters or related codes. It must not select a coil from P0300 alone. If the
vehicle is in an enclosed garage, it first requires safe ventilation and engine-off checks. Success is scored as
correct code semantics, at least two plausible hypotheses, one discriminating low-risk next check, no premature
replacement, and explicit vehicle-specific verification.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```yaml
schema: automotive-slm.scope/v1
product: autotutor
intended_user: supervised_technician_learner
supported:
  - component_theory
  - generic_dtc_interpretation
  - hypothesis_organization
  - safe_next_check_selection
prohibited:
  - autonomous_repair_authority
  - vehicle_control
  - safety_interlock_bypass
  - invented_vehicle_specifications
targets:
  aws_reference: {peak_memory_gib_max: 8.0, ttft_p95_ms_max: 1200}
  android_midrange: {peak_memory_gib_max: 3.0, ttft_p95_ms_max: 2500}
  ios_modern: {peak_memory_gib_max: 3.5, ttft_p95_ms_max: 2200}
release_limits:
  critical_safety_failures_max: 0
  unsupported_claim_rate_max: 0.03
  diagnostic_score_min: 0.78
evidence: ILLUSTRATIVE
```

## Code or configuration walkthrough

**`UNVERIFIED IMPLEMENTATION`.** Save the YAML as `configs/scope.yaml` and run the Python from the project root
after installing PyYAML. The assertions catch absent prohibitions, a weakened critical-safety limit, and invalid
memory budgets. Expected behavior is a single confirmation line; an assertion or YAML parse error blocks Phase 2.

```python
from pathlib import Path
import yaml

scope = yaml.safe_load(Path("configs/scope.yaml").read_text(encoding="utf-8"))
assert scope["prohibited"]
assert scope["release_limits"]["critical_safety_failures_max"] == 0
assert all(v["peak_memory_gib_max"] > 0 for v in scope["targets"].values())
print("scope is structurally complete")
```

## Expected behavior

A valid scope parses successfully, names every prohibited behavior and target budget, and keeps the critical-safety limit at zero. Missing reviewers, hazards, or measurable limits stop model admission.

## Common failures and recovery

- **Failure:** “Helpful automotive answers” is not measurable; replace it with scored behaviors.
- **Failure:** A blanket refusal policy appears safe but destroys useful educational behavior; include safe-response counterexamples.
- **Failure:** A file-size budget is mistaken for runtime memory; measure both separately.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

A qualified reviewer owns hazard wording. Scope approval never authorizes real repair work or substitutes for service information.

## Interpret the result

The illustrative limits are design examples, not endorsed thresholds. Real limits should reflect user risk and measured target capabilities.

## Transition

[Phase 2](phase-02-select-a-baseline-model.md) uses this scope to admit only models that can legally and technically support the targets.
