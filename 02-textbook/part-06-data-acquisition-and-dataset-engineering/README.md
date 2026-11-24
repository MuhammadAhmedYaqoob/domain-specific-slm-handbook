<a id="part-06"></a>

# Part VI — Data Acquisition and Dataset Engineering

Domain learning is bounded by source rights, coverage, provenance, quality, and objective alignment. This part treats
datasets as versioned products with lineage—not folders of text.

## Conceptual map

A source is discovered, rights-reviewed, acquired, parsed, normalized, deduplicated, quality-scored, and assigned to a
purpose-specific split. Only then does it become a training or evaluation record. Provenance remains traceable through
every transformation. Synthetic generation adds a model, prompt, sampling policy, and verification burden; it does not
erase the need for authority.

## Learning outcomes and dependencies

Readers should be able to discover lawful sources, construct a source register, normalize and deduplicate content,
prevent leakage, design CPT/SFT/preference/evaluation records, and validate synthetic data. Read after Part II’s
objective definitions and before executing any domain training. It feeds Parts VII, VIII, X, and XI.

## Chapters

1. [CH-23 — Finding Domain-Specific Training Data](ch-23-finding-domain-specific-training-data.md): discovery,
   licensing, provenance, authority, and coverage.
2. [CH-24 — Dataset Collection and Processing](ch-24-dataset-collection-and-processing.md): ingestion, parsing,
   cleaning, deduplication, PII/secrets handling, lineage, and versioning.
3. [CH-25 — Dataset Design for Different Training Objectives](ch-25-dataset-design-for-different-training-objectives.md):
   separate schemas and sampling policies for CPT, SFT, preference learning, and evaluation.
4. [CH-26 — Synthetic Data and Knowledge Generation](ch-26-synthetic-data-and-knowledge-generation.md): teacher
   generation, diversity, verification, contamination, and provenance.

## Decision checkpoint

No training dataset is accepted without source-level rights, immutable manifests, split policy, duplicate/leakage
analysis, domain and safety review, quality statistics, and reproducible transformation lineage. Synthetic data never
becomes ground truth merely because it is fluent.

### Practice before continuing

Choose one public technical document and draft a source record: owner, URL/version, license or permission, allowed use,
authority, update frequency, extraction method, and exclusion conditions. Discovery is complete only when the rights
question has an answer.

## Cross-links

[Part II](../part-02-training-and-domain-adaptation/README.md) ·
[Part VII](../part-07-multi-domain-slm-engineering/README.md) · [Safety](../front-matter/safety-and-responsible-use.md)
