# Learner milestones

## How to use the milestones

Milestones are observable demonstrations of competence, not calendar or manuscript statuses. Complete them in
dependency order, retain the named evidence, and ask another engineer to reproduce a representative artifact. Reading
alone does not satisfy a milestone, and a documentation check does not establish implementation or experiment success.

## M0 — Evidence discipline and navigation

- **Learner outcome:** navigate the complete package and apply its evidence vocabulary without overstating a claim.
- **Dependencies:** `prompt.txt`.
- **Paths:** `BOOKMAP.md`, `00-governance/`, `01-planning/`, `15-audit-ledgers/`.
- **Exit gate:** stable paths and IDs; all planning files authored; owners/dependencies/gates explicit; links, fences,
  Mermaid and Markdown lint pass; statuses distinguish authored specifications, outlines and unexecuted work.
- **Evidence:** validation logs and clean review record.

## M1 — Verified research and recency baseline

- **Learner outcome:** verify a fast-moving claim from a primary source and define its expiry trigger.
- **Dependencies:** M0.
- **Paths:** `01-planning/initial-research-survey.md`, `01-planning/fast-moving-topics-watchlist.md`,
  `12-reference-library/`, compatibility/evidence ledgers.
- **Exit gate:** primary/official source records for every chapter cluster; model/license/runtime/AWS shortlist checked;
  limitations and expiry triggers recorded; no “latest” or unsupported superior-performance claims.
- **Evidence:** bibliography/source records, dated ledger and review.

## M2 — Foundations, adaptation, and PEFT

- **Learner outcome:** explain the model computation, build an objective-specific dataset, and train a reproducible
  LoRA adaptation.
- **Dependencies:** M1.
- **Paths:** CH-01–09, corresponding assessments and research comparison sections.
- **Exit gate:** chapter definition-of-done, FIG-02–06 and TBL-01–05/11, cited equations/examples, statically reviewed
  listings and assessment keys.
- **Evidence:** chapter review records and artifact ledger.

## M3 — Compression and edge deployment

- **Learner outcome:** convert a qualified artifact, account for memory, and validate offline inference on physical
  hardware.
- **Dependencies:** M1–M2 and M4 experiment contracts.
- **Paths:** CH-10–22, `09-edge-guide/`, relevant comparison report sections.
- **Exit gate:** FIG-07–19/25 and TBL-06–17; runtime compatibility records; executable-intent conversion/app paths;
  explicit physical-device validation instructions; no unmeasured performance claim.
- **Evidence:** reviews, locked listings, compatibility ledger and blocked-cell report.

## M4 — Data and evaluation design

- **Learner outcome:** create rights-aware dataset lineage, leakage-safe splits, risk-stratified evaluation, and a
  preregistered protocol before adaptation.
- **Dependencies:** M0–M1.
- **Paths:** CH-23–32, `05-dataset-guide/`, `06-experiment-framework/`.
- **Exit gate:** rights/provenance and objective dataset specifications; schemas; domain evaluation rubrics; every
  formal protocol and metric/environment/provenance/Pareto specification reviewed; known-answer fixtures designed.
- **Evidence:** schema/protocol reviews and link/fence/lint validation. No run is required to close the specification
  portion; executable status remains unverified.

## M5 — AWS implementation

- **Learner outcome:** deploy, load-test, secure, roll back, cost, and tear down an exact AWS serving tuple.
- **Dependencies:** M1, M3–M4 and approved AWS target.
- **Paths:** CH-33–36, `08-aws-guide/`, AWS protocol.
- **Exit gate:** pinned IaC/container/endpoint implementation, security review, smoke/load/rollback/teardown execution
  in an exact Region and dated cost evidence.
- **Evidence:** raw traces, IaC plans, endpoint revisions, billing/price capture and teardown proof.

## M6 — Automotive flagship execution

