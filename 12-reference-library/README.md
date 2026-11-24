# Glossary and reference library

This library helps readers learn the vocabulary of small-language-model engineering and judge the evidence behind
technical claims. Start with the glossary when a concept is unfamiliar, use the abbreviations list while reading a
chapter, and consult the definition register when two similar terms appear interchangeable. The bibliography then
shows where claims originate and how strongly a source can support them.

The library is also the canonical terminology and source-control layer for the book. Reading a paper, model card, or
vendor page does not turn its claims into project execution evidence.

## Contents

- [Glossary](glossary.md) — concepts across language modeling, adaptation, compression, data, evaluation, edge
  deployment, AWS, security, and lifecycle.
- [Abbreviations](abbreviations.md) — canonical expansions and usage notes.
- [Definition register](definition-register.md) — normative resolutions for overloaded terms; it overrides casual usage
  elsewhere in the repository.
- [Bibliography](bibliography.md) — categorized primary papers and official documentation cited by authored repository
  content.
- [Source record tutorial](source-record-template.md) — a worked paper review, instructions, and reusable evidence
  record.
- [Model checkpoint record tutorial](model-checkpoint-record-template.md) — a worked checkpoint review, instructions,
  and reusable checkpoint record.
- [Runtime record tutorial](runtime-record-template.md) — a worked runtime review, instructions, and reusable
  compatibility record.

## Suggested learning path

1. Read a chapter and expand unfamiliar initials in [abbreviations](abbreviations.md).
2. Look up the concept in the [glossary](glossary.md), paying special attention to its example and related terms.
3. Use the [definition register](definition-register.md) before comparing models, memory, speed, or support claims.
4. Follow a bibliography link to the original paper or official documentation.
5. Use the worked records to evaluate whether the source supports the exact claim you want to make.

## Source classes

1. **Primary research:** original paper or archival report. Its claims are `PAPER-REPORTED`, never automatically
   reproduced.
2. **Official documentation:** maintainer, standards body, platform owner, model card, license, or official repository.
   Inspection supports `DOC-VERIFIED`; it does not prove execution.
3. **Vendor material:** official performance or capability claims remain `VENDOR-REPORTED` until independently tested.
4. **Independent reproduction:** project or third-party execution with auditable artifacts. Use `EXECUTED` or
   `REPRODUCED` only when the reproducibility standard is satisfied.
5. **Secondary source:** survey, tutorial, or commentary. Useful for discovery, but not preferred support for
   consequential claims.

## Maintenance rules

- Use stable keys from the [bibliography](bibliography.md) and link the exact source at the claim site.
- Record access dates for mutable web documentation and immutable revisions, releases, DOI values, or arXiv identifiers
  where available.
- Never infer checkpoint compatibility from a family, runtime support from a format, or commercial permission from
  public weight access.
- Never guess authors, dates, licenses, versions, benchmarks, hardware support, or publication status. Record the
  exact verification action needed, such as “check Crossref using the title before approval,” and keep the record out
  of the approved state until that check is complete.
- Recheck mutable model, runtime, AWS, pricing, licensing, and API records at experiment freeze and release approval.
- Preserve the ten evidence labels defined by governance exactly.

## Research boundary

The bibliography consolidates primary and official sources actually cited in the repository as of **2026-11-23**. It is
not exhaustive. No entry asserts that this project reproduced a paper, converted a model, ran a runtime, deployed to
AWS, or tested a physical device.

Apply the [evidence and citation standard](../00-governance/evidence-and-citation-standard.md),
[terminology standard](../00-governance/terminology-standard.md), and
[reproducibility standard](../00-governance/reproducibility-standard.md). Navigation begins at the
[bookmap](../BOOKMAP.md).
