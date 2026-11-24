# CH-24: Dataset Collection and Processing

## Status and objectives

**AUTHORED / UNVERIFIED.** Every command, dependency, API, algorithm, and code listing below is
`UNVERIFIED IMPLEMENTATION`: none was executed, linted, benchmarked, or compatibility-tested for this chapter. URLs,
versions, and licenses require release-time verification. Diagrams and numerical examples are `ILLUSTRATIVE`; no
performance claim is made.

This chapter turns authorized sources from [CH-23](ch-23-finding-domain-specific-training-data.md) into immutable,
attributable dataset snapshots for the objective-specific builders in
[CH-25](ch-25-dataset-design-for-different-training-objectives.md). It is offline preparation, not runtime retrieval,
and complies with the [no-runtime-RAG standard](../../00-governance/no-runtime-rag-standard.md).

## 24.1 The contract: evidence before text

Downloading is not dataset engineering. A usable record must identify the received bytes, source owner, rights,
acquisition time, transformations, review decisions, and every published descendant. Preserve four distinct entities:

| Entity           | Identity                               | Example                       |
| ---------------- | -------------------------------------- | ----------------------------- |
| Source           | reviewed registry ID                   | publisher/API plus rights     |
| Artifact         | SHA-256 of original bytes              | one HTML response or PDF      |
| Document/segment | artifact + extraction recipe + locator | page, section, table row      |
| Example          | objective builder + parent IDs         | CPT span, SFT turn, test item |

A pipeline should be **idempotent**: the same immutable input, code, configuration, and declared tools produce the same
logical output. Network acquisition and some OCR/model operations are not inherently deterministic, so capture response
bytes, validators, tool/model versions, prompts, seeds where effective, and timestamps.

<a id="fig-22"></a> **FIG-22 — Dataset engineering pipeline (`ILLUSTRATIVE`).** Alt text: registered sources pass rights
and acquisition gates, immutable raw bytes are extracted and normalized, privacy, quality, and safety checks quarantine
failures, duplicate families are controlled, objective-specific datasets are split, registered, and connected to
takedown lineage. Not measured; arrows show conceptual relationships only.

```text
source registry + rights
        │
        ▼
API / permitted HTTP / upload ──► reject (policy, auth, MIME, size)
        │
        ▼
RAW bytes + receipt + SHA-256
        │
        ▼
HTML / PDF / OCR / table / image extraction ──► quarantine
        │
        ▼
Unicode + text + unit normalization
        │
        ▼
rights + PII + language + quality + safety gates
        │
        ▼
exact dedup → near-duplicate clusters → contradiction candidates
        │
        ▼
group-aware split → leakage audit
        │
        ▼
Parquet/JSONL snapshot + manifest + checksums + lineage
```

Raw storage is append-only. Corrections create a derived version; they never overwrite evidence.

## 24.2 Acquisition, APIs, and ethical scraping

The registry must record canonical URL, owner, acquisition method, jurisdiction, license/SPDX expression or custom
terms, allowed training/redistribution, attribution, robots decision, rate limit, retention, and reviewer. Public
accessibility is not permission. Unknown rights fail closed to quarantine.

Prefer, in order, an official bulk export, documented API, conditional HTTP fetch, and only then a permitted crawler.
API connectors need explicit pagination termination, response-schema validation, cursor checkpoints, bounded timeouts,
`Retry-After`, exponential backoff with jitter for `429/5xx`, and permanent handling of `400/401/403/404`. Store
API/version, parameters with secrets removed, response validators, and bytes. Credentials belong in a secret manager or
environment, never URLs, logs, or manifests.

Before crawling:

1. confirm terms, license, and intended training use; obtain permission where needed;
2. honor RFC 9309 robots rules, while recognizing robots permission is not copyright permission;
3. identify the crawler, bound rate/concurrency, cache, and use conditional requests;
4. never bypass authentication, paywalls, access controls, or anti-bot measures;
5. stop on changed terms, robots policy, or denial responses; and
6. retain source-ID/hash-based takedown capability.

Treat all payloads as hostile. Allowlist HTTPS hosts and paths; validate every redirect; reject loopback, link-local,
private-network and non-HTTP addresses; defend against DNS rebinding and server-side request forgery (SSRF). Bound
bytes, decompression ratio, redirects, archive depth, PDF pages, image pixels, parser time, and total work. Parse
without privilege, cloud credentials, or unrestricted egress. Scan malware and never execute macros, JavaScript,
embedded files, shell snippets, or downloaded models.

