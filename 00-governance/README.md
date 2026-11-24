# Governance Guide

Governance answers a practical question: **what may a reader safely believe and do after reading this book?** These
rules keep an explanation, a reported result, a worked example, and a tested implementation from being confused with
one another.

The rules apply to every Markdown deliverable in the repository. **MUST** means a release cannot proceed without the
requirement or an explicitly allowed waiver. **SHOULD** means the normal practice; a reviewer may accept a documented
reason to differ. **MAY** marks an option.

## Why these rules matter

Imagine a table saying that a model produces 18 tokens per second on a phone. That number has very different meanings
depending on whether it was:

- measured by this project on a named device;
- copied from a vendor page under different conditions;
- calculated from an assumption; or
- invented only to explain how a chart works.

Without a label and trace, a learner may make an expensive or unsafe decision from the wrong kind of evidence.
Governance makes the distinction visible.

## Read the standards in this order

1. [Evidence and citation](evidence-and-citation-standard.md) — decide what supports a claim.
2. [Reproducibility](reproducibility-standard.md) — turn an experimental question into auditable evidence.
3. [Code listings](code-listing-standard.md) — tell readers exactly what code was and was not tested.
4. [Figures and tables](figure-table-standard.md) — prevent educational graphics from looking like measurements.
5. [Safety and risk](safety-and-risk-standard.md) — increase controls as plausible harm increases.
6. [No-runtime-RAG](no-runtime-rag-standard.md) — preserve the book's central architecture constraint.
7. [Terminology](terminology-standard.md) — keep similar concepts distinct.
8. [Cross-references](cross-reference-standard.md) — keep the book navigable as files move.
9. [Definition of done](definition-of-done.md) — decide when an artifact may advance toward release.

If two rules conflict, apply safety and legal controls first, then evidence integrity, reproducibility, architecture, and
presentation. Record unresolved conflicts in [open issues](../15-audit-ledgers/open-issues-and-omissions.md); do not
quietly choose the easier rule.

## A learner-oriented authoring workflow

For each section:

1. **Teach the idea.** State the problem, intuition, mechanism, and practical consequence.
2. **Mark checkable claims.** Give each material claim an evidence label and trace it to a source or project artifact.
3. **Separate teaching from observation.** Label hypothetical numbers and diagrams `ILLUSTRATIVE`.
4. **Make code honest.** Say whether it ran, where it ran, and how the reader can validate it.
5. **Assess risk.** Identify plausible harm and add expert review or prohibited-use limits where needed.
6. **Check navigation.** Use stable IDs and relative links.
7. **Record the state.** Update the applicable ledger instead of relying on prose alone.

### Good example

> `PAPER-REPORTED`: The cited study measured lower perplexity after domain-adaptive pretraining under its stated
> dataset and model conditions. This project has not reproduced that result.

This tells the learner both what the source found and what remains to be validated locally.

### Anti-example

> Domain-adaptive pretraining definitely improves every specialist model.

This overgeneralizes one possible result, hides its source, and gives no conditions under which the statement might
fail.

## Records that preserve the evidence

Use the ledgers for [claims](../15-audit-ledgers/evidence-status-ledger.md),
[recency and compatibility](../15-audit-ledgers/compatibility-and-recency-ledger.md),
[implementations](../15-audit-ledgers/implementation-verification-ledger.md),
[figures and tables](../15-audit-ledgers/figure-table-ledger.md),
[safety reviews](../15-audit-ledgers/safety-and-domain-review-ledger.md),
[links and terms](../15-audit-ledgers/cross-link-and-terminology-ledger.md), and
[readiness](../15-audit-ledgers/deliverable-readiness-ledger.md).

Use the populated repository forms for a [source record](../12-reference-library/source-record-template.md),
[experiment plan](../06-experiment-framework/templates/experiment-plan-template.md),
[run record](../06-experiment-framework/templates/run-record-template.md), and
[experiment result](../06-experiment-framework/templates/experiment-result-template.md). A form is useful only when its
fields contain real, scoped information; unfilled prompts are not evidence.

## Rules that never disappear

- A claim receives no stronger label than its evidence supports.
- A procedure that has not run remains a procedure, not a result.
- The primary product does not retrieve documents at answer time.
- High-risk material requires the stated expert and safety reviews.
- Every stable artifact has one ID and one canonical location.
- A measured result links to its protocol, environment, and raw artifacts.
- Executable-looking code states its validation level.
- Overall maturity stops at the lowest unsatisfied mandatory gate.

These standards are authoritative policy. That means the rules themselves are in force; it does **not** mean the book,
its code, or its experiments have passed them. A change to an evidence label, risk tier, architectural prohibition, ID
scheme, or release gate must explain the reason, migration, affected records, reviewer, date, and compatibility impact.
