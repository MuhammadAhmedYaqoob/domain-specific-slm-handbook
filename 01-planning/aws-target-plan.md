# AWS learning and target-selection plan

## Learning objective and prerequisites

The learner will choose the least complex AWS training and serving tuple that satisfies a measured workload, then
defend the choice with compatibility, reliability, security, performance, and cost evidence. Complete CH-10, CH-28–34,
and the baseline evaluation before provisioning. You should already understand containers, IAM concepts, network
boundaries, GPU memory accounting, percentiles, and open-loop load tests.

Official documentation currently establishes that AWS offers self-managed and managed container paths for model
training/serving, but that does not establish compatibility or economy for this project. Exact targets are chosen on
execution day from current evidence.

The order matters: freeze the artifact and workload before comparing services; calculate feasibility before paying for
pilots; test the cheapest plausible tuple before adding orchestration; and measure tails and failure costs before
optimizing nominal price. Training and serving are separate decisions because their memory, utilization, scaling, and
recovery patterns differ.

## Target profiles

### AWS-S — Small development and low-concurrency inference

Intended for environment validation, short adaptation runs where feasible, single-model functional serving and low
request concurrency. Prefer the least complex qualified design: one isolated account/project, one Region, object
storage, digest-pinned container registry, one compute target, authentication, logs/metrics, budget alarms and automatic
teardown. CPU is a valid candidate only if the exact model and SLO qualify; GPU is not assumed.

Choose this profile first because it teaches the complete deployment loop without hiding the model behind unnecessary
platform complexity. A single EC2 host offers maximum observability and operational responsibility; a SageMaker AI
endpoint trades some control for managed lifecycle features. Compare both only when the learning goal or operating
requirements justify the additional run.

### AWS-T — Higher-throughput production inference

Intended only after offered-load, context/output distribution, availability and latency SLOs exist. Candidate designs
may use managed SageMaker AI endpoints, EC2 autoscaling or EKS if Kubernetes requirements independently justify it.
Multi-replica scaling, batching, health replacement, canary/blue-green, rollback, capacity failure and observability are
required. Complexity must be justified by measured sustained successful work.

Training targets are separate tuples from serving targets. A training accelerator is selected from model/method memory
and duration requirements; it does not automatically become the serving target.

## Practical laboratory sequence

1. Calculate weight, KV-cache, runtime-buffer, storage, and expected request-rate bounds on paper.
2. Query current Region availability, quotas, images, and prices; save dated evidence.
3. Run a CPU or smallest-accelerator smoke deployment with one known-answer request.
4. Run process-cold, warm batch-one, and stepped offered-load trials.
5. Exercise authentication, invalid input, overload, unhealthy-host replacement, rollback, and teardown.
6. Compare EC2 and SageMaker AI only with the same artifact, workload distribution, and allocation formula.

**Illustrative laboratory schedule:** day 1 freezes requirements and IaC; day 2 builds and scans the image; day 3 runs
the small tuple; day 4 runs load and failure drills; day 5 analyzes cost and writes the decision. Substitute actual
queue, build, and run times after the smoke test. Never reserve a five-day endpoint merely because this teaching
schedule spans five days.

## Hardware and service selection procedure

1. **Freeze workload.** Record model/tokenizer/export hashes, precision, maximum context/KV policy, request arrival
   model, concurrency, input/output distributions, batch policy, SLOs, availability, retention and security
   requirements.
2. **Calculate feasibility bounds.** Estimate weight, KV-cache, activation/runtime-buffer and container/storage needs
   with assumptions visible. Estimates shortlist candidates; they never establish fit.
3. **Resolve Region and account constraints.** Record allowed Regions, data residency, service availability, account
   quotas, capacity reservation needs, network egress and organization controls.
4. **Enumerate current candidates.** Query authoritative EC2/SageMaker instance documentation and APIs for CPU/GPU
   architecture, accelerator memory, host memory, storage/network, AMI/driver/CUDA support and quota. Do not copy a
   timeless instance list into this plan.
5. **Resolve runtime compatibility.** For each exact vLLM, SGLang, TensorRT-LLM, llama.cpp server, ONNX Runtime or other
   candidate, record model architecture, precision/quantization, kernel, image digest, driver/CUDA and instance
   dependencies. Unsupported or fallback paths remain visible.
