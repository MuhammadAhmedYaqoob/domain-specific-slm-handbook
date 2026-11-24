<a id="references"></a>

# References and Source Navigation

This page teaches how to move from a claim to a source and back. Canonical records are in the
[reference library](../../12-reference-library/README.md), supported by the
[evidence-status ledger](../../15-audit-ledgers/evidence-status-ledger.md) and chapter-level reference sections.

## Source hierarchy

Prefer, in order appropriate to the claim:

1. peer-reviewed paper or original preprint/revision for a research method;
2. official model card, license, repository, and technical report for a checkpoint;
3. official runtime/API/hardware/cloud documentation for capabilities;
4. immutable project execution artifacts for project results;
5. reputable secondary synthesis only when primary material is unavailable or the synthesis itself is the subject.

Marketing pages, snippets, blogs, and benchmark leaderboards may locate a primary source but should not silently replace
it.

## Required source record

Each `SRC-*` record should contain title, author/organization, source type, stable URL or DOI/arXiv identifier,
publication and access dates, version/revision, license where relevant, archived snapshot or content hash when
permitted, supported claims, limitations, and reviewer.

### Example trace

For “runtime R supports format F,” locate official versioned runtime documentation. Record the version, access date,
URL, supported scope, and limitations; label that statement `DOC-VERIFIED`. Create a separate compatibility claim for
the project artifact and device. It remains a planned test until conversion, load, operator coverage, and inference
evidence exist. One source does not prove both claims.

## Citation rules

- Cite near the claim, not only at chapter end.
- Point to the exact paper section/table/figure or documentation section when practical.
- Distinguish `PAPER-REPORTED`, `VENDOR-REPORTED`, and `DOC-VERIFIED`.
- Do not cite a survey for a numerical result when the original result is available.
- Do not fabricate authors, dates, DOI, arXiv IDs, URLs, versions, or access dates.
- Preserve contradictory sources and explain scope/version differences.
- Attach model, dataset, code, and documentation versions to compatibility claims.

## Recency-sensitive sources

Recheck model licenses/cards, runtime/operator support, mobile OS and SDK requirements, AWS service
behavior/quotas/prices, hardware specifications, and security guidance at every release cutoff. Record “as of” dates. A
current documentation check does not retroactively verify old behavior.

## Bibliography generation

Generate publication bibliographies only from validated source records. Deduplicate by DOI/arXiv/URL plus version,
preserve display names and title casing from authoritative metadata, and fail the build on unresolved citation keys.
Chapter-local short keys must map unambiguously to canonical records.

## Reader workflow

1. Find the evidence label beside the claim.
2. Follow its citation or `SRC-*` key to the canonical record.
3. Check model, version, hardware, workload, and date.
4. Read limitations and conflicting sources, not only the quoted result.
5. Decide whether the source supports understanding, a hypothesis, or a release decision.

If any link in that chain is missing, treat the claim as unsupported for decision-making.

## Reproduction references

A source citation plus similar output is insufficient for `REPRODUCED`. Link the prior claim, independent protocol,
exact artifacts and environment, raw results, comparison tolerance, and reviewer. Otherwise retain `PAPER-REPORTED` or
`VENDOR-REPORTED`.

Related: [Evidence labels](../front-matter/evidence-labels.md),
[evidence standard](../../00-governance/evidence-and-citation-standard.md), and
[experiment checklist](appendix-experiment-reporting-checklist.md).
