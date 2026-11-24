# Data pipeline implementation

## Concept, purpose, prerequisites, and outputs

Dataset engineering converts legally usable source artifacts into immutable objective-specific records. The pipeline preserves provenance because normalized text alone cannot answer who owned the source, which terms applied, where a statement appeared, or whether related examples leaked across splits.

**Prerequisites:** an approved source registry, a reviewer roster, isolated parsing tools, storage for immutable raw bytes, and a secret split key. **Inputs:** source records, acquired bytes, extraction locators, review decisions, and objective metadata. **Outputs:** receipts, quarantines, normalized rows, duplicate clusters, root-family splits, JSONL/Parquet snapshots, manifests, and checksums.

### Worked example

A reviewed automotive case `case-auto-001` yields one accepted diagnostic dialogue. The builder normalizes Unicode, computes a content SHA-256, assigns every descendant of that root to one deterministic split, writes the corresponding Parquet partition, and records file size and digest in `manifest.json`. Expected behavior is idempotent logical output from identical inputs. Exact validation compares two clean builds, traces the row back to raw bytes and rights terms, proves no family crosses splits, and rejects a deliberately duplicated row and an unknown-rights fixture.

## Contract

> Verification note: commands and listings require execution in the stated target environment before operational use.

Raw bytes are append-only. A publishable row must trace to a rights-reviewed source, original artifact hash, extraction
locator, transformation chain, review decision, split group, and immutable snapshot. Unknown rights, parser/security
findings, prohibited personally identifiable information (PII), missing lineage, or evaluation leakage fail closed.

## Pipeline and storage

```text
approved source registry
  → bounded acquisition + receipt + SHA-256
  → sandboxed HTML/PDF/OCR/table extraction
  → Unicode/text/unit normalization
  → rights/security/PII/language/quality gates
  → exact + near duplicate clusters
  → scoped contradiction candidates + expert adjudication
  → root-family split before chunking/synthesis
  → objective views: CPT/SFT/preference/KD/calibration/evaluation
  → schema + leakage + distribution validation
  → immutable JSONL/Parquet snapshot + manifest + checksums
```

**Listing DATA-L01 — extraction path `configs/data.yaml`.**

```yaml
schema: slm-data/v1
status: example_configuration
root: data
limits:
  download_bytes: 50000000
  redirects: 0
  pdf_pages: 1000
  image_pixels: 50000000
normalization: { unicode_display: NFC, unicode_matching: NFKC_CASEFOLD }
dedup: { word_shingle_size: 5, candidate_jaccard: 0.85 }
splits: { train: 0.80, validation: 0.10, test: 0.10 }
hard_gates: [rights, malware, pii, lineage, safety, leakage]
objectives:
  - cpt
  - sft
  - preference
  - distillation_response
  - quantization_calibration
  - pruning_calibration
  - evaluation
```

## Source and record schemas

**Listing DATA-L02 — extraction path `schemas/source.schema.json`.**

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "required": ["source_id", "canonical_url", "owner", "rights", "acquisition", "review"],
  "properties": {
    "source_id": { "type": "string", "pattern": "^[a-z0-9][a-z0-9._-]+$" },
    "canonical_url": { "type": "string", "pattern": "^https://" },
    "owner": { "type": "string", "minLength": 1 },
    "rights": {
      "type": "object",
      "additionalProperties": false,
      "required": ["basis", "training_allowed", "redistribution_allowed", "terms_sha256"],
      "properties": {
        "basis": { "type": "string" },
        "training_allowed": { "type": "boolean" },
        "redistribution_allowed": { "type": "boolean" },
        "terms_sha256": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
        "attribution": { "type": "string" }
      }
    },
    "acquisition": { "enum": ["manual_upload", "documented_api", "reviewed_https"] },
    "jurisdiction": { "type": ["string", "null"] },
    "review": {
      "type": "object",
      "required": ["status", "reviewer"],
      "properties": { "status": { "const": "approved" }, "reviewer": { "type": "string" } }
    }
  }
}
```

**Listing DATA-L03 — extraction path `schemas/sft.schema.json`.**

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "required": ["id", "domain", "task", "root_family_id", "risk_tier", "source_ids", "messages", "review_status"],
  "properties": {
    "id": { "type": "string", "minLength": 1 },
    "domain": { "type": "string", "minLength": 1 },
    "task": { "type": "string", "minLength": 1 },
    "root_family_id": { "type": "string", "minLength": 1 },
    "risk_tier": { "enum": ["low", "moderate", "high", "critical"] },
    "source_ids": { "type": "array", "minItems": 1, "uniqueItems": true, "items": { "type": "string" } },
    "messages": {
      "type": "array",
      "minItems": 2,
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["role", "content"],
        "properties": {
          "role": { "enum": ["system", "user", "assistant"] },
          "content": { "type": "string", "minLength": 1 }
        }
      }
    },
    "review_status": { "enum": ["expert_accepted", "editorial_accepted"] }
  }
}
```

