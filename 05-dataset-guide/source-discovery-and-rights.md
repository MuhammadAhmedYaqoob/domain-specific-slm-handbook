# Source discovery and rights

## Why discovery starts with a knowledge map

A search engine ranks pages, not fitness for training. If collection starts with broad keywords, the resulting corpus
usually overrepresents easy explanatory prose and underrepresents revisions, hazards, regional variants, negative
cases, and evidence needed for evaluation. Start with the decisions the model should support, decompose them into
entities and relationships, and mark every coverage cell as `covered`, `candidate`, `gap`, or `excluded`.

For an automotive charging-system lesson, for example, the knowledge structure includes battery state, charging
voltage, engine operating condition, ambient conditions, instrument connection, observed symptom, possible causes,
safe tests, and model-specific limits. “Battery” alone is not a useful discovery query. A better query combines a stable
identifier, exact variant, document type, revision, and authority.

## Rights are a chain of planned actions

“Open” is incomplete. An engineer needs an answer for each planned act:

- access and retain the original;
- extract text, tables, or images;
- transform and annotate;
- use for a named training or evaluation objective;
- share records with reviewers;
- redistribute examples or a dataset;
- release weights or adapters trained from it;
- use commercially, if applicable;
- honor attribution, deletion, expiry, and geographic restrictions.

Record the governing instrument and its version. A code repository license may cover software but not bundled data. A
government portal can contain third-party material. A permissive document may embed restricted photographs. When the
answer is uncertain, quarantine the asset and seek a qualified review; do not turn uncertainty into a guessed approval.

## A repeatable discovery procedure

1. Write a capability-and-hazard matrix and identify empty cells.
2. Search authoritative catalogs using identifiers and revision terms.
3. Add complementary professional, scholarly, educational, operational, and community tracks.
4. Record candidates before downloading content.
5. Capture source ownership, authority scope, version, locality, update mechanism, and likely restrictions.
6. Review the exact asset and intended acts.
7. Approve only named objectives; keep evaluation-only sources out of training.
8. Preserve rejected candidates and reasons so another engineer does not repeat the same search.

Discovery ends with a portfolio, not one “best” source. Independent source families reduce correlated errors and make
contradictions visible.

## Status and decision rule

**AUTHORED / STATICALLY REVIEWED / NOT LEGALLY APPROVED.** Candidate entry points and methods derive from
[CH-23](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-23-finding-domain-specific-training-data.md).
Public access, an API, a repository license, government authorship, or a robots allowance does not by itself permit
model training.

Every candidate is `reject`, `unknown_quarantine`, `metadata_only`, `evaluation_only`, or
`approved_for_named_objectives`. Approval applies to an exact asset/version and planned acts; it never applies
automatically to a hostname.

## Discovery method

Start with a matrix:

```text
capability × jurisdiction × effective time × asset/population variant
× hazard × language × training objective
```

For every uncovered cell, search independently across:

1. regulator and government catalogs;
2. manufacturer and component-owner portals;
3. standards-body catalogs;
4. scholarly indexes, DOI registries, institutional repositories, and dataset registries;
5. open educational resource catalogs and library records;
6. documented APIs and bulk-data catalogs;
7. professional associations and commissioned experts;
8. forums and field reports for vocabulary and candidate cases only;
9. archived editions, errata, recalls, corrections, and change feeds.

Use stable identifiers—standard/revision, DOI, product/model/year, code, label Set ID, vocabulary ID, scientific
name—not only keywords. Preserve negative searches and rejection reasons; a gap becomes an expert-authoring requirement,
not permission to invent facts.

## Source-class qualification

