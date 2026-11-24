# Prompt coverage ledger

## What is checked and why

This ledger maps `prompt.txt` to teachable repository content. A file count alone is not enough: each row states what
the learner should be able to understand or do after studying that content. This matters because a complete-looking
book can still omit the reasoning, practice, or evidence boundaries needed for independent engineering work.

**Status key:** `Covered` means substantive teaching content and navigation exist. `Covered, external validation not performed` means
the lesson and procedure exist but the claimed real-world outcome has not been demonstrated. `Not yet performed`
means the required external run or review has no retained record.

## Learning-path map

<!-- markdownlint-disable MD013 MD060 -->
| Prompt area | Repository content | Learner outcome | Current status and interpretation |
| --- | --- | --- | --- |
| Foundations and architectures | CH-01–03 | Explain language-model evolution, trace token generation through a decoder-only Transformer, and compare small-model candidates without treating family names as proof of support | Covered; model facts still need candidate-date refresh |
| Training and adaptation | CH-04–09 | Choose among scratch training, CPT/DAPT/TAPT, SFT, preference methods, full tuning, LoRA, and QLoRA; explain their data and compute trade-offs | Covered, external validation not performed; no project training run |
| Compression | CH-10–15 | Calculate memory needs and design quantization, pruning, distillation, and combined experiments while separating file-size reduction from actual speed | Covered, external validation not performed; no compressed artifact or benchmark |
| Edge and mobile | CH-16–22 | Match model, format, runtime, and hardware; reason about memory, prefill/decode, KV cache, battery, and thermal limits; follow Android and iOS deployment paths | Covered, external validation not performed; no app build or physical-device run |
| Data engineering | CH-23–26 | Discover legally usable sources and build objective-specific, traceable, deduplicated, leakage-controlled datasets with human checks | Covered, external validation not performed; no rights-approved project corpus |
| Multi-domain systems | CH-27–28 | Compare independent models, adapters, routing, and shared models; design a reusable no-runtime-RAG platform | Covered, external validation not performed; no two-domain implementation |
| Evaluation and experiments | CH-29–32 | Build domain and safety suites, control benchmark conditions, report uncertainty, and use Pareto analysis without fabricating results | Covered; result tables correctly remain empty until runs occur |
| AWS | CH-33–36 | Select services, train and serve models, secure delivery, monitor cost, and plan rollback from cloud to edge | Covered, external validation not performed; no AWS account/Region execution |
| Automotive project | CH-37 and 17 phase records | Follow an end-to-end specialist-model project from scope and data through training, optimization, cloud/mobile deployment, and evidence-based selection | Covered, external validation not performed; 0/17 phases have retained execution evidence |
| Additional domains | CH-38–41 | Adapt the platform to pharmacy, motorcycle/truck, beekeeping, zookeeping, and a new domain while applying risk-specific limits | Covered; qualified domain review not yet performed |
| Production, safety, and cost | CH-42–44 | Identify supply-chain, privacy, reliability, monitoring, lifecycle, and cost controls and distinguish estimates from bills | Covered; current prices, drills, and external reviews not yet performed |
| Research and architecture | CH-45–47 | Turn open questions into controlled studies, stage a realistic benchmark matrix, and explain the complete production architecture | Covered; hypotheses and diagrams are plans, not project results |
<!-- markdownlint-restore -->

## Required teaching artifacts

<!-- markdownlint-disable MD013 -->
| Requirement | Check result | Why learners need it | How to close the remaining gap |
| --- | --- | --- | --- |
| 47 chapters and 14 parts | 47/47 chapters and 14/14 indexes contain substantive material | Provides a progressive path rather than disconnected notes | Independent technical and editorial review of the frozen text |
| Required comparisons | 25/25 table IDs exist once | Makes alternatives and trade-offs explicit | Populate measured-result schemas only from controlled project runs |
| Required visuals | 26/26 prompt visuals plus 7/7 CH-47 diagrams exist once | Makes data flow, architecture, and trade-offs easier to reason about | Run accessibility and rendered-output review; add measured points only with raw data |
| Learning tools | 11/11 tools are documented and registered | Lets learners calculate memory, cost, parameters, and trade-offs | Extract, lock dependencies, test fixtures and failures, and retain outputs |
| Assessments | 47/47 chapter assessments include tasks and rubrics | Gives learners a way to demonstrate understanding | Curriculum reviewer checks solutions, scoring, and capstone progression |
| Deliverables | 12/12 requested deliverable areas contain useful content | Connects theory, implementation, research, and deployment | Complete each deliverable's empirical and review steps |
| Decision profiles | 8/8 hardware/cloud profiles and 17 criteria are specified | Teaches that one model cannot be “best” without a target and constraints | Benchmark exact candidates; no profile can name a winner yet |
<!-- markdownlint-restore -->

## Final pedagogical pass

The 2026-11-24 learner-first pass sampled every numbered top-level section and read all 47 chapters against the same
sequence: establish the problem in plain language, explain why it matters, introduce the concept before requiring it,
show a worked or illustrative example, interpret the example, and provide practice or an assessment. Checklist-heavy
reference records were also checked for explanation and a filled example. All chapters contain substantive teaching
content and connect to a chapter assessment; no chapter is only a status sheet, checklist, or governance record.

Concrete identifiers in reusable listings are explicitly illustrative. Learners are told to replace them with values
from their own reviewed registry and rerun validation. Shell, Terraform, Docker, Kotlin, and template expressions such
as `${var.region}` remain only where they are executable language interpolation rather than unresolved replacement
tokens.

## Example: passing and failing coverage evidence

**Pass:** the quantization requirement points to CH-11, where a learner can explain scales and zero points, compare
formats and algorithms, calculate memory, follow an experiment protocol, and see that measured results are absent.

**Fail:** a row says only “quantization — complete” or points to a heading with no explanation, example, procedure, or
learning outcome. A diagram or status word by itself also fails.

## Domains, implementation, and startup work

All ten domain classes in the prompt are represented: automotive, motorcycle, heavy truck, pharmacy, beekeeping,
zookeeping and animal care, agricultural equipment, industrial machinery, electronics repair, and future specialist
domains. The implementation material covers the requested Python and ML stack, compression libraries, Docker, AWS,
infrastructure as code, edge runtimes, Kotlin/JNI, and Swift/SwiftUI. All 12 requested pre-writing activities also have
substantive artifacts.

These records show that a learner has a blueprint. They do not show that dependencies resolve, data rights are
approved, code runs, or a model meets a target.

## Final prompt disposition

**Content completeness:** covered as a coherent learning package. The reader is taught all 19 stated success outcomes,
including GPT internals, specialization, compression, data, evaluation, Android/iOS, AWS, multi-domain operation, and
evidence-based selection.

**Real-world validation:** not yet performed for the exact model/data/runtime/hardware combinations. Close this by
following the [implementation ledger](implementation-verification-ledger.md), [safety ledger](safety-and-domain-review-ledger.md),
and [open issues](open-issues-and-omissions.md), preserving every run and review against one frozen candidate.
