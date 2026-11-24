# Study and technical-writing workflow

## Purpose and learning state model

Use this workflow to convert reading and experiments into material that another engineer can understand and reproduce.
Writing is part of learning: it exposes gaps in mechanism, evidence, and assumptions.

Allowed content states are:

`OUTLINE → RESEARCHING → DRAFTED → TECHNICALLY_REVIEWED → VALIDATED_AS_APPLICABLE → AUDITED → PUBLICATION_READY`

A chapter can be understood conceptually while its implementation remains `UNVERIFIED IMPLEMENTATION` and its trials
remain `PLANNED EXPERIMENT`. Keep conceptual mastery, implementation verification, and experimental evidence separate.

## Learner responsibilities

- **Learner:** derive mechanisms, run exercises, retain evidence, and state uncertainties.
- **Study partner:** reproduce one result and challenge causal claims.
- **Domain reviewer:** review terminology, intended use, hazards, and escalation criteria.
- **Technical reviewer:** inspect equations, implementation, measurements, and trade-offs.

One person may perform several responsibilities, but safety-critical content and final release decisions require an
independent qualified reviewer.

## Per-chapter learning cycle

1. **Preview the contract**
   - Read the chapter outcome and prerequisites in the [detailed contents](detailed-table-of-contents.md).
   - Attempt the prerequisite exit questions before reading the chapter.
   - Record the decision the chapter should enable, not merely its vocabulary.

2. **Build a concept map**
   - Write the problem, intuition, mechanism, inputs, outputs, resource costs, assumptions, and failure modes.
   - Connect each new concept to an earlier chapter and identify any forward reference.
   - Explain the map aloud without notes; revisit prerequisites if the explanation fails.

3. **Build the source and claim map**
   - Prefer original papers, standards, official docs/model cards and reproducibility work.
   - Create full source records; identify exact claim, applicability, limitation, evidence label, checked date and
     expiry trigger.
   - Triangulate consequential claims. A vendor benchmark is not independent evidence.
   - Exit test: no central claim depends on an uncitable snippet, invented identifier, or expired compatibility record.

4. **Write the teaching sequence**
   - Use problem → intuition → definition → mechanism → illustration/example → necessary mathematics → implementation →
     strengths/weaknesses/resources → alternatives → domain/edge application → references.
   - Define symbols and units; distinguish fact, reported finding, project hypothesis, estimate and illustration.
   - Exit test: a study partner can reproduce the explanation and numerical example.

5. **Reconstruct tables and figures**
   - Reserve IDs in the [figure/table ledger](../15-audit-ledgers/figure-table-ledger.md).
   - Follow the [diagram plan](diagram-plan.md). Provide caption, alt text, legend, units, editable source and
     provenance.
   - Measured charts require immutable data, protocol, and analysis. Educational graphics must be labelled
     `ILLUSTRATIVE`.
   - Exit test: the visual remains understandable in grayscale and without the surrounding prose.

6. **Run the smallest useful implementation**
   - Put canonical executable-intent material in the relevant companion and link it from the chapter.
   - Pin dependency candidates; show prerequisites, commands, expected artifacts, failure handling and validation.
   - Until executed in the declared environment, label `UNVERIFIED IMPLEMENTATION`. Never fabricate console output.
   - Experiments use `06-experiment-framework/`; preregister before confirmation and retain all attempts.
   - Exit test: a clean-environment smoke run produces the documented artifact or a retained, explained failure.

7. **Validate in proportion to the claim**
   - Static: syntax/schema/type/link/security review where available.
   - Software: clean install, smoke, fixture, save/resume, conversion and parity.
   - Cloud: exact Region/service/image/instance execution, security, load, cost and teardown.
   - Edge: signed release on named physical device, placement, memory, cold/warm/sustained/lifecycle/update evidence.
   - Promote only the claim actually tested. Documentation verification is not runtime validation.

8. **Complete retrieval practice**
   - Add review questions, engineering exercises, practical assignment and milestone task with answer/rubric path.
   - Assess mechanism and trade-off reasoning, not trivia alone; avoid leaking sealed project tests.
   - Exit test: answer the review questions after a 48-hour delay and complete the practical task without copying the
     chapter procedure.

9. **Obtain specialist review**
   - Technical review for every chapter; data/rights, security, cloud, edge, statistics or domain review as applicable.
   - Pharmacy, hazardous repair and animal-care content require qualified safety review and explicit limits.
   - Exit test: blocking findings are resolved or recorded as explicit limitations.

10. **Run an integration audit**
    - Validate relative links/anchors, Mermaid, fences, Markdown lint, code identifiers, terminology, CH/FIG/TBL
      numbering and reciprocal links.
    - Check that no planned result is represented as executed.
    - Update prompt coverage, evidence, compatibility, figure/table and implementation ledgers.
    - Gate: zero unexplained validation errors.

11. **Apply the definition of done**
    - The Managing Editor reviews the [definition of done](../00-governance/definition-of-done.md), unresolved
      limitations, evidence statuses and publication scope.
    - `PUBLICATION_READY` requires required reviews and artifacts; missing physical/cloud execution may be acceptable
      only when the text explicitly teaches an unverified procedure rather than claims a result.

## Change control

- Source, model, runtime, dependency, platform or AWS changes reopen affected recency and compatibility checks.
- Metric, dataset, gate or hypothesis changes after preregistration create a new plan; never rewrite observed history.
- Canonical-path changes require redirect/migration, link audit and ledger updates.
- Corrections create superseding records and retain prior hashes.
- A reviewer cannot waive rights, critical safety, identity, sealed leakage or fabricated-evidence prohibitions.

## Required handoff packet

Every peer-review request includes the chapter path/commit, concept map, dependency list, source/claim map,
figures/tables/listings, validation records, assessment answers, and unresolved limitations. This packet allows the
reviewer to test understanding rather than proofread prose only.

## Worked weekly pattern

**Illustrative 12-hour week:** two hours previewing and deriving, three hours reading primary material, three hours
implementing, two hours measuring and interpreting, one hour writing, and one hour delayed retrieval practice. Replace
these allocations with measured time after two weeks. If implementation consumes the week, retain the failure, reduce
the fixture, and do not omit interpretation or retrieval practice.

## Roadmap completion sweep

Check all 47 chapters, 12 deliverables, 26 mandatory plus seven production figures, 25 tables, 11 learning tools, and
eight target classes. Verify the glossary, bibliography, licenses, links, fences, Mermaid, and Markdown lint; obtain
expert, legal, security, and accessibility review where applicable; and publish unresolved limitations. Unexecuted
validation remains visible.

## Cross-links

- [Governance](../00-governance/README.md)
- [Chapter template](../02-textbook/front-matter/chapter-authoring-template.md)
- [Milestones](milestones.md)