<!-- markdownlint-disable MD013 -->
| Class                                   | Useful for                                 | Required rights/evidence checks                                       | Default caution                                      |
| --------------------------------------- | ------------------------------------------ | --------------------------------------------------------------------- | ---------------------------------------------------- |
| official references and safety guidance | law, labels, recalls, identifiers, hazards | exact jurisdiction, effective date, third-party material, reuse terms | authority scope is narrower than “all domain truth”  |
| manufacturer manuals                    | model-specific procedure and warnings      | contract, edition, portal terms, redistribution/model-release rights  | ownership of a copy is not training permission       |
| standards                               | terminology and requirements               | exact licensed edition and permitted uses                             | catalog abstracts do not license full text           |
| academic/open datasets                  | observations, annotations, methods         | dataset—not code/paper—license, consent, privacy, DUA                 | benchmark cannot also support final claims           |
| OER/books                               | coherent teaching                          | exact license/version, element-level exclusions, edition              | `NC`, `ND`, `SA`, quotations, and images need review |
| structured databases/APIs               | normalized records and updates             | database rights, field exceptions, API contract, limits               | open endpoint can contain restricted fields          |
| forums                                  | natural questions and failure language     | platform/author rights, PII, deletion, API/robots terms               | anecdote is not safety ground truth                  |
| expert-authored material                | rare and high-risk gaps                    | contributor agreement, credentials, conflicts, confidentiality        | evaluation authors must be separated                 |
| open corpora                            | linguistic breadth                         | upstream composition, opt-outs, PII, licenses                         | coarse provenance and benchmark contamination        |
| synthetic data                          | controlled variants                        | grounding-source and teacher terms, full lineage                      | generator is not an authority                        |
<!-- markdownlint-restore -->

## Non-compensatory gates

1. **Identity:** canonical origin, owner/controller, title, version, dates, locator, upstream relationship.
2. **Rights:** exact instrument and whether it permits acquire, copy, transform, train, evaluate, redistribute
   examples/data, release weights/adapters, and commercial use.
3. **Access policy:** terms version, API plan, robots evidence, rate limits; never bypass authentication, paywalls,
   CAPTCHAs, or controls.
4. **Privacy/confidentiality:** lawful basis, consent scope, minimization, sensitive fields, retention, deletion,
   security.
5. **Authority/safety:** what claim types the source can establish, applicability, qualifications, conflicts,
   supersession.
6. **Technical acquisition:** approved endpoint/method, pagination, schema, quotas, checksums, refresh signal.
7. **Evaluation firewall:** disposition and source-family split fixed before content is exposed to builders or teachers.

Any unresolved planned act fails closed. Fair-use/fair-dealing questions require jurisdiction-specific counsel and are
not a default corpus strategy.

## Source and rights register

**Listing DG-SR-01 — source record fields.** The populated record below demonstrates every field. In production, the
schema validator defines required types and enumerations; documentation should not use blank values that look
release-ready.

Decisions are append-only. A new review supersedes an old decision; it does not erase it. Asset- and field-level
exceptions override site-level summaries.

### Fully populated illustrative record

The following fictional record demonstrates how the fields work. The URL uses the reserved `.example` domain and is
not a real source or an approval.

```yaml
source_id: "src-auto-oer-charging-v2"
canonical_url: "https://training.example/automotive/charging-system-v2.pdf"
publisher: "Example Technical College"
source_class: "open_educational_resource"
title: "Charging-System Measurement Fundamentals, second edition"
jurisdictions: ["US-general-education"]
languages: ["en-US"]
capability_tags: ["automotive.electrical.charging", "measurement.voltage", "diagnostic-sequencing"]
version:
  edition_or_release: "2.0"
  published_at: "2026-02-12"
  effective_from: "2026-02-12"
  effective_to: null
  supersedes: "src-auto-oer-charging-v1"
rights:
  status: "approved_for_named_objectives"
  instrument_url: "https://training.example/licenses/charging-v2.txt"
  instrument_sha256: "8a2d2fd46e56dc70a03e67a9c9f0aa73ffab018acb149f8d509f38a58423f5d2"
  checked_at: "2026-11-20T14:30:00Z"
  asset_scope: "PDF text and author-created diagrams; excludes two credited photographs"
  permits: ["copy", "extract_text", "transform", "cpt", "sft", "internal_review"]
  obligations: ["retain_attribution", "publish_change_notice"]
  exceptions: ["pages 18-19 photographs excluded"]
  reviewer: "rights-review-014"
access:
  method: "approved_https_download"
  terms_url: "https://training.example/terms/2026-01.txt"
  robots_url: "https://training.example/robots.txt"
  credentials_required: false
  rate_limit_reference: "one conditional request per day"
provenance:
  upstream_origin: "publisher-authored"
  acquisition_receipt_uri: "receipts/2026/11/src-auto-oer-charging-v2.json"
  raw_sha256: "2f3ea9a3f13b1ad50685d00460e771bc709f6464cd595414d13d6c6f85b52133"
quality:
  authority_scope: "general measurement education; not vehicle specifications"
  known_limitations: ["12 V examples only", "no hybrid or EV high-voltage procedures"]
  coverage: "battery and alternator concepts, instrument setup, evidence-first diagnosis"
  bias: "single educational institution and US terminology"
freshness:
  stated_cadence: "annual"
  checked_at: "2026-11-20"
  next_review_at: "2027-02-12"
  urgent_triggers: ["publisher correction", "license change", "safety incident"]
restrictions:
  pii_or_sensitive_fields: []
  attribution: "Title, edition, publisher, and source URL"
  redistribution: "derived text records internal only"
  model_release: "requires separate approval before public weight release"
  deletion_or_takedown: "remove excluded source descendants within 30 days"
disposition:
  status: "approved"
  allowed_objectives: ["cpt", "sft"]
  split_family_id: "fam-auto-charging-oer-example-college"
approvals:
  rights: "rights-decision-2026-044"
  privacy: "privacy-na-2026-031"
  domain: "auto-review-2026-117"
  safety: "safety-review-2026-088"
```

