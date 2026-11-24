# CH-23: Finding Domain-Specific Training Data

**Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

> **Chapter status (2026-11-23): AUTHORED, STATICALLY REVIEWED, NOT LEGALLY APPROVED.** Official source entry points and
> selected API documentation were checked during authoring. No corpus was downloaded, no code listing was executed, and
> no source except the specifically quoted openFDA terms below is represented as cleared for model training. URLs,
> terms, licenses, robots rules, API limits, and regulations remain time-sensitive and must be rechecked on the
> acquisition date.

## 23.1 Learning objectives and scope

After this chapter, the reader should be able to:

1. convert a domain capability statement into a source-discovery map and coverage matrix;
2. distinguish public access, open licensing, public-domain status, contractual permission, and technical accessibility;
3. build a source register that preserves provenance, versions, restrictions, freshness, and safety authority;
4. apply rights, robots, terms-of-service, privacy, quality, duplication, and evaluation-separation gates before
   collection;
5. find complementary sources for automotive, motorcycle, heavy-truck, pharmacy, beekeeping, zookeeping, industrial,
   agricultural-equipment, and electronics specialists;
6. score source quality and risk without hiding veto conditions inside an average;
7. acquire only approved sources through reproducible, rate-limited workflows; and
8. keep source discovery and offline dataset construction consistent with the book's **no-runtime-RAG** architecture.

This chapter is about **finding and qualifying** sources. Extraction, optical character recognition (OCR),
normalization, personally identifiable information (PII) handling, document segmentation, and immutable dataset
production are developed in [CH-24](ch-24-dataset-collection-and-processing.md). Objective-specific record design and
split construction belong to [CH-25](ch-25-dataset-design-for-different-training-objectives.md); synthetic generation
belongs to [CH-26](ch-26-synthetic-data-and-knowledge-generation.md).

## 23.2 Why discovery is an engineering problem

The naïve plan—search the Web, download relevant text, and train—fails for four independent reasons:

- **Accessible is not reusable.** A document can be readable without permission to reproduce, transform, bulk-download,
  or use it for machine-learning training.
- **Relevant is not authoritative.** A highly ranked forum answer may be unsafe, obsolete, copied, or applicable to a
  different model year, jurisdiction, species, formulation, or voltage class.
- **Large is not complete.** Millions of generic repair posts can still omit pneumatic-brake inspection, pediatric
  contraindications, queen-rearing seasonality, or species-specific welfare.
- **Training data is not evaluation data.** If the same manual, recall notice, question, or paraphrase appears on both
  sides, the benchmark measures exposure rather than generalization.

A **source** is a versioned origin from which records may be derived: an API endpoint, dataset release, document
collection, repository, book edition, standards publication, expert engagement, or governed synthetic pipeline. **Source
discovery** is the controlled process that turns a capability requirement into candidate sources, evidence about those
sources, an acquisition decision, and a traceable snapshot. It is not merely URL collection.

The central design unit is:

> **capability × jurisdiction × time × asset variant × hazard × training objective**

For example, “brake knowledge” is too broad. “Explain, but do not authorize, an air-brake pre-trip inspection for a U.S.
Class 8 tractor, distinguishing current federal minimum rules from manufacturer procedure, with a qualified-technician
escalation path” is discoverable and reviewable.

## 23.3 Evidence-first discovery lifecycle

<a id="fig-ch23-22"></a> **FIG-CH23-22 — Dataset source discovery and qualification pipeline.** `ILLUSTRATIVE`;
companion to canonical FIG-22 in CH-24; no measured performance is implied. Alt text: requirements become search tracks,
candidates pass legal and technical gates, approved snapshots enter a source register and are partitioned into training
or sealed evaluation collections.

```text
Capability taxonomy + hazards + jurisdictions + dates
                         |
                         v
       Search tracks: official / academic / standards /
       open data / manuals / OER / experts / communities
                         |
                         v
              Candidate source record
                         |
          +--------------+---------------+
          |                              |
          v                              v
  Rights and policy gates        Evidence and quality review
  license / terms / robots       authority / provenance /
  API / privacy / contract       coverage / freshness / bias
          |                              |
          +--------------+---------------+
                         v
        REJECT | QUARANTINE | EVALUATION-ONLY | APPROVE
                         |
                         v
      Immutable raw snapshot + hashes + acquisition log
                         |
               +---------+---------+
               |                   |
               v                   v
        training family       sealed evaluation family
        deduplication barrier + source-family holdout
```

Approval is asymmetric: every gate must pass; one unresolved veto is enough to quarantine a source.

### 23.3.1 Start with a capability and hazard taxonomy

Build a hierarchical inventory before searching. Each leaf needs:

- intended user and permitted use;
- concepts, procedures, diagnostic tasks, and terminology;
- asset/species/drug scope and exclusions;
- jurisdiction, language, units, and date range;
- expected answer authority—education, screening, inspection, diagnosis, prescribing, repair authorization;
- credible ground-truth authority;
- consequence of a wrong answer;
- update trigger and acceptable staleness;
- data objective: continued pretraining (CPT), supervised fine-tuning (SFT), preference data, calibration, or
  evaluation.

Search both names and identifiers. Useful identifier families include OBD-II diagnostic trouble codes, NHTSA campaign
numbers, vehicle identification number (VIN) attributes, SAE/ISO standard numbers, Code of Federal Regulations sections,
RxNorm concept unique identifiers (RxCUIs), National Drug Codes (NDCs), Structured Product Labeling (SPL) Set IDs,
taxonomic names, equipment model and serial families, component part numbers, error codes, and Digital Object
Identifiers (DOIs).

### 23.3.2 Run complementary search tracks

One query engine should not determine the corpus. Search:

1. official regulator and government portals;
2. manufacturer, professional-association, and standards-body catalogs;
3. scholarly indexes, institutional repositories, and dataset registries;
4. open educational resource (OER) catalogs and libraries;
5. structured APIs and bulk-download catalogs;
6. professional forums and question-answer archives, for candidate discovery only until rights and quality clear;
7. expert organizations and commissioned-author routes;
8. existing corpus documentation and dataset cards;
9. archived editions and change histories; and
10. adjacent domains for shared foundations, while tagging the boundary.

Use domain synonyms and exclusions. Distinguish passenger-car hydraulic brakes from heavy-truck air brakes; an approved
indication from an off-label discussion; and managed-care husbandry from companion-animal advice.

### 23.3.3 Preserve negative discovery results

A missing authoritative source is a result, not an invitation to fill the gap with generated text. Record the query,
catalogs searched, date, filters, and why candidates failed. This supports targeted expert authoring and prevents
repeated unproductive searches.

## 23.4 Source classes: value, limits, and qualification

<a id="tbl-19"></a> **TBL-19 — Domain-source class decision matrix.** `STATICALLY REVIEWED`. This is a decision aid, not
a license opinion. Every row still requires a source-specific record.