Critical records require `expert_accepted`; the generic schema cannot verify reviewer authorization, so the registry
validator must.

## Rights-first acquisition

Prefer official bulk export, then documented API, then reviewed conditional HTTPS. Never bypass authentication,
paywalls, robots controls, rate limits, or anti-bot systems. Robots permission is not copyright permission. Treat every
payload as hostile: allowlist scheme/host/path, validate resolved addresses, cap bytes and decompression, reject
private/link-local targets, scan malware, and parse without credentials or unrestricted egress.

**Listing DATA-L04 — extraction path `src/slm_impl/data/acquire.py`.**

```python
from __future__ import annotations
import argparse, hashlib, ipaddress, json, socket, urllib.parse, urllib.request
from datetime import datetime, timezone
from pathlib import Path
import yaml

MAX_BYTES = 50_000_000

def sha256(data: bytes) -> str:
    return hashlib.sha256(data).hexdigest()

def validate_url(url: str) -> None:
    parsed = urllib.parse.urlsplit(url)
    if parsed.scheme != "https" or not parsed.hostname or parsed.username:
        raise ValueError("only credential-free HTTPS URLs are allowed")
    for info in socket.getaddrinfo(parsed.hostname, 443, type=socket.SOCK_STREAM):
        ip = ipaddress.ip_address(info[4][0])
        if not ip.is_global:
            raise ValueError(f"non-global destination rejected: {ip}")

class NoRedirect(urllib.request.HTTPRedirectHandler):
    def redirect_request(self, *args, **kwargs):
        raise urllib.error.HTTPError(args[0], args[1], "redirect blocked", {}, None)

def main() -> None:
    p=argparse.ArgumentParser()
    p.add_argument("--registry",type=Path,required=True)
    p.add_argument("--root",type=Path,default=Path("data"))
    a=p.parse_args()
    sources=yaml.safe_load(a.registry.read_text("utf-8"))["sources"]
    opener=urllib.request.build_opener(NoRedirect)
    for source in sources:
        if source["review"]["status"] != "approved" or \
           not source["rights"]["training_allowed"]:
            raise RuntimeError(f"{source['source_id']}: rights gate failed")
        url=source["canonical_url"]; validate_url(url)
        req=urllib.request.Request(url,headers={
            "User-Agent":"GovernedSLMData/0.1 (handbook-operator@example.invalid)"})
        with opener.open(req,timeout=30) as response:
            data=response.read(MAX_BYTES+1)
            if len(data)>MAX_BYTES: raise RuntimeError("size limit exceeded")
            if response.url != url: raise RuntimeError("effective URL changed")
            media=response.headers.get_content_type()
        digest=sha256(data)
        raw=a.root/"raw"/"sha256"/digest[:2]/digest
        raw.parent.mkdir(parents=True,exist_ok=True)
        if raw.exists() and raw.read_bytes()!=data: raise RuntimeError("hash collision")
        raw.write_bytes(data)
        receipt={"source_id":source["source_id"],"sha256":digest,
                 "bytes":len(data),"media_type":media,
                 "retrieved_utc":datetime.now(timezone.utc).isoformat(),
                 "rights_basis":source["rights"]["basis"]}
        out=a.root/"receipts"/f"{digest}.json"; out.parent.mkdir(parents=True,exist_ok=True)
        out.write_text(json.dumps(receipt,indent=2,sort_keys=True),"utf-8")
if __name__=="__main__": main()
```

This restricted connector intentionally omits APIs, retries, caching, and redirects. Add each through a separately
reviewed connector. A production network sandbox must also defend against DNS rebinding between validation and
connection.

## Extraction rules

- **HTML:** parse Document Object Model structure; remove scripts/navigation while retaining warnings, headings, tables,
  captions, code, and locator metadata.
