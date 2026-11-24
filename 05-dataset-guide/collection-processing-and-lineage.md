# Collection, processing, and lineage

## From document to evidence-preserving records

Collection is a compiler pipeline. Source files are the input language; canonical records are an intermediate
representation; objective datasets are target-specific builds. A good compiler does not discard source locations or
silently change meaning, and neither should a data pipeline.

Three copies serve different purposes:

- **raw artifact:** exact bytes used to prove provenance and reproduce extraction;
- **canonical representation:** structured text, tables, units, scope, and review decisions;
- **objective view:** only the fields and rendering needed for one learning or evaluation task.

Never edit raw artifacts in place. A corrected parser creates a new extraction linked to the same raw hash. A corrected
source creates a new artifact and usually a new source version.

## Extraction before cleaning

Extraction asks, “What structure did the publisher communicate?” Cleaning asks, “Which representation helps analysis?”
Reversing them loses evidence. For example, flattening a warning box into surrounding prose may teach a model that the
warning is an ordinary optional step. Flattening a table can detach a value from its row, unit, or footnote.

For every extracted block, retain a locator, block type, order, parent heading, and confidence. Review OCR at the
character classes most likely to change meaning: decimal points, minus signs, inequality symbols, units, identifiers,
and negation. `0.5 A`, `5 A`, and `O.S A` can be one OCR confusion with very different consequences.

## Normalization and units as typed interpretation

Normalization should make comparison possible without pretending ambiguity has disappeared. Keep:

1. the exact raw expression;
2. an NFC display form;
3. a matching-only form for duplicate search;
4. a parsed typed value when interpretation is justified.

A unit record should contain value, unit, physical dimension, canonical value, conversion rule, precision, condition,
and review status. Never convert an ambiguous gallon, ton, or decimal separator until locale and context are known.
Converting does not authorize rounding; `12.0 V` may communicate different precision from `12 V`.

```json
{
  "raw": "2.50 ± 0.05 bar at 20 °C",
  "parsed": {
    "value": 2.5,
    "uncertainty": 0.05,
    "unit": "bar",
    "dimension": "pressure",
    "condition": "temperature = 20 °C"
  },
  "canonical": {
    "value": 250.0,
    "uncertainty": 5.0,
    "unit": "kPa",
    "rule_id": "ucum-pressure-v1"
  },
  "review_status": "human_verified"
}
```

This is an illustrative representation, not a domain recommendation.

## Deduplication is family construction

Exact hashes find byte-identical files. Normalized hashes find formatting variants. Token shingles and similarity
search find near copies. None identifies the conceptual family by itself. Editions, translations, quoted passages,
mirrors, templates, and synthetic paraphrases need explicit lineage edges.

Use a two-step process:

1. retrieve candidates with conservative exact, lexical, and semantic methods;
2. classify the relationship as `same_bytes`, `format_variant`, `excerpt`, `revision`, `translation`,
   `template_sibling`, `synthetic_derivative`, or `unrelated`.

Do not delete all duplicates. Choose a canonical training representative while retaining provenance and attribution.
Evaluate deduplication thresholds on manually labeled pairs by document type; boilerplate-heavy manuals behave
differently from short Q&A.

## Contradictions are scoped data

Two sentences disagree only after scope is aligned. Represent each claim as:

```text
subject + predicate + value + unit + polarity + asset/population
+ conditions + jurisdiction + effective interval + evidence source
```

Then classify the apparent conflict:

- **true conflict:** same scope and time, incompatible claims;
- **supersession:** a newer revision replaces an older one;
- **applicability difference:** different model, formulation, taxon, or operating condition;
- **terminology difference:** equivalent concepts use different labels;
- **extraction error:** OCR or table parsing changed the claim;
- **unresolved:** evidence is insufficient.

Preserve both claims and the adjudication. Frequency is not authority: ten copied pages do not outweigh one current,
applicable controlling source.

## Quality scoring without hiding vetoes

Use a vector such as provenance completeness, extraction fidelity, authority fit, applicability completeness,
freshness, coherence, verifiability, and reviewer confidence, each on a documented scale. A weighted score can order a
review queue:

```text
priority_score = 0.20 provenance + 0.20 extraction + 0.15 authority
               + 0.15 applicability + 0.10 freshness
               + 0.10 coherence + 0.10 verifiability
```