<!-- markdownlint-disable MD013 -->
| Source class                         | Best contribution                                                     | Licensing and restrictions gate                                                                                                                            | Provenance and quality checks                                                                     | Coverage and freshness                                                              | Duplication and evaluation rule                                                                            |
| ------------------------------------ | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Public technical documentation       | terminology, concepts, interfaces, official procedures                | “Publicly viewable” is not a license; capture copyright notice, terms, download permission, trademark and redistribution limits                            | publisher identity, revision, applicability, supersession, citations                              | usually narrow and product/version specific; monitor revision feed or page metadata | group mirrors and translations by canonical document family; do not split across train/test                |
| Open datasets                        | structured observations, labels, identifiers, incidents               | verify dataset-level license, field-level exceptions, privacy, attribution/share-alike/noncommercial conditions                                            | data dictionary, collection method, missingness, sampling frame, known biases                     | release cadence and observation window must be distinct                             | split by entity/time/event, not random rows; reserve later releases where feasible                         |
| Research repositories                | papers, code, supplementary files, benchmark definitions              | paper, code, data, and model can have different licenses; repository license may not cover linked data                                                     | authorship, venue/status, commit or DOI, methods, retractions, repository-to-paper match          | often a frozen research snapshot                                                    | hold out named benchmark families and derivatives; detect copied benchmark records                         |
| Academic datasets                    | task labels, expert annotations, research protocols                   | read data-use agreement, consent scope, redistribution and commercial restrictions; ethics approval is not a reuse license                                 | annotation protocol, adjudication, inter-rater evidence, population and institution bias          | usually bounded by study period and geography                                       | never train on the benchmark used to claim performance; hold out institution/time where possible           |
| Technical manuals                    | precise procedures, specifications, warnings, diagrams                | commonly copyrighted or contract-gated; obtain written permission or a license covering extraction and training; owner possession is not corpus permission | exact make/model/serial/model-year, revision, service-bulletin supersession, safety prerequisites | deep but product-local; often stale after revisions                                 | hash page/text/image derivatives; keep same procedure and edition family in one split                      |
| Open educational materials           | explanations, worked examples, pedagogical sequence                   | verify exact Creative Commons version and element-level exceptions; NC, ND, and SA terms need project-specific review                                      | named author/institution, references, correction process, educational level                       | broad foundations; may lag operational changes                                      | deduplicate against textbooks, slides, and mirrors; assess on newly authored items                         |
| Official reference documents         | law, labels, recalls, nomenclature, classifications                   | government origin does not settle rights worldwide; inspect agency terms, third-party material, seals, and database terms                                  | issuing authority, effective date, amendment history, jurisdiction, legal status                  | high authority for defined scope; monitor effective/withdrawal dates                | separate versions; do not mix current and superseded text without temporal labels                          |
| Domain books with appropriate rights | coherent deep exposition and rare-topic coverage                      | use verified public-domain status, applicable open license, publisher/author agreement, or commissioned rights; buying a copy grants no training right     | edition, author expertise, references, errata, peer/editorial review                              | coherent but publication-lagged                                                     | detect scans, quotations, instructor banks, and later editions as one rights/duplication family            |
| Public technical discussion forums   | real terminology, edge cases, failure narratives, conversational form | check platform terms, author rights, API rules, deletion obligations, usernames/PII, robots; visibility is not consent                                     | reputation is weak evidence; require corroboration, spam filtering, date/locale/product tags      | broad and fresh but noisy and popularity-biased                                     | conversation-level split; cluster cross-posts and copied manuals; never use as sole safety ground truth    |
| Structured domain databases          | normalized entities and relations                                     | database rights, API terms, proprietary vocabulary fields, query limits and redistribution can differ from record copyright                                | schema, identifiers, constraints, update process, provenance per field                            | strong breadth and machine readability; semantics can change                        | version-aware entity split; keep snapshots and deprecated IDs; evaluation must not be direct lookup replay |
| Expert-authored explanations         | gap supplying, local practice, reviewed reasoning, uncertainty language | contributor agreement must state compensation, ownership/license, confidentiality, training/evaluation uses, withdrawal handling                           | credentials, conflict disclosure, independent review, source citations, reviewer identity         | targets rare/high-risk gaps; schedule recertification                               | author/template holdout; prevent style dominance; separately commission evaluation cases                   |
| Official safety guidance             | hazard controls, stop conditions, PPE, escalation                     | capture jurisdiction and legal status; standards incorporated by reference may remain copyrighted                                                          | regulator/recognized authority, effective date, hierarchy over informal advice                    | narrow but release-critical; changes trigger urgent review                          | never dilute through majority voting with forums; maintain dedicated safety evaluation                     |
| Open training corpora                | linguistic breadth and possible domain fragments                      | “open” may describe access or code, not every upstream work; audit composition, opt-outs, PII, takedown and license compatibility                          | corpus card, source inventory, filtering, dedup method, contamination reports                     | large but provenance may be coarse and snapshot age unclear                         | assume benchmark and web duplication risk until measured; fragments need source traceability               |
| Synthetic datasets                   | controlled scenarios, balance, format, counterfactuals                | respect grounding-source rights and teacher terms; generated output may carry contractual restrictions and factual risk                                    | generator/model/version/prompt, grounding citations, validators, acceptance/rejection logs        | fast to refresh, but cannot create authority                                        | cluster by prompt/template/fact; split underlying facts and documents before generation                    |
| Human-authored Q&A collections       | instruction format, misconception handling, conversational relevance  | contributor terms, privacy, workplace confidentiality, patient/customer data, platform rights                                                              | author credentials, answer citations, adjudication, date and locale                               | targeted but expensive; question distribution may be artificial                     | hold out authors, scenarios, source facts, and templates; writers must not see sealed tests                |
<!-- markdownlint-restore -->

### 23.4.1 Public domain, open license, and permission are different

Use an explicit evidence state; never use the ambiguous value `open`:

- `public_domain_verified`—the exact work and jurisdictional basis were checked;
- `licensed_verified`—the exact license text, version, licensor, scope, and obligations were captured;
- `permission_contract`—a signed agreement covers the planned uses;
- `restricted_evaluation_only`—use is approved only for controlled evaluation;
- `metadata_only`—only factual metadata or a citation may be retained;
- `unknown_quarantine`—no ingestion or model exposure;
- `rejected`—a gate failed.

An open API can serve restricted records; an open-source repository can contain third-party manuals; a Creative Commons
page can contain excluded images; a federal page can reproduce third-party submissions. Evaluate rights at **asset and
field level**, not only hostname level.

## 23.5 Mandatory decision gates

### Gate A — identity and provenance

Proceed only if the collector can identify the publisher/controller, canonical URL or persistent identifier, title,
version, publication/effective dates, acquisition path, and relationship to upstream sources. A mirror without canonical
provenance is a lead, not a training source.

### Gate B — rights

Ask:

1. Who owns or controls the text, data, images, annotations, and database arrangement?
2. What exact instrument permits copying, transformation, storage, model training, internal distribution, model release,
   and commercial use?
3. Does it contain attribution, notice, share-alike, noncommercial, no-derivatives, field-of-use, geographic, time, or
   downstream-use constraints?
4. Are there third-party components or user submissions?
5. Does permission cover weights, adapters, extracted datasets, examples, and the intended release model?

If a planned act is unclear, quarantine and obtain counsel or written permission. **Fair use/fair dealing is
jurisdiction- and fact-specific; this chapter does not prescribe it as a corpus strategy.**

### Gate C — terms, robots, and access controls

Terms and API contracts can impose conditions independent of copyright. Record the accepted terms version and
account/API plan. Archive `robots.txt` as a policy signal, but do not treat robots permission as a license or robots
denial as the only reason to stop. Never bypass authentication, CAPTCHAs, paywalls, rate limits, anti-bot controls,
geographic restrictions, or technical protection measures. Prefer an official bulk export or API.

### Gate D — privacy, confidentiality, and sensitive data

Reject or quarantine patient records, prescriptions linked to people, vehicle owner details, VINs linked to individuals,
employee records, exact locations of threatened species, facility-security layouts, customer tickets, unpublished
diagnostics, and employer-confidential procedures unless a lawful reviewed process authorizes them. Public availability
does not remove privacy, professional-secrecy, or safety concerns.

### Gate E — safety authority

Assign a contextual authority tier:

- **A1:** binding law, regulator-approved label, official recall, controlling safety rule;
- **A2:** current manufacturer procedure or recognized consensus standard applicable to the asset;
- **A3:** peer-reviewed evidence, professional guidance, extension/veterinary/clinical reference;
- **B:** qualified expert explanation grounded in A-tier evidence;
- **C:** community report, anecdote, unverified observation.

A regulator may define legal requirements but not a repair sequence; a manufacturer manual may define a sequence but not
independent comparative evidence. C-tier material can reveal questions and vocabulary but cannot establish a high-risk
answer.

### Gate F — technical acquisition

Confirm the permitted method, endpoint, pagination, authentication, call limits, retry policy, file sizes, checksums,
schema, and update mechanism. Estimate storage and review effort. Technical harvestability is not approval.

### Gate G — evaluation separation

Choose the disposition **before content acquisition**: training candidate, development-only, sealed evaluation, safety
challenge set, metadata/index only, or rejected. If a benchmark supports final claims, block its source family, mirrors,
answer explanations, translations, synthetic paraphrases, and upstream passages from training.