- **PDF:** classify pages as born-digital, scanned, or hybrid; retain page/block coordinates and reading-order warnings.
  Never silently accept empty or scrambled text.
- **OCR:** record engine/version/language/preprocessing/confidence and retain page crops for review. Digits, decimal
  points, minus signs, units, part numbers, doses, and torque values need high-risk review.
- **Tables:** preserve multi-row headers, units, footnotes, row keys, raw cells, spans, and missing-value reason. Do not
  flatten `not reported`, `not applicable`, and `redacted`.
- **Images:** OCR visible text; a vision-generated description is synthetic and needs evidence coordinates and expert
  review.

Parser, OCR, language-model, and contradiction-detector output is untrusted. A model score prioritizes review; it never
certifies rights or facts.

## Normalization, deduplication, split, publication

**Listing DATA-L05 — extraction path `src/slm_impl/data/build.py`.**

```python
from __future__ import annotations
import argparse, hashlib, hmac, json, re, unicodedata
from collections import defaultdict
from pathlib import Path
import pyarrow as pa, pyarrow.parquet as pq

CONTROL=re.compile(r"[\x00-\x08\x0b\x0c\x0e-\x1f\x7f]")
SPACE=re.compile(r"[^\S\n]+")

def normalize(raw: str) -> tuple[str,str]:
    display=unicodedata.normalize("NFC",raw.replace("\r\n","\n").replace("\r","\n"))
    display=CONTROL.sub("",display)
    display="\n".join(SPACE.sub(" ",x).strip() for x in display.splitlines()).strip()
    match=unicodedata.normalize("NFKC",display).casefold().replace("\u00a0"," ")
    return display,SPACE.sub(" ",match).strip()

def stable_split(group: str,key: bytes) -> str:
    u=int.from_bytes(hmac.new(key,group.encode(),hashlib.sha256).digest()[:8],"big")/2**64
    return "train" if u<.8 else "validation" if u<.9 else "test"

def file_hash(path: Path) -> str:
    h=hashlib.sha256()
    with path.open("rb") as f:
        for block in iter(lambda:f.read(1<<20),b""): h.update(block)
    return h.hexdigest()

def main() -> None:
    p=argparse.ArgumentParser()
    p.add_argument("--validated-jsonl",type=Path,required=True)
    p.add_argument("--split-key",type=Path,required=True)
    p.add_argument("--output",type=Path,required=True)
    p.add_argument("--pipeline-commit",required=True)
    a=p.parse_args(); key=a.split_key.read_bytes()
    if len(key)<32: raise ValueError("split key must contain at least 256 bits")
    rows=[]; ids=set(); roots=defaultdict(set); hashes=defaultdict(list)
    for number,line in enumerate(a.validated_jsonl.read_text("utf-8").splitlines(),1):
        row=json.loads(line)
        if row["id"] in ids: raise ValueError(f"duplicate id line {number}")
        ids.add(row["id"])
        if row["decision"]!="accept": raise ValueError(f"unaccepted {row['id']}")
        row["text_nfc"],row["text_match"]=normalize(row["text_raw"])
        row["content_sha256"]=hashlib.sha256(row["text_match"].encode()).hexdigest()
        row["split"]=stable_split(row["root_family_id"],key)
        roots[row["root_family_id"]].add(row["split"])
        hashes[row["content_sha256"]].append(row["id"]); rows.append(row)
    if any(len(v)>1 for v in roots.values()): raise RuntimeError("root crosses splits")
    exact={k:v for k,v in hashes.items() if len(v)>1}
    if exact: raise RuntimeError(f"exact duplicates remain: {exact}")
    a.output.mkdir(parents=True,exist_ok=False); files=[]
    for split in ("train","validation","test"):
        target=a.output/f"split={split}"/"part-00000.parquet"; target.parent.mkdir()
        selected=sorted((r for r in rows if r["split"]==split),key=lambda r:r["id"])
        pq.write_table(pa.Table.from_pylist(selected),target,compression="zstd")
        files.append({"path":target.relative_to(a.output).as_posix(),
                      "bytes":target.stat().st_size,"sha256":file_hash(target)})
    manifest={"schema":"slm-data-manifest/v1","pipeline_commit":a.pipeline_commit,
              "row_count":len(rows),"files":files,
              "status":"execution_requires_review"}
    (a.output/"manifest.json").write_text(
        json.dumps(manifest,sort_keys=True,separators=(",",":")),"utf-8")
if __name__=="__main__": main()
```

