# Evidence and Citation Standard

## The question every claim must answer

A reader should be able to ask, “How do we know this, under what conditions, and when was it checked?” Every checkable
claim MUST therefore identify its evidence type, scope, date, and important limitations. A label describes where the
support came from; it is not a score for truth.

## Choose the evidence label

| Label | Plain-language meaning | What must be linked |
| --- | --- | --- |
| `REPRODUCED` | An independent person or team repeated a prior result | original and independent runs, raw artifacts, comparison |
| `EXECUTED` | This project ran the exact revision on the named target | run/result ID, environment, command, output, date |
| `PAPER-REPORTED` | A paper reports the statement | source ID and page, section, table, or figure |
| `VENDOR-REPORTED` | A product or model owner reports the statement | source ID, exact product/version, access date |
| `DOC-VERIFIED` | Current authoritative documentation explicitly supports it | source ID, version, scope, access date |
| `STATICALLY REVIEWED` | A person inspected code or design without target execution | artifact ID, reviewer, scope, date |
| `UNVERIFIED IMPLEMENTATION` | Runnable-looking material has not passed target execution | listing ID, prerequisites, validation steps, known limits |
| `PLANNED EXPERIMENT` | A method to answer a question exists, but no result exists | plan ID, protocol, acceptance rule |
| `ILLUSTRATIVE` | An example or number exists only to teach | explicit statement and assumptions |
| `ESTIMATE` | A formula or forecast produced the value | formula, inputs, date, uncertainty |

### Worked example

Suppose a vendor page says its runtime supports an Android GPU backend.

- Correct: `VENDOR-REPORTED` for the support statement, with runtime version and access date.
- Not yet justified: `EXECUTED`, because this project has not run the model on Android.
- After a successful project test: add a separate `EXECUTED` claim scoped to the exact phone, model, runtime, and test.

### Anti-example

> The model is mobile-ready and achieves excellent speed.

“Mobile-ready,” “excellent,” the device, the workload, and the evidence are all undefined. The sentence cannot guide a
purchase or deployment decision.

## Which statements need labels

Label the first nearby occurrence of empirical results, benchmarks, compatibility, support, prices, checkpoint
properties, licenses, maintenance status, implementation outcomes, physical-device behavior, and numerical estimates.
In a table, label every row or include a clear evidence column.

A definition such as “perplexity is the exponential of average cross-entropy” does not need a label when the derivation
is shown. It still needs a citation if the statement is attributable, disputed, non-obvious, or depends on a particular
implementation.

## Make one claim say one thing

Every material claim MUST have an immutable ID such as `CLM-EDGE-0042` and a record in the
[evidence ledger](../15-audit-ledgers/evidence-status-ledger.md). Record:

- the exact claim and canonical location;
- one evidence label;
- source IDs and, for execution, run or artifact IDs;
- model, version, hardware, dataset, workload, domain, and exclusions;
- publication or observation date and last verification date;
- expiry date or the event that requires rechecking;
- limitations, owner, and reviewer.

Good atomic claim:

> `CLM-EDGE-0042`: Runtime 0.4.2 documents CPU inference for model family X on Android arm64 as of 2026-11-20.

Bad combined claim:

> Runtime X supports every phone and is faster and more accurate.

One documentation page is unlikely to support every part of the second sentence.

## Cite the source that actually supports the words

Give every source an ID such as `SRC-DOC-0118` and complete the
[source record](../12-reference-library/source-record-template.md). Include authors or owner, title, date, publication
status, version or DOI/arXiv ID, canonical URL, access date, contribution, relevant finding, limitations, reproduction
availability, and affected claims.

Use a relative repository citation such as
`[Attention Is All You Need](../12-reference-library/bibliography.md#vaswani2017attention)`. For papers, include a
locator such as “Section 4.2” or “Table 3.” A search result, generated answer, URL shortener, or unsourced snippet is a
discovery aid, not final evidence.

Prefer original papers, standards, official documentation, and official repositories. Use independent reproduction
when available. Vendor material is appropriate for documenting a vendor's product, but comparative performance should
also have independent support. Identify preprints, corrections, retractions, and superseding versions.

## Keep fast-moving claims current

Model licenses, APIs, runtime support, AWS services, instance types, and prices can change after the prose is written.
Record them in the [recency ledger](../15-audit-ledgers/compatibility-and-recency-ledger.md).

- Recheck fast-moving claims within 90 days of a publication candidate.
- Recheck prices within 30 days.
- Use a shorter validity period when a source or deprecation notice requires it.
- Recheck immediately when a pinned version, license, API, URL, hardware target, or support matrix changes.
- If a claim expires, verify it again, downgrade it, or remove it. Do not keep an old date as if it were current.

Historical statements should name their historical date; they do not claim current support.

## Handle disagreement and uncertainty

Present conflicting credible evidence with each source's conditions. Do not average results collected on different
models, datasets, or hardware. Lack of evidence is not proof that something is impossible. Words such as “best,”
“fastest,” “safe,” and “production-ready” require a declared comparison set, decision rule, and supporting evidence.

## Short author workflow

1. Highlight material claims.
2. Split compound claims.
3. Assign IDs and the strongest justified labels.
4. Link primary sources or run artifacts with exact locators.
5. Record scope, dates, and limitations.
6. Update the evidence and recency ledgers.
7. Ask a reviewer to check every high-risk and execution claim.

The reviewer MUST reject missing raw evidence, broader wording than the evidence supports, stale verification, or a
label promoted without justification. This file defines authoritative policy; it does not promote any other claim.