Weights are illustrative and must be calibrated. Rights, privacy, malware, missing lineage, and safety failures are
separate Boolean vetoes. A record with a score of 0.97 and prohibited rights still fails.

## Status and invariant

**AUTHORED PIPELINE SPECIFICATION / NOT EXECUTED.** This operational summary derives from
[CH-24](../02-textbook/part-06-data-acquisition-and-dataset-engineering/ch-24-dataset-collection-and-processing.md). No
connector, parser, OCR engine, quality threshold, or dependency is represented as tested here.

Every released row must trace backward to authorized original bytes and forward to every snapshot/model descendant:

```text
source decision → raw artifact → extraction → canonical segment
→ review/dedup/split → objective record → dataset manifest → run/release
```

Raw evidence is append-only. Corrections, reprocessing, and source changes create new versions.

## Pipeline stages

1. **Authorize:** resolve source and rights records; unknown or expired approval blocks collection.
2. **Acquire:** prefer official bulk export, documented API, conditional HTTP, then expressly permitted crawling.
3. **Receipt:** store original bytes, SHA-256, request parameters without secrets, timestamps, response metadata, policy
   snapshots, and failures.
4. **Sandbox:** treat PDFs, archives, HTML, images, XML, and office files as hostile; bound bytes, pages, pixels,
   expansion, redirects, and parser time.
5. **Extract:** preserve structural locators, reading order, tables, captions, warnings, page regions, and extraction
   confidence.
6. **Normalize:** retain raw text; create NFC display and separate matching forms; parse units without erasing the raw
   expression.
7. **Gate:** rights, malware, PII/confidentiality, language, quality, relevance, safety, and lineage.
8. **Deduplicate/adjudicate:** exact and near duplicates, versions, mirrors, contradictions, and canonical families.
9. **Split:** assign connected root families before chunking, paraphrasing, or synthesis.
10. **Publish:** immutable objective views, checksums, validation reports, dataset bill of materials, and lineage graph.

## Secure acquisition

Connectors must allowlist schemes, hosts, paths, and redirects; reject loopback, link-local, private-network, and
unapproved destinations; use bounded timeouts/retries; honor documented `Retry-After`; checkpoint pagination; and fail
on repeated cursors or schema drift. Never bypass controls. Credentials belong in a secret manager/environment and must
not appear in URLs, manifests, or logs.

Downloaded payloads execute with no privilege, cloud credentials, or unrestricted egress. Scan archives; reject
traversal, bombs, excessive nesting, macros, embedded executables, and downloaded models. Parser failure produces
quarantine, never an accepted empty record.

## Extraction contracts

### HTML

Parse the DOM rather than regular expressions. Preserve headings, lists, code, warnings, captions, tables, and order.
Store effective/canonical URL, encoding evidence, author/dates, language hint, and a stable structural locator.
Boilerplate removal must be evaluated because it can delete warnings.

### PDF and OCR

Classify each page as born-digital, scanned, or hybrid. Preserve page, block coordinates, reading order, rotation, and
warnings. For OCR, record renderer, engine/version, language pack, preprocessing, boxes, and confidence. Human-review
low-confidence quantities, negation, identifiers, and safety tables. Never reconstruct redacted text.

### Tables and images

Tables retain captions, hierarchical headers, row keys, raw cell values, parsed values, units, spans, and footnotes.
Distinguish missing, not applicable, redacted, and not measured. Images retain source region/hash/caption; strip
released EXIF location/device metadata. Model-generated descriptions are synthetic annotations requiring provenance and
review.

## Canonical segment

**Listing DG-CP-01 — canonical segment.** The populated example below demonstrates the complete record. A production
schema still permits explicit `null` or `und` when the source truly does not establish a value; do not infer missing
facts from filenames.

### Populated canonical example

