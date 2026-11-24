# Domain-Specific Small Language Models

**Foundations, data engineering, adaptation, compression, edge deployment, AWS infrastructure, evaluation, and
production MLOps**

This book teaches you to turn a general pretrained model into a measured, deployable specialist SLM. You will make the
same linked decisions an engineering team must make: what the model may do, which data it may learn from, how to adapt
it, how to shrink it, how to test it, and whether a particular AWS or edge release is justified. Domain capability is
learned during training and represented in model parameters or trained adapters.

The primary inference architecture does **not** retrieve documents, query a vector database, or delegate answers to a
remote teacher. That constraint makes offline operation possible, but it also makes factual freshness, uncertainty,
scope control, and model updates central design problems. A trained model is not an exact database.

## What you will build

By the end, you should be able to produce a versioned dataset, a baseline and adapted checkpoint, a controlled
evaluation report, at least one compressed artifact, an exact-target cloud or edge benchmark, and a release decision
that another engineer can audit. The automotive project in Part X demonstrates how those artifacts fit together; the
remaining domain chapters show what must change rather than encouraging blind reuse.

## Start here

1. Read the [title and edition statement](front-matter/title-page.md), [preface](front-matter/preface.md), and
   [audience and prerequisites](front-matter/audience-and-prerequisites.md).
2. Choose a route in [How to use this book](front-matter/how-to-use-this-book.md).
3. Learn the [evidence labels](front-matter/evidence-labels.md),
   [notation and conventions](front-matter/notation-and-conventions.md), and
   [safety boundary](front-matter/safety-and-responsible-use.md) before using empirical claims or implementation
   listings.
4. Use the [final engineering decision framework](back-matter/final-decision-framework.md) only after collecting
   evidence with Parts VIII and XIII.

If you want the shortest practical route, read Parts I, II, III, IV, VI, and VIII; build the Part X baseline; then add
Part V for edge delivery or Part IX for AWS. Do not skip evaluation design: write the scorecard before training.

## Book structure

<!-- markdownlint-disable MD013 -->
| Part                                                                                                                     | Purpose                                                                        | Chapters |
| ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ | -------- |
| [I — Language Model Foundations](part-01-language-model-foundations/README.md)                                           | Language-model history, Transformer mechanics, and modern SLM selection        | CH-01–03 |
| [II — Training and Domain Adaptation](part-02-training-and-domain-adaptation/README.md)                                  | Pretraining, continued pretraining, SFT, and preference optimization           | CH-04–07 |
| [III — Parameter-Efficient Fine-Tuning](part-03-parameter-efficient-fine-tuning/README.md)                               | Full tuning, PEFT, LoRA, and QLoRA                                             | CH-08–09 |
| [IV — Model Compression and Optimization](part-04-model-compression-and-optimization/README.md)                          | Memory, quantization, pruning, distillation, and combined pipelines            | CH-10–15 |
| [V — Edge AI and Mobile Inference](part-05-edge-ai-and-mobile-inference/README.md)                                       | Hardware, runtimes, optimization, Android, iOS, and other edge targets         | CH-16–22 |
| [VI — Data Acquisition and Dataset Engineering](part-06-data-acquisition-and-dataset-engineering/README.md)              | Rights-aware discovery, processing, objective-specific datasets, and synthesis | CH-23–26 |
| [VII — Multi-Domain SLM Engineering](part-07-multi-domain-slm-engineering/README.md)                                     | Domain organization and a reusable training platform                           | CH-27–28 |
| [VIII — Evaluation, Benchmarking, and Experiments](part-08-evaluation-benchmarking-and-research-experiments/README.md)   | Quality, safety, edge benchmarks, and controlled ablations                     | CH-29–32 |
| [IX — AWS Training, Optimization, and Deployment](part-09-aws-training-optimization-and-deployment/README.md)            | AWS infrastructure, training, serving, and edge delivery                       | CH-33–36 |
| [X — Complete Hands-On Project](part-10-complete-hands-on-project/README.md)                                             | Automotive SLM blueprint from scope through selection                          | CH-37    |
| [XI — Extending to Other Domains](part-11-extending-the-platform-to-other-domains/README.md)                             | Pharmacy, motorcycle/truck, animal-care, and domain onboarding                 | CH-38–41 |
| [XII — Production Engineering, Safety, and Maintenance](part-12-production-engineering-safety-and-maintenance/README.md) | Security, monitoring, lifecycle, and cost                                      | CH-42–44 |
| [XIII — Experimental Research Roadmap](part-13-experimental-research-roadmap/README.md)                                  | Fifteen research questions and a staged benchmark matrix                       | CH-45–46 |
| [XIV — Final Engineering Architecture](part-14-final-engineering-architecture/README.md)                                 | Proposed production system                                                     | CH-47    |
<!-- markdownlint-restore -->

The [detailed table of contents](../01-planning/detailed-table-of-contents.md) is the canonical CH-01–CH-47 registry.
Each part introduction explains its conceptual map, prerequisites, practical output, and readiness check.

## Back matter

- [Final decision framework](back-matter/final-decision-framework.md)
- [Mathematical reference](back-matter/appendix-mathematical-reference.md)
- [Hardware and memory reference](back-matter/appendix-hardware-and-memory-reference.md)
- [Experiment reporting checklist](back-matter/appendix-experiment-reporting-checklist.md)
- [Detailed technical index](back-matter/index-plan.md)
- [Figure index](back-matter/figure-index.md) and [table index](back-matter/table-index.md)
- [Glossary](back-matter/glossary-pointer.md) and [references](back-matter/references-pointer.md)
- [Unresolved research, errata, and limitations](back-matter/errata-and-limitations.md)

## Evidence and maturity

The chapters combine research, worked examples, implementation designs, and experiment protocols. Most implementations
and experiments remain unexecuted. Each chapter states its evidence boundary. `PAPER-REPORTED`, `DOC-VERIFIED`, and
`VENDOR-REPORTED` do not mean this project reproduced a result; `UNVERIFIED IMPLEMENTATION` does not mean code ran;
`PLANNED EXPERIMENT` is not a finding.

Repository maturity is controlled by the [definition of done](../00-governance/definition-of-done.md) and
[deliverable readiness ledger](../15-audit-ledgers/deliverable-readiness-ledger.md). This navigation page makes no
release-readiness claim.

## Architectural and safety boundary

- Domain capability is learned into weights or trained adapters.
- Runtime vector search, document retrieval, dynamic chunk injection, and remote-teacher substitution are prohibited in
  the primary inference path.
- Training does not guarantee exact storage or recall. Expect stale knowledge, conflicts, hallucinations, and imperfect
  calibration.
- Pharmacy, hazardous repair, industrial, electrical, and animal-care content is educational or assistive only.
  Qualified verification and escalation remain mandatory.

See the binding [governance standards](../00-governance/README.md), [assessments](../13-assessments/README.md), and
[open issues and omissions](../15-audit-ledgers/open-issues-and-omissions.md).
