# Phase 12: Combine optimizations with successive halving

[Previous phase](phase-11-pruning.md) · [Next phase](phase-13-evaluate.md)

## Learning objectives

- Order transformations according to compatibility.
- Attribute the marginal value of each added operation.
- Spend physical-device and cloud budget only on credible survivors.

## Theory and purpose

Optimization order changes distributions and artifact identity. Merge adapters before ordinary release
quantization; recover after pruning; recalibrate after any weight-changing operation. Applying every technique
creates an uninterpretable, fragile pipeline. Successive halving runs cheap compatibility and development checks
first, then spends expensive sealed, thermal, energy, and production-load tests on survivors.

## Prerequisites

- Accepted single-method artifacts and explicit target budgets.
- Immutable parent hashes and resumable orchestration.
- Development suites separate from sealed Phase 13 evaluation.

## Step-by-step method

1. Enumerate only technically compatible sequences with a hypothesis for each operation.
2. Stage load, template, and deterministic smoke checks.
3. Stage development quality, safety, and retention checks.
4. Stage file, memory, startup, TTFT, and decode checks on target-like hardware.
5. Advance a small survivor set to sealed and sustained tests.
6. Preserve failures and compare each child with its immediate parent to estimate marginal value.

## Worked automotive example

**`ILLUSTRATIVE` halving.** Twelve candidates enter. Two fail load/template parity. Of ten, three fail the 0.78
development quality floor and one has a safety regression. Six reach target-like tests; two exceed 3.0 GiB and
one throttles by 35%. Three survive: LoRA-merged Q4, mixed-student Q4, and INT8 cloud. The structured-pruned Q4
pipeline is eliminated because pruning adds 14 GPU-hours yet is slower than its Q4 parent.

## Filled artifact example

The following artifact is fully populated for the worked example. Its values are **`ILLUSTRATIVE`**, not
measurements from this repository.

```yaml
schema: automotive-slm.combined-decision/v1
evidence: ILLUSTRATIVE
entrants: 12
stages:
  compatibility: {entered: 12, advanced: 10}
  development_quality_safety: {entered: 10, advanced: 6}
  target_like_performance: {entered: 6, advanced: 3}
survivors:
  - auto-sft-lora-q4-r1
  - auto-mixed-student-q4-r1
  - auto-sft-int8-aws-r1
rejected_example:
  id: auto-structured-q4-r1
  reason: added_14_gpu_hours_and_slower_than_parent
```

## Code or configuration walkthrough

**`ILLUSTRATIVE` calculation.** The output makes resource allocation visible. A real orchestrator must additionally
hash inputs and outputs, preserve stdout/stderr and exit status, reject changed resume state, and never overwrite
a run.

```python
stages = [("compatibility", 12, 10), ("development", 10, 6), ("target_like", 6, 3)]
for name, entered, advanced in stages:
    assert 0 <= advanced <= entered
    print(name, {"eliminated": entered - advanced, "advance_rate": advanced / entered})
```

## Expected behavior

Candidates advance through compatibility, development, and target-like stages without changing thresholds. Every child can be compared with its immediate parent to isolate marginal value.

## Common failures and recovery

- **Failure:** Quantizing before an ordinary adapter merge changes or blocks the intended release path.
- **Failure:** Reusing pre-pruning calibration ignores changed weight distributions.
- **Failure:** Relaxing a quality floor to retain a fast candidate invalidates preregistration.

Recovery means returning to the last accepted immutable parent, correcting the cause in a new run, and retaining
the failed evidence. Never weaken a preregistered limit merely to advance an attractive candidate.

## Safety checks

Every transformation creates a new artifact and must repeat safety regression; parent approval does not transfer automatically.

## Interpret the result

Survivor count is not a success metric. A small, explainable set gives Phase 13 stronger statistical and review focus.

## Transition

[Phase 13](phase-13-evaluate.md) evaluates the three illustrative survivors once on sealed expert-grounded cases.
