# Learning assessments

These 47 chapter companions are designed to improve learning, not only measure it. Each combines retrieval practice,
model answers, worked reasoning, evidence critique, an engineering walkthrough, safety transfer, and a chapter-specific
rubric. The assessment is not evidence that an experiment or deployment ran.

## Learning cycle

1. **Retrieve before rereading.** Answer the chapter’s concept questions closed-book. Retrieval is effortful; that
   effort strengthens later recall better than repeatedly recognizing familiar prose.
2. **Correct with explanation.** Compare with the model answer, explain why your reasoning differed, and recompute the
   worked example. Copying the answer is not correction.
3. **Apply in an artifact.** Complete the engineering exercise with immutable inputs, tests, raw outputs, failures, and
   an honest evidence label.
4. **Review on a spacing schedule.** Reattempt missed questions after 2 days, 1 week, 3 weeks, and 8 weeks. Shorten or
   extend the interval based on successful recall, not calendar completion.
5. **Interleave related chapters.** Mix adjacent distinctions—for example CPT versus SFT, quantization versus pruning,
   and prefill versus decode—so the learner must choose the correct mechanism rather than repeat a chapter pattern.

Keep a review log with chapter, item, date, confidence before checking, result, misconception, and next review date.
Confidence without correctness is a calibration signal and should trigger an earlier review.

## Feedback and scoring

Each chapter is scored out of 100:

- retrieval and concepts: 20;
- mechanism/calculation: 15;
- evidence evaluation: 15;
- engineering decision: 15;
- domain, edge, AWS, and safety transfer: 10;
- engineering artifact: 20; and
- milestone handoff: 5.

Passing requires 75 overall and at least half credit in every dimension. Fabricated evidence, hidden leakage or failed
runs, false execution status, prohibited runtime RAG, or unsafe consequential advice cannot be offset by points
elsewhere. Model answers are concise exemplars, not strings that must be copied; equally rigorous alternatives earn
credit when assumptions and evidence are explicit.

## Milestone projects

The milestone handoffs form one cumulative engineering portfolio:

- **CH-01–CH-09 — Model and adaptation brief:** architecture, checkpoint, training objective, SFT, preference, PEFT, and
  LoRA decisions.
- **CH-10–CH-15 — Compression package:** resource baseline, quantization, pruning, distillation, additional methods, and
  an ordered pipeline.
- **CH-16–CH-22 — Edge qualification:** device envelopes, hardware/backend evidence, Android/iOS designs, sustained
  optimization, and a third edge platform.
- **CH-23–CH-28 — Data and platform package:** rights-aware sources, lineage-preserving datasets, objective views,
  synthetic-data controls, multi-domain architecture, and reusable orchestration.
- **CH-29–CH-32 — Evaluation package:** metric definitions, domain suites, controlled device benchmarks, ablations, and
  Pareto rules.
- **CH-33–CH-36 — AWS and delivery package:** least-privilege infrastructure, recoverable training, serving, signing,
  resumable delivery, and rollback.
- **CH-37 — Automotive capstone:** integrate phases 1–17 and make separate best-accuracy, mobile, balanced, low-memory,
  and AWS-cost selections.
- **CH-38–CH-44 — Domain and operations qualification:** pharmaceutical, mechanic, animal-care, onboarding, security,
  lifecycle, and total-cost gates.
- **CH-45–CH-47 — Research and production defense:** preregistration, staged benchmark campaign, and final architecture.

Milestones link immutable prior artifacts rather than copying them. Superseded versions remain traceable with a reason
for replacement.

## Assessment index