## 23.6 Source register

The source register is the acquisition control plane. Decisions are append-only; a new record supersedes rather than
silently rewrites an old decision.

**Listing CH23-01 — Source-register record. `UNVERIFIED IMPLEMENTATION`.** Format: YAML. Purpose: define minimum
governance metadata. Dependencies/hardware: none. Input: human and automated review. Output: one source decision record.
Expected failures: ambiguous rights, missing history, conflicting restrictions. Security: never store credentials or
sensitive raw records here.

```yaml
source_id: src-us-fda-openfda-drug-label-2026q4
canonical_url: https://open.fda.gov/apis/drug/label/
publisher: U.S. Food and Drug Administration
source_class: structured_domain_database
title: openFDA drug label endpoint
jurisdictions: [US]
languages: [en]
domain_tags: [pharmacy, labeling]
capability_tags: [indications, contraindications, warnings, dosage_forms]

rights:
  status: licensed_verified # example only; requires reviewer sign-off
  instrument_url: https://open.fda.gov/terms/
  instrument_version_or_captured_at: "2026-11-23T00:00:00Z"
  asset_scope: "endpoint records; inspect warnings and third-party fields"
  permits: [copy, transform, commercial_use]
  obligations: [retain_source_attribution_as_project_policy]
  exceptions: ["records marked as third-party/non-public-domain"]
  reviewer: null
  reviewed_at: null

access:
  method: api
  robots_url: https://open.fda.gov/robots.txt
  terms_url: https://open.fda.gov/terms/
  rate_limit_reference: https://open.fda.gov/apis/authentication/
  credentials_required: false
  approved_user_agent: "project-name/contact-email"

provenance:
  upstream_origin: "FDA public data; endpoint-specific provenance required"
  publication_date: null
  effective_date: null
  version_field: "meta.last_updated"
  acquisition_started_at: null
  raw_snapshot_uri: null
  raw_sha256: null
  request_log_uri: null
  schema_hash: null

quality:
  authority_tier: A1
  collection_method_reviewed: false
  known_limitations: ["not independently validated for clinical or production use"]
  geographic_bias: US
  temporal_coverage: null
  population_or_asset_coverage: "endpoint-defined"
  missingness_report_uri: null
  contradiction_notes: null

freshness:
  stated_cadence: null
  checked_at: "2026-11-23"
  next_check_at: null
  expiry_trigger: "meta.last_updated or FDA terms/schema change"

restrictions:
  pii_expected: false
  sensitive_fields: []
  attribution_text: "Data provided by the U.S. Food and Drug Administration"
  redistribution: "recheck exact asset"
  model_release: "awaiting_legal_review"
  deletion_or_takedown_process: null

disposition:
  objective: null # cpt | sft | calibration | evaluation
  split_family_id: family-fda-label
  evaluation_exclusion_families: [family-fda-label]
  status: quarantine
  approvers: { domain: null, legal: null, privacy: null, safety: null }
```