```json
{
  "segment_id": "seg-auto-charge-000184",
  "source_id": "src-auto-oer-charging-v2",
  "artifact_sha256": "2f3ea9a3f13b1ad50685d00460e771bc709f6464cd595414d13d6c6f85b52133",
  "locator": {
    "page": 7,
    "section": "Interpreting a charging complaint",
    "region": "warning-box-2"
  },
  "text_raw": "A diagnostic code identifies a monitored condition; test the circuit before replacing a component.",
  "text_nfc": "A diagnostic code identifies a monitored condition; test the circuit before replacing a component.",
  "text_match": "a diagnostic code identifies a monitored condition test the circuit before replacing a component",
  "language": {
    "tag": "en-US",
    "confidence": 0.998,
    "detector": "language-detector-3.1"
  },
  "units": [],
  "rights_basis_id": "rights-decision-2026-044",
  "authority_scope": "general diagnostic sequencing",
  "jurisdiction": ["US-general-education"],
  "effective_from": "2026-02-12",
  "effective_to": null,
  "quality_components": {
    "provenance": 1.0,
    "extraction": 1.0,
    "authority_fit": 0.8,
    "applicability": 0.75,
    "freshness": 1.0,
    "verifiability": 1.0
  },
  "risk_tier": "moderate",
  "root_family_id": "fam-auto-charging-oer-example-college",
  "duplicate_cluster_id": "dup-auto-code-condition-004",
  "parent_ids": ["artifact:src-auto-oer-charging-v2"],
  "decision": "accept",
  "reason_codes": ["RIGHTS_APPROVED", "EXTRACTION_VERIFIED", "DOMAIN_REVIEWED"],
  "transform": {
    "code_revision": "git:4d56b7c",
    "config_sha256": "0f2f4d58dbf57ebf2d9be3d77bbf75ad25399cdb3c2a05b92e8805fc3d9af682"
  }
}
```

The authority score is below 1.0 because a general educational source cannot supply model-specific service limits.

## Normalization

- preserve `text_raw`;
- use Unicode NFC for authoritative display;
- use NFKC/case-folding only in a separate comparison field;
- surface bidirectional, zero-width, and disallowed control characters;
- never auto-correct names, codes, identifiers, measurements, or part numbers;
- store unit expression, parsed value, dimension, canonical conversion, rule version, locale, precision,
  range/inequality, and uncertainty;
- leave ambiguous `ton`, `gal`, decimal separators, and references unresolved.

## Quality, privacy, and safety gates

Quality is a vector: provenance, rights clarity, extraction integrity, coherence, relevance, attribution, freshness, and
review confidence. Any aggregate is for ranking only. Hard failure includes unknown/prohibited rights, malware,
prohibited PII, secrets, missing lineage, failed safety review, or final-test ancestry.

PII/sensitive handling is purpose-led: minimize collection, classify direct/quasi-identifiers, quarantine contextual
cases, apply approved deletion/redaction/pseudonymization, encrypt and restrict raw evidence, and maintain
retention/takedown rules. Hashing identifiers is not anonymization. Exact zoo locations, industrial security
architecture, patient records, VIN-owner links, and confidential procedures need special controls or exclusion.

Every decision is `accept`, `reject`, `quarantine`, or `needs_review` with rule/version and reason codes.

## Duplicates, versions, and contradictions

- hash raw bytes and normalized match text;
- retrieve near-duplicate candidates with declared shingling/MinHash/LSH or equivalent;
- manually calibrate thresholds by document type;
- cluster mirrors, editions, translations, OCR copies, quoted passages, templates, and synthetic descendants;
- guard against transitive over-clustering;
- select canonical records by rights, authority, applicability, freshness, completeness, and extraction quality;
- retain all attribution and lineage.

Contradictions are scoped claims
`(subject, predicate, value/unit, negation, context, jurisdiction, effective interval, source)`. A conflict may be a
revision, locale difference, different asset/population, or OCR error. Preserve both and obtain expert adjudication;
frequency cannot override current applicable authority.

## Leakage-safe splitting

Random row splitting fails because rows are not independent. One manual can produce hundreds of chunks; one expert case
can produce Q&A, paraphrases, preference pairs, and synthetic dialogues. If siblings land in train and test, the score
measures recognition rather than generalization.

Build a graph in which nodes are records and edges mean shared source, document, edition, translation, case, asset,
author, template, prompt, or synthetic ancestry. Compute connected components, assign each component once, and only then
render rows. For time-forward evaluation, apply the date boundary after family grouping and quarantine families that
span both periods unless the protocol explicitly handles them.

