# Learner roadmap

## Mission

This directory turns the textbook into a sequenced course for engineers who want to build, measure, and deploy a
domain-specific Small Language Model (SLM). It explains what to learn first, why that dependency matters, what to build
after each block, and how to avoid spending compute before data, evaluation, and deployment feasibility are understood.
The primary product learns domain capability through training; runtime retrieval and remote-teacher substitution remain
outside the primary inference architecture.

## Entry prerequisites

Readers should be able to write and debug Python, train a small PyTorch model, interpret loss curves, use Git and a
terminal, and explain vectors, matrices, gradients, probability, and train/validation/test splits. Familiarity with
Docker, REST APIs, Linux processes, and one cloud platform is helpful but not required. Before the first GPU exercise,
complete the environment and dependency guidance in the
[implementation handbook](../03-implementation-handbook/environment-and-dependencies.md).

If any prerequisite is weak, use a one-week bridge: implement next-token cross-entropy on a toy sequence model, profile
one PyTorch forward pass, containerize it, and calculate confidence intervals for repeated latency measurements. The
exit test is the ability to explain every line and reproduce the run from a clean environment.

## Recommended learning order

1. Read the [research survey](initial-research-survey.md) to distinguish established methods from target-dependent
   claims.
2. Follow the [dependency map](learning-dependency-map.md): foundations, data and evaluation, adaptation, compression,
   deployment, then production.
3. Use the [detailed contents](detailed-table-of-contents.md) as the chapter-by-chapter syllabus and the
   [diagram plan](diagram-plan.md) as an active-recall guide.
4. Build the [automotive baseline](automotive-baseline-plan.md) through the staged
   [experiment sequence](hands-on-experiment-sequence.md).
5. Choose deployment hardware only after reading the [AWS](aws-target-plan.md) and
   [edge](edge-target-plan.md) target plans.
6. Demonstrate competence at the checkpoints in [milestones](milestones.md), then use the
   [authoring workflow](authoring-workflow.md) to turn findings into reproducible technical notes.

The order is deliberate. Data and evaluation come before training so that improvement cannot be defined after seeing
results. Compression follows a qualified high-precision model so quality loss has a valid reference. Runtime probes
precede large training runs because an unsupported operator or tokenizer can invalidate an otherwise successful model.
AWS and physical-edge qualification come last because deployment measurements are meaningful only for a fixed artifact
and workload.

## How to study each block

For each chapter, use four passes: preview the outcome and prerequisites; derive or diagram the mechanism without notes;
run the smallest relevant exercise; then write a one-page decision memo that states when the method should and should
not be used. Do not proceed when a prerequisite exit test is missing. Record failed runs because they teach
compatibility boundaries and prevent repeated work.

## Worked study schedules

**Illustrative 24-week schedule (12 hours per week):** weeks 1–3, CH-01–03; weeks 4–6, CH-23–25 and CH-29–30; weeks
7–10, CH-04–09; weeks 11–14, CH-10–15; weeks 15–18, CH-16–22 and one early export probe; weeks 19–20, CH-31–32; weeks
21–22, CH-33–36; weeks 23–24, CH-37 capstone and decision report. Substitute measured reading, exercise, and compute
times after the first two weeks; keep the sequence, but expand any block whose exit test is not met.

**Illustrative 12-week intensive schedule (30 hours per week):** combine each pair of the blocks above into one week,
reserve weeks 9–10 for the automotive build, week 11 for AWS and edge qualification, and week 12 for analysis and
reproduction by another person. Replace the assumed hours with the team’s measured throughput and hardware queue time.

## Evidence and completion

All study and experiment work inherits [governance](../00-governance/README.md), especially the
[evidence standard](../00-governance/evidence-and-citation-standard.md), [definition of
done](../00-governance/definition-of-done.md), [figure/table
standard](../00-governance/figure-table-standard.md), and no-runtime-RAG rule. `DOC-VERIFIED` and `PAPER-REPORTED`
describe inspected sources; they do not establish local reproduction. A learner completes this roadmap only after
producing a rights-approved dataset snapshot, untouched baseline, trained adaptation, measured optimization comparison,
one AWS deployment, one physical-device run, and a target-specific decision with limitations.

## Navigation

- [Bookmap](../BOOKMAP.md)
- [Canonical textbook](../02-textbook/README.md)
- [Experiment framework](../06-experiment-framework/README.md)
- [Automotive case study](../07-automotive-case-study/README.md)