The record deliberately narrows authority and objectives. It does not claim that a general handout establishes
vehicle-specific thresholds, and its excluded pages cannot silently enter OCR or image-derived data.

## Verification questions for engineers

Before promotion, another engineer should be able to answer from stored evidence:

- Which exact bytes were reviewed, and do their hashes still match?
- Which clauses permit each intended act?
- Are third-party elements excluded at page or field level?
- Does the source have authority for the claim type being extracted?
- What event forces re-review?
- Can every descendant be found if permission changes?

If any answer depends on memory, a browser bookmark, or an unstored conversation, provenance is incomplete.

## Quality, authority, and coverage

Score authority, provenance, coverage, freshness, consistency, verifiability, structural usability, and language fit
separately. Use a score to prioritize review, never to override rights, privacy, security, or safety vetoes. Count
independent source families, not mirrors.

Suggested contextual authority labels:

- `A1`: controlling law, approved label, official recall, controlling rule;
- `A2`: applicable current manufacturer procedure or recognized standard;
- `A3`: peer-reviewed/professional evidence in scope;
- `B`: qualified expert derivative grounded in A-tier evidence;
- `C`: community report or unverified observation.

The applicable authority depends on the claim: a law is not a repair procedure; a terminology mapping is not a treatment
recommendation.

## Approved acquisition procedure

<!-- markdownlint-disable MD013 -->

`propose → register → rights/privacy/safety review → metadata-only dry run → acquire → hash → validate schema/counts → archive policy and receipt → quarantine → promote`

<!-- markdownlint-restore -->

Acquisition code must consume an allowlisted, approved manifest; use identifiable, rate-limited requests; redact secrets
from receipts; preserve raw response bytes and validators; fail on schema/policy drift; and never decide rights itself.
API pagination and retry behavior are endpoint-specific.

## Monitoring and withdrawal

Monitor terms, licenses, owner, schema, version/effective date, recalls/corrections, source availability, and safety
notices. Urgent changes block new releases until adjudicated. A withdrawal request traverses lineage to every segment,
objective view, dataset, run, adapter/model, and package; affected artifacts are rebuilt or retired.

## Exit criteria

- every admitted artifact resolves to an approved source/rights record;
- all obligations are materialized in manifests/notices;
- unknown rights and prohibited access are zero;
- source families and final-evaluation dispositions are frozen;
- domain, rights, privacy, and safety approvers are named;
- acquisition and takedown paths are tested before scale.

## Cross-links

- [Dataset guide](README.md)
- [Collection and lineage](collection-processing-and-lineage.md)
- [Synthetic governance](synthetic-data-governance.md)
- [Evidence standard](../00-governance/evidence-and-citation-standard.md)
