# Open issues and omissions

## How to use this list

An issue remains open when the repository lacks evidence needed to rely on a claim, implementation, or release. The
severity reflects consequence: **Critical** can invalidate safety, legality, or the central project outcome; **High**
blocks reproducibility or defensible decisions; **Medium** blocks publication quality. Closing a row requires the
listed evidence, not a change from “open” to “done.”

<!-- markdownlint-disable MD013 MD060 -->
| ID / severity | What is missing and why it matters | Current responsibility | Evidence that closes the issue |
| --- | --- | --- | --- |
| AUD-001 / Critical | Training, compression, conversion, deployment, and benchmarks are not yet performed, so no project outcome is demonstrated | Unassigned; name experiment lead before execution | Reviewed `RUN-*`/`RES-*` bundles with exact tuples, commands, failures, raw artifacts, assertions, uncertainty, and hashes |
| AUD-002 / Critical | Physical Android, iOS, ARM, Pi, Jetson, laptop, accelerator, battery, and thermal checks are not yet performed, so edge feasibility is unknown | Unassigned; name edge test lead and device custodians | Declared cohorts pass correctness, parity, memory, latency, energy, thermal, lifecycle, offline, update, and rollback gates |
| AUD-003 / Critical | AWS deployment, quota, Region, canary, rollback, cleanup, price, and billing checks are not yet performed | Unassigned; name cloud and cost leads | Reproducible deployment/load/failure/rollback/cleanup records plus dated rates and reconciled bill |
| AUD-004 / Critical | Qualified safety and domain review is not yet performed, so specialist use cannot be approved | Unassigned; appoint reviewers with recorded qualifications | Signed reviews of frozen content, data, suites, artifact, critical outputs, remediation, retest, and residual risk |
| AUD-005 / Critical | A rights-approved real corpus and sealed benchmark do not yet exist, so training legality and evaluation integrity are unknown | Unassigned; name data and rights reviewers | Source permissions, lineage, hashes, PII/dedup/leakage reports, authoritative ground truth, split controls, and approvals |
| AUD-006 / High | Material claims do not all have atomic `CLM-*` and complete `SRC-*` records, so evidence can be hard to trace | Unassigned; name evidence editor | Every material claim has one scope, label, exact locator, date, limitation, expiry, and independent review |
| AUD-007 / High | Most chapter listings lack complete extraction and run manifests, so learners cannot prove they ran the same code | Unassigned; name implementation maintainer | Stable listing IDs and markers, exact-byte hashes, locks, lint/type/tests, target runs, outputs, and review |
| AUD-008 / High | Locks, image digests, model/runtime revisions, and SBOMs are unresolved, so clean reproduction is unlikely | Unassigned; name build maintainer | Clean builds from immutable manifests on a second environment with matching assertions |
| AUD-009 / High | Mutable model, runtime, API, and license facts need candidate-date refresh, so some choices may become stale | Unassigned; name compatibility reviewer | Recheck within the defined window; freeze revisions and licenses; record changes and affected claims |
| AUD-010 / High | Current prices and comparable target measurements are absent, so none of eight target profiles can name a winner | Unassigned; name cost and decision reviewers | Dated prices and controlled measurements satisfy hard gates; reviewed Pareto and sensitivity analysis supports each selection |
| AUD-011 / High | Independent technical, evidence, statistical, rights/privacy, editorial, and release reviews are not yet performed | Unassigned; release coordinator must identify separate qualified reviewers | All approvals cite the same candidate hash and all findings are closed or explicitly release-blocking |
| AUD-012 / Medium | Visual accessibility and rendered-publication review are not yet performed | Unassigned; name accessibility reviewer | Contrast, grayscale/print, text alternative, screen-reader order, narrow-layout, and font-size checks pass |
| AUD-013 / Medium | Publication manifest, final render, checksums, release notes, errata, and rollback archive do not yet exist | Unassigned; name publication maintainer | Immutable candidate archive contains every artifact, hash, approval, known limit, errata path, and rollback instruction |
<!-- markdownlint-restore -->

## Passing and failing closure evidence

**Pass for AUD-001:** a failed out-of-memory training attempt and a successful corrected run are both preserved; exact
data/model/config/environment hashes, metrics, checkpoints, and reviewed conclusions are linked.

**Fail:** a status cell says “complete,” a screenshot shows a progress bar, or only the successful summary is retained.
That evidence cannot establish the exact run, hide no failures, or support reproduction.

## Truthful omissions

The repository does not claim a successful model run, rights-approved corpus, measured graph, current dollar price,
physical-device performance, best candidate, or professional-use approval. Learning sections use explicitly
illustrative worked values to teach calculations and interpretation; project result registries do not substitute
unrelated reports or assumptions for measurements.

## Current disposition and order of work

The 302-file documentation corpus passes the final marker, navigation, table, fence, heading, equation, structured
snippet, and configured Markdown-lint checks. Start by freezing a candidate and approving data rights; then resolve
dependencies and listings; execute training/evaluation; test cloud and physical devices; complete expert and
independent reviews; finally render and archive the release. The package remains a documentation-complete `DRAFT`,
not evidence of a production-ready SLM.