6. **Choose service envelope.** Compare EC2 and SageMaker AI for operational ownership, startup, scaling, update,
   observability, security and allocated cost. Consider EKS only with an existing Kubernetes operational requirement;
   Lambda may orchestrate but is not presumed to host the model.
7. **Capture current cost.** Preserve dated AWS Price List API or calculator inputs and, after execution, allocated
   billing evidence for exact Region, currency, purchase model, storage, transfer, load balancing/API, logs/metrics and
   failed capacity.
8. **Pilot cheapest plausible tuples.** Deploy through hashed IaC, verify placement and output parity, then run
   process-cold, batch-one and stepped open-loop load. Record OOM, throttling, failed scale-out and 5xx.
9. **Qualify and clean up.** Select only a tuple passing all hard gates. Destroy pilot resources automatically and
   archive teardown evidence.

## Worked cost method

**Illustrative example:** suppose a dated price query returns compute rate `C`, storage rate `S`, and transfer rate `T`.
For a trial lasting `h` hours, storing `g` GB-month-equivalent, transferring `e` GB, and completing `n` valid requests,
allocated cost per valid request is `(C × h + S × g + T × e + observability and API charges) / n`. If 2% of attempted
requests fail, report cost per attempted request as well. Replace every symbol with the execution-day API or bill value,
record Region/currency/purchase model, and include idle startup, failed scale-out, and teardown delay. The example
provides the method, not a dollar estimate.

## Required architecture controls

- Separate development, test and production identities and artifact promotion.
- Least-privilege IAM; no long-lived credentials in code or images.
- Private networking and controlled egress where required; encryption in transit and at rest with managed keys selected
  by policy.
- Immutable model, tokenizer, container and IaC identifiers; vulnerability and model-file safety checks.
- Request validation, authentication/authorization, rate limits, non-sensitive correlation IDs and privacy-aware logs.
- Health/readiness checks that include model load and known-answer behavior, not process liveness alone.
- CloudWatch or approved equivalent metrics for queue, TTFT, token timing, errors, memory/utilization, scaling and
  spend.
- Canary or blue-green release, previous qualified revision, rollback drill and artifact revocation.
- Tags, budgets, alarms, idle shutdown, Spot interruption handling where used, and teardown proof.
- No runtime RAG and no remote teacher answering production requests.

## Qualification gates

- **Identity/rights:** exact model and dependencies are allowed for intended use.
- **Compatibility:** image starts on exact AMI/driver/instance; expected accelerator placement occurs; no silent
  unsupported fallback.
- **Semantic/safety:** served output matches the approved artifact within registered tolerance and passes complete
  regression.
- **Memory/reliability:** no disallowed OOM, crash, timeout or corruption at registered workload.
- **Performance:** client-observed p95/p99 TTFT/end-to-end and sustained throughput meet frozen SLO at offered load;
  overload behavior is retained.
- **Operations:** scale, unhealthy-host replacement, canary, rollback, interruption and teardown drills pass.
- **Security:** IAM/network/secrets/logging/image review passes.
- **Cost:** allocated cost per attempted and successful work unit uses dated evidence and includes
  idle/failure/observability overhead.

Failure of any hard gate yields `REJECT`, `UNSUPPORTED` or `INCONCLUSIVE`; a cheaper instance does not compensate for
safety or reliability failure.

## Required evidence paths

- Plan and attempts: `06-experiment-framework/templates/` and `results/`.
- Protocol: [AWS inference benchmark](../06-experiment-framework/protocols/aws-inference-benchmark.md).
- Small target decision:
  [small AWS inference deployment](../14-decision-framework/targets/small-aws-inference-deployment.md).
- Throughput decision:
  [higher-throughput AWS production deployment](../14-decision-framework/targets/higher-throughput-aws-production-deployment.md).
- Implementation companion: [AWS guide](../08-aws-guide/README.md).
- Fast-moving checks: [watchlist W-10/W-11](fast-moving-topics-watchlist.md).

## Publication rule

Report the exact date, Region, service, instance, count, image/runtime, purchase model, workload and allocation formula
with every compatibility, performance or cost claim. Never extrapolate one Region, instance size, load level or runtime
to another.

## Learner exit test

Given a new model revision and workload, produce two feasible tuples, reject unsupported ones with evidence, deploy the
cheapest plausible survivor, meet the frozen security and SLO gates, calculate allocated cost from dated inputs, and
recreate the environment and teardown from hashed IaC.