## 24.3 Extraction by medium

### HTML

Keep original bytes. Decode from BOM/HTTP/parser evidence and record encoding plus replacement-character count. Parse
the DOM, not regex. Remove scripts, styles, navigation, banners, and repeated chrome while retaining headings, lists,
code, warnings, captions, tables, and order. Preserve canonical/effective URL, author, publication/update date, language
hint, and CSS/XPath-like locators. Generic boilerplate removal can delete safety callouts; evaluate it on hand-labeled
domain pages.

### PDF and OCR

PDF is page description, not semantic structure. Classify each page as born-digital, scanned, or hybrid. For embedded
text, preserve page, block coordinates, reading order, font clues, and extraction warnings. Multi-column text,
ligatures, rotation, formulas, headers, and footnotes are common failures. Repair line-end hyphens only with contextual
rules.

For scanned pages, render in a sandbox, then apply orientation, deskew, denoise, and OCR. Record engine/version,
language pack, preprocessing, bounding boxes, and confidence. Confidence is not correctness: `0/O`, `1/l/I`, minus
signs, decimals, superscripts, and units are safety-critical errors. Hybrid pages require overlap reconciliation to
avoid duplicating an invisible OCR layer. Never infer hidden redacted text. Low-confidence quantities and safety tables
require human review.

### Tables, records, and images

Tables preserve caption, page/region, multi-level headers, row keys, cell raw value, parsed type, unit, spans, and
footnotes. Distinguish `not_reported`, `not_applicable`, and `redacted`; do not flatten all to null. Never coerce
identifiers (`00123`), ranges (`3–5`), inequalities (`<0.1`), or formulas into lossy numbers. Validate CSV/JSON/XML/API
schemas before normalization.

Images retain hash, source region, dimensions, color mode, caption/alt text, and EXIF policy. Remove GPS/device EXIF
from published derivatives, retaining protected audit data only when lawful. OCR extracts visible text. Vision models
may propose descriptions, but those are synthetic annotations requiring model/prompt provenance, evidence coordinates,
and qualified review. Do not infer dosage, torque, wiring, animal condition, or machine state without validated
task-specific review.

## 24.4 Unicode, cleaning, units, and metadata

Retain `text_raw`; produce NFC `text_nfc` for authoritative display and a separate NFKC/case-folded `text_match` for
matching. NFKC can alter mathematical, typographic, identifier, and unit distinctions, so never apply it globally to
authoritative text. Normalize line endings, remove disallowed controls, and surface bidirectional/zero-width characters
for security review. Preserve headings, warnings, negation, punctuation, code, and paragraph boundaries. Never
auto-correct part numbers, drug names, diagnostic codes, or measurements.

Unit normalization is parsing plus dimensional validation, not replacement. Store raw and canonical forms:

```json
{
  "raw": "35 psi",
  "value": 35,
  "unit_raw": "psi",
  "value_si": 241316.505,
  "unit_si": "Pa",
  "dimension": "pressure",
  "conversion_rule": "psi_to_Pa@v1"
}
```

For affine conversion \(y=ax+b\), \(x\) is source value, \(a\) scale, \(b\) offset, and \(y\) canonical value.
Temperature needs offsets; pressure generally does not. Preserve precision, locale, inequalities, ranges, and
uncertainty. Leave `ton`, `gal`, `oz`, decimal comma, and other ambiguities unresolved until context establishes
convention. Safety-critical conversions need tested definitions and expert review.

Metadata includes source/effective URL, owner, title/authors, publication/update/retrieval UTC dates, media type,
language, jurisdiction, license/terms snapshot, attribution, allowed uses, redistribution, hashes,
extractor/version/config, structural locator, parent IDs, quality signals, decisions, and takedown state. Unknown must
be explicit, never inferred from filenames.

## 24.5 Quality, language, PII, and filtering

Quality is a vector. Let \(q_i\in[0,1]\) represent provenance, rights clarity, extraction integrity, coherence,
relevance, attribution, and review confidence. A ranking score is

\[ Q=\prod_i\max(\epsilon,q_i)^{w_i},\quad \sum_iw_i=1. \]

