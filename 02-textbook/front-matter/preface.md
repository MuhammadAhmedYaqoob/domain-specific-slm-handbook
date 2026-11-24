<a id="preface"></a>

# Preface

Large general-purpose models can conceal the engineering problem behind abundant compute, remote services, and large
context windows. A specialist SLM cannot. Its vocabulary, data rights, token budget, parameter budget, memory layout,
runtime kernels, safety behavior, and update strategy must agree. This book treats those choices as one coupled system.

The motivating deployments span two environments. AWS supplies elastic training, controlled evaluation, artifact
governance, and scalable serving. Edge and mobile targets impose hard limits on storage, RAM, thermal headroom, power,
latency, and operator connectivity. Neither target is a secondary export. A credible model must have an evidence-backed
path to both.

## The central constraint

This project deliberately excludes runtime retrieval-augmented generation from primary inference. The model does not
search a corpus or add retrieved chunks to each prompt. Knowledge acquisition occurs offline through licensed data
engineering, continued pretraining, supervised instruction tuning, preference optimization, and optional trained
adapters. Evaluation may use external references as ground truth, but those references are not fed into normal
inference.

That constraint does not turn parameters into a database. Models compress patterns imperfectly; they can omit rare
facts, merge conflicting sources, reproduce stale information, or invent plausible answers. The proper engineering
response is measured scope, calibration, refusal and escalation behavior, versioned releases, and qualified human
verification—not claims of exact memorization.

## What the book connects

Parts I–IV establish language modeling, adaptation, PEFT, and compression. Parts V–IX connect those methods to physical
devices, datasets, multi-domain organization, evaluation, and cloud infrastructure. Parts X–XIV turn the components into
a domain project, extension process, production lifecycle, research program, and final architecture.

The result is intentionally a textbook plus engineering record:

- equations explain mechanisms and assumptions;
- diagrams expose system boundaries;
- listings are governed artifacts rather than implied execution;
- benchmark protocols distinguish quality, safety, latency, memory, energy, and cost;
- decision gates prevent an attractive average score from hiding a safety or deployability failure.

## The learning pattern

Each major topic follows the same loop:

1. name the engineering problem and the baseline;
2. build an intuitive model of the mechanism;
3. calculate a small example by hand;
4. implement or configure the smallest meaningful experiment;
5. measure quality and systems behavior under controlled conditions;
6. decide whether the evidence supports keeping, revising, or rejecting the change.

For example, “use 4-bit quantization” is not a conclusion. The book first asks which tensors and kernels use 4-bit
values, estimates ideal storage, verifies runtime support, measures peak memory and sustained decode, and checks whether
domain and safety behavior regressed. The method is accepted only for the exact artifact-runtime-device tuple that
passes.

## Evidence discipline

Claims carry labels defined in [Evidence labels](evidence-labels.md). In particular:

- a cited paper supports `PAPER-REPORTED`, not `REPRODUCED`;
- current product documentation supports `DOC-VERIFIED`, not measured compatibility;
- runnable-looking code remains `UNVERIFIED IMPLEMENTATION` until execution evidence is retained;
- proposed benchmark matrices remain `PLANNED EXPERIMENT`;
- design numbers are `ILLUSTRATIVE` or `ESTIMATE` unless a stronger trace exists.

This edition contains substantial research synthesis and implementation designs. Device, model, compatibility, cost,
and benchmark claims still require the evidence named beside them. The [figure index](../back-matter/figure-index.md),
[table index](../back-matter/table-index.md), and [limitations register](../back-matter/errata-and-limitations.md) state
those boundaries directly.

## Safety and stewardship

Domain specialization can increase persuasive fluency without ensuring correctness. Pharmacy, hazardous maintenance,
industrial, electrical, and animal-care workflows therefore use non-compensatory safety gates: a model that fails
critical safety tests is not rescued by strong aggregate accuracy. R4 autonomous prescribing or operation is outside
release scope. See [Safety and responsible use](safety-and-responsible-use.md).

## A note to practitioners

Use the book as a sequence of decisions, not a catalog of fashionable techniques. Establish requirements and measurement
first. Compare a baseline. Change one controlled factor where possible. Retain artifacts. Report negative results. Ship
only the simplest architecture that passes the declared quality, safety, memory, latency, energy, cost, licensing, and
maintainability gates.
