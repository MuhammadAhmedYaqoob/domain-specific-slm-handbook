<a id="evidence-labels"></a>

# Evidence Labels

Evidence labels answer one plain question: **how do we know this?** They prevent a paper's result, a documentation
statement, an unrun code sample, and a project measurement from being mistaken for one another. Labels describe the
trace supporting a claim; they are not quality grades. The binding rules are in the
[evidence and citation standard](../../00-governance/evidence-and-citation-standard.md).

<!-- markdownlint-disable MD013 -->
| Label                       | Permitted meaning                                                             | Minimum trace                                                                      |
| --------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `REPRODUCED`                | An independent execution repeated a prior result under a declared protocol.   | Prior claim, protocol, environment, artifacts, raw output, comparison criterion    |
| `EXECUTED`                  | This project ran the stated code or test on the named target.                 | Command/config, versions, target, timestamp, logs or measurements, artifact hashes |
| `PAPER-REPORTED`            | The cited paper reports the claim; this project did not reproduce it.         | Precise citation and relevant section/table/figure                                 |
| `VENDOR-REPORTED`           | A vendor or model owner reports the claim.                                    | Dated first-party source and product/version                                       |
| `DOC-VERIFIED`              | Current authoritative documentation supports the API, feature, or constraint. | Dated documentation URL and version/context                                        |
| `STATICALLY REVIEWED`       | Syntax, API use, or control flow was inspected without target execution.      | Reviewer, date, reviewed artifact/version, review scope                            |
| `UNVERIFIED IMPLEMENTATION` | Executable-looking code/config exists but was not run on the declared target. | Artifact location and explicit unexecuted boundary                                 |
| `PLANNED EXPERIMENT`        | A protocol exists but no valid result exists.                                 | Hypothesis, variables, controls, metrics, stop conditions                          |
| `ILLUSTRATIVE`              | Numbers or flows exist only to teach a concept.                               | Visible label and assumptions sufficient to avoid factual interpretation           |
| `ESTIMATE`                  | A calculated or forecast value follows declared assumptions.                  | Formula/model, inputs, units, date, uncertainty or sensitivity                     |
<!-- markdownlint-restore -->

## Non-equivalence rules

- `PAPER-REPORTED` and `VENDOR-REPORTED` never imply `REPRODUCED`.
- `DOC-VERIFIED` establishes documented support, not successful integration or performance.
- `STATICALLY REVIEWED` never implies execution.
- `UNVERIFIED IMPLEMENTATION` must not be called tested, working, compatible, or production-ready.
- `PLANNED EXPERIMENT` contains no finding.
- `ILLUSTRATIVE` values must not enter a benchmark comparison.
- An `ESTIMATE` does not become `EXECUTED` merely because a script calculated it.

## Learn the labels through one example

Suppose a runtime manual says INT4 is supported, a paper reports lower memory, and this repository contains an export
script:

- “Runtime R documents INT4 for operator set O in version V” is `DOC-VERIFIED`.
- “Paper P reports 35% lower peak memory under workload W” is `PAPER-REPORTED`.
- “The export script should create an INT4 artifact” is `UNVERIFIED IMPLEMENTATION` until run.
- “The artifact occupied 1.2 GiB peak on device D under protocol Q” is `EXECUTED` only with retained logs and versions.
- Repeating paper P's protocol independently within its comparison tolerance may become `REPRODUCED`.
- Using 1.0 GiB merely to demonstrate a formula is `ILLUSTRATIVE`.
- Forecasting 1.2 GiB from a documented formula and declared inputs is an `ESTIMATE`.

Notice that several labels can be true at once because they refer to different claims.

## Claim-level usage

Place a label near every empirical performance, compatibility, cost, resource, safety, or implementation-status claim. A
section-level label is acceptable only when its scope is unambiguous. If adjacent claims have different traces, label
them separately.

For mixed evidence, split the statement:

> `DOC-VERIFIED`: Runtime X documents operator Y in version Z.  
> `PLANNED EXPERIMENT`: Export and execute the project artifact on target device D.

## Promotion rules

Evidence is promoted only when the stronger label’s trace exists. Running a notebook may promote its own behavior from
`UNVERIFIED IMPLEMENTATION` to `EXECUTED`; it does not promote a cited benchmark to `REPRODUCED` unless the reproduction
protocol and comparison criteria are satisfied.

## Staleness and supersession

Documentation, vendor specifications, prices, model licenses, cloud instance types, runtime support, and device firmware
change. Record an as-of date and version. A newer source may supersede a claim but does not erase the historical trace.
Conflicts belong in the claim/source registers and [limitations log](../back-matter/errata-and-limitations.md).

## Reader test

Before repeating a claim, ask: **who observed it, on what artifact and target, under which protocol, when, and where is
the retained evidence?** If the text cannot answer, use the weakest honest label or omit the claim.

### Quick practice

“A vendor model card lists an 8K context window” is `VENDOR-REPORTED`, not `EXECUTED`. “Our test loaded an 8K prompt
without crashing” is `EXECUTED`, but it does not prove answer quality across the full context. Label the smallest claim
the evidence actually supports.