For `0.9, 0.8, 0.7` with equal weights, \(Q\approx0.796\). The geometric mean penalizes weak components, but hard gates
override it: forbidden rights, malware, prohibited PII, failed safety review, or missing lineage always
reject/quarantine. Record every component, threshold, rule/model version, and reason. Calibrate scores against experts
and audit retention by source, language, demographic/geographic coverage, and risk so “cleaning” does not erase minority
material.

Language detection is uncertain for short, mixed, transliterated, and code-heavy text. Detect on blocks, retain
probability/margin and detector version, permit `und`, and review low-margin/unexpected cases. Do not reject tables
merely because cells are short.

PII handling is purpose-led: minimize collection; classify direct/quasi-identifiers, sensitive attributes, credentials
and secrets; detect with rules plus reviewed entity models; quarantine contextual cases; redact, aggregate,
pseudonymize, or delete under policy; encrypt and restrict access; define retention, takedown, and memorization tests.
Unsalted hashing remains linkable and dictionary-attackable. Stable pseudonyms should use keyed HMAC with separately
protected keys, but remain pseudonymous—not anonymous. Obtain jurisdiction-specific privacy/legal review.

Apply cheap hard gates first:

```text
rights/security/MIME/size → parse integrity → PII → language →
quality/relevance → exact dedup → near dedup → contradictions
```

Every result is `accept`, `reject`, `quarantine`, or `needs_review` with reason codes. Retries are bounded. Parser
crashes, unknown licenses, OCR/model outages, and empty extraction never become accepted empty records.

## 24.6 Duplicates, contradictions, and versions

SHA-256 raw bytes identify artifacts; SHA-256 of documented match text identifies equivalent extraction. Near duplicates
use word/character shingles. For sets \(A,B\),

\[ J(A,B)=\frac{|A\cap B|}{|A\cup B|}. \]

MinHash estimates Jaccard and locality-sensitive hashing (LSH) retrieves candidates without quadratic all-pairs
comparison; SimHash is an alternative. Candidate similarity is not truth. Cluster accepted edges, select a canonical
record by rights, authority, quality, recency, and completeness, and retain all attributions. Guard against transitive
chaining. Segment-level dedup avoids collapsing manuals that only share boilerplate. Deduplicate before splitting.

Contradiction detection operates on scoped claims: subject, predicate, value/unit, negation, equipment/drug/species,
jurisdiction, conditions, source authority, and effective interval. Retrieve same-subject/predicate candidates; compare
normalized quantities and dates; optionally use natural language inference (NLI) for review priority. NLI probability is
not factual adjudication. Conflicts may be revisions, regional rules, different conditions, or OCR errors. Preserve
both, and require experts for safety-critical conflicts; majority frequency must not overrule a current authoritative
source.

Version three layers independently: source release/ETag/content hash; pipeline Git commit + lock + config hash;
immutable dataset manifest ID. New revisions supersede rather than overwrite. A deletion ledger traverses lineage to
rebuild snapshots without a withdrawn source, preserving restricted audit evidence only when lawful.

## 24.7 Splits and leakage

Random rows leak correlated chunks. Assign an entire source family, document, edition/translation, duplicate cluster,
synthetic family, entity, author, or event to one split. A deterministic assignment uses

\[ u=\operatorname{int}(\operatorname{HMAC}_k(group)_{0:8})/2^{64}, \] <!-- markdownlint-disable-line MD013 MD049 -->

then maps \(u\) to configured intervals. Protect the key. Freeze test IDs before generation; never expose test content
to teachers or judges.

Audit exact/near matches, shared ancestors/clusters, copied answer strings, temporal cutoff violations, entity/event
overlap, synthetic/paraphrase descendants, and known benchmark contamination. Balance only after group constraints and
report actual distributions. Test data is access-controlled and never used for thresholds, prompts, or generator
validation.

## 24.8 Synthetic validation, storage, and lineage

Synthetic output is a derivative, not evidence. Record generator/checkpoint, tokenizer, prompt template, decoding, seed
where effective, timestamp, and parent source spans. Gate schema/roles; duplicate and test contamination; offline source
support; deterministic quantities/units/codes; answerability, ambiguity, uncertainty and safety; diversity/template
rates; and risk-stratified expert review. A model judge prioritizes review but cannot certify truth. Keep a synthetic
flag so mixture effects can be ablated.

