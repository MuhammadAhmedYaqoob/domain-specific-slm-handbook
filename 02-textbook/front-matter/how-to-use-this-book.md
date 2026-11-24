<a id="how-to-use"></a>

# How to Use This Book

## Read claims before recipes

Begin with [Evidence labels](evidence-labels.md), [Notation](notation-and-conventions.md), and
[Safety](safety-and-responsible-use.md). Chapters combine established concepts, sourced observations, implementation
designs, and proposed experiments. Their labels determine what may be repeated as a result.

## Four reading tracks

### Complete engineering track

Read Parts I–IX in order, execute the gates in Part VIII, then complete Parts X–XIV. This route preserves the dependency
from requirements and data to deployment and maintenance.

### Research track

Read Parts I–IV, VI–VIII, and XIII. Use the
[experiment checklist](../back-matter/appendix-experiment-reporting-checklist.md) to convert a chapter hypothesis into a
controlled protocol. Retain negative and null results.

### Edge/mobile track

Read CH-02, CH-10, CH-13–15, Part V, CH-30, CH-36, CH-43, and CH-47. Treat conversion success on a desktop as
insufficient: validate operators, peak memory, TTFT, decode throughput, energy, thermal behavior, and output quality on
the exact device.

### Cloud/platform track

Read Parts II–IV, VI–IX, XII, and XIV. Emphasize artifact provenance, IAM, network boundaries, cost attribution,
observability, rollback, and parity between evaluated and served artifacts.

### Fast prototype track

Read CH-01–03, CH-06, CH-08–11, CH-23–25, CH-29–31, and CH-37. Produce a small SFT adapter, one quantized export, and a
baseline-versus-candidate report. This route is suitable for learning, not for bypassing the safety, licensing, or
exact-target gates required for release.

## Dependency map

The conceptual spine is:

1. **Requirements and foundations** — CH-01–03.
2. **Learning method** — CH-04–09.
3. **Compression and deployability** — CH-10–22.
4. **Data and domain organization** — CH-23–28.
5. **Measurement** — CH-29–32.
6. **Cloud and edge delivery** — CH-33–36.
7. **Integrated project and extension** — CH-37–41.
8. **Production lifecycle and research** — CH-42–46.
9. **Architecture decision** — CH-47.

Do not postpone evaluation until step 8. Define the Part VIII scorecard before training, then execute it after each
material change.

## How to work through a chapter

1. Record the chapter’s prerequisites and acceptance criteria.
2. Separate factual claims from illustrative examples and estimates.
3. Follow links to canonical governance or source records.
4. Treat code according to its execution label.
5. Complete conceptual questions before implementation.
6. Run only within declared safety and resource boundaries.
7. Capture environment, seeds, artifacts, raw measurements, and failures.
8. Update decision and limitations records instead of silently changing assumptions.

Use a three-column notebook while reading: **concept**, **evidence**, and **decision impact**. If a paper reports faster
decode, put the claim under evidence; do not put “choose this method” under decision impact until the runtime and target
match your own system.

## Hands-on sequence

The shortest defensible implementation sequence is: establish a frozen baseline; construct rights-aware training and
evaluation splits; choose adaptation; train and retain artifacts; evaluate domain quality and safety; compress one
factor at a time; export to candidate runtimes; benchmark cloud and physical edge targets; apply non-compensatory gates;
select, document, release, monitor, and preserve rollback.

### A one-week study sprint

- Day 1: define one safe task, one prohibited task, and one baseline.
- Day 2: create ten training examples and a separate ten-case evaluation set to learn the schemas.
- Day 3: run or inspect one SFT path and preserve its configuration.
- Day 4: estimate weight and KV-cache memory, then export one candidate format.
- Day 5: measure quality, peak memory, TTFT, and decode under one frozen workload.
- Day 6: analyze failures and repeat only the change justified by that analysis.
- Day 7: write an accept, revise, or reject record. Label all made-up teaching numbers `ILLUSTRATIVE`.

## Cross-reference conventions

Chapters are `CH-01`–`CH-47`; required figures are `FIG-01`–`FIG-26`; required tables are `TBL-01`–`TBL-25`. Links
should target canonical files and explicit HTML anchors where available. The
[detailed index](../back-matter/index-plan.md), [figure index](../back-matter/figure-index.md), and
[table index](../back-matter/table-index.md) provide entry points.

## Completion is gated

Reading a chapter or generating an artifact is not completion. Use the
[definition of done](../../00-governance/definition-of-done.md), [assessment materials](../../13-assessments/README.md),
and [final decision framework](../back-matter/final-decision-framework.md).