- **Learner outcome:** execute the automotive lineage from approved scope through at least one target decision, or
  produce evidence-based rejection and blocked records.
- **Dependencies:** M1–M5 as required by phase.
- **Paths:** CH-37, `07-automotive-case-study/`.
- **Exit gate:** all feasible phases produce immutable accepted/rejected/inconclusive records; unavailable resources are
  explicit; no phase is called executed without raw evidence; at least one qualified target required for a release
  claim.
- **Evidence:** phase manifests, datasets, checkpoints, exports, reviews, failures and decisions.

## M7 — Domain extension and production controls

- **Learner outcome:** separate reusable platform elements from domain-specific data, safety, regulation, and
  evaluation requirements.
- **Dependencies:** M4 and lessons from M6.
- **Paths:** CH-38–44, `10-multi-domain-architecture/`, domain guides.
- **Exit gate:** pharmaceutical/mechanical/animal-care delta plans, new-domain template, security threat model,
  lifecycle and cost controls, qualified-review requirements and rollback.
- **Evidence:** domain/safety/security/legal/FinOps reviews.

## M8 — Confirmatory experiments and decisions

- **Learner outcome:** run confirmatory trials and make uncertainty-aware, target-specific Pareto decisions.
- **Dependencies:** qualified M6 candidates and available targets.
- **Paths:** CH-45–46, `11-research-roadmap/`, experiment results, decision framework.
- **Exit gate:** preregistered confirmation runs, retained failures, uncertainty/multiplicity analysis, separate target
  fronts and five final automotive decision categories or explicit no-candidate outcomes.
- **Evidence:** content-addressed plans, attempts, raw data, analyses and reviewed decisions.

## M9 — Integration and full audit

- **Learner outcome:** audit the complete evidence chain, documentation, implementation status, safety, and known
  limitations.
- **Dependencies:** M1–M8 deliverables intended for release.
- **Paths:** entire repository and `15-audit-ledgers/`.
- **Exit gate:** prompt coverage; chapter/deliverable completeness; links/fences/Mermaid/lint;
  IDs/captions/index/glossary; licenses/citations; code status; reproducibility; safety/expert reviews; unresolved
  omissions and errata.
- **Evidence:** signed audit checklist and zero unexplained errors.

## M10 — Reproducible portfolio release

- **Learner outcome:** publish a versioned, independently reviewable learning portfolio whose claims match its
  evidence.
- **Dependencies:** M9.
- **Paths:** release snapshot.
- **Exit gate:** technical, domain-safety, security, legal/license, accessibility and copy-edit approvals; immutable
  release manifest; all claims carry correct evidence status; unexecuted work remains clearly labelled.
- **Evidence:** approval record, release hash, archive and errata channel.

## Dependency and change rule

Milestones are sequential where named, but writing may proceed in parallel when prerequisites are satisfied. A changed
source, dependency, target or safety rule reopens affected milestones. Blocked cloud, hardware, rights or expert-review
work is reported as `BLOCKED`; scope reduction requires Publication Board approval and prompt-coverage disclosure.

## Worked pacing examples

**Illustrative part-time pacing:** M0–M1 in weeks 1–2, M2 in weeks 3–8, M4 in parallel during weeks 4–7, M3 in weeks
9–14, M5 in weeks 15–16, M6 in weeks 17–21, and M7–M10 in weeks 22–24. **Illustrative intensive pacing:** complete the
same groups in 12 weeks while preserving every dependency and review gate. Replace these durations after measuring the
first foundation, training, conversion, and deployment tasks; do not compress a milestone whose competence evidence is
incomplete.

At each milestone, conduct a 30-minute oral defense, a clean-environment reproduction of one artifact, and a written
decision memo. Record the measured preparation and execution time to improve the remaining schedule.

## Cross-links

- [Bookmap](../BOOKMAP.md)
- [Definition of done](../00-governance/definition-of-done.md)
- [Authoring workflow](authoring-workflow.md)
