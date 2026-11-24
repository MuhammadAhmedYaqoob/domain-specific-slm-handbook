# Fast-moving topics verification guide

## Why and how to verify

Model cards, licenses, framework APIs, conversion support, mobile operating systems, accelerator kernels, AWS
availability, and prices can change faster than textbook prose. The learning goal is therefore not memorizing a
snapshot; it is learning a repeatable verification method. Only links explicitly inspected in the
[initial survey](initial-research-survey.md) carry its 2026-11-23 documentation check, and documentation checks do not
establish an executed compatibility tuple.

Every record below has an accountable role, canonical affected paths, upstream dependency, verification gate, and
trigger. The owner writes exact versions, immutable revisions, authoritative URLs, checked date, supported/unsupported
combinations, and execution evidence to the
[compatibility and recency ledger](../15-audit-ledgers/compatibility-and-recency-ledger.md). “Latest” is prohibited.

For every watch record, use this six-step loop:

1. State the exact decision and tuple that depends on the claim.
2. Inspect the primary source, immutable revision or release notes, license, and known limitations.
3. Record the checked date, version, affected paths, and next trigger.
4. Reproduce the smallest local smoke test that could disprove compatibility.
5. Run parity, safety, resource, and target tests in proportion to the eventual claim.
6. Mark the outcome with the allowed vocabulary and retain unsupported paths and failures.

Verification occurs at three times: when a topic is first studied, immediately before implementation, and immediately
before a release decision. A scheduled review never overrides an event trigger such as a changed model, runtime, OS,
driver, Region, price, license, security advisory, or regulation.

## Watch records

### W-01 — Model checkpoints and licenses

- **Owner:** Model Research Lead.
- **Paths:** CH-03, CH-37, `07-automotive-case-study/phase-02-select-a-baseline-model.md`, target records under
  `14-decision-framework/targets/`.
- **Dependencies:** publisher model card, weight repository, license text, tokenizer/template, immutable revision.
- **Gate:** identity hash, license/use approval, base/instruct classification, architecture/config inspection,
  model-card limitation capture, and target export probe.
- **Trigger:** candidate change, repository revision, license change, release planning, or 30 days since selection
  evidence.

### W-02 — Training stack and PEFT

- **Owner:** Training Lead.
- **Paths:** CH-04–09, CH-34, implementation handbook, automotive phases 06–09.
- **Dependencies:** Python, PyTorch, CUDA/driver, Transformers, Datasets, PEFT, TRL, Accelerate, bitsandbytes, tokenizer
  and model architecture.
- **Gate:** resolved lock, clean-environment install, dependency check, deterministic smoke train, save/resume, adapter
  merge parity, and license/security scan.
- **Trigger:** any dependency/model/CUDA change or failed reproducibility run.

### W-03 — Quantization formats and kernels

- **Owner:** Compression Lead.
- **Paths:** CH-10–11, CH-15, CH-18–22, quantization protocol, automotive phases 10/12/15.
- **Dependencies:** source model, calibration manifest, converter revision, format, runtime kernel, backend, target
  hardware.
- **Gate:** conversion completes; unsupported/fallback operators recorded; source/export semantic and safety parity
  passes; file, loaded/peak-memory and latency measured on exact target.
- **Trigger:** runtime/converter/model/OS/backend change or new target.

### W-04 — Pruning and sparse execution

- **Owner:** Compression Lead.
- **Paths:** CH-12/15/31–32, pruning protocol, automotive phases 11–12.
- **Dependencies:** pruning code, recovery training, dense/sparse representation, supported kernel and hardware.
- **Gate:** parameter/shape/sparsity inventory, recovery evidence, exact executable artifact, and measured
  latency/memory against dense control. Nominal zeros alone fail.
- **Trigger:** library/kernel update, hardware change, or changed sparsity pattern.

### W-05 — Distillation tooling and teacher policy

- **Owner:** Distillation Lead; Safety Reviewer approves teacher data.
- **Paths:** CH-13/26/37, distillation protocol, automotive phase 09.
- **Dependencies:** teacher license/terms, prompt and data rights, tokenizer/logit alignment, expert review, student
  architecture.
- **Gate:** retained teacher provenance and outputs, factual/safety sampling, cost record, leakage check, loss-path
  smoke test, and student comparison to direct-data control.
- **Trigger:** teacher/model/API/terms/prompt/data change.

### W-06 — Edge runtimes and formats

- **Owner:** Edge Runtime Lead.
- **Paths:** CH-18–22/31/36, `09-edge-guide/`, edge protocol, automotive phase 15.
- **Dependencies:** llama.cpp/GGUF, ExecuTorch, LiteRT-LM, ONNX Runtime Mobile, Core ML, MLX Swift, MLC LLM, MNN, NCNN
  or vendor runtime; exact model operators and tokenizer.
- **Gate:** official support evidence plus local export/load/generation; backend placement proof; semantic parity;
  memory, sustained latency, lifecycle, crash and offline-update tests.
- **Trigger:** runtime commit/release, converter or model revision, app SDK/OS/firmware change, or 30 days before
  release.

### W-07 — Android acceleration

