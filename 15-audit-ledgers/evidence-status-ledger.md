# Evidence status ledger

## What is checked and why

This ledger checks whether important statements tell the reader **how we know, under what conditions, and when they
were checked**. This prevents a paper result, vendor statement, estimate, and project measurement from being treated
as interchangeable evidence.

## How to interpret evidence labels

<!-- markdownlint-disable MD013 -->
| Label | Meaning in this repository | Passing example | Failing example |
| --- | --- | --- | --- |
| `PAPER-REPORTED` | A cited paper reports the bounded finding | Source ID plus page/table locator, model, dataset, and limitation | “Research proves quantization is lossless” with no locator or scope |
| `VENDOR-REPORTED` | A product owner reports a capability | Exact product/version, vendor URL, access date, and wording | Extending a vendor claim to unsupported devices |
| `DOC-VERIFIED` | Authoritative documentation explicitly supported the statement on a date | Runtime 0.x docs name an operator/backend and supported platform | Assuming that documented backend support proves one model exports |
| `STATICALLY REVIEWED` | A person inspected content or code without target execution | Named reviewer, immutable artifact, checklist, findings, and date | An author rereads changing files and calls the code tested |
| `UNVERIFIED IMPLEMENTATION` | Concrete code or commands exist but lack target-run evidence | Listing ID, prerequisites, limits, and exact validation steps | Calling a complete-looking listing `EXECUTED` |
| `PLANNED EXPERIMENT` | A protocol and acceptance rule exist but no result does | Hashed plan, controls, metrics, repetition, and empty result record | A graph with invented points |
| `ILLUSTRATIVE` | The item teaches a concept rather than reports a result | Caption says it is conceptual and names assumptions | A conceptual diagram presented as measured architecture performance |
| `ESTIMATE` | A formula and inputs produce a forecast | Memory formula, units, inputs, date, and uncertainty | A dollar or latency number with no source or calculation |
| `EXECUTED` | This project ran an exact revision on a named target | `RUN-*`, environment, command, logs, output, date, and hashes | An upstream benchmark or successful import only |
| `REPRODUCED` | An independent run repeated a prior result | Original and independent runs, raw data, and comparison | The same operator reruns the same notebook |
<!-- markdownlint-restore -->

## Current evidence records

<!-- markdownlint-disable MD013 -->
| Content group | Current evidence | What the status permits the learner to conclude | Validation method and responsibility |
| --- | --- | --- | --- |
| Foundations and established methods, CH-01–15 | Paper-backed draft synthesis | Cited authors reported bounded findings; project reproduction is not implied | Responsibility unassigned: split material claims into `CLM-*` records and complete `SRC-*` records with locators and limits |
| Model and checkpoint facts, CH-03 and model survey | Documentation checked at the research freeze; immutable revisions are incomplete | Facts were supported for the cited version/date only | Responsibility unassigned: freeze repository revision, files, license, tokenizer, and model card; recheck within 90 days of a candidate |
| Runtime, mobile, and AWS capabilities | Documentation-level support only | A documented feature may be considered for testing, not treated as target compatibility | Responsibility unassigned: pin the full tuple and retain export, load, parity, placement, and target-run evidence |
| Executable listings | `UNVERIFIED IMPLEMENTATION` | The learner has a concrete blueprint and validation path | Responsibility unassigned: register, extract, hash, lint, test, execute, and independently review every material listing |
| Benchmarks and final choices | `PLANNED EXPERIMENT`; no project results | The reader can run a controlled study but cannot cite a winner | Responsibility unassigned: approve hashed plans, retain all attempts/raw data, quantify uncertainty, and review conclusions |
| Costs and prices | Formulae and estimates; current capture not yet performed | Learners can calculate cost when supplied with dated rates and usage | Responsibility unassigned: capture official Region-specific rates within 30 days and reconcile estimates with bills |
| Safety and domain suitability | Authored controls; qualified approval not yet performed | Risks and required checks are taught, but no model is approved for reliance | Responsibility unassigned: experts review frozen content, data, suites, outputs, and intended-use limits |
<!-- markdownlint-restore -->

## Current gaps and how to close them

1. Assign stable `CLM-*` IDs to each material claim and keep each claim atomic.
2. Replace incomplete or direct-only citations with `SRC-*` records containing exact locators, dates, scope, and limits.
3. Freeze model cards, repositories, datasets, and documentation by revision or hash for experiments.
4. Create `RUN-*` and `RES-*` bundles for every attempted execution, including failures and raw artifacts.
5. Obtain an independent evidence review of the same frozen candidate.

No qualifying project `EXECUTED` or `REPRODUCED` record was found. Therefore training, conversion, compression,
deployment, benchmark, physical-device, and price outcomes remain not yet performed. A prose improvement cannot change
that status; a changed dependency, model, data set, runtime, OS, hardware target, or workload requires re-execution or
a reviewed equivalence argument.