Build connected components over source family, edition/translation, document, case/event, entity/site/author, duplicate
cluster, synthetic seed/template, teacher prompt, and counterfactual family. One component receives one split. Use
temporal and entity holdouts when evaluating future/generalization behavior.

Audit:

- exact/normalized/near/semantic overlap;
- common ancestors and source families;
- answer, rationale, rubric, ontology, or label overlap;
- benchmark exposure to teachers/judges;
- test-informed prompt, threshold, data, checkpoint, compression, or calibration choices;
- temporal/supersession violations.

Semantic matching is an offline audit tool, not runtime RAG. No detector proves absence; record residual uncertainty.

## Storage and manifest

Use object storage for originals, JSONL for audit/review, Parquet for typed releases, a transactional registry for
decisions/edges, and specialized shards only where justified. Avoid tiny files and mutable “latest” paths.

**Listing DG-CP-02 — dataset manifest.** Use the fully populated illustrative manifest below as the teaching record.
Production schemas should reject omitted approvals, files, validation reports, and limitations.

### Populated manifest example

```yaml
dataset_id: "automotive-sft-2026.11.0-illustrative"
status: "illustrative_not_released"
objective: "sft"
created_at: "2026-11-22T18:00:00Z"
schema_id: "sft-messages/v2"
parent_dataset_ids: ["automotive-canonical-2026.11.0"]
source_register_sha256: "8f398090f683d1bb3e43d0f477f46ea28f57a8f4f928bee3c9751f693f0c4478"
lineage_graph_sha256: "325d70210e38b3359ca7db949a3d648b188f51b678384f5ecf74315b4a43684d"
pipeline_git_commit: "4d56b7c"
resolved_config_sha256: "2a0185dc917af223ad6f3e7881b5e842d4c885080cb874ed66fca8b0ca780b86"
environment_lock_sha256: "5a970d05b847f2f49d951fb4a7cffad28a1ab8299f517e574f386e3e1b9f3f7d"
split_policy_id: "connected-family-v2"
counts_by_split: {train: 8200, validation: 900, test: 0}
distributions:
  risk: {low: 4100, moderate: 3300, high: 1700}
  authority: {A2: 3900, A3: 2700, B: 2500}
rejection_reasons: {unknown_rights: 311, duplicate: 826, unsupported_claim: 142}
files:
  - path: "train-000.parquet"
    bytes: 2849107
    sha256: "9d66a83c88dcb8c38cb7fcfa4fb72207047cb0b19ded2a0321e516b6e0a94186"
validation_reports:
  schema: "report-schema-2026-11-22"
  rights: "report-rights-2026-11-22"
  privacy: "report-privacy-2026-11-22"
  duplicates: "report-duplicates-2026-11-22"
  leakage: "report-leakage-2026-11-22"
  safety: "report-safety-2026-11-22"
approvals:
  domain: "approval-domain-118"
  rights: "approval-rights-045"
  safety: "approval-safety-091"
limitations:
  - "Illustrative counts and hashes; no dataset was built."
  - "No final evaluation rows are included."
```

## Worked mini-pipeline

1. Register one approved PDF and store its bytes and receipt.
2. Extract heading and warning blocks with page locators.
3. Compare OCR/rendered text against the page; quarantine uncertain quantities.
4. Normalize Unicode and parse only unambiguous units.
5. Create atomic claims and attach applicability.
6. Retrieve duplicate candidates and classify family relationships.
7. Adjudicate conflicting revisions; mark old facts as historical rather than mixing them.
8. Compute quality components and apply hard gates.
9. Construct family components and assign train or validation.
10. Render an SFT view, write immutable files, hash them, and produce reports.
11. Select an unrelated source/case family for final evaluation.
12. Rehearse source withdrawal and verify all descendants are discoverable.

## Publication and withdrawal gates

Block publication for checksum mismatch, mutable inputs, missing approvals, unresolved rights/privacy/safety issues,
critical contradictions, duplicate clusters crossing sealed splits, test-derived synthetic records, or incomplete
lineage. Before release, prove that deleting one source identifies and can rebuild every affected dataset descendant.

## Cross-links

- [Dataset guide](README.md)
- [Source and rights](source-discovery-and-rights.md)
- [Objective datasets](training-objective-datasets.md)
- [Reproducibility standard](../00-governance/reproducibility-standard.md)