- **Owner:** Android Lead.
- **Paths:** CH-17–20/31, Android guide and target records.
- **Dependencies:** Android API/NDK/Gradle/Kotlin, ABI, CPU/GPU/NPU backend, vendor SDK, app lifecycle and packaging.
- **Gate:** signed release/profileable build on named physical device; placement and fallback evidence;
  cold/warm/sustained/lifecycle runs.
- **Trigger:** device/OS/security patch, SDK/runtime/app change, or release candidate.

### W-08 — Apple acceleration

- **Owner:** iOS Lead.
- **Paths:** CH-17–19/21/31, iOS guide and modern-iPhone target.
- **Dependencies:** Xcode/Swift/iOS, Core ML/Metal/MLX Swift or ExecuTorch, signing, model format and operators.
- **Gate:** signed Release build on named physical iPhone; compute-unit/placement evidence; parity, memory, thermal,
  energy where measurable, background and update tests.
- **Trigger:** iOS/Xcode/runtime/model/app change or release candidate.

### W-09 — Other edge and WebGPU

- **Owner:** Edge Systems Lead.
- **Paths:** CH-22/31, ARM/Jetson/laptop targets.
- **Dependencies:** OS/kernel/driver/browser, CPU ISA, GPU/NPU, cooling, runtime and WebGPU implementation.
- **Gate:** exact environment inventory and native benchmark; browser support or simulator output never substitutes for
  physical target evidence.
- **Trigger:** browser/driver/runtime/firmware change or new target.

### W-10 — AWS compute, services, and prices

- **Owner:** Cloud Lead; FinOps Reviewer approves cost evidence.
- **Paths:** CH-33–36/44, `08-aws-guide/`, AWS protocol and target records.
- **Dependencies:** Region, account quota, EC2/SageMaker AI/ECR/S3/IAM/VPC/CloudWatch APIs, instance/AMI/driver/CUDA,
  purchase model and Price List.
- **Gate:** current Region availability/quota; digest-pinned image; deployed smoke and load tests; IAM/network/security
  review; dated price/bill capture; automated teardown.
- **Trigger:** each execution day for price/availability, image or driver change, service/API announcement, Region
  change, or 14 days before release.

### W-11 — Server inference runtimes

- **Owner:** Serving Lead.
- **Paths:** CH-35, AWS guide and AWS inference protocol.
- **Dependencies:** vLLM, SGLang, TensorRT-LLM, llama.cpp server, ONNX Runtime, TGI if maintained, model
  architecture/quantization and accelerator.
- **Gate:** official support record, pinned image, output parity, load/stability/observability/security tests and
  unsupported-feature record.
- **Trigger:** runtime/image/model/driver change or security advisory.

### W-12 — MLOps and infrastructure tools

- **Owner:** Platform Lead.
- **Paths:** CH-28/33–36/42–43/47, implementation/AWS/multi-domain companions.
- **Dependencies:** MLflow or selected tracker, Docker, AWS CLI/SDK, CDK or Terraform, CI and artifact signer.
- **Gate:** lock/digest, schema migration and rollback, least-privilege review, reproducible IaC plan/apply/destroy,
  immutable registry lineage.
- **Trigger:** dependency or schema change, provider API change, security advisory, or deployment.

### W-13 — Data authorities and regulation

- **Owner:** Data Governance Lead; Domain Safety Reviewer co-owns consequential sources.
- **Paths:** CH-23–26/30/38–41/42, dataset guide and domain records.
- **Dependencies:** source license/terms, jurisdiction, version/effective date, provenance and qualified reviewer
  availability.
- **Gate:** rights approval, authoritative-version pin, update schedule, content review, conflict resolution and
  descendant takedown path.
- **Trigger:** source update, terms/regulation change, jurisdiction change, complaint or safety incident.

### W-14 — Benchmarks, evaluators, and contamination

- **Owner:** Evaluation Lead.
- **Paths:** CH-29–32/45–46, experiment framework and assessments.
- **Dependencies:** benchmark version, item roots, evaluator/rubric, model exposure, statistical plan.
- **Gate:** frozen manifests, contamination review, scorer fixtures, blind/qualified review where required, uncertainty
  and multiplicity plan.
- **Trigger:** benchmark/evaluator/model/data change or evidence of leakage.

## Review outcomes

Allowed outcomes are `CURRENT_DOCS_ONLY`, `EXECUTION_VERIFIED`, `STALE`, `UNSUPPORTED`, `BLOCKED`, or `RETIRED`.
Documentation-only verification must never populate an executed compatibility cell. Stale evidence blocks release but
does not erase historical records.

## Learner exercise and exit test

Choose one model checkpoint, one quantizer, one edge runtime, and one AWS serving runtime. Build four linked records that
pin identity and versions, cite official support, preserve a smoke-test result, and identify exactly what would reopen
each claim. Then change one dependency version and trace every invalidated conclusion. The exercise is complete when
another engineer can rerun the checks without interpreting “current,” “supported,” or “compatible” informally.

## Cross-links

- [Initial survey](initial-research-survey.md)
- [Compatibility ledger](../15-audit-ledgers/compatibility-and-recency-ledger.md)
- [AWS target procedure](aws-target-plan.md)
- [Edge target procedure](edge-target-plan.md)