The sample reflects a checked fact: openFDA's terms say that unless otherwise noted its content is offered under CC0
1.0, while marked third-party material may not be public domain; the service also has limits and disclaims
clinical/production validation ([terms](https://open.fda.gov/terms/), [API overview](https://open.fda.gov/apis/)).
`DOC-VERIFIED` on 2026-11-23. This does **not** clear every FDA, DailyMed, or third-party asset.

## 23.7 Quality, coverage, and risk scoring

A score prioritizes review; it never overrides a veto.

$$
Q_s = 0.25A + 0.15P + 0.15C + 0.10F + 0.10X + 0.10V + 0.10S + 0.05L
$$

Each term is 0–5: authority $A$, provenance $P$, capability coverage $C$, freshness $F$, consistency $X$,
independent verifiability $V$, structural usability $S$, and language/locale fit $L$. A source with scores
$5,5,3,4,4,5,4,5$ has $Q_s=4.40$.

Risk remains separate:

$$
R_s = \max(R_{\text{rights}},R_{\text{privacy}},R_{\text{safety}},R_{\text{security}}) + 0.4R_{\text{stale}} +
0.3R_{\text{bias}} + 0.3R_{\text{dup}},
$$

with each component 0–5. The maximum preserves catastrophic dimensions. Suggested policy:

- unresolved rights, prohibited access, personal clinical data, confidential records, or dangerous security detail →
  **veto/quarantine**, regardless of $Q_s$;
- $Q_s \ge 4.0$, no component risk above 2 → priority review;
- $3.0 \le Q_s < 4.0$ → potentially useful with corrective controls;
- $Q_s < 3.0$ → discovery vocabulary, negative examples, or rejection;
- safety-critical claims require applicable A-tier evidence and qualified review.

**Supplemental quality/risk rubric.** `ILLUSTRATIVE`; thresholds are proposed project policy, not empirical findings.

<!-- markdownlint-disable MD013 -->
| Dimension        | 0                               | 3                                 | 5                                                                          |
| ---------------- | ------------------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| Authority        | anonymous/unaccountable         | known practitioner or institution | controlling regulator/label or applicable manufacturer/consensus authority |
| Provenance       | origin unknown                  | publisher and date known          | field-to-origin lineage, version, method, and hash preserved               |
| Coverage         | incidental mentions             | major common topics               | mapped common, rare, variant, and high-risk cells                          |
| Freshness        | undated/superseded              | within ordinary review window     | current effective version with monitored change feed                       |
| Rights risk      | permission contradicted/unknown | conditional and reviewable        | exact permission verified for all planned acts                             |
| Safety risk      | unsafe/unqualified              | educational with gaps             | hazards, stop conditions, prerequisites, and escalation reviewed           |
| Duplication risk | copied/mirrored origin unknown  | detectable family overlap         | canonical families identified and isolated                                 |
<!-- markdownlint-restore -->

Coverage is measured against the requirements matrix, not token count. Track approved independent source families,
A-tier presence, expert-reviewed examples, unresolved contradictions, current/superseded coverage, rare high-consequence
cases, language, and region. Twenty mirrors of one manual are one evidence family.

## 23.8 Duplication, contamination, and evaluation separation

Duplication control begins during discovery:

1. normalize canonical URLs and persistent identifiers;
2. hash original bytes and normalized text;
3. record edition, translation, mirror, citation, and derivation relationships;
4. apply exact/near-duplicate detection later in CH-24;
5. group documents by **source family** before splitting;
6. group synthetic records with source facts, generator prompt, and templates;
7. maintain a denylist of benchmark questions, answers, repositories, and mirrors.

Use multiple barriers:

- **source-family holdout:** editions, mirrors, translations, answer keys, and derivatives stay together;
- **entity holdout:** selected drugs, models, species/institutions, or fault families remain unseen where transfer is
  tested;
- **temporal holdout:** earlier snapshots train, later valid changes test;
- **author/institution holdout:** tests beyond one writing style/site;
- **template/fact holdout:** paraphrases of test facts remain excluded;
- **sealed access:** only the evaluation custodian sees final items.

Random row splitting is usually inadequate because documents, events, and templates generate correlated rows.

## 23.9 Permitted-source acquisition workflow

The workflow is: propose → register → legal/privacy/safety decision → metadata dry run → acquire → hash → validate
counts/schema → snapshot terms and logs → quarantine raw data → promote after review.

### 23.9.1 API acquisition with an approval manifest

**Listing CH23-02 — Allowlisted JSON API snapshotter. `UNVERIFIED IMPLEMENTATION`.** Python 3.12; dependencies:
`httpx==0.28.1`, `pydantic==2.12.4` (examples, not compatibility-tested). Hardware: workstation. Input:
reviewer-approved manifest and environment variables. Output: response, metadata, SHA-256. Expected failures: HTTP 429,
schema drift, incomplete pagination, changed terms, interruption. Security: secrets stay out of manifests/logs; this
code does not decide rights.

```python
from __future__ import annotations
import hashlib, json, os, time
from datetime import datetime, timezone
from pathlib import Path
from urllib.parse import urlparse
import httpx
from pydantic import BaseModel, HttpUrl

class ApprovedRequest(BaseModel):
    source_id: str
    url: HttpUrl
    allowed_hosts: list[str]
    rights_approved: bool
    terms_checked_at: str
    terms_url: HttpUrl
    params: dict[str, str | int] = {}
    output_dir: Path
    contact: str

def acquire(spec: ApprovedRequest) -> None:
    host = (urlparse(str(spec.url)).hostname or "").lower()
    if not spec.rights_approved:
        raise PermissionError("Source is not approved in the rights register")
    if host not in {h.lower() for h in spec.allowed_hosts}:
        raise PermissionError(f"Host {host!r} is not allowlisted")
    spec.output_dir.mkdir(parents=True, exist_ok=True)
    headers = {"Accept": "application/json",
               "User-Agent": f"domain-slm-dataset/0.1 ({spec.contact})"}
    if key := os.getenv("SOURCE_API_KEY"):
        headers["Authorization"] = f"Bearer {key}"
    with httpx.Client(timeout=60.0, follow_redirects=False) as client:
        response = client.get(str(spec.url), params=spec.params, headers=headers)
        if response.status_code == 429:
            raise RuntimeError("Rate limited; follow documented retry policy")
        response.raise_for_status()
        raw = response.content
    stamp = datetime.now(timezone.utc).strftime("%Y%m%dT%H%M%SZ")
    digest = hashlib.sha256(raw).hexdigest()
    (spec.output_dir / f"{spec.source_id}-{stamp}.json").write_bytes(raw)
    (spec.output_dir / f"{spec.source_id}-{stamp}.metadata.json").write_text(
        json.dumps({
            "source_id": spec.source_id,
            "requested_url": str(response.request.url),
            "retrieved_at": stamp,
            "status_code": response.status_code,
            "sha256": digest,
            "bytes": len(raw),
            "content_type": response.headers.get("content-type"),
            "etag": response.headers.get("etag"),
            "last_modified": response.headers.get("last-modified"),
            "terms_url": str(spec.terms_url),
            "terms_checked_at": spec.terms_checked_at,
            "redirects_followed": False,
        }, indent=2), encoding="utf-8")
    time.sleep(1.0)  # Replace with endpoint-specific reviewed policy.
```

Pagination must be endpoint-specific. Never assume `limit`, cursors, total counts, or retry headers are portable. Fail
closed when pages repeat, schema changes, or disclaimers change.

### 23.9.2 NHTSA recall acquisition example

NHTSA documents recalls and complaints APIs at [NHTSA Datasets and APIs](https://www.nhtsa.gov/nhtsa-datasets-and-apis);
the page states recall metadata frequency as daily and gives requests. `DOC-VERIFIED` on 2026-11-23. The API is
authoritative for reported recall records, not a repair manual. **Rights still require recorded agency-policy review.**

**Listing CH23-03 — Single NHTSA recall query. `UNVERIFIED IMPLEMENTATION`.** Python 3.12; `httpx==0.28.1`. Input:
make/model/year. Output: raw JSON and SHA-256. Expected failures: naming mismatch, no results, API drift. Safety: never
convert recall text into repair authorization.

```python
import hashlib
from pathlib import Path
import httpx

params = {"make": "Honda", "model": "Civic", "modelYear": 2020}
response = httpx.get(
    "https://api.nhtsa.gov/recalls/recallsByVehicle",
    params=params,
    headers={"User-Agent": "domain-slm-research/0.1 (contact@example.org)"},
    timeout=60.0,
    follow_redirects=False,
)
response.raise_for_status()
raw = response.content
Path("raw/nhtsa-recalls-2020-honda-civic.json").write_bytes(raw)
Path("raw/nhtsa-recalls-2020-honda-civic.sha256").write_text(
    hashlib.sha256(raw).hexdigest() + "\n", encoding="ascii"
)
```

### 23.9.3 Robots and terms preflight

This informs a human decision; it cannot approve crawling.

**Listing CH23-04 — Robots/policy preflight. `UNVERIFIED IMPLEMENTATION`.** Python 3.12 standard library. Input:
candidate URL, user agent, terms URL. Output: evidence bundle. Expected failures: absent robots, dynamic terms,
redirects, robots rules that do not express contract terms. Security: does not crawl content or bypass controls.

```python
from datetime import datetime, timezone
from pathlib import Path
from urllib.parse import urljoin
from urllib.request import Request, urlopen
from urllib.robotparser import RobotFileParser
import hashlib, json

TARGET = "https://example.org/manuals/"
TERMS = "https://example.org/terms"
USER_AGENT = "domain-slm-research/0.1"
OUT = Path("policy-evidence/example-org")
OUT.mkdir(parents=True, exist_ok=True)

robots_url = urljoin(TARGET, "/robots.txt")
rp = RobotFileParser()
rp.set_url(robots_url)
rp.read()
records = []
for kind, url in [("robots", robots_url), ("terms", TERMS)]:
    with urlopen(Request(url, headers={"User-Agent": USER_AGENT}), timeout=30) as r:
        body = r.read()
        records.append({"kind": kind, "url": r.geturl(), "status": r.status,
                        "sha256": hashlib.sha256(body).hexdigest(),
                        "retrieved_at": datetime.now(timezone.utc).isoformat()})
        (OUT / f"{kind}.bin").write_bytes(body)
(OUT / "decision-input.json").write_text(json.dumps({
    "target": TARGET,
    "user_agent": USER_AGENT,
    "robots_allows_target": rp.can_fetch(USER_AGENT, TARGET),
    "evidence": records,
    "human_decision_required": True,
    "warning": "robots permission is neither copyright nor ToS approval",
}, indent=2), encoding="utf-8")
```

### 23.9.4 Repository, document, and human workflows

For a repository, pin a commit, archive license/notices, enumerate submodules and large-file pointers, inspect
per-directory licenses, record upstream datasets, and scan history for secrets/PII. A root `LICENSE` does not cover
imported manuals or data.

For PDFs/books, register ISBN/DOI/report number, edition, publisher, rights basis, and element-level exceptions. Acquire
through the permitted channel; preserve bytes/checksum; record whether extraction, OCR, diagrams, tables,
redistribution, and model release are covered. Never remove digital rights management or access controls.

For experts, use signed terms covering topic/hazard scope, permitted references, confidentiality, ownership/license for
contributions and downstream artifacts, attribution, conflicts, credentials, independent review, revision/expiry,
withdrawal, and sealed-test isolation.

## 23.10 Domain discovery plans

The URLs below are **candidate entry points, not blanket approvals**. Each must pass the gates above on acquisition day.
“Authority” describes subject-matter role, not copyright status.

### 23.10.1 Automotive mechanics

**Capability map.** Combustion/electric powertrains; lubrication/cooling; fuel/emissions; OBD-II concepts and diagnostic
trouble codes; scan-data interpretation; 12/48 V and high-voltage electrical safety; starting/charging; networks;
steering/suspension; hydraulic braking/electronic stability; manual/automatic/CVT transmissions; service intervals;
recalls; safe diagnosis.

<!-- markdownlint-disable MD013 -->
| Track                            | Candidate authoritative examples                                                                                                                                                                                   | Coverage, freshness, restrictions                                                                                                           | Use and separation                                                                   |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Safety/recall                    | [NHTSA datasets and APIs](https://www.nhtsa.gov/nhtsa-datasets-and-apis), [investigations/recalls resources](https://www.nhtsa.gov/resources-investigations-recalls), [vPIC APIs](https://vpic.nhtsa.dot.gov/api/) | vehicle identity, complaints, campaigns; track API changes and manufacturer-supplied fields; complaints are reports, not verified diagnoses | use only after rights review; reserve campaigns by date/model family for evaluation  |
| Emissions/OBD regulation         | [EPA vehicle and engine certification overview](https://www.epa.gov/ve-certification/overview-certification-and-compliance-vehicles-and-engines), [eCFR Title 40](https://www.ecfr.gov/current/title-40)              | U.S. requirements; eCFR is continuously updated but not the official legal edition; incorporated standards may have separate rights         | teach regulation and temporal/jurisdiction labels; do not infer OEM repair procedure |
| Standards/identifiers            | [SAE standards catalog](https://www.sae.org/standards/)                                                                                                                                                            | OBD, diagnostic, and network standards may be paywalled/copyrighted; catalog metadata is not permission to ingest standards                 | license required; keep revision families together                                    |
| Manufacturer service information | OEM portals, owner manuals, bulletins, wiring/repair information                                                                                                                                                   | highest model specificity; often account-, contract-, region-, VIN-, or subscription-restricted; revisions/prerequisites critical           | acquire only with explicit training rights; hold out OEM/model/year families         |
| Education/research               | accredited programs, verified OER, university repositories                                                                                                                                                         | useful explanations may simplify or lag technology                                                                                          | verify exact asset license; expert-review procedures and safety                      |
| Field experience                 | permitted technician Q&A and commissioned case narratives                                                                                                                                                          | valuable symptom language; copying, unsafe-shortcut, regional, and survivorship bias                                                        | discover questions; ground answers in A-tier evidence; conversation-family split     |
<!-- markdownlint-restore -->

**Quality controls.** Tag make, model, model year, engine/motor, transmission, market, VIN applicability, tool
prerequisites, torque units, revision, and whether a step requires high-voltage depowering, lifting, restraint systems,
refrigerant handling, or road testing. Recall/complaint frequency is not failure probability without a denominator. VINs
can become personal data when linked to owners.

**Safety/evaluation.** Qualified technicians review high-voltage isolation, airbags/pretensioners, brakes, steering,
lifting, fuel, refrigerant, and torque-critical content. Tests must include “consult current OEM procedure,” wrong-model
traps, units, superseded bulletins, incomplete scan-code evidence, and refusal to equate a code with a confirmed failed
part.

### 23.10.2 Motorcycle mechanics

**Capability map.** Two-/four-stroke and single/multi-cylinder engines; ignition/charging; carburetion/electronic
injection; chain/belt/shaft drive; clutches/sequential transmissions; front/rear suspension; tires;
conventional/combined/ABS braking; chassis geometry; electric motorcycles; fault diagnosis.

**Discovery plan.**

- Start with [NHTSA datasets](https://www.nhtsa.gov/nhtsa-datasets-and-apis) for U.S. motorcycle recalls/complaints and
  [EPA annual certification data](https://www.epa.gov/compliance-and-fuel-economy-data/annual-certification-data-vehicles-engines-and-equipment)
  for applicable engine/vehicle records. Verify filters and rights.
- Search manufacturer portals by exact model, market, year, engine code, ABS variant, and revision. Service manuals,
  wiring, torque tables, and diagnostic software are typically controlled; obtain permission rather than scraping dealer
  systems.
- Search regulators/research catalogs for braking, helmet, tire, and crash evidence. Population statistics do not define
  repair procedure.
- Evaluate licensed vocational curricula and commissioned technicians for pedagogy.
- Use forums only for symptom language and rare-case leads. Modifications, racing, emissions tampering, and improvised
  repairs need exclusion or explicit risk labels.

Cross engine architecture, induction, cooling, ignition, final drive, brake technology, suspension, road/off-road use,
market, model year, and high voltage. Prioritize chain slack, axle alignment, tire direction/pressure, brake
contamination, steering-head adjustment, suspension preload assumptions, and unapproved modifications.

Group shared platforms and rebadged models. Hold out whole model families, not pages. Recalls/OEM changes trigger
review. Evaluation tests cross-model caution and escalation, not memorized torque values without current context.

### 23.10.3 Heavy-duty trucks and commercial vehicles

**Capability map.** Diesel/fuel injection/turbocharging; EGR/DPF/SCR/DEF aftertreatment; SAE J1939 concepts;
electric/alternative fuel; transmission/driveline; hydraulics; pneumatic brakes; suspension; preventive maintenance;
inspections; duty-cycle-aware diagnosis.

<!-- markdownlint-disable MD013 -->
| Track             | Candidate examples                                                                                                                                                                                                                                                                  | Critical qualification                                                                                                 |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Safety/operations | [FMCSA Open Data Program](https://www.fmcsa.dot.gov/registration/fmcsa-data-dissemination-program), [Large Truck and Bus Crash Facts](https://www.fmcsa.dot.gov/safety/data-and-statistics/large-truck-and-bus-crash-facts), [eCFR Title 49](https://www.ecfr.gov/current/title-49) | different refresh cycles, reporting, redactions, denominators, and legal meanings; violations are not repair diagnoses |
| Emissions         | [EPA heavy-duty supporting material](https://www.epa.gov/regulations-emissions-vehicles-and-engines/control-air-pollution-new-motor-vehicles-heavy-duty), [mobile-source research](https://www.epa.gov/moves/mobile-source-emission-factors-research)                               | testing/regulation is not an OEM troubleshooting tree; preserve year, engine class, duty cycle, assumptions            |
| Vehicle safety    | [NHTSA FARS](https://www.nhtsa.gov/research-data/fatality-analysis-reporting-system-fars), [NHTSA datasets](https://www.nhtsa.gov/nhtsa-datasets-and-apis)                                                                                                                          | crash data supports hazard prioritization, not causal repair claims                                                    |
| Standards         | [SAE J1939 catalog entry](https://www.sae.org/standards/content/j1939da_202510/) and SAE catalog                                                                                                                                                                                    | verify current revision and license; never use unofficial copies of paid standards                                     |
| OEM/component     | engine, transmission, brake, axle, aftertreatment, hydraulic, telematics, body-builder documents                                                                                                                                                                                    | truck configurations combine manufacturers; record serial/calibration/interfaces; portal terms may prohibit bulk use   |
<!-- markdownlint-restore -->

Model each truck as a graph: chassis/VIN, engine family/calibration, transmission, axles, brake controller,
aftertreatment, body/upfit, trailer, jurisdiction. Preserve suspect parameter number (SPN), failure mode identifier
(FMI), occurrence count, lamp state, and operating conditions where permitted; a code is not a parts instruction.

Pneumatic brakes, steering, wheel ends, lifting, pressure release, hydraulics, high voltage, alternative fuels,
emissions, and roadside criteria require specialist review. Never teach defeating safety/emissions systems. Hold out
fleet, component family, model year, and fault family; test energy isolation, tractor-trailer interactions, derate
misconceptions, and “insufficient configuration” behavior.

### 23.10.4 Pharmacy and pharmaceutical knowledge

**Intended-use limit.** The project may support education, terminology, label comprehension, and bounded informational
workflows. It is **not an autonomous prescriber, dispenser, interaction checker, diagnosis system, or substitute for a
pharmacist/clinician and current approved labeling**.

**Capability map.** Ingredients/products; classes; formulations/routes/strengths; indications; contraindications; boxed
warnings/precautions; adverse reactions; interactions; dosing concepts; renal/hepatic/pregnancy/lactation/age
considerations; storage; recalls/shortages; regulatory status; uncertainty/escalation.

<!-- markdownlint-disable MD013 -->
| Source layer          | Current authoritative entry point                                                                                                                                                                                                                                                  | Provenance, rights, freshness, and limits                                                                                                                               |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Machine-readable FDA  | [openFDA APIs](https://open.fda.gov/apis/), [drug label endpoint](https://open.fda.gov/apis/drug/label/), [terms](https://open.fda.gov/terms/)                                                                                                                                     | terms say CC0 unless noted and warn some third-party works are not public domain; metadata includes last-updated information; not validated for clinical/production use |
| Current SPL           | [DailyMed Web Services](https://dailymed.nlm.nih.gov/dailymed/app-support-web-services.cfm), [DailyMed](https://dailymed.nlm.nih.gov/dailymed/)                                                                                                                                    | API documents current SPL resources/version history; separately verify NLM terms and each payload                                                                       |
| Normalized vocabulary | [RxNorm API](https://lhncbc.nlm.nih.gov/RxNav/APIs/RxNormAPIs.html), [RxNav terms](https://lhncbc.nlm.nih.gov/RxNav/TermsofService.html)                                                                                                                                           | docs say most API calls need no license, with a proprietary-endpoint exception; distributions/source vocabularies may differ                                            |
| Approval/regulatory   | [Drugs@FDA](https://www.accessdata.fda.gov/scripts/cder/daf/), [Orange Book files](https://www.fda.gov/drugs/drug-approvals-and-databases/orange-book-data-files), [drug safety communications](https://www.fda.gov/drugs/drug-safety-and-availability/drug-safety-communications) | distinguish labeling, history, equivalence, and communications; preserve effective/withdrawal dates                                                                     |
| Clinical/research     | [PubMed](https://pubmed.ncbi.nlm.nih.gov/), regulator assessments, reviews, guidelines                                                                                                                                                                                             | abstract/full-text rights differ; one paper is not necessarily current care; record design/population/corrections                                                       |
| International         | [EMA medicines](https://www.ema.europa.eu/en/medicines), [WHO publications](https://www.who.int/publications), national regulators                                                                                                                                                 | never merge jurisdiction-specific labels/products without qualification; rights/languages vary                                                                          |
<!-- markdownlint-restore -->

Critical distinctions:

- brand, ingredient, dose form, package, and jurisdiction are not interchangeable;
- absence of an adverse-event report is not evidence of safety, and spontaneous reports do not directly give
  incidence/causality;
- preserve SPL Set ID, version, section, and effective date;
- interaction relevance can depend on dose, route, timing, patient factors, and evidence;
- terminologies normalize entities but do not prove recommendations.

Do not collect patient narratives, prescriptions, or case records without an approved clinical-data process establishing
lawful basis, authorization, de-identification, governance, security, and rights. De-identification does not grant
training rights. Never copy subscription databases without an explicit contract.

Monitor label/regulator changes on a risk basis; urgent communications trigger review. Store valid-from/to dates.
Offline weights can become stale; releases need cutoff, jurisdiction, and verification/escalation behavior.

Licensed pharmacists review labels, formulations, interactions/contraindications, and escalation;
physicians/toxicologists review relevant high-risk clinical areas; regulatory counsel reviews intended use. Seal a
safety set for contraindications, duplicate ingredients, pregnancy/pediatrics/geriatrics, renal/hepatic ambiguity,
route/strength confusion, obsolete products, emergencies, and refusal to prescribe. A confident fabricated dose or
contraindication is release-blocking.

### 23.10.5 Beekeeping and apiary management

**Capability map.** Honey-bee biology/behavior; equipment; inspections; queen/colony lifecycle; nutrition; swarming;
seasonal management; Varroa/other pests; brood disease; pesticides; sampling; overwintering; honey handling;
pollination; registration/movement rules.

Candidate tracks:

- [USDA APHIS National Honey Bee Surveys](https://www.aphis.usda.gov/plant-pests-diseases/honey-bee-surveys) links
  official survey protocols and surveillance goals. Search results showed the page modified 2025-07-07 during
  verification; that observation is not a license.
- [USDA NASS Bee and Honey](https://data.nass.usda.gov/Surveys/Guide_to_NASS_Surveys/Bee_and_Honey/index.php) and
  [Quick Stats](https://quickstats.nass.usda.gov/) support survey discovery; verify API key rules, sampling frames,
  revisions, and terms.
- [USDA National Agricultural Library beekeeping portal](https://www.nal.usda.gov/animal-health-and-welfare/beekeeping)
  links guides and data; each work needs its own rights record.
- [FAO TECA beekeeping resources](https://www.fao.org/teca/en/technologies/8410) and national agencies can add global
  practice; verify license, bee species, climate, locale, and pesticide regime.
- University extension and peer-reviewed research add regional management. Institutional authorship is not automatically
  an open license.
- State/provincial agriculture and pesticide authorities are required for local registration, movement, treatments, and
  notifiable disease.

Tag species/subspecies, scale, objective, climate, hemisphere, season/phenology, forage, pest pressure,
treatment/jurisdiction, sampling method, and operation size. Survey statistics retain denominator/method. Historical
treatments may be unapproved or ineffective.

Review pesticide/veterinary-product legality, residues, PPE, allergy/anaphylaxis escalation, smoker/fire, lifting, food
hygiene, and reportable disease. Never infer local approval from another country. Reviewers: experienced
beekeeper/apiary inspector and, for disease/treatments, apiculture scientist or veterinarian.

Hold out extension institution, climate region, season, disease event, and publication family. Test look-alike
conditions, insufficient evidence, resistance uncertainty, and regional escalation.

### 23.10.6 Zookeeping, animal care, and husbandry

**Capability map.** Taxonomy/natural history; species-specific housing; environment; nutrition; social grouping;
enrichment; behavior/training; welfare indicators; quarantine/biosecurity; reproduction; transport; recordkeeping;
keeper safety; veterinary escalation.

<!-- markdownlint-disable MD013 -->
| Track                        | Entry point                                                                                                                                                                              | Restrictions and quality concerns                                                                        |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Professional care            | [AZA Animal Care Manuals](https://www.aza.org/animal-care-manuals/), [AZA wellbeing](https://www.aza.org/animal-wellbeing-resources)                                                     | expert compilations; public/member access does not establish training rights; retain edition and caveats |
| Regional guidance            | [EAZA resources](https://www.eaza.net/resources/), [EAZA welfare](https://www.eaza.net/welfare-resources/)                                                                               | public and controlled resources differ; verify every document/translation; regional standards differ     |
| International welfare/health | [WOAH Terrestrial Code](https://www.woah.org/en/what-we-do/standards/codes-and-manuals/terrestrial-code-online-access/), [WAZA welfare](https://www.waza.org/priorities/animal-welfare/) | authority scope differs; verify edition, chapter, rights, and managed-care applicability                 |
| U.S. regulatory              | [USDA APHIS Animal Welfare](https://www.aphis.usda.gov/animal-welfare), [eCFR Title 9](https://www.ecfr.gov/current/title-9)                                                             | legal minima are not complete best practice; applicability/effective dates matter                        |
| Population records           | [WAZA studbooks](https://www.waza.org/priorities/conservation/international-studbooks/), [Species360](https://species360.org/)                                                           | records may be access-controlled, ownership-governed, sensitive, and unsuitable without agreement        |
| Science                      | [PubMed](https://pubmed.ncbi.nlm.nih.gov/), zoological journals/repositories                                                                                                             | often small populations; never universalize one institution/species result                               |
<!-- markdownlint-restore -->

“Big cat,” “primate,” or “bird” is insufficient. Tag scientific name, subspecies/population, age, sex,
reproductive/social/medical context, institution climate, enclosure context, and evidence level. Separate wild ecology
from managed-care recommendations.

Exclude exact threatened-animal locations, enclosure security details, dangerous-animal response vulnerabilities,
personal data, unpublished veterinary records, and transfer/trade records unless specifically authorized. Individual
records are not generic care guidance.

The model may educate and structure observations; it must not diagnose/treat, prescribe diets/medication, authorize
restraint/transport, or replace protocols/veterinarians. Veterinarians, taxon specialists, nutritionists, welfare
scientists, and keepers review respective areas.

Hold out species, institution, manual family, and assessment tool. Test taxonomy ambiguity, minimum standard versus best
practice, enrichment risk, zoonosis/biosecurity, abnormal behavior escalation, and missing veterinary context.

### 23.10.7 Agricultural equipment maintenance

Cover diesel/electric powertrains, hydraulics, power take-off (PTO), implements, steering/brakes, control buses,
precision agriculture, rollover protection, guarding, lockout, and storage.

Candidate authority begins with [OSHA agricultural standards](https://www.osha.gov/agricultural-operations/standards),
[29 CFR Part 1928](https://www.ecfr.gov/current/title-29/subtitle-B/chapter-XVII/part-1928),
[NIOSH agricultural safety](https://www.cdc.gov/niosh/agriculture/about/index.html), manufacturer/operator manuals,
extension programs, and ASABE/ISO/SAE catalogs. OSHA/eCFR defines U.S. legal scope, not model-specific procedure.
Standards/OEM portals often require licenses.

Matrix: tractor/power unit × implement × hitch/PTO × hydraulic class × control network × market/year/task. High-risk
review covers stored energy, raised implements, PTO entanglement, rollover protection, guarding, grain/confined spaces,
overhead electricity, chemical application, and autonomy. Hold out platform and implement families.

### 23.10.8 Industrial machinery and manufacturing

Cover drives, bearings/lubrication, pneumatics/hydraulics, electrical power, programmable logic controllers, sensors,
robotics, guarding, lockout/tagout, process control, condition monitoring, and records.

Candidate sources include [OSHA 1910](https://www.osha.gov/laws-regs/regulations/standardnumber/1910),
[NIOSH publications](https://www.cdc.gov/niosh/docs/), [NIST Manufacturing](https://www.nist.gov/manufacturing),
[CISA ICS advisories](https://www.cisa.gov/news-events/ics-advisories), OEM manuals, and IEC/ISO/NFPA catalogs. CISA
advisories are security evidence, not maintenance procedure. Standards require rights review. Plant procedures can
contain trade secrets, security architecture, worker data, and local hazards.

Qualify plant, machine model, controller/firmware, safety context, energy sources, and modification state. Never turn an
online PLC example into a safety-interlock change. Maintenance, electrical, controls, functional-safety, and
occupational-safety reviewers are required. Hold out plants/OEM families and test “site procedure required.”

### 23.10.9 Electronics and electrical repair

Cover circuits, components, schematics, instruments, soldering/rework, analog/digital/power electronics, fault
isolation, batteries, electrostatic discharge, mains/high voltage, RF exposure, and device repairability.

Candidate sources include [NIST publications](https://www.nist.gov/publications),
[OSHA electrical](https://www.osha.gov/electrical), [CPSC recalls](https://www.cpsc.gov/Recalls), manufacturer
datasheets/application notes/service documents, licensed OER, and IEC/IEEE/NFPA catalogs. Verify each manufacturer's
terms. Community repair guides may have page-level licenses and third-party media; public editability does not imply
commercial training permission.

Tag voltage/energy class, isolation category, chemistry, model/revision, measurement category, earth/reference topology,
firmware, and instrument rating. Review de-energization, stored charge, isolation, arc flash, damaged lithium cells,
lasers, RF/microwave, CRT/high voltage, and fire. Evaluation penalizes energized probing without prerequisites and holds
out device/schematic families.

## 23.11 Cross-domain mixture design

<!-- markdownlint-disable MD013 -->
| Shared family                       | Useful across                                  | Boundary to preserve                                                             |
| ----------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------- |
| electrical fundamentals/measurement | vehicle, agricultural, industrial, electronics | voltage/energy, grounding, measurement category, intrinsic safety, qualification |
| hydraulics                          | truck, agricultural, industrial                | pressure, fluid, accumulator, load support, OEM procedure                        |
| combustion/emissions                | vehicle domains                                | duty cycle, regulatory class, aftertreatment, fuel, year                         |
| diagnostic reasoning                | maintenance domains                            | evidence ordering transfers; values/procedures do not                            |
| biology/welfare                     | beekeeping, zookeeping                         | taxa, individual context, veterinary authority, region                           |
| structured identifiers              | pharmacy and technical domains                 | identifiers normalize entities; they do not prove advice                         |
<!-- markdownlint-restore -->

Store shared documents once with multiple tags. Measure mixture weights by unique source family and tokens so large
shared corpora do not erase small domains. Safety rules remain domain-specific.

## 23.12 Expert review as source and release control

Roles include domain lead, rights/privacy reviewer, safety authority, data curator, independent experts, evaluation
custodian, and adjudicator. Review 100% of high-risk records and all low-authority-derived records; use risk-weighted
samples for low-risk strata with stated thresholds.

Use labels such as `correct`, `correct_with_context`, `obsolete`, `wrong_jurisdiction`, `unsafe`, `unsupported`,
`rights_issue`, and `needs_adjudication`. Agreement is not truth when reviewers share a misconception.

Every safety-critical accepted record exposes:

1. claim and scope;
2. source version;
3. prerequisites and stop conditions;
4. uncertainty;
5. reviewer role/date;
6. adjudication history;
7. expiry/change trigger.

Experts must not silently repair source text while retaining its original authority label. A correction is a new
expert-authored derivative with new provenance.

## 23.13 No-runtime-RAG boundary and edge implications

Sources here are used **offline** to build/evaluate datasets. The primary deployed model does not fetch documents, query
vectors, select chunks, or call a teacher at inference. Approved text influences parameters/adapters through training;
evaluation references remain outside.

Consequences:

- training does not guarantee exact recall or citation;
- conflicting sources may be blended;
- weights do not expose supporting versions;
- labels, recalls, regulations, and procedures change;
- compression does not solve staleness;
- offline devices may miss urgent updates.

Controls are knowledge-cutoff/jurisdiction metadata, uncertainty/refusal training, conservative model-specific values,
frequent safety updates, signed releases, and instructions to verify current labels/recalls/manuals through an
authorized channel outside the primary response. A deterministic local calculator or identifier parser is not RAG if it
fetches no knowledge, but still needs validation.

Cleaner, nonduplicated data can reduce wasted training tokens; it does not prove a small/compressed model retains rare
facts. Test through sealed benchmarks after every optimization.

## 23.14 Worked example: deciding whether a brake document may enter training

Assume the capability matrix has an uncovered cell: “explain why an OBD-II code does not by itself authorize replacing
a brake-system component.” A search finds four apparently useful items:

1. a current regulator recall record available through a documented API;
2. an OEM service bulletin visible after accepting portal terms;
3. a forum post that reproduces most of that bulletin; and
4. a technician commissioned to write a model-independent explanation.

The learner should resist ranking these by technical detail alone. The recall record has strong provenance and may
establish that a defect campaign exists, but it is not a diagnostic procedure. The OEM bulletin may contain the exact
procedure, but visibility after login does not establish permission to copy it into training. The forum copy has weaker
provenance and cannot acquire rights that the poster did not have. The commissioned explanation can be used only after
the contributor agreement covers training, derivatives, confidentiality, and downstream release.

An acquisition review could produce these records:

```yaml
- source_id: recall-api-2026-0042
  role: authoritative_event_record
  rights_state: licensed_verified
  authority_tier: A1
  training_disposition: approved_after_asset_review
  limitation: "establishes recall scope, not repair diagnosis"
  split_family_id: brake-campaign-0042

- source_id: oem-bulletin-17b
  role: model_specific_procedure
  rights_state: unknown_quarantine
  authority_tier: A2
  training_disposition: prohibited_pending_written_permission
  limitation: "portal access is not a training license"
  split_family_id: brake-campaign-0042

- source_id: forum-copy-991
  role: discovery_lead
  rights_state: rejected
  authority_tier: C
  training_disposition: rejected
  limitation: "probable copied bulletin; origin and permission unresolved"
  split_family_id: brake-campaign-0042

- source_id: expert-note-2026-013
  role: bounded_explanation
  rights_state: permission_contract
  authority_tier: B
  training_disposition: approved_after_independent_technical_review
  limitation: "must not invent model-specific values or procedures"
  split_family_id: brake-campaign-0042
```

The shared `split_family_id` is deliberate. If the recall becomes evaluation material, neither the expert explanation
nor a synthetic paraphrase derived from the same campaign may enter training. Grouping by URL would miss this semantic
relationship.

**Bad decision:** “Use the forum post because it is public and more detailed.” This confuses access with permission,
copied detail with provenance, and popularity with authority.

**Better decision:** use the recall to define event scope, obtain explicit permission for the bulletin or omit it,
commission an independently reviewed explanation for the general concept, and reserve a different campaign or model
family for evaluation. The resulting corpus may be smaller, but every accepted statement has a defensible role.

This example also shows why a source register stores rejected candidates. Six months later, a new collector can see
that the forum copy was rejected because of provenance and rights, rather than rediscovering and ingesting it.

## 23.15 Practical project plan

1. Write intended/prohibited uses.
2. Define capability, hazard, jurisdiction, variant, and freshness cells.
3. Nominate evaluation custodian and reserve benchmark families.
4. Search official, structured, academic, OER, manual, expert, and community tracks.
5. Create source records before acquisition.
6. Capture rights, terms, API, robots, privacy, and safety evidence.
7. Apply vetoes, then score quality/risk.
8. Run metadata-only pilot and estimate review cost.
9. Approve a small diverse portfolio rather than merely the largest corpus.
10. Snapshot, hash, log, and quarantine.
11. Profile coverage, contradictions, freshness, and duplicates.
12. Commission experts for uncovered/high-risk cells.
13. Build training and sealed evaluation families independently.
14. Obtain domain, safety, rights, privacy, and release approvals.
15. Monitor changes and revoke superseded/withdrawn material.

Bandwidth is rarely the dominant cost. Rights analysis, expert review, versioning, and evaluation custody are. Pharmacy,
safety-critical repair, industrial controls, and animal care require multidisciplinary reviewers.

<a id="2315-exercises"></a>

## 23.16 Exercises

1. **Rights triage.** Register one readable OEM manual, one openFDA endpoint, one forum thread, and one Creative Commons
   course. Identify permitted, conditional, and unknown acts without downloading content.
2. **Coverage map.** Build a 40-cell automotive matrix spanning subsystem, year, jurisdiction, and hazard. Propose three
   independent source families for weak cells.
3. **Evaluation firewall.** Group manuals, mirrors, synthetic Q&A, and a benchmark into derivation families. Explain why
   random rows leak.
4. **Freshness drill.** Design change detection for DailyMed SPL, NHTSA recalls, and one apiary regulation, including
   urgent triggers and model-release response.
5. **Authority conflict.** Adjudicate forum versus OEM and old textbook versus current regulator without majority
   voting.
6. **Expert commissioning.** Draft a 20-record pneumatic-brake assignment covering rights, citations, confidentiality,
   review, and sealed-test isolation.
7. **Acquisition review.** Statically inspect CH23-02–04; add endpoint-specific pagination/rate handling for an approved
   API; report execution status honestly.
8. **Low-resource domain.** Document failed searches for one zoo species or regional beekeeping context and propose
   reviewed expert authoring instead of synthetic invention.
9. **Safety gate.** Define five release-blocking errors each for pharmacy, agricultural machinery, and electronics; name
   the reviewer profession.
10. **No-runtime-RAG audit.** Trace one source from acquisition through offline inference and identify accidental
    retrieval dependencies.

## 23.17 Summary and transition

Source discovery begins with a capability and hazard gap, not a search box. Public access, open licensing,
public-domain status, and contractual permission are different claims. Provenance explains where an asset came from,
which version it represents, and how descendants can be corrected or withdrawn. Authority determines what a source can
support: a recall can establish an event without establishing a repair procedure, and a forum can reveal vocabulary
without becoming safety ground truth. Rights, privacy, access, and safety are vetoes; quality scores only prioritize
the candidates that survive.

CH-24 now takes the approved and quarantined decisions as inputs. It preserves original bytes, extracts auditable
records, cleans without erasing meaning, groups duplicates and contradictions, and constructs leakage-resistant splits.

## 23.18 Honest status and unresolved limitations

- This chapter is `STATICALLY REVIEWED`, not legally approved.
- Listings CH23-01–04 are `UNVERIFIED IMPLEMENTATION`; none was run.
- Selected official pages were checked on 2026-11-23. No API, license, standard, regulation, or URL is guaranteed
  unchanged.
- Only the quoted openFDA CC0/exception statement was verified against an explicit reuse instrument during authoring.
  All other examples are **candidates requiring exact license/terms verification**.
- No corpus was acquired, profiled, deduplicated, or measured; quality thresholds and the diagram are `ILLUSTRATIVE`.
- No attorney or domain professional approved this chapter. Legal, pharmacist, veterinarian, technician, beekeeper,
  zookeeper, and safety reviews remain gates.
- URLs identify entry points but do not imply endorsement, completeness, training permission, or fitness.
- Other jurisdictions require local legal, regulatory, language, and practice research.
- The assessment and project ledgers were intentionally not edited because this task authorizes only this chapter.

## 23.19 References and primary entry points

Accessed 2026-11-23. These are discovery references; rights are per asset.

1. NHTSA. “NHTSA Datasets and APIs.”
   [https://www.nhtsa.gov/nhtsa-datasets-and-apis](https://www.nhtsa.gov/nhtsa-datasets-and-apis). `DOC-VERIFIED`.
2. U.S. FDA. “openFDA Terms of Service.” [https://open.fda.gov/terms/](https://open.fda.gov/terms/). `DOC-VERIFIED`.
3. U.S. FDA. “About the openFDA API.” [https://open.fda.gov/apis/](https://open.fda.gov/apis/). `DOC-VERIFIED`.
4. U.S. NLM. “DailyMed Web Services.”
   [https://dailymed.nlm.nih.gov/dailymed/app-support-web-services.cfm](https://dailymed.nlm.nih.gov/dailymed/app-support-web-services.cfm).
   `DOC-VERIFIED`.
5. U.S. NLM. “RxNorm API.”
   [https://lhncbc.nlm.nih.gov/RxNav/APIs/RxNormAPIs.html](https://lhncbc.nlm.nih.gov/RxNav/APIs/RxNormAPIs.html).
   `DOC-VERIFIED`.
6. USDA APHIS. “National Honey Bee Surveys.”
   [https://www.aphis.usda.gov/plant-pests-diseases/honey-bee-surveys](https://www.aphis.usda.gov/plant-pests-diseases/honey-bee-surveys).
   `DOC-VERIFIED`.
7. USDA NASS. “Honey Bee Surveys and Reports.”
   [https://data.nass.usda.gov/Surveys/Guide_to_NASS_Surveys/Bee_and_Honey/index.php](https://data.nass.usda.gov/Surveys/Guide_to_NASS_Surveys/Bee_and_Honey/index.php).
   `DOC-VERIFIED`.
8. FMCSA. “Open Data Program.”
   [https://www.fmcsa.dot.gov/registration/fmcsa-data-dissemination-program](https://www.fmcsa.dot.gov/registration/fmcsa-data-dissemination-program).
   `DOC-VERIFIED`.
9. EPA. “Heavy-Duty Engine and Vehicle Standards—Supporting Testing, Research and Modeling.”
   [https://www.epa.gov/regulations-emissions-vehicles-and-engines/control-air-pollution-new-motor-vehicles-heavy-duty](https://www.epa.gov/regulations-emissions-vehicles-and-engines/control-air-pollution-new-motor-vehicles-heavy-duty).
   `DOC-VERIFIED`.
10. AZA. “Animal Care Manuals.” [https://www.aza.org/animal-care-manuals/](https://www.aza.org/animal-care-manuals/).
    `DOC-VERIFIED` for description, not rights.
11. EAZA. “Resources” and “Animal Welfare Resources.”
    [https://www.eaza.net/resources/](https://www.eaza.net/resources/);
    [https://www.eaza.net/welfare-resources/](https://www.eaza.net/welfare-resources/). `DOC-VERIFIED` for descriptions,
    not rights.
12. OSHA. “Agricultural Operations—Standards” and “General Industry Standards.”
    [https://www.osha.gov/agricultural-operations/standards](https://www.osha.gov/agricultural-operations/standards);
    [https://www.osha.gov/laws-regs/regulations/standardnumber/1910](https://www.osha.gov/laws-regs/regulations/standardnumber/1910).
    `DOC-VERIFIED`.
13. U.S. GPO. Electronic Code of Federal Regulations. [https://www.ecfr.gov/](https://www.ecfr.gov/). `DOC-VERIFIED` for
    access; eCFR states it is not the official legal edition.

## Cross-links

- [Part index](README.md)
- [CH-24](ch-24-dataset-collection-and-processing.md)
- [CH-25](ch-25-dataset-design-for-different-training-objectives.md)
- [CH-26](ch-26-synthetic-data-and-knowledge-generation.md)
- [CH-23 assessment](../../13-assessments/ch-23-assessment.md)
- [Evidence standard](../../00-governance/evidence-and-citation-standard.md)
- [Safety standard](../../00-governance/safety-and-risk-standard.md)
- [No-runtime-RAG standard](../../00-governance/no-runtime-rag-standard.md)