Use original object bytes for evidence, JSONL for streaming/review, Parquet for typed columnar snapshots, Arrow IPC for
interchange, a transactional database for registry/decisions/edges, and WebDataset shards where multimodal sequential
training warrants their deletion cost. Avoid tiny Parquet files and high-cardinality partitions.

Each manifest records schema/objective, counts/bytes/distributions, split policy, thresholds and rejection reasons,
Git/config/lock/tool versions, every file path/size/SHA-256, parent snapshots, approvals, and limitations. Lineage is a
directed acyclic graph with `wasDerivedFrom`, `used`, `wasGeneratedBy`, `supersedes`, and `invalidates`; every row must
trace backward to bytes/rights and forward to snapshots.

## 24.9 Reusable Python project

All listings are `UNVERIFIED IMPLEMENTATION`.

```text
domain-data-pipeline/
├── pyproject.toml
├── configs/pipeline.yaml
├── src/domain_data/{models,transform,dedup_split,pipeline}.py
├── tests/test_pipeline.py
└── data/{raw/sha256,receipts,quarantine,validated,curated,manifests}/
```

**CH24-L01 — dependencies/configuration.**

```toml
# pyproject.toml — UNVERIFIED IMPLEMENTATION
[build-system]
requires = ["hatchling==1.27.0"]
build-backend = "hatchling.build"
[project]
name = "domain-data-pipeline"
version = "0.1.0"
requires-python = ">=3.11,<3.13"
dependencies = ["pydantic==2.10.6","PyYAML==6.0.2","pyarrow==18.1.0"]
[project.optional-dependencies]
dev = ["pytest==8.3.4","ruff==0.8.4","mypy==1.13.0"]
[project.scripts]
domain-data = "domain_data.pipeline:main"
```

```yaml
# configs/pipeline.yaml — UNVERIFIED IMPLEMENTATION
schema_version: 1
paths: { root: data }
quality: { minimum: 0.70 }
splits: { train: 0.80, validation: 0.10, test: 0.10 }
dedup: { shingle_words: 5, jaccard: 0.85 }
hard_gates: [rights, malware, pii, lineage, leakage, safety]
```

**CH24-L02 — canonical records.**

```python
# src/domain_data/models.py — UNVERIFIED IMPLEMENTATION
from datetime import datetime
from typing import Literal
from pydantic import BaseModel, Field

class Rights(BaseModel):
    license_expression: str | None
    training_allowed: bool
    redistribution_allowed: bool
    attribution: str
    reviewer: str

class Segment(BaseModel):
    segment_id: str
    source_id: str
    artifact_sha256: str
    locator: str
    text_raw: str
    text_nfc: str
    text_match: str
    language: str = "und"
    quality_components: dict[str, float]
    quality_score: float = Field(ge=0, le=1)
    leakage_group_id: str
    duplicate_cluster_id: str
    synthetic: bool = False
    parent_ids: list[str] = []
    decision: Literal["accept","reject","quarantine","needs_review"]
    reason_codes: list[str] = []
    created_at: datetime
```

**CH24-L03 — deterministic transforms.**

```python
# src/domain_data/transform.py — UNVERIFIED IMPLEMENTATION
import math, re, unicodedata
CONTROL = re.compile(r"[\x00-\x08\x0b\x0c\x0e-\x1f\x7f]")
SPACE = re.compile(r"[^\S\n]+")

def normalize(raw: str) -> tuple[str, str]:
    nfc = unicodedata.normalize("NFC", raw.replace("\r\n","\n").replace("\r","\n"))
    nfc = CONTROL.sub("", nfc)
    nfc = "\n".join(SPACE.sub(" ", x).strip() for x in nfc.splitlines()).strip()
    match = unicodedata.normalize("NFKC", nfc).casefold().replace("\u00a0"," ")
    return nfc, SPACE.sub(" ", match).strip()

def quality(q: dict[str,float], w: dict[str,float]) -> float:
    if q.keys() != w.keys() or not math.isclose(sum(w.values()), 1.0):
        raise ValueError("components and normalized weights must match")
    if any(not 0 <= x <= 1 for x in q.values()):
        raise ValueError("component outside [0,1]")
    return math.exp(sum(w[k] * math.log(max(1e-9, q[k])) for k in q))
```

**CH24-L04 — hashes, near duplicates, stable groups.**

