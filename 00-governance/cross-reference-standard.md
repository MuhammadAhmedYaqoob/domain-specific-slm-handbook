# Cross-Reference Standard

<!-- markdownlint-disable MD033 -->

## Why stable links matter

This book is read in a repository, rendered as documentation, and used to extract code and records. Heading text,
pagination, and hosting URLs can change. Stable IDs, explicit anchors, and relative links let a reader find the same
artifact after those changes.

## Use one canonical location

- Internal links MUST be relative Markdown links.
- Do not use a local absolute path, `file://` URL, editor link, or branch-specific web URL as the canonical reference.
- External URLs belong in source records and citations.
- Directory and file names use lowercase kebab-case and `.md`, except established root files such as `README.md`.
- A filename describes its durable subject, not an author, date, status, version, or the word “final.”
- Each artifact has one canonical file and anchor. Other files link to it rather than copy its controlling text.
- A rename updates inbound links and ledger entries in the same change.

Good link:

```markdown
[quantization benchmark protocol](../06-experiment-framework/protocols/quantization-benchmark.md)
```

Anti-example:

Anti-example text: “click here in the writer's local `final-v3.md` file.”

The anti-example is machine-specific, uses weak link text, and embeds a mutable status in the filename.

## Stable IDs

| Artifact | Pattern | Populated example |
| --- | --- | --- |
| Chapter | `CH-01` through `CH-47` | `CH-11` |
| Claim | area plus four digits | `CLM-EDGE-0042` |
| Source | type plus four digits | `SRC-DOC-0118` |
| Figure/table | reserved numeric range | `FIG-07`, `TBL-12` |
| Listing | area plus four digits | `LST-TRAIN-0017` |
| Experiment/run/result | area plus four digits | `EXP-QUANT-0003`, `RUN-QUANT-0019`, `RES-QUANT-0003` |
| Hazard/decision | domain or area plus four digits | `HAZ-PHARM-0004`, `DEC-EDGE-0008` |
| Definition | descriptive slug | `TERM-TIME-TO-FIRST-TOKEN` |

Registries are the [detailed contents](../01-planning/detailed-table-of-contents.md),
[evidence ledger](../15-audit-ledgers/evidence-status-ledger.md),
[figure/table ledger](../15-audit-ledgers/figure-table-ledger.md),
[implementation ledger](../15-audit-ledgers/implementation-verification-ledger.md),
[results registry](../06-experiment-framework/results/README.md),
[safety ledger](../15-audit-ledgers/safety-and-domain-review-ledger.md), and
[definition register](../12-reference-library/definition-register.md).

IDs are uppercase, immutable, and never reused. They move with their content. Deleted records keep a tombstone or
supersession link so old citations still explain what happened.

## Anchors and link wording

Place an explicit lowercase anchor immediately before a stable heading:

```html
<a id="fig-07"></a>
```

A link then targets `chapter-file.md#fig-07`. Do not rely on an automatically generated heading slug for
release-critical references.

Link text should name the destination, such as “quantization calibration protocol,” rather than “here” or “Section 4.”
On first nearby use, include the ID and title: “Figure FIG-07 — Quantization calibration flow.” Avoid “above,” “below,”
and page numbers because extraction and pagination move content.

<a id="all-markdown-extraction-convention"></a>

## All-Markdown extraction convention

Markdown remains the source of truth for extractable code, commands, data, formulas, captions, and records.

For listing `LST-TRAIN-0017`, put these exact marker forms on separate lines around one complete artifact:

```html
<!-- extract:LST-TRAIN-0017:start -->
<!-- extract:LST-TRAIN-0017:end -->
```

Markers MUST match, occur once, remain unnested, and appear start before end. The enclosed artifact includes its
metadata and exactly one primary fenced block or table. Fences declare a language. Extraction preserves UTF-8 bytes and
the declared final-newline policy.

The artifact manifest records the ID, source path, anchor, output path, SHA-256, extraction tool and version, and UTC
time. Output extension, executable permission, and newline conversion are declared. Secret injection or undocumented
rewriting is prohibited. See the [code listing standard](code-listing-standard.md) for executable content.

## Validation workflow

Before review and release, automation MUST verify:

1. every relative target and explicit anchor exists;
2. registered IDs, anchors, and extraction markers are unique;
3. every registry path resolves to its canonical artifact;
4. every deliverable appears in the [bookmap](../BOOKMAP.md) or a linked directory index;
5. filename case matches on case-sensitive systems;
6. fences and extraction marker pairs are balanced; and
7. external source URLs have been checked and retain access dates.

Record results and justified exceptions in the
[cross-link and terminology ledger](../15-audit-ledgers/cross-link-and-terminology-ledger.md). A known broken internal
link blocks release. This standard is authoritative.
