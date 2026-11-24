# Audit ledgers

## What this section is for

These ledgers help a learner answer two different questions:

1. **Can I learn the requested subject from this repository?**
2. **Has the project proved that its code, models, deployments, and safety controls work in the real world?**

The answer to the first question is largely yes. The answer to the second is not yet. On 2026-11-24, the repository
contains all 47 requested chapters, 14 part indexes, 25 required comparison tables, 26 required teaching visuals,
seven additional production-architecture diagrams, 11 learning tools, 12 deliverable areas, 47 assessments, 17
automotive project phases, and eight final decision profiles. Those counts show content coverage; they do not prove a
successful training run, mobile build, AWS deployment, benchmark, rights review, or expert safety review.

## How to read a status

<!-- markdownlint-disable MD013 -->
| Status | Plain-language interpretation | Passing evidence | Evidence that does not pass |
| --- | --- | --- | --- |
| `DRAFT` | Useful material exists, but some evidence or review is still missing | A substantive chapter with sources, limits, and exercises | A heading or checklist entry by itself |
| `VERIFIED` | The stated claims or artifact behavior have traceable evidence for a defined scope | A source-backed claim or retained run tied to exact versions | “This should work” without a source or run |
| `REVIEWED` | Required independent reviewers approved the same frozen version | Signed reviews naming the candidate hash | Informal comments on changing files |
| `EXECUTED` | This project ran the exact command on the named target | Command, environment, logs, outputs, date, and artifact hashes | A code listing or vendor tutorial |
| `REPRODUCED` | An independent run repeated a reported result | Original evidence plus an independent run and comparison | Re-running only the author's own script |
| `DOC-VERIFIED` | Authoritative documentation supported a bounded statement on a recorded date | Exact documentation, version, access date, and scope | Assuming support for every model or device |
| `UNVERIFIED IMPLEMENTATION` | Runnable-looking instructions exist but have not passed target execution | This remains the honest status until a retained run exists | Treating complete-looking code as tested |
| `PLANNED EXPERIMENT` | A protocol exists, but there are no results yet | Inputs, controls, metrics, acceptance rule, and blank result fields | Invented or borrowed measurements |
| `ILLUSTRATIVE` / `ESTIMATE` | Material teaches or forecasts rather than reports a project result | Assumptions and formula or explanatory caption | Presenting the value as measured |
<!-- markdownlint-restore -->

The complete vocabulary is defined in the
[definition of done](../00-governance/definition-of-done.md) and
[evidence standard](../00-governance/evidence-and-citation-standard.md).

## Current conclusion

**Content completeness:** documentation-complete `DRAFT`. The requested learning path and practical blueprints are
present.

**Real-world validation:** not yet performed for project training, model conversion, compression, AWS deployment,
mobile builds, physical-device measurements, current prices, final model selection, or qualified domain approval.
Therefore the package is not `VERIFIED`, `REVIEWED`, `RELEASE-CANDIDATE`, or publication-ready.

To close that gap, freeze one candidate revision; resolve exact dependencies, models, data, and targets; perform the
documented checks; retain successful and failed runs; obtain rights, safety, technical, statistical, and editorial
reviews; then have every reviewer approve the same candidate hash.

## Ledger guide

- [Prompt coverage](prompt-coverage-ledger.md): what learners are taught and the outcome of each content block.
- [Evidence status](evidence-status-ledger.md): which kinds of statements are supported and which are not.
- [Compatibility and recency](compatibility-and-recency-ledger.md): what can change and how to verify exact tuples.
- [Safety and domain review](safety-and-domain-review-ledger.md): hazards, expert checks, and release stops.
- [Cross-links and terminology](cross-link-and-terminology-ledger.md): navigation, lint, naming, and scan results.
- [Open issues and omissions](open-issues-and-omissions.md): remaining work in practical order.
- [Deliverable readiness](deliverable-readiness-ledger.md): what each deliverable can currently be used for.
- [Implementation verification](implementation-verification-ledger.md): how to turn listings into tested artifacts.
- [Figures and tables](figure-table-ledger.md): whether required teaching aids exist and whether they contain data.

Re-run the relevant ledger after a source, license, API, dependency, model, data snapshot, target device, price, run,
review, or release decision changes. Until named people are assigned, each responsibility is explicitly unassigned;
the closure method in the applicable ledger remains the source of truth.