- [CH-01: Evolution of Language Models](ch-01-assessment.md)
- [CH-02: Transformer Architecture in Depth](ch-02-assessment.md)
- [CH-03: Modern SLM Architectures](ch-03-assessment.md)
- [CH-04: Pretraining Fundamentals](ch-04-assessment.md)
- [CH-05: Continued Pretraining and Domain Adaptation](ch-05-assessment.md)
- [CH-06: Supervised Fine-Tuning](ch-06-assessment.md)
- [CH-07: Preference Optimization and Post-Training](ch-07-assessment.md)
- [CH-08: Full Fine-Tuning Versus PEFT](ch-08-assessment.md)
- [CH-09: LoRA and Its Evolution](ch-09-assessment.md)
- [CH-10: The Fundamentals of Model Compression](ch-10-assessment.md)
- [CH-11: Quantization — Complete Research and Implementation](ch-11-assessment.md)
- [CH-12: Pruning — Complete Research and Implementation](ch-12-assessment.md)
- [CH-13: Knowledge Distillation — Complete Research and Implementation](ch-13-assessment.md)
- [CH-14: Additional Model Compression Techniques](ch-14-assessment.md)
- [CH-15: Combining Optimization Techniques](ch-15-assessment.md)
- [CH-16: Edge AI Fundamentals](ch-16-assessment.md)
- [CH-17: Mobile Hardware Architecture](ch-17-assessment.md)
- [CH-18: Edge Inference Frameworks](ch-18-assessment.md)
- [CH-19: Mobile Inference Optimization](ch-19-assessment.md)
- [CH-20: Android Deployment](ch-20-assessment.md)
- [CH-21: iOS Deployment](ch-21-assessment.md)
- [CH-22: Other Edge Devices](ch-22-assessment.md)
- [CH-23: Finding Domain-Specific Training Data](ch-23-assessment.md)
- [CH-24: Dataset Collection and Processing](ch-24-assessment.md)
- [CH-25: Dataset Design for Different Training Objectives](ch-25-assessment.md)
- [CH-26: Synthetic Data and Knowledge Generation](ch-26-assessment.md)
- [CH-27: Single-Domain Versus Multi-Domain Strategies](ch-27-assessment.md)
- [CH-28: Reusable Domain Training Platform](ch-28-assessment.md)
- [CH-29: Model Quality Evaluation](ch-29-assessment.md)
- [CH-30: Domain-Specific Evaluation Suites](ch-30-assessment.md)
- [CH-31: Compression and Edge Benchmarking](ch-31-assessment.md)
- [CH-32: Optimization Experiments and Ablation Studies](ch-32-assessment.md)
- [CH-33: AWS Infrastructure for SLM Development](ch-33-assessment.md)
- [CH-34: Training SLMs on AWS](ch-34-assessment.md)
- [CH-35: AWS Inference Serving](ch-35-assessment.md)
- [CH-36: AWS-to-Edge Model Delivery](ch-36-assessment.md)
- [CH-37: Build an Automotive Mechanic SLM End to End](ch-37-assessment.md)
- [CH-38: Pharmaceutical Knowledge SLM](ch-38-assessment.md)
- [CH-39: Motorcycle and Truck Mechanic SLMs](ch-39-assessment.md)
- [CH-40: Beekeeping and Zookeeping SLMs](ch-40-assessment.md)
- [CH-41: New Domain Onboarding](ch-41-assessment.md)
- [CH-42: Model Security and Reliability](ch-42-assessment.md)
- [CH-43: Monitoring and Model Lifecycle](ch-43-assessment.md)
- [CH-44: Cost Optimization](ch-44-assessment.md)
- [CH-45: Research Questions and Hypotheses](ch-45-assessment.md)
- [CH-46: Benchmark Matrix](ch-46-assessment.md)
- [CH-47: Proposed Production System](ch-47-assessment.md)

## Governing standards

- [Definition of done](../00-governance/definition-of-done.md)
- [Evidence and citation](../00-governance/evidence-and-citation-standard.md)
- [Reproducibility](../00-governance/reproducibility-standard.md)
- [Safety and risk](../00-governance/safety-and-risk-standard.md)
- [Cross-reference](../00-governance/cross-reference-standard.md)
- [No-runtime-RAG](../00-governance/no-runtime-rag-standard.md)
