# Source record tutorial

A source record answers three questions: **What is this source? What does it actually support? What still needs
checking?** The first record is a realistic illustration, not evidence that this project reproduced the paper.

## Worked example: the LoRA paper

### Identity

- **Citation key:** `hu2022lora`
- **Record status:** reviewed
- **Source class:** primary research
- **Title:** “LoRA: Low-Rank Adaptation of Large Language Models”
- **Authors:** Edward J. Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, and
  Weizhu Chen
- **Publication:** ICLR 2022 conference paper
- **Archival identifiers:** arXiv:2106.09685; OpenReview `nZeVKeeFYf9`
- **Canonical URL:** https://openreview.net/forum?id=nZeVKeeFYf9
- **Code:** https://github.com/microsoft/LoRA
- **Accessed:** 2026-11-23
- **Language:** English

### Authority and rights

- **Why primary:** the authors introduce and evaluate LoRA in this paper.
- **Paper access:** public through OpenReview.
- **Code license:** MIT according to the repository license; verify the license at the pinned commit before reuse.
- **Project rights:** citing the paper is distinct from copying code, training data, or model weights. Review each
  artifact's terms independently.
- **Vendor role:** several authors were affiliated with Microsoft; treat performance results as paper-reported, not as
  independent project measurements.

### Technical review

- **Problem:** full fine-tuning stores and updates a complete parameter set for every task.
- **Contribution:** LoRA freezes pretrained weights and learns low-rank matrices whose product forms an additive update
  to selected weight matrices.
- **Reported findings:** the paper reports large reductions in trainable parameters and competitive task quality in its
  tested model, dataset, and hyperparameter settings. Use the paper's tables for exact values and conditions.
- **Methods:** experiments cover Transformer language models and natural-language tasks; the exact model and dataset
  differ by experiment.
- **Limits stated or found:** low rank is not guaranteed to suit every target; results do not prove parity for a new
  domain; a merged adapter does not reduce dense inference size; the paper does not establish mobile compatibility.
- **Project use:** foundational support for the LoRA mechanism and a baseline for PEFT experiments.
- **Must not support:** claims that LoRA always matches full fine-tuning, reduces KV-cache memory, or runs faster on a
  particular phone.

### Reproduction and claim traceability

- **Official code:** repository URL above; pin a commit before execution.
- **Project execution:** not executed in this illustrative record.
- **Integrity:** no artifact downloaded; record hashes after download.

| Claim location | Supported statement | Source location | Evidence | Qualification |
| --- | --- | --- | --- | --- |
| Chapter 9, LoRA mechanism | LoRA learns low-rank additive updates while freezing base weights. | Sections 2–4 | `PAPER-REPORTED` | Mechanism only |
| Chapter 9, efficiency | The paper reports fewer trainable parameters in its experiments. | Tables 1–2 | `PAPER-REPORTED` | Preserve model and task conditions |

### Recency and decision

- **Recency check:** on 2026-11-23, confirm the OpenReview record and inspect the official repository release history.
- **Decision:** approved for explaining the original method; new empirical claims require project runs.
- **Next check:** before freezing code dependencies or publishing a comparison.

## Field-by-field instructions

1. **Identity:** copy the title and ordered authors from the primary record, not a search snippet. Prefer a DOI,
   proceedings page, arXiv identifier, release tag, or commit. Record the access date for mutable pages.
2. **Authority and rights:** explain why the source is original or official. Link exact terms. Describe permitted acts
   factually; request legal review when terms are ambiguous.
3. **Technical review:** separate the problem, method, findings, and limitations. Preserve model, data, hardware,
   metric, and uncertainty conditions around every number.
4. **Artifacts:** pin code, data, and model revisions and hash downloaded files. “Code available” does not mean
   reproduced.
5. **Traceability:** make one row per consequential claim and point to a page, section, table, figure, or documentation
   heading.
6. **Recency:** for mutable sources, state how and when status was checked and name the event that triggers another
   check.
7. **Unknown metadata:** do not leave a blank or write a vague unresolved marker. State an action, such as “query
   Crossref by exact title and compare authors before approval.”

## Reusable copy

Replace the descriptive example values below. They deliberately show the expected form and level of detail.

### Identity

- **Citation key:** `lead-author-year-short-title`
- **Record status:** candidate requiring metadata review
- **Source class:** primary research paper
- **Title:** Exact title copied from the proceedings record
- **Authors or organization:** Ordered author list copied from the primary source
- **Publication:** 2026 conference paper; verify against the proceedings page
- **Identifiers:** DOI copied from Crossref and arXiv identifier copied from arXiv
- **Canonical URL:** Direct DOI, proceedings, or official documentation URL
- **Immutable revision:** DOI, release tag, or full commit hash
- **Accessed:** 2026-11-24
- **Language:** English

### Authority and rights

- **Why primary or official:** The authors introduce the method in this paper
- **License or terms:** Exact license name, version, and direct terms URL
- **Rights relevant to project:** Factual summary of reading, code reuse, training, modification, and redistribution
- **Legal review:** required because redistribution language needs review
- **Conflicts or vendor role:** Author affiliation or “none identified after checking the paper”

### Technical review

- **Problem and scope:** One paragraph defining the problem and experimental boundary
- **Main contribution:** One paragraph describing the new mechanism
- **Reported findings:** Exact result with model, data, baseline, metric, and uncertainty
- **Methods and environment:** Dataset, split, model, hardware, software, sample size, and repetitions
- **Source limitations:** Limitations stated by the authors
- **Reviewer limitations:** Transfer, confounding, licensing, recency, and external-validity concerns
- **Project applicability:** Direct explanation, conditional experiment, historical context, or comparison only
- **Claims this source cannot support:** Explicitly excluded inferences

### Evidence and review

- **Code and artifacts:** Pinned URLs, revisions, and hashes, or a stated verification action before approval
- **Project execution:** Not executed; create run record `run-YYYYMMDD-method-device` before using `EXECUTED`
- **Claim row:** Chapter section; exact statement; source section/table; evidence label; scope limit
- **Recency check:** 2026-11-24; inspect official release history and status notice
- **Decision:** Candidate until metadata, rights, and claim locations pass review
- **Next check:** At experiment freeze and before release approval

### Acceptance checks

- [ ] Primary-source title, authors, date, status, and identifiers agree.
- [ ] URLs resolve to the intended work and mutable sources have access dates.
- [ ] Findings retain their experimental conditions and evidence labels.
- [ ] Rights are recorded without unsupported legal conclusions.
- [ ] Every consequential claim has a precise source location and qualification.
- [ ] Every unknown has a named verification action and blocks approval where necessary.

Cross-links: [bibliography](bibliography.md) ·
[evidence standard](../00-governance/evidence-and-citation-standard.md) ·
[reproducibility standard](../00-governance/reproducibility-standard.md)
