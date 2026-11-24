# Deliverable readiness ledger

## What is checked and why

This ledger asks what each promised deliverable is useful for **today** and what would be needed before relying on it
for production or publication. This distinction matters because a complete curriculum can be valuable even when its
code, measurements, rights, and safety claims have not yet been independently validated.

`DRAFT` means substantive content exists. `VERIFIED` would require traceable evidence for claims and claimed behavior.
`REVIEWED` would require all applicable independent reviewers to approve the same frozen version. No deliverable has
reached either higher state.

<!-- markdownlint-disable MD013 MD060 -->
| Deliverable | What exists and what learners gain | Current status | Validation method |
| --- | --- | --- | --- |
| A — Technical textbook | 47 chapters, 14 parts, front/back matter, figures, tables, and navigation teach the complete subject path | `DRAFT`; content complete, claim and editorial review not yet performed | Normalize material claims/sources; technical and copy editors review a frozen render; resolve findings |
| B — Engineering handbook | Procedures, commands, troubleshooting, and 11 tools teach how to implement the workflows | `DRAFT`; all executable material remains unverified | Extract exact listings; resolve locks; lint/type/test; execute on every claimed target; independently repeat critical paths |
| C — Research comparison report | Comparisons teach how model, method, runtime, cloud, and edge choices differ | `DRAFT`; source recency and project reproduction incomplete | Refresh mutable facts; complete source locators; reproduce selected decision-critical findings under controlled conditions |
| D — Domain dataset guide | Ten domain classes and a reusable rights-aware process teach dataset design and governance | `DRAFT`; real-source approval and pipeline run not yet performed | Approve licenses; ingest hashed sources; run lineage, PII, duplicate, leakage, contradiction, and quality checks |
| E — Benchmark framework | Protocols, schemas, and blank result records teach controlled evaluation | `DRAFT`; no project benchmark results | Freeze test data and plans; validate scorers; retain every run/failure and raw result; review statistics |
| F — Automotive case study | A 17-phase end-to-end blueprint teaches specialization through deployment and selection | `DRAFT`; 0/17 phases have retained execution evidence | Approve rights and safety; execute every phase; compare candidates; make five selections from measured evidence |
| G — AWS guide | Training, serving, security, observability, cost, rollback, and delivery guidance | `DRAFT`; named cloud execution not yet performed | Freeze account/Region/quota/images; deploy, load-test, canary, roll back, tear down, and reconcile cost |
| H — Edge guide | Android, iOS, runtime, hardware, update, and benchmark guidance | `DRAFT`; physical-device validation not yet performed | Build and sign apps; install exact exports; test offline behavior, lifecycle, updates, quality, memory, latency, battery, and heat |
| I — Multi-domain architecture | Strategies and contracts teach reusable no-runtime-RAG model management | `DRAFT`; multi-domain system not yet implemented | Train/test at least two domains; measure interference and isolation; exercise independent updates and rollback |
| J — Research roadmap | Questions, hypotheses, priorities, budgets, risks, and decision rules teach staged investigation | `DRAFT`; experiments not yet performed | Approve hashed protocols; execute in stages; link results, uncertainty, and decisions back to each hypothesis |
| K — Glossary and references | Bibliography, terms, abbreviations, definitions, and source templates support study | `DRAFT`; metadata and independent evidence review incomplete | Complete IDs, dates, locators, versions, and limitations; check definition conflicts and source support |
| L — Learning assessments | 47 chapter tasks, rubrics, assignments, and milestones support practice | `DRAFT`; curriculum trial not yet performed | Subject and curriculum reviewers solve samples, check rubrics, test progression, and observe a capstone cohort |
<!-- markdownlint-restore -->

## Passing and failing evidence

**Pass for the edge guide:** a frozen guide and exact artifact produce signed Android and iOS builds that install and
run offline on named devices; retained tests cover quality, lifecycle, update failure, rollback, memory, latency,
battery, and thermal behavior.

**Fail:** every section contains code and screenshots, but no exact artifact was built or installed. This can still be
a useful `DRAFT`; it is not target-verified.

## Overall readiness

- Content-complete deliverables: **12/12**.
- Deliverables at `VERIFIED` or above: **0/12**.
- Retained project run bundles: **0**.
- Qualified domain/safety approvals: **0**.
- Evidence-based final target selections: **0/8**.

Responsibility for each remaining method is unassigned until a named person accepts it. Publication requires one
candidate hash, immutable evidence, applicable rights and expert approvals, target qualification, independent review,
and a release record. Content completeness does not substitute for those real-world checks.
