# Definition of Done

## “Done” means evidence, not effort

A long chapter may still need review. A short script may be fully verified. Completion depends on the gates an artifact
has passed, not the time spent on it or the percentage of a checklist that looks complete.

The applicable state is the highest state for which **every** mandatory condition is satisfied. Missing evidence,
unavailable artifacts, unresolved critical risk, or unperformed target execution MUST remain visible.

## Maturity states in plain language

| State | What the reader can rely on | What moves it forward |
| --- | --- | --- |
| `PLANNED` | The need, owner, and acceptance rule are known | write substantive content |
| `DRAFT` | Useful content exists, but evidence or review may be incomplete | self-audit, IDs, sources, known limitations |
| `VERIFIED` | Claims and artifacts have traceable evidence at the stated level | independent applicable reviews |
| `REVIEWED` | Required technical, editorial, safety, rights, and statistical reviews passed | freeze and integrate a candidate |
| `RELEASE-CANDIDATE` | One immutable snapshot passes all release gates | final approval and publication |
| `RELEASED` | The approved version is published with hashes and rollback information | maintain or supersede it |
| `DEPRECATED` | The artifact remains traceable but is no longer recommended | point readers to its replacement |
| `REVOKED` | Approval was withdrawn because of risk, rights, integrity, or defects | contain harm and publish guidance |

`BLOCKED` explains why a pre-release artifact cannot advance; it is not progress by itself. `SUPERSEDED` links an old
immutable record to a newer one without erasing history.

### Example

A complete Android tutorial with imports, build commands, and troubleshooting is useful `DRAFT` material. If no one has
built it on the named Android version and device, its code remains `UNVERIFIED IMPLEMENTATION`. Clear validation
instructions make it a strong lesson, but do not turn it into an `EXECUTED` implementation.

### Anti-example

> The mobile chapter is 100% done because every heading contains text.

Filled headings do not prove compatibility, successful execution, safe behavior, or review.

## When a chapter is reviewed

A chapter or standalone guide MAY reach `REVIEWED` only when:

- it teaches its scope coherently and maps to the [coverage ledger](../15-audit-ledgers/prompt-coverage-ledger.md);
- terminology is consistent and fact, report, estimate, proposal, and illustration are distinguishable;
- material claims have IDs, labels, sources, scope, dates, and limitations;
- required figures, tables, listings, exercises, and links are present and useful;
- visuals follow the [figure and table standard](figure-table-standard.md);
- code follows the [listing standard](code-listing-standard.md), with unexecuted code labeled honestly;
- experiments follow the [reproducibility standard](reproducibility-standard.md);
- intended use, risk, rights, privacy, security, expert review, and no-runtime-RAG checks are complete where applicable;
- links, anchors, IDs, fences, extraction markers, spelling, and Markdown validation pass; and
- open limitations have an owner, severity, next action, and release decision.

Finishing one chapter does not finish its part or the whole repository.

## When code or an experiment is done

An implementation is **instructionally complete** when a learner has the files, dependencies, commands, input/output
contract, safety and cost warnings, cleanup steps, and exact validation checks. It is **target-verified** only after the
exact revision runs on every target claimed and the logs and assertions are retained.

An experiment is complete only when the approved hashed plan, every attempted run, failures and deviations, raw data,
analysis, uncertainty, bounded conclusion, and required reviews are preserved. `EXECUTED` means the run happened; it
does not automatically mean success, statistical significance, safety, or release fitness.

## A release-candidate workflow

1. **Freeze one candidate.** Record the commit, dependencies, data/model manifests, and artifact hashes.
2. **Check content and evidence.** Confirm coverage, citations, recency, terminology, and accessible explanations.
3. **Check implementations and experiments.** Confirm labels match retained run evidence.
4. **Check architecture and safety.** Run no-runtime-RAG, rights, security, privacy, hazard, and expert gates.
5. **Check the publication.** Validate Markdown, links, anchors, IDs, extraction, navigation, and rendered output.
6. **Resolve blockers.** Every remaining issue needs a recorded owner and release disposition.
7. **Approve the same snapshot.** All reviewers must reference the identical candidate hash.

The supporting records are the [readiness ledger](../15-audit-ledgers/deliverable-readiness-ledger.md),
[evidence ledger](../15-audit-ledgers/evidence-status-ledger.md),
[implementation ledger](../15-audit-ledgers/implementation-verification-ledger.md),
[safety ledger](../15-audit-ledgers/safety-and-domain-review-ledger.md), and
[open-issues ledger](../15-audit-ledgers/open-issues-and-omissions.md).

## Waivers and non-waivable controls

A non-safety mandatory rule may be waived only where its standard allows this. Record the requirement, artifact hash,
reason, compensating control, risk, owner, independent approver, expiry, and remediation.

No waiver may permit fabricated evidence, missing data rights, broken artifact integrity, `R4` functionality, or runtime
RAG in the primary architecture.

## Release and maintenance

For a release, publish the version, date, candidate hash, checksums, supported targets, evidence boundary, known
limitations, migration and rollback instructions, and superseded version. New evidence that invalidates safety, rights,
integrity, or a material claim requires triage and may require revocation or a corrected release.

Project schedules do not override these gates. This file defines authoritative completion rules; it does not declare
the repository or any artifact complete.
