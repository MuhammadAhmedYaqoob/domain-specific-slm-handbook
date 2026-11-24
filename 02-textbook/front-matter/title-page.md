<a id="title-page"></a>

# Domain-Specific Small Language Models

## Foundations, Data Engineering, Adaptation, Compression, Edge Deployment, AWS Infrastructure, Evaluation, and Production MLOps

**A research-grounded engineering textbook and implementation companion**

**Audience:** AI engineers, ML researchers, mobile and cloud inference engineers, data engineers, technical leads, and
domain experts collaborating on specialist language models.

**Edition:** Research draft, first edition  
**Research cutoff:** 23 November 2026  
**Canonical chapter range:** CH-01–CH-47  
**Maintainer:** Project authoring and review team

This edition develops one end-to-end question:

> How can a compact language model acquire useful specialist capability during training, run credibly on both AWS and
> constrained edge hardware, and remain measurable, maintainable, and safe without runtime retrieval?

## Reader promise

This is not a catalog of model names. It is a guided path from first principles to a defensible release decision. Every
major technique is connected to a problem, mechanism, worked example, implementation path, measurement method, and
failure condition. When the repository has no measured result, the book teaches how to obtain one instead of inventing
a benchmark.

The intended outcome is practical judgment: you should be able to explain why a candidate learns the desired behavior,
what it costs in memory and compute, which runtime can execute it, what evidence supports its quality, and when it must
not be released.

The textbook distinguishes sourced claims, project execution, proposed implementations, estimates, and planned
experiments. A polished listing is not proof of execution, and a paper- or vendor-reported benchmark is not a project
result. Consult [Evidence labels](evidence-labels.md) before interpreting empirical statements.

## Architectural declaration

The primary inference path is **closed-book with respect to external document retrieval**: no vector database, runtime
chunk search, dynamic context augmentation from a corpus, or remote-teacher fallback. Domain capability must be
represented in base-model parameters, continued-pretraining updates, supervised fine-tuning updates, preference-training
updates, or selected trained adapters. Offline collection, synthesis, labeling, validation, and external evaluation
references are allowed.

## Safety declaration

The book does not authorize autonomous prescribing, diagnosis, hazardous repair, safety-interlock bypass, or unreviewed
control of physical systems. High-consequence outputs require domain-qualified verification and explicit escalation. See
[Safety and responsible use](safety-and-responsible-use.md).

## Navigation and authority

- [Textbook contents](../README.md)
- [How to use this book](how-to-use-this-book.md)
- [Governance standards](../../00-governance/README.md)
- [Detailed chapter registry](../../01-planning/detailed-table-of-contents.md)
- [Errata and limitations](../back-matter/errata-and-limitations.md)

Publication metadata such as copyright, licensing, authorship, and redistribution terms is maintained by the publisher
or repository owner and is separate from the technical evidence taught in this edition.