```python
# src/domain_data/dedup_split.py — UNVERIFIED IMPLEMENTATION
import hashlib, hmac

def digest(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()

def shingles(text: str, n: int = 5) -> set[tuple[str,...]]:
    words = text.split()
    return {tuple(words[i:i+n]) for i in range(max(0, len(words)-n+1))}

def jaccard(a: set, b: set) -> float:
    return len(a & b) / len(a | b) if a | b else 1.0

def split(group: str, key: bytes, train=.8, validation=.1) -> str:
    raw = hmac.new(key, group.encode(), hashlib.sha256).digest()
    u = int.from_bytes(raw[:8], "big") / 2**64
    return "train" if u < train else "validation" if u < train+validation else "test"

def assert_no_cluster_leak(rows: list[dict]) -> None:
    seen = {}
    for row in rows:
        old = seen.setdefault(row["duplicate_cluster_id"], row["split"])
        if old != row["split"]:
            raise ValueError("duplicate cluster crosses splits")
```

**CH24-L05 — fail-closed immutable publication.**

```python
# src/domain_data/pipeline.py — UNVERIFIED IMPLEMENTATION
import argparse, hashlib, json, os
from pathlib import Path
import pyarrow as pa, pyarrow.parquet as pq

def file_hash(path: Path) -> str:
    h = hashlib.sha256()
    with path.open("rb") as f:
        for block in iter(lambda: f.read(1 << 20), b""): h.update(block)
    return h.hexdigest()

def publish(rows: list[dict], out: Path, metadata: dict) -> None:
    if not rows or any(r["decision"] != "accept" for r in rows):
        raise ValueError("only a nonempty accepted set may publish")
    out.mkdir(parents=True, exist_ok=False)
    files = []
    for name in ("train","validation","test"):
        target = out / f"split={name}" / "part-00000.parquet"
        target.parent.mkdir()
        selected = sorted((r for r in rows if r["split"] == name),
                          key=lambda r:r["segment_id"])
        pq.write_table(pa.Table.from_pylist(selected), target, compression="zstd")
        files.append({"path":target.relative_to(out).as_posix(),
                      "bytes":target.stat().st_size,"sha256":file_hash(target)})
    manifest = {**metadata,"rows":len(rows),"files":files}
    (out/"manifest.json").write_text(
        json.dumps(manifest,sort_keys=True,separators=(",",":")),encoding="utf-8")

def main() -> None:
    p=argparse.ArgumentParser()
    p.add_argument("--validated-jsonl",type=Path,required=True)
    p.add_argument("--output",type=Path,required=True)
    a=p.parse_args()
    commit=os.environ.get("PIPELINE_GIT_COMMIT")
    if not commit: raise RuntimeError("Git commit is required")
    rows=[json.loads(x) for x in a.validated_jsonl.read_text(encoding="utf-8").splitlines()]
    publish(rows,a.output,{"pipeline_version":commit,"status":"UNVERIFIED IMPLEMENTATION"})
```

**CH24-L06 — invariant tests and commands.**

```python
# tests/test_pipeline.py — UNVERIFIED IMPLEMENTATION
import pytest
from domain_data.transform import normalize, quality
from domain_data.dedup_split import assert_no_cluster_leak, split

def test_normalization_and_quality():
    assert normalize("Cafe\u0301\r\n35\u00a0psi") == ("Café\n35 psi","café\n35 psi")
    assert quality({"a":.9,"b":.8},{"a":.5,"b":.5}) == pytest.approx((.9*.8)**.5)

def test_group_stability_and_leakage():
    assert split("manual-7",b"test") == split("manual-7",b"test")
    with pytest.raises(ValueError):
        assert_no_cluster_leak([
          {"duplicate_cluster_id":"c1","split":"train"},
          {"duplicate_cluster_id":"c1","split":"test"}])
```

```bash
# UNVERIFIED IMPLEMENTATION
python -m venv .venv
python -m pip install -e ".[dev]"
ruff check src tests
mypy src
pytest -q
export PIPELINE_GIT_COMMIT="$(git rev-parse HEAD)"
domain-data --validated-jsonl data/validated/segments.jsonl \
  --output data/curated/snapshot=automotive-2026-11-23.1
```

Generate a target-specific lock with hashes before execution. The intentionally absent acquisition, HTML/PDF/OCR,
language, PII, MinHash/LSH, unit-registry, contradiction-review, and validated-assembly modules must be implemented and
reviewed before claiming an end-to-end pipeline; hiding omissions would be less reproducible than stating them.

