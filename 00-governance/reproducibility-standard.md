# Reproducibility Standard

## What reproducibility gives the learner

A reproducible experiment lets another engineer determine what you intended, what actually ran, what failed, what
changed, and why the evidence supports the conclusion. It does not require bit-for-bit identical floating-point output.
It requires controlled inputs, declared tolerances, preserved artifacts, and instructions that do not depend on hidden
author knowledge.

## The experiment lifecycle

Use this sequence:

`QUESTION → APPROVED PLAN → RUNS → RESULT → REVIEW → QUALIFIED, REJECTED, OR INCONCLUSIVE`

Use the repository's [experiment plan](../06-experiment-framework/templates/experiment-plan-template.md),
[run record](../06-experiment-framework/templates/run-record-template.md),
[result record](../06-experiment-framework/templates/experiment-result-template.md), and
[failure report](../06-experiment-framework/templates/failure-report-template.md).

Approve and hash the plan before inspecting the primary outcome. You may perform later exploratory analysis, but label
it exploratory; do not rewrite the original decision rule after seeing the result.

## A worked planning example

Question: Does 4-bit quantization reduce peak memory enough for the target phone without unacceptable domain-quality
loss?

A useful plan fixes:

- baseline: one exact float checkpoint and tokenizer revision;
- variant: one exact quantizer, configuration, and exported artifact;
- primary metrics: peak resident memory and domain exact-match score;
- practical rule: memory must remain below the phone's declared budget and exact-match loss must not exceed the
  preapproved tolerance;
- controls: same prompts, context length, output limit, runtime version, device state, and trial order;
- repetition: one warm-up followed by 30 measured paired prompts per variant;
- outcome: qualify, reject, or report inconclusive according to the preapproved rule.

This is a fully explained way to validate the question. It remains `PLANNED EXPERIMENT` until runs exist.

Anti-example:

> Try a few quantizers and keep whichever output looks best.

This invites cherry-picking and gives no stable baseline, metric, repetition, or acceptance rule.

## Freeze inputs before execution

The plan MUST identify:

- hypothesis, measured quantity, baseline, variants, controls, primary and secondary metrics, meaningful difference, and
  decision rule;
- exact model, tokenizer, revision, license, chat template, adapter state, and quantization/export configuration;
- dataset IDs and hashes, split manifest, rights and provenance, preprocessing hash, leakage boundary, and rubric;
- source commit, exact command, working directory, resolved configuration, and dependency lock or image digest;
- seeds, determinism settings, trial count, stopping and exclusion rules, missing-data treatment, and statistical method;
- hardware, operating system, drivers, firmware, runtime, libraries, device state, workload, instrumentation, and
  environmental controls; and
- expected artifacts, retention and access, budget, safety gate, and reviewers.

Do not use mutable identifiers such as `latest`, an unpinned branch, an unversioned URL, or a dataset without a
snapshot.

## Record every attempt

Crashes, out-of-memory events, invalid runs, timeouts, manual aborts, and budget stops are data about the method. Give
every attempt a run ID and terminal state. Record UTC start and end, operator, resolved command and configuration,
environment, seed, logs, input/output hashes, resources, costs, deviations, and failure linkage.

A retry is a new run; never overwrite the failed attempt. For inference, record warm-up, cache state, prompt and output
lengths, concurrency, sampling, and timing boundaries. Follow the
[edge protocol](../06-experiment-framework/protocols/edge-device-benchmark.md) or
[AWS protocol](../06-experiment-framework/protocols/aws-inference-benchmark.md) for those targets.

## Report the complete result

Include every planned attempt and denominator, not only successes. State units, analysis unit, aggregation, uncertainty,
paired or unpaired design, multiple-comparison handling, practical threshold, and deviations under the
[statistical protocol](../06-experiment-framework/protocols/statistical-analysis.md).

Do not rank results collected under incompatible conditions. Preserve raw measurements as immutable,
content-addressed artifacts. Derived tables and figures link their transformation code/configuration and source-data
hash.

Use evidence labels precisely:

- `PLANNED EXPERIMENT`: the question and protocol exist, but no valid result exists.
- `EXECUTED`: at least one traceable run exists; it may have failed.
- `REPRODUCED`: an independent party reran the preserved procedure and compared declared tolerances.
- `UNVERIFIED IMPLEMENTATION`: code exists without sufficient target execution.
- `STATICALLY REVIEWED`: inspection only.

One phone, operating system, accelerator, runtime, model revision, or workload does not establish another.

## Build the reproduction package

A qualified result needs the plan, runs, results, failures, resolved machine-readable configuration, code revision,
dependency lock, input/output hashes, acquisition instructions, ordered commands, raw logs, measurements, analysis,
generated artifacts, environment manifest, nondeterminism and tolerance, limitations, troubleshooting, review
decisions, and supersession history.

Do not embed credentials, personal data, restricted licensed data, or signing keys. Document the secure provisioning
role and process instead.

## Independent reproduction workflow

1. Start only from the preserved package.
2. Provision the declared environment and inputs.
3. Run the documented commands without undocumented author intervention.
4. Record every deviation and failed attempt.
5. Compare results with the declared tolerance, not with an expectation of identical bits.
6. Publish the independent run and comparison.

If hardware, access, cost, or legal restrictions prevent this, retain `EXECUTED` and explain the barrier. Do not promote
the result to `REPRODUCED`. This standard is authoritative and does not claim that any planned repository experiment
has run.
