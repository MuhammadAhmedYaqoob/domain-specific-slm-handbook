<a id="part-10"></a>

# Part X — Complete Hands-On Project

This part integrates the previous nine parts into one automotive-mechanic SLM blueprint. It is a governed reference
workflow, not evidence that the listed infrastructure, training, conversion, deployment, or benchmarks have been
executed.

## Conceptual map

The project is a chain of evidence: scope determines hazards and metrics; sources determine allowable data; data and
objective determine adaptation; the baseline makes improvement measurable; compression creates candidates; cloud and
edge qualification test exact artifacts; release gates choose survivors; monitoring and rollback keep the decision
reversible.

## Learning outcomes and dependencies

Readers should be able to move from intended use and risk classification through source governance, objective-specific
datasets, baseline selection, CPT/SFT/adapters, compression, evaluation, AWS packaging, edge qualification, and final
selection. Complete Parts I–IX first or use their checkpoints as mandatory review links.

## Chapter

[CH-37 — Build an Automotive Mechanic SLM End to End](ch-37-build-an-automotive-mechanic-slm-end-to-end.md) provides the
project structure, configuration contracts, safety gates, training and delivery designs, benchmark plan, and decision
record.

## Required deliverables

Produce frozen requirements, source/data manifests, model/tokenizer hashes, training records, evaluation suites, safety
review, compression lineage, signed release manifest, cloud and physical-device reports, cost evidence, rollback plan,
and an approved decision record. Keep `UNVERIFIED IMPLEMENTATION` labels until execution traces exist.

### Learner milestone

Complete a small-scale dry run before spending on full training: ten source records, a tiny transformed dataset, a
frozen evaluation slice, one baseline report, and one configuration-to-artifact lineage trace. The values are not
production evidence, but the exercise exposes broken contracts early.

## Safety and architecture checkpoint

Hazardous repair instructions require qualified review and R2–R3 controls. Runtime retrieval, vector search, dynamic
corpus injection, and remote-teacher fallback are excluded from primary inference.

## Cross-links

[Safety](../front-matter/safety-and-responsible-use.md) ·
[Experiment checklist](../back-matter/appendix-experiment-reporting-checklist.md) ·
[Part XI](../part-11-extending-the-platform-to-other-domains/README.md)