## 24.10 Worked example: from two manuals to one leakage-safe claim family

This example follows a fictional, rights-approved training document so the transformations can be inspected without
presenting operational repair instructions. Suppose a 2024 HTML service note and a 2025 PDF revision contain:

```text
2024 HTML: Warning — isolate stored energy. Normal test pressure is 35 psi.
2025 PDF OCR: Warning - isolate stored energy. Normal test pressure is 36 psl.
```

The PDF OCR has confused `psi` with `psl`. A mirror also contains the 2024 sentence with navigation text. The correct
pipeline does not immediately choose the newest or majority value.

### Step 1 — preserve artifacts and receipts

```jsonl
{"artifact_id":"sha256:8b61f6f0e40f89f4f7dd11c4b9f0de4349db73fca0a34309b8b7469a9218674a","source_id":"src-approved-html","media_type":"text/html","retrieved_at":"2026-11-20T10:15:00Z","rights_basis_id":"permission-042","effective_date":"2024-03-01"}
{"artifact_id":"sha256:52766e43c802e174a2276a46980d3944d9f080f3e7b6ca71da92e2327b41196d","source_id":"src-approved-pdf","media_type":"application/pdf","retrieved_at":"2026-11-20T10:17:00Z","rights_basis_id":"permission-042","effective_date":"2025-02-14"}
```

Hashes identify received bytes, not truth. Both artifacts remain immutable even if one is later rejected.

### Step 2 — extract with locators and uncertainty

```jsonl
{"segment_id":"seg-2024-warning","artifact_id":"sha256:8b61f6f0e40f89f4f7dd11c4b9f0de4349db73fca0a34309b8b7469a9218674a","locator":"css:main section[data-id='warning']","text_raw":"Warning — isolate stored energy. Normal test pressure is 35 psi.","extraction_method":"dom-main-v2","warnings":[]}
{"segment_id":"seg-2025-warning","artifact_id":"sha256:52766e43c802e174a2276a46980d3944d9f080f3e7b6ca71da92e2327b41196d","locator":"page=7,bbox=112,204,903,330","text_raw":"Warning - isolate stored energy. Normal test pressure is 36 psl.","extraction_method":"ocr-example-v1","warnings":["low_confidence_token:psl","quantity_requires_review"]}
```

**Anti-example:** silently changing `psl` to `psi` because it “looks obvious.” That destroys evidence about the OCR
error and may turn a wrong number into authoritative-looking text. Keep raw text, propose a correction in a derived
field, and require review against the page image.

### Step 3 — normalize for display and matching

NFC normalization preserves the em dash in the authoritative display. A separate matching representation may
case-fold and standardize spacing:

```json
{"segment_id":"seg-2024-warning","text_nfc":"Warning — isolate stored energy. Normal test pressure is 35 psi.","text_match":"warning — isolate stored energy. normal test pressure is 35 psi."}
```

Do not delete `Warning`, negation, units, or sentence boundaries as “boilerplate.” Repeated safety text can be both
duplicated and essential; deduplication controls exposure without erasing its meaning.

### Step 4 — deduplicate at the right granularity

Assume the normalized 2024 sentence has word 3-shingles:

\[
A=\{\text{warning isolate stored},\text{isolate stored energy},\text{stored energy normal},
\text{energy normal test},\text{normal test pressure},\text{test pressure 35},
\text{pressure 35 psi}\}.
\]

The mirror has the same seven content shingles plus three navigation shingles. Its Jaccard similarity is
\(7/(7+3)=0.70\). If a calibrated candidate threshold is 0.65, it is reviewed as a near duplicate. The 2025 revision
has changed quantity tokens, so similarity alone must not collapse it into 2024; revisions remain linked but distinct.

**Anti-example:** hashing whole HTML pages only. Navigation changes defeat exact matching, leaving duplicated content in
the corpus. Segment-level shingles reveal the shared substantive passage.

### Step 5 — represent, do not vote away, the contradiction

```jsonl
{"claim_id":"claim-pressure-2024","subject":"fictional-system-test","predicate":"normal_pressure","value":35,"unit":"psi","effective_from":"2024-03-01","source_segment":"seg-2024-warning","status":"superseded_candidate"}
{"claim_id":"claim-pressure-2025","subject":"fictional-system-test","predicate":"normal_pressure","value":null,"unit":null,"effective_from":"2025-02-14","source_segment":"seg-2025-warning","status":"quarantine_ocr_quantity"}
```