The listing deliberately requires a previously validated assembly. Before production publication add JSON Schema
validation, rights/reviewer authorization, malware/PII gates, lineage directed-acyclic-graph checks, near-duplicate
clustering, contradiction review, temporal rules, tokenizer rendering, and objective-specific checks.

Near duplicates use candidate retrieval such as MinHash/locality-sensitive hashing followed by reviewed similarity.
Cluster before splitting and guard against transitive chaining. Contradictions need subject, predicate, value/unit,
conditions, jurisdiction, product/version, and effective interval; majority frequency must not overrule current
authority.

## Objective views

Do not reuse one JSONL for every stage:

<!-- markdownlint-disable MD013 -->
| View                     | Required payload                                               | Forbidden input           |
| ------------------------ | -------------------------------------------------------------- | ------------------------- |
| CPT                      | contiguous text, boundaries, authority/source/token accounting | sealed evaluation         |
| SFT                      | exact `messages`, template/loss policy, expert-approved target | hidden evaluator state    |
| Preference               | prompt, candidates, rubric, tie/both-bad, generator/reviewer   | forced false preference   |
| Distillation             | teacher revision/decoding/raw+accepted output/evidence         | unreviewed teacher claims |
| Quantization calibration | representative rendered inputs, lengths/tails                  | test roots                |
| Pruning calibration      | inputs, masks/statistic policy, mixture                        | test roots                |
| Evaluation               | prompt, ground truth/rubric, risk, seal/access log             | any optimization path     |
<!-- markdownlint-restore -->

Example SFT record:

<!-- markdownlint-disable MD013 -->
```json
{
  "id": "auto-001",
  "domain": "automotive",
  "task": "diagnostic_dialogue",
  "root_family_id": "case-auto-001",
  "risk_tier": "high",
  "source_ids": [
    "approved-source-17"
  ],
  "messages": [
    {
      "role": "system",
      "content": "Distinguish observations from hypotheses."
    },
    {
      "role": "user",
      "content": "A misfire code appeared. Which component should I replace?"
    },
    {
      "role": "assistant",
      "content": "The code identifies a detected condition, not a failed component. Preserve evidence and follow the exact vehicle procedure before replacement."
    }
  ],
  "review_status": "expert_accepted"
}
```
<!-- markdownlint-restore -->

This is `ILLUSTRATIVE`, not approved advice or training data.

## Commands and validation

```powershell
# Verification note: not executed here
.\.venv-data\Scripts\python.exe src/slm_impl/data/acquire.py `
  --registry configs/sources.yaml --root data
.\.venv-data\Scripts\python.exe src/slm_impl/data/build.py `
  --validated-jsonl data/validated/segments.jsonl `
  --split-key configs/secrets/split-key.bin `
  --pipeline-commit 08d7ac0eedd64c9d751783ed8d2178834c21b879 `
  --output data/curated/snapshot=automotive-sft-2026-11-24-v1
```

Exact validation:

1. schema-test valid, malformed, missing, extra-field, and Unicode fixtures;
2. rerun build twice and compare logical rows and hashes;
3. prove every published row reaches original bytes and a rights record;
4. prove no source/root/duplicate/synthetic family crosses a sealed split;
5. scan train/calibration against evaluation by hash, near-match, lineage, and manual semantic sample;
6. report distributions by source/domain/task/risk/difficulty/language and input/target tokens;
7. manually inspect PDF order, OCR quantities, units, warnings, and tables;
8. simulate source withdrawal and prove descendants are identified;
9. reject an unknown license, prohibited PII, parser failure, empty extraction, and unresolved critical contradiction;
10. hash all files and the manifest in a clean verification process.

Expected failures: changed terms, `401/403/429`, malformed or malicious documents, OCR corruption, ambiguous units,
giant boilerplate duplicate clusters, inadequate reviewer authority, cross-split lineage, mutable output path, and
insufficient temporary disk.

## Completion record

The acquisition boundary, schemas, extraction policy, normalization, exact deduplication, deterministic root splitting,
immutable publication, objective separation, commands, failures, and validation gates are authored. Production-grade
parser/OCR/PII/near-dedup/contradiction modules and all execution evidence remain unimplemented here and must be
completed before an end-to-end claim.