The second claim deliberately has no normalized value until the crop is checked. If review confirms `36 psi`, an
expert must decide whether the difference is a revision, variant, or source error. If it confirms `35 psi`, the OCR
record is corrected as a new derivative. Two mirrors of 2024 do not outvote a valid 2025 revision, and “newer” does not
override an applicability mismatch.

### Step 6 — form a root group before splitting

Put the HTML, PDF revision, mirror, extracted segments, corrected OCR, and later synthetic Q&A in one root family:

```json
{"root_family_id":"family-fictional-pressure-procedure","members":["seg-2024-warning","seg-2025-warning","mirror-2024-warning"],"split":"train","assignment_basis":"HMAC(group)","derivatives_inherit_split":true}
```

If the family is assigned to train, none of its paraphrases may become validation or test. A separate independently
authored procedure family supplies evaluation.

### Step 7 — interpret the result

The training view may use the reviewed warning and only the version-scoped quantity that passed applicability review.
The dataset manifest reports one unique claim family, not three “examples.” This prevents duplicated evidence from
inflating coverage. The example also demonstrates the distinct jobs of:

- **cleaning:** make text processable while preserving meaning;
- **deduplication:** control repeated evidence and gradient exposure;
- **contradiction handling:** preserve scoped alternatives until adjudicated;
- **splitting:** estimate generalization across independent roots; and
- **lineage:** make correction, withdrawal, and audit possible.

## 24.11 Scale, failure, security, and safety

Downloads scale with bandwidth/quotas; OCR with pages/pixels/CPU; parsing with malformed inputs; exact dedup linearly
with bytes; naive near-dedup quadratically with segments; PII/NLI/vision with model compute; publication with schema
evolution and tiny files. Start with files plus SQLite when auditable; move to object storage, transactional catalogs,
queues, sandbox workers, and distributed columnar processing only when measurements justify them. Track CPU/GPU hours,
peak memory, bytes, failures, quarantine, and review load.

Release is blocked by unknown/incompatible rights, missing attribution, malware/parser findings, secrets/prohibited PII,
lineage gaps, checksum mismatch, cross-split duplicates, test-derived synthetic data, unsupported synthetic claims,
unresolved obsolete/current safety conflicts, or absent expert approval. Use least privilege, encryption, separated
raw/curated roles, tamper-evident logs, dependency scanning, signed/checksummed upstream artifacts where available,
differential update review, and rollback.

Licenses are record-level. “Open” is not a license. Preserve notices, attribution, custom terms, database rights,
commercial/share-alike restrictions, consent/privacy, and removal obligations in a dataset bill of materials. SPDX
improves notation but does not replace legal review.

SLMs have finite capacity: more noisy data can worsen useful specialization. Even curated parameters have imperfect
recall, stale facts, contradictions, hallucinations, and memorization risk. Pharmacy, braking, electrical, lifting,
industrial, and animal-welfare corpora require authoritative sources, qualified reviewers, intended-use limits,
escalation, and release gates; training never creates autonomous professional authority.

<a id="2411-troubleshooting-and-exercises"></a>

## 24.12 Troubleshooting and exercises

| Symptom                    | Cause                        | Safe response                                |
| -------------------------- | ---------------------------- | -------------------------------------------- |
| empty PDF text             | scan/encryption/font mapping | quarantine; OCR selected pages               |
| nonsensical order          | columns/positioned glyphs    | layout-aware extraction and review           |
| confident wrong OCR number | look-alike glyph             | inspect crop; quantity/unit review           |
| manual in every split      | split before grouping        | rebuild from source/duplicate groups         |
| giant near-dup cluster     | boilerplate/chaining         | remove boilerplate; representative threshold |
| illegal source passes      | quality used as gate         | fail closed on rights                        |
| table rejected as language | short/mixed cells            | retain `und`; use surrounding context        |
| implausible conversion     | locale/unit ambiguity        | preserve raw; reject and review              |
| old snapshot changes       | mutable path                 | publish a new immutable snapshot             |
| deletion cannot propagate  | absent lineage               | block release and rebuild                    |

Exercises:

1. Implement sandboxed hybrid-PDF extraction and explain overlap failure modes.
2. Design a table schema with multi-row headers, units, footnotes, and missing reasons.
3. Evaluate MinHash/LSH precision and recall on 500 expert-labeled pairs without tuning on test pairs.
4. Group editions, translations, quotations, and synthetic paraphrases for leakage-safe splitting.
5. Add temperature/torque conversion with locale, dimension, precision, and property tests.
6. Threat-model SSRF, DNS rebinding, decompression bombs, malicious PDFs, and credential leakage; test every enforced
   bound.
7. Create risk-stratified synthetic review for pharmacy, including mandatory expert gates.
8. Model two conflicting torque specifications as scoped, versioned claims and adjudicate them.
9. Prove every Parquet row traces to raw bytes, rights, code/config, and split group.

## 24.13 Summary and transition

Collection converts an approved source decision into immutable bytes and a receipt. Extraction creates located,
versioned segments rather than anonymous text. Cleaning must preserve safety language, identifiers, units, and raw
forms. Exact and near deduplication identify repeated evidence; they do not decide factual authority. Contradiction
detection creates review candidates scoped by variant, jurisdiction, condition, and time. Group-aware splitting keeps
all descendants of one evidence family on one side of the evaluation boundary.

CH-25 begins after these steps. It asks a different question: which view of the governed records is fit for CPT, SFT,
diagnostic training, preference optimization, calibration, or sealed evaluation? A clean segment is not automatically
a correct target for any of them.

## 24.14 References

1. Koster et al. (2022), _RFC 9309: Robots Exclusion Protocol_. <https://www.rfc-editor.org/rfc/rfc9309> <!-- markdownlint-disable-line MD013 MD049 -->
2. Fielding, Nottingham, Reschke (2022), _RFC 9110: HTTP Semantics_. <https://www.rfc-editor.org/rfc/rfc9110> <!-- markdownlint-disable-line MD013 MD049 -->
3. Davis and Whistler, _Unicode Standard Annex #15_. <https://unicode.org/reports/tr15/> <!-- markdownlint-disable-line MD013 MD049 -->
4. Unicode Consortium, _UTS #39: Unicode Security Mechanisms_. <https://unicode.org/reports/tr39/> <!-- markdownlint-disable-line MD013 MD049 -->
5. ISO, _ISO 32000-1:2008, PDF 1.7_. <https://www.iso.org/standard/51502.html> <!-- markdownlint-disable-line MD013 MD049 -->
6. Smith (2007), “An Overview of the Tesseract OCR Engine,” ICDAR. DOI: 10.1109/ICDAR.2007.4376991.
7. Broder (1997), “On the Resemblance and Containment of Documents.” DOI: 10.1109/SEQUEN.1997.666900.
8. Charikar (2002), “Similarity Estimation Techniques from Rounding Algorithms.” DOI: 10.1145/509907.509965.
9. Apache Parquet format. <https://parquet.apache.org/docs/file-format/>
10. W3C (2013), _PROV-DM_. <https://www.w3.org/TR/prov-dm/> <!-- markdownlint-disable-line MD013 MD049 -->
11. Gebru et al. (2021), “Datasheets for Datasets.” DOI: 10.1145/3458723.
12. Bender and Friedman (2018), “Data Statements for NLP.” DOI: 10.1162/tacl_a_00041.
13. Pushkarna, Zaldivar, Kjartansson (2022), “Data Cards.” DOI: 10.1145/3531146.3533231.
14. NIST (2010), _SP 800-122: Protecting PII_. DOI: 10.6028/NIST.SP.800-122. <!-- markdownlint-disable-line MD013 MD049 -->
15. SPDX specification. <https://spdx.github.io/spdx-spec/>
16. OWASP, _SSRF Prevention Cheat Sheet_. <!-- markdownlint-disable-line MD013 MD049 -->
    <https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html>

## 24.15 Honest completion record

The chapter covers every prompt topic, but it is not executed or production-ready. Required follow-up is to verify
references and dependency licenses/APIs; lock and scan environments; implement omitted production stages; test
adversarial fixtures; calibrate quality/dedup thresholds on labeled domain data; perform legal/privacy/security/safety
review; validate links and identifiers; and update the assessment/audit ledgers in a separately authorized change. Until
then, status remains **AUTHORED / UNVERIFIED** with no claim of execution.
