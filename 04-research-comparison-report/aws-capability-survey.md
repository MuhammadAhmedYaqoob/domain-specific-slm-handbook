# Amazon Web Services capability survey for domain-specific Small Language Models

**Research date:** 2026-11-23  
**Evidence status:** `DOC-VERIFIED` unless a paragraph is explicitly labeled otherwise. This means the cited
documentation was checked; it does **not** mean runtime behavior, performance, quota availability, or cost was
independently reproduced.  
**Project scope:** training, adapting, evaluating, registering, serving, monitoring, and distributing small language
models (SLMs), primarily in the 0.5B–8B range, without runtime retrieval-augmented generation (RAG).

## Executive decision

For an initial, small-scale specialist-SLM project, use the smallest architecture that meets the experiment:

1. Amazon Simple Storage Service (S3) for immutable datasets, checkpoints, evaluation outputs, and release artifacts.
2. Amazon Elastic Compute Cloud (EC2) with one right-sized GPU, or an Amazon SageMaker AI training job when managed job
   lifecycle and reproducibility are worth the service premium.
3. AWS Identity and Access Management (IAM), AWS Key Management Service (KMS), Amazon Virtual Private Cloud (VPC),
   Amazon CloudWatch, and AWS CloudTrail as the security and operations baseline.
4. Amazon Elastic Container Registry (ECR) only when containers are used.
5. A single EC2 inference host or a SageMaker real-time endpoint for the first deployment. Prefer vLLM on NVIDIA GPUs
   for a mainstream Hugging Face-format model; benchmark llama.cpp when CPU execution or a GGUF artifact is an
   intentional requirement.
6. Add Amazon Elastic Container Service (ECS), AWS Step Functions, AWS Secrets Manager, and Amazon API Gateway only when
   their operational function is actually needed.
7. Defer Amazon Elastic Kubernetes Service (EKS), multi-node training, P-class/Blackwell clusters, Amazon FSx for
   Lustre, and elaborate multi-model serving until measured workload requirements justify them.

Amazon Bedrock is not a universal replacement for EC2 or SageMaker. Its managed customization and Custom Model Import
paths are useful only when the exact model architecture, Region, customization method, and serving constraints are
supported. Bedrock does not provide the low-level runtime and kernel control required for this project's
runtime-comparison experiments.

## Learner lesson: build the smallest AWS system that answers the experiment

### Problem, intuition, and historical reason

Amazon Web Services (AWS) offers overlapping infrastructure, managed machine-learning, container, and model services.
The problem is not service availability; it is choosing the minimum control plane that keeps experiments reproducible,
secure, observable, and affordable. Cloud platforms grew by separating concerns—object storage, compute, identity,
networking, orchestration, serving, and audit—so adopting every service at once increases cost and failure surface
without improving the model.

### Mechanism and concrete example

Amazon Simple Storage Service (S3) holds immutable data and artifacts. Amazon Elastic Compute Cloud (EC2) provides host
and accelerator control. Amazon SageMaker AI wraps jobs and endpoints in managed lifecycle contracts. Identity and
Access Management (IAM), Key Management Service (KMS), Virtual Private Cloud (VPC), CloudWatch, and CloudTrail provide
access, encryption, isolation, operations, and audit boundaries. For an automotive LoRA experiment, one encrypted GPU
EC2 host can read a versioned S3 dataset, write checkpoints to S3, emit metrics to CloudWatch, and shut down
automatically. SageMaker becomes useful when managed job isolation, Spot restart, or endpoint operations justify its
different price and control model.

### Strengths, limitations, compatibility, and decision

EC2 maximizes runtime and kernel control but leaves patching, recovery, and shutdown to the team. SageMaker reduces
infrastructure lifecycle work while imposing service contracts and separate pricing. Bedrock can simplify supported
managed customization/import but cannot host arbitrary architectures or experimental kernels. Elastic Container
Service (ECS) is a simpler first orchestrator than Elastic Kubernetes Service (EKS) for many small deployments.
Trainium and Inferentia require the Neuron software path; they are not drop-in CUDA replacements.

Start with S3, security/audit controls, one right-sized compute choice, pinned software, and automatic cleanup. Add a
service only when it solves a named operational problem. Avoid P-class clusters, Kubernetes, distributed filesystems,
or vendor-specific accelerators until a single-node baseline exposes the need.

## Evidence and interpretation rules

- `DOC-VERIFIED`: the statement is supported by an official AWS or serving-runtime source linked directly in this
  survey.
- `VENDOR-REPORTED`: a performance or savings statement is attributed to its vendor and is not treated as independently
  measured.
- `ESTIMATE`: a planning value that must be replaced by a dated, region-specific observation.
- Availability is a tuple, not a global fact: **service + feature + instance type + accelerator + Region + Availability
  Zone + account quota + date**.
- A runtime's general model-family support does not establish that a particular checkpoint, quantization, adapter,
  attention variant, or chat template works. Validate the exact artifact.
- No benchmark or successful deployment is claimed in this document.

## Small-scale priority classification

<!-- markdownlint-disable MD013 -->
| Capability                                     |                                   Initial classification | Why                                                                               |
| ---------------------------------------------- | -------------------------------------------------------: | --------------------------------------------------------------------------------- |
| S3                                             |                                                Essential | Durable source of truth for datasets, checkpoints, reports, and release artifacts |
| IAM                                            |                                                Essential | Least-privilege access and workload roles                                         |
| KMS                                            |                                                Essential | Encryption-key control for sensitive datasets and artifacts                       |
| VPC and security groups                        |                                                Essential | Network isolation for training and serving                                        |
| CloudWatch                                     |                                                Essential | Logs, metrics, alarms, and operational diagnosis                                  |
| CloudTrail                                     |                                                Essential | Account/API audit trail                                                           |
| EC2 **or** SageMaker AI training               |                                         Essential choice | Compute is required; using both is optional                                       |
| ECR                                            |            Essential with containers; otherwise optional | Versioned image registry                                                          |
| Infrastructure as Code (IaC)                   |          Essential before repeatable/shared environments | Prevents console-only, irreproducible infrastructure                              |
| Secrets Manager                                | Optional initially; essential when runtime secrets exist | Do not create secrets merely to justify the service                               |
| API Gateway                                    |                                                 Optional | Managed public API controls; not required for private experiments                 |
| ECS                                            |                                                 Optional | Simpler managed container orchestration than Kubernetes                           |
| Step Functions                                 |                                                 Optional | Durable multi-step orchestration and retries                                      |
| Lambda                                         |                                                 Optional | Control-plane glue, not GPU model serving                                         |
| Bedrock                                        |                                    Optional, conditional | Managed models/customization/import with explicit compatibility limits            |
| SageMaker Model Registry and managed endpoints |                                                 Optional | Useful governance/hosting layer; not required for a one-host prototype            |
| EKS                                            |                             Likely unnecessary initially | Kubernetes control-plane and platform burden exceed small-project needs           |
| FSx for Lustre                                 |                             Likely unnecessary initially | Valuable for shared high-throughput multi-node I/O, not a default for one GPU     |
| Multi-node P/Trainium clusters                 |                             Likely unnecessary initially | The target SLM size and PEFT methods should be tested on one accelerator first    |
<!-- markdownlint-restore -->

This classification is a project recommendation, not an AWS service capability claim.

The priority table is a dependency lesson, not a shopping list. Storage, identity, encryption, networking,
observability, audit, and one compute path form the minimum governed system. Containers, orchestration, registries, and
public API services are conditional. Large-cluster services are deferred because their operational burden cannot be
justified by a 0.5B–8B experiment before measurement.

## Compute survey

### Current EC2 accelerator choices

AWS's current-generation list and accelerated-computing specification are the authoritative starting points, but the
separate Region table must be checked before design or purchasing:

- [EC2 current-generation instance types](https://docs.aws.amazon.com/ec2/latest/instancetypes/instance-types.html)
- [Accelerated-computing specifications](https://docs.aws.amazon.com/ec2/latest/instancetypes/ac.html)
- [Instance types by Region](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-regions.html)
- [Accelerated-computing overview](https://aws.amazon.com/ec2/instance-types/accelerated-computing/)

<!-- markdownlint-disable MD013 -->
| Family group                                | Accelerator documented by AWS                                                                       | Best-fit role in this project                                                                |             Small-scale priority |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------: |
| G4dn                                        | NVIDIA T4                                                                                           | Compatibility tests, modest inference, legacy CUDA path                                      |                         Optional |
| G5                                          | NVIDIA A10G                                                                                         | Single-GPU PEFT and general inference where available                                        |                   Core candidate |
| G6                                          | NVIDIA L4                                                                                           | Cost-conscious inference and smaller-model PEFT                                              |                   Core candidate |
| G6e                                         | NVIDIA L40S                                                                                         | More memory/throughput than G6 for training or serving                                       |               Advanced candidate |
| G7 / G7e                                    | NVIDIA Blackwell-generation graphics/inference GPUs; G7e uses RTX PRO 6000 Blackwell Server Edition | Newer inference/training option; validate Region, software image, quota, and runtime kernels |               Advanced candidate |
| P4d / P4de                                  | NVIDIA A100                                                                                         | Multi-GPU or memory-heavy training                                                           |    Usually unnecessary initially |
| P5 / P5e / P5en                             | NVIDIA H100/H200 variants                                                                           | Large/distributed training and high-throughput inference                                     |    Usually unnecessary initially |
| P6-B200 / P6-B300                           | NVIDIA Blackwell/Blackwell Ultra                                                                    | Very large training/inference                                                                | Unnecessary for initial SLM work |
| P6e-GB200 and related UltraServer offerings | Grace Blackwell systems                                                                             | Frontier-scale workloads                                                                     |                      Unnecessary |
| Trn1 / Trn1n / Trn2                         | AWS Trainium                                                                                        | Training and supported Neuron inference after porting/compilation validation                 |              Optional experiment |
| Inf2                                        | AWS Inferentia2                                                                                     | Supported Neuron inference with a different software path from CUDA                          |              Optional experiment |
<!-- markdownlint-restore -->

`DOC-VERIFIED`: AWS lists these families as accelerated-computing options in its current instance guide. Exact
accelerator memory, networking, Spot support, operating-system support, and instance sizes differ by subtype; consult
the specification rather than inferring one subtype from its family name.

`VENDOR-REPORTED`: AWS makes comparative performance claims on the [P6](https://aws.amazon.com/ec2/instance-types/p6/)
and [G7e](https://aws.amazon.com/ec2/instance-types/g7e/) product pages. Those claims are not project benchmarks and
must not be used to predict this workload without a controlled trial.

The compute table should be read from workload to hardware, not from newest family to presumed superiority. G-class
single-GPU instances are the first qualification band for this project. P-class and multi-node systems enter only when
measured memory or throughput cannot be met. AWS silicon enters as a separate compiler/runtime experiment with its own
support matrix.

### Practical GPU selection process

1. Freeze the exact model, maximum context, training method, precision, optimizer, batch/accumulation plan, and serving
   concurrency.
2. Estimate weights, gradients, optimizer state, activations, temporary buffers, and key-value (KV) cache separately. A
   model file fitting in GPU memory is not evidence that training or serving will fit.
3. Select the least expensive candidate with adequate accelerator memory and supported kernels.
4. Check the [EC2 Region table](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-regions.html), account
   quotas, and actual launch capacity on the research date.
5. Run a short representative job and record out-of-memory behavior, tokens/s, time to first token, utilization, wall
   time, and all billed resources.
6. Promote to P-class, multi-GPU, or multi-node only after the single-GPU result demonstrates a real bottleneck.

For 0.5B–8B SLM experiments, G5/G6/G6e-class single-GPU instances are the first candidates to evaluate. This is an
engineering recommendation, not a compatibility or fit guarantee.

### AWS silicon qualification

Trainium and Inferentia can be attractive only after the Neuron toolchain supports the exact architecture and
operations. They are not drop-in CUDA replacements. The current
[AWS Neuron documentation](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/) and
[Neuron model migration and support matrix](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/inference-legacy/nxd-inference/developer_guides/migrate-from-tnx-to-nxdi.html)
must be checked at implementation time.

The separate [vLLM Neuron plugin](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/vllm-neuron/) is documented as
beta and its own support matrix is narrower than vLLM's CUDA support. Treat each model recipe and feature as
independently gated; do not infer LoRA, quantization, or parallelism support from the vLLM name alone.

## EC2 versus SageMaker AI

<!-- markdownlint-disable MD013 -->
| Decision dimension      | Self-managed EC2                                                                             | SageMaker AI training/hosting                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Runtime/kernel control  | Highest: AMI, driver, container, disk, process, and network are directly controlled          | Bring-your-own-container is possible, but jobs/endpoints follow SageMaker contracts               |
| Setup burden            | User manages provisioning, patching, shutdown, checkpoint transfer, scheduling, and recovery | Managed jobs provision and release infrastructure; logs and artifacts integrate with AWS services |
| Experiment transparency | Direct host access simplifies low-level runtime comparison                                   | Better standardized job records, but managed layers can obscure host details                      |
| Reproducibility         | Requires disciplined IaC, immutable images, scripts, and run manifests                       | Job specifications and isolated jobs help, but code/image/data versions still must be pinned      |
| Spot handling           | User implements checkpointing, interruption response, relaunch, and fleet strategy           | Managed Spot Training handles job interruption/restart mechanics when configured correctly        |
| Inference               | Full control with EC2/ECS/EKS and any compatible server                                      | Managed real-time, asynchronous, serverless, and batch options; feature support differs           |
| Cost structure          | Raw compute plus storage/network/operations                                                  | Managed-service rates and related resources; do not compare only nominal instance names           |
| Best initial fit        | Runtime research, one-host prototype, maximum control                                        | Repeatable training jobs or managed endpoints where reduced operations justify cost               |
<!-- markdownlint-restore -->

`DOC-VERIFIED`: SageMaker AI documents
[distributed training](https://docs.aws.amazon.com/sagemaker/latest/dg/distributed-training-get-started.html),
[Managed Spot Training](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training.html),
[checkpointing](https://docs.aws.amazon.com/sagemaker/latest/dg/model-checkpoints.html), and multiple
[inference options](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model-options.html).

Recommended sequence:

- Start on EC2 when comparing vLLM, llama.cpp, ONNX Runtime, TensorRT-LLM, and SGLang because host-level control is
  central to the experiment.
- Use SageMaker training jobs when clean job isolation, managed lifecycle, job metadata, managed Spot handling, or
  organization controls outweigh the service premium.
- Do not assume an `ml.*` SageMaker instance and an apparently corresponding EC2 instance have identical price,
  availability, software environment, or operational behavior.

### SageMaker inference choices

`DOC-VERIFIED`: SageMaker documents:

- **Real-time inference:** persistent managed endpoint for sustained low-latency/high-throughput traffic.
- **Serverless inference:** intermittent traffic with managed scaling and cold-start trade-offs. The current SageMaker
  feature matrix does not list GPU support for serverless inference.
- **Asynchronous inference:** queued requests, larger payloads/longer processing, and scale-to-zero behavior.
- **Batch Transform:** offline dataset inference without a persistent endpoint.

Sources: [inference options](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model-options.html),
[deployment options](https://docs.aws.amazon.com/sagemaker/latest/dg/how-it-works-deployment.html), and the
[current feature matrix](https://docs.aws.amazon.com/sagemaker/latest/dg/model-deploy-feature-matrix.html).

For an interactive SLM, a GPU-backed real-time endpoint is the relevant managed baseline. Serverless should not be
presented as GPU LLM hosting unless the current feature matrix changes and the exact configuration is verified.

SageMaker [multi-model endpoints](https://docs.aws.amazon.com/sagemaker/latest/dg/multi-model-endpoints.html) can share
a fleet among compatible models, including GPU-backed models, but occasional model-load latency and framework/container
constraints make them a later optimization. A shared base model with hot-swappable adapters is not automatically
equivalent to a SageMaker multi-model endpoint and needs runtime-specific validation.

## Spot capacity and failure recovery

### Self-managed EC2 Spot

Spot is appropriate only for interruption-tolerant training, conversion, evaluation, or batch jobs.

Required design:

1. Write atomic, resumable checkpoints to local storage at a measured interval.
2. Copy checkpoints and run manifests to durable S3 storage; do not rely on instance store.
3. Persist optimizer/scheduler state, random-number-generator state, dataloader progress or an equivalent deterministic
   resume cursor, and dataset/model revisions.
4. Poll instance metadata and consume EventBridge events for Spot interruption and rebalance signals.
5. Make startup idempotent: discover the latest valid checkpoint, verify its hash, and resume.
6. Diversify across compatible instance types and Availability Zones only after confirming that the software and
   numerical behavior remain acceptable.
7. Keep an On-Demand path for deadline-bound or non-resumable stages.

`DOC-VERIFIED`: an
[EC2 rebalance recommendation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/rebalance-recommendations.html) can
precede the two-minute interruption notice but is best effort and is not guaranteed to arrive earlier. AWS recommends
price-capacity-optimized for most Spot workloads and documents capacity-optimized for high interruption-cost workloads
in its
[Fleet allocation strategies](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-fleet-allocation-strategy.html).
[Capacity Rebalancing](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-fleet-capacity-rebalance.html) can launch
replacements for at-risk Fleet instances but does not replace application checkpoint correctness.

### SageMaker Managed Spot Training

Set managed Spot explicitly, configure maximum runtime and maximum wait, and write framework checkpoints to the
configured local checkpoint path. SageMaker synchronizes that path to S3 and restores it on restart.

`DOC-VERIFIED`: Managed Spot does **not** automatically fall back to On-Demand when Spot capacity is unavailable. The
job can wait until its maximum wait is exhausted. Checkpointing is strongly recommended for jobs that are not short.
Sources: [Managed Spot Training](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training.html),
[checkpoint behavior](https://docs.aws.amazon.com/sagemaker/latest/dg/model-checkpoints.html), and
[`CheckpointConfig`](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CheckpointConfig.html).

`VENDOR-REPORTED`: AWS advertises potential discounts “up to” a percentage for Managed Spot. This survey deliberately
does not copy that volatile maximum into a budget.

## Storage and artifact management

### Essential baseline

**Amazon S3 — essential.** Use versioned, access-controlled buckets/prefixes for:

- raw, accepted, and rejected dataset snapshots;
- model-source manifests and licenses;
- training checkpoints and final weights;
- evaluation inputs, raw outputs, and reports;
- edge artifacts, checksums, signatures, and release manifests;
- immutable run manifests that bind code, image, data, model, and configuration revisions.

S3 is durable object storage, not a low-latency POSIX training filesystem. Stream, stage, cache, or shard data according
to measured access behavior. Official sources: [S3 documentation](https://docs.aws.amazon.com/s3/) and
[S3 security best practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html).

**Amazon EBS — essential for most EC2 hosts.** Use encrypted EBS for the operating system and persistent host-attached
work/cache space. Record volume type, size, provisioned performance, encryption key, snapshot policy, and
delete-on-termination behavior. Do not confuse an EBS snapshot with an application-consistent training checkpoint.
Source: [EBS documentation](https://docs.aws.amazon.com/ebs/).

**Instance store — optional scratch only.** Local NVMe can accelerate caches and temporary checkpoints, but data is
ephemeral. Copy authoritative state to S3/EBS before termination.

### Scale-up storage

**Amazon FSx for Lustre — optional; usually unnecessary initially.** It is relevant when multiple accelerator nodes need
high-throughput shared access to datasets/checkpoints. AWS's EKS AI/ML storage guidance distinguishes S3, EBS, EFS, and
FSx options and recommends benchmarking for the actual workload:
[EKS AI/ML storage](https://docs.aws.amazon.com/eks/latest/best-practices/aiml-storage.html).

**Amazon EFS — optional.** A managed shared filesystem can simplify shared configuration or modest shared files, but it
should not be assumed to match the I/O profile of distributed training. Benchmark before adoption.

## Container registry and software environments

**Amazon ECR — essential when containers are used.** Store immutable, vulnerability-scanned training and inference
images. Deploy by image digest, not a mutable tag. Apply lifecycle policies without deleting images referenced by
reproducibility records. Source: [ECR documentation](https://docs.aws.amazon.com/ecr/).

For EC2, AWS [Deep Learning AMIs](https://docs.aws.amazon.com/dlami/latest/devguide/what-is-dlami.html) reduce initial
driver/framework setup. For containers,
[AWS Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/what-is-dl-containers.html)
provide framework images. Pin the exact AMI ID or image digest and record driver, CUDA/ROCm/Neuron, framework, compiler,
runtime, and kernel versions. “Latest” is not reproducible.

For multi-node NVIDIA training, Elastic Fabric Adapter (EFA), placement, security groups, NCCL, and storage must be
configured together. The
[DLAMI EFA guide](https://docs.aws.amazon.com/dlami/latest/devguide/tutorial-efa-launching.html) documents the required
EFA-capable setup. This is advanced scope, not a default.

## Security baseline

<!-- markdownlint-disable MD013 -->
| Service/control                                      | Project use                                                         |                  Priority |
| ---------------------------------------------------- | ------------------------------------------------------------------- | ------------------------: |
| IAM roles and policies                               | Workload identities, least privilege, short-lived credentials       |                 Essential |
| KMS                                                  | Customer-controlled encryption keys and key-policy separation       |                 Essential |
| VPC, private subnets, security groups, VPC endpoints | Network boundaries and private service access                       |                 Essential |
| Secrets Manager                                      | API keys/tokens that cannot use IAM roles; rotation where supported |     Conditional essential |
| CloudTrail                                           | Audit AWS API activity and configuration changes                    |                 Essential |
| S3 Block Public Access, bucket policies, versioning  | Protect datasets and artifacts                                      |                 Essential |
| ECR image scanning and digest pinning                | Reduce container supply-chain risk                                  | Essential with containers |
<!-- markdownlint-restore -->

Sources: [IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html),
[KMS best practices](https://docs.aws.amazon.com/prescriptive-guidance/latest/aws-kms-best-practices/),
[VPC security best practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html),
[Secrets Manager best practices](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html), and
[CloudTrail security best practices](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/best-practices-security.html).

Minimum controls:

- use separate roles for data preparation, training, evaluation, deployment, and runtime;
- deny public S3/ECR access unless an explicit reviewed distribution requirement exists;
- encrypt S3, EBS, logs, registries, and endpoint data with documented keys and retention;
- place GPU hosts/endpoints in private subnets where practical and use Systems Manager rather than open SSH ingress;
- never bake Hugging Face tokens or API credentials into images, AMIs, model files, notebooks, or logs;
- validate model licenses and use safe serialization formats where possible;
- hash, scan, and approve model/container artifacts before promotion;
- keep training records and user prompts free of unnecessary personal or sensitive data.

CloudTrail is an audit source, not application observability. CloudWatch is an operations source, not a substitute for a
tamper-aware governance trail.

## Orchestration and serving platform choices

### ECS

**Classification: optional, preferred before EKS for a small containerized service.** ECS can run GPU containers on EC2
capacity while AWS manages container scheduling/service lifecycle. Use it when automatic replacement, service
deployments, and task definitions are needed but Kubernetes portability is not. Source:
[choosing an AWS container service](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/choosing-aws-container-service.html).

### EKS

**Classification: likely unnecessary initially.** EKS is justified when the organization already operates Kubernetes,
requires Kubernetes APIs/ecosystem, needs sophisticated multi-service scheduling, or plans a shared inference platform.
It introduces cluster upgrades, add-ons, node lifecycle, GPU device plugins/operators, networking, storage drivers,
observability, and policy work. Do not deploy EKS solely because a runtime publishes a Helm chart.

### Lambda

**Classification: optional control-plane glue; inappropriate for hosting the GPU SLM process.** Use Lambda for
lightweight event validation, manifest updates, notifications, cleanup triggers, or orchestration callbacks. Lambda
execution/storage/runtime constraints and lack of general GPU instances make it unsuitable as the model server. Source:
[Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html).

### Step Functions

**Classification: optional.** Use Step Functions for durable workflows that coordinate preprocessing, training,
evaluation, approval, conversion, and deployment with explicit states, retries, timeouts, and failure paths. Keep large
payloads in S3 and pass references. Source:
[Step Functions service integrations](https://docs.aws.amazon.com/step-functions/latest/dg/integrate-services.html).

### API Gateway

**Classification: optional.** It can provide managed authentication integration, throttling, request routing, and public
API controls. Before placing it in front of token streaming, verify the selected API type, integration, timeout,
payload, and streaming behavior. An Application Load Balancer or direct private endpoint may be simpler. Sources:
[API Gateway quotas](https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html) and
[API types](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-types.html).

## Monitoring, audit, and cost controls

### CloudWatch

Collect:

- host/container health, CPU, memory, disk, network, and GPU metrics;
- model-load success/failure and cold-start duration;
- request count, queue depth, concurrency, latency, time to first token, inter-token latency, generated tokens, and
  failure class;
- out-of-memory events, process restarts, Spot interruptions, and checkpoint age;
- model ID, artifact hash, runtime image digest, and deployment ID as bounded-cardinality dimensions or structured logs;
- quality/safety signals only with privacy review and explicit retention.

GPU metrics generally require an agent/exporter such as NVIDIA DCGM rather than appearing automatically in every
CloudWatch configuration. Source:
[CloudWatch agent and NVIDIA GPU metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-NVIDIA-GPU.html).

### CloudTrail

Retain an organization/account trail as appropriate, protect its destination, validate logs where required, and alert on
high-risk changes: role/policy edits, key-policy edits, public bucket changes, trail changes, model artifact access, and
deployment mutations. Source:
[CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html).

### Cost Management

Use AWS Budgets, Cost Explorer, cost-allocation tags, and anomaly detection; add automatic shutdown for nonproduction
GPU resources. Sources:
[AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html),
[Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html), and
[Cost Anomaly Detection](https://docs.aws.amazon.com/cost-management/latest/userguide/manage-ad.html).

## Infrastructure as Code

**Classification: essential before repeatable or shared deployment.**

- **AWS Cloud Development Kit (CDK):** strong fit for teams wanting typed, code-first AWS infrastructure synthesized to
  CloudFormation. Source: [AWS CDK Guide](https://docs.aws.amazon.com/cdk/v2/guide/home.html).
- **Terraform:** strong fit where the team already uses HashiCorp Configuration Language, needs provider/ecosystem
  consistency, or manages more than AWS. Use the official
  [Terraform AWS Provider documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) and pin
  provider/module versions.

Choose one primary ownership path per resource. Do not let CDK, Terraform, console edits, and service-generated
resources silently compete. Store plans/templates in version control; use remote state locking where applicable; scan
changes; separate environments/accounts; and require review for IAM, networking, encryption, public exposure, and
deletion.

## Amazon Bedrock applicability

### What Bedrock can do

`DOC-VERIFIED`: Bedrock provides managed foundation-model access and supports model customization methods for specific
models and Regions. The current fine-tuning table must be read directly because model IDs and Regions change:
[fine-tuning support](https://docs.aws.amazon.com/bedrock/latest/userguide/custom-model-fine-tuning.html) and
[custom-model overview](https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html).

`DOC-VERIFIED`:
[Custom Model Import](https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-import-model.html)
accepts only documented architectures and configurations, in documented Regions, and has feature limitations. The page
currently documents selected Llama, Mistral/Mixtral, Flan, GPTBigCode, Qwen, and other architecture paths. This is not
permission to assume every checkpoint or modification in those families imports successfully.

### Where Bedrock fits

- managed baseline/teacher generation, subject to data governance and model terms;
- managed fine-tuning or continued pretraining when the exact listed model meets requirements;
- managed serving for an imported, supported architecture when Bedrock's operational model is preferable;
- a comparison point against self-hosted specialist models.

### Where Bedrock does not fit

- low-level comparison of vLLM, TGI, llama.cpp, ONNX Runtime, TensorRT-LLM, and SGLang;
- arbitrary custom kernels, unsupported architectures, unsupported quantization formats, or unrestricted container
  control;
- an assumption that all externally trained LoRA adapters can be imported or dynamically switched;
- a mechanism to avoid verifying Region, quota, import limits, model license, inference features, or price;
- runtime RAG for the primary product, which remains prohibited by project governance.

**Small-scale classification: optional and conditional.** Bedrock can reduce infrastructure work, but may constrain
model choice, training control, serving runtime, portability, and experimental observability.

## Serving runtime survey

### Decision matrix

<!-- markdownlint-disable MD013 -->
| Runtime            | Officially documented strengths                                                                                              | Important limits/qualification                                                                                               | AWS fit                                                               |                    Small-scale priority |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | --------------------------------------: |
| vLLM               | OpenAI-compatible server, continuous batching, broad Hugging Face ecosystem, distributed serving, NVIDIA/AMD/Intel/CPU paths | Exact architecture, quantization, kernel, GPU compute capability, and feature support vary; Neuron is a separate beta plugin | Strong default on NVIDIA EC2/SageMaker BYOC; conditional on Neuron    |                     Essential benchmark |
| Hugging Face TGI   | Continuous batching, streaming, metrics/tracing, quantization, mature container interface                                    | Officially in maintenance mode since 2025-11-12; HF recommends vLLM or SGLang for new endpoints                              | Existing deployments only unless a specific compatibility reason wins |        Likely unnecessary for new build |
| llama.cpp server   | GGUF, CPU-first portability, many hardware backends, quantized models, lightweight OpenAI-compatible server                  | Different artifact/toolchain; GPU throughput and model-feature support must be benchmarked; not a CUDA-serving drop-in       | Strong for CPU/GGUF and cloud-edge artifact parity                    | Essential benchmark for GGUF/CPU target |
| ONNX Runtime GenAI | ONNX portability and execution providers; CPU/CUDA and other hardware paths; generative loop APIs                            | GenAI API is marked preview; model export/operator compatibility is checkpoint-specific                                      | Useful when ONNX is a release target, not automatically best server   |                     Optional experiment |
| TensorRT-LLM       | NVIDIA-specific optimization, quantization, paged attention, in-flight batching, multi-GPU serving                           | Linux/NVIDIA-focused; hardware-specific support matrix and build/runtime complexity                                          | Advanced NVIDIA EC2/SageMaker BYOC option                             |                      Advanced practical |
| SGLang             | OpenAI-compatible serving, prefix-aware RadixAttention, broad model/hardware paths, distributed serving                      | Fast-moving compatibility; exact backend, quantization, and checkpoint must be verified                                      | Strong alternative to vLLM on supported EC2/SageMaker BYOC            |                          Core benchmark |
<!-- markdownlint-restore -->

### vLLM

Official sources:

- [installation and hardware platforms](https://docs.vllm.ai/en/stable/getting_started/installation/)
- [GPU installation and requirements](https://docs.vllm.ai/en/stable/getting_started/installation/gpu/)
- [supported models](https://docs.vllm.ai/en/stable/models/supported_models.html)
- [quantization compatibility](https://docs.vllm.ai/en/stable/features/quantization/)
- [parallelism and scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/)
- [OpenAI-compatible server](https://docs.vllm.ai/en/stable/serving/openai_compatible_server.html)

`DOC-VERIFIED`: vLLM documents NVIDIA CUDA, AMD ROCm, Intel XPU, and CPU platforms. That platform statement does not
mean every feature is equivalent across them. On AWS, NVIDIA G/P families are the lowest-friction path for the
mainstream CUDA build. AWS Neuron uses the separate
[vLLM Neuron plugin](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/vllm-neuron/), currently labeled beta; use
its model/feature matrix rather than the main vLLM matrix.

Recommended role: first GPU-serving baseline for supported Hugging Face-format models. Pin the container digest and
runtime version, then test the exact checkpoint, tokenizer, chat template, quantization, maximum context, streaming,
structured output, and LoRA behavior.

### Hugging Face Text Generation Inference (TGI)

Official sources:

- [TGI engine status and migration guidance](https://huggingface.co/docs/inference-endpoints/engines/tgi)
- [TGI repository](https://github.com/huggingface/text-generation-inference)
- [NVIDIA installation](https://huggingface.co/docs/text-generation-inference/installation_nvidia)
- [quantization](https://huggingface.co/docs/text-generation-inference/conceptual/quantization)

`DOC-VERIFIED`: Hugging Face marks TGI as maintenance mode as of 2025-11-12 and recommends vLLM or SGLang alternatives
for new Inference Endpoints. Existing TGI deployments remain a valid migration/compatibility concern, but selecting it
for a new platform now creates a maintenance risk.

Recommended role: compatibility benchmark only when an existing TGI image, endpoint, or feature dependency must be
retained. Do not call it unmaintained or broken; “maintenance mode” is the precise vendor status.

### llama.cpp server

Official sources:

- [llama.cpp repository](https://github.com/ggml-org/llama.cpp)
- [build and hardware backends](https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/build.md)
- [HTTP server documentation](https://github.com/ggml-org/llama.cpp/blob/HEAD/tools/server/README.md)

`DOC-VERIFIED`: llama.cpp documents CPU and multiple accelerator backends, GGUF artifacts, quantized inference,
continuous batching, and OpenAI-compatible server routes. It is the most relevant cloud runtime when the approved edge
artifact is GGUF or CPU deployment is a deliberate cost/portability experiment.

Qualification: verify that the chosen architecture converts correctly to GGUF and that tokenizer, chat template,
rope/context configuration, quantization, and any adapter merge preserve quality. Cloud-edge artifact parity can
simplify release management, but does not imply performance parity.

### ONNX Runtime GenAI

Official sources:

- [Generate API status](https://onnxruntime.ai/docs/genai/)
- [supported configuration/model types and providers](https://onnxruntime.ai/docs/genai/reference/config.html)
- [installation variants](https://onnxruntime.ai/docs/genai/howto/install.html)
- [source-independent support and configuration matrix](https://onnxruntime.ai/docs/genai/reference/config.html)

`DOC-VERIFIED`: the Generate API is labeled preview. It supplies token generation, sampling, logits processing, and
KV-cache management over supported ONNX models and execution providers. ONNX Runtime's broad provider ecosystem is
useful for cross-target experiments, but export success and generic ONNX execution are not proof that a full
autoregressive model is optimized or supported by the GenAI layer.

Recommended role: optional portability experiment, especially if the edge or Windows path already uses ONNX. Keep the
preview API risk explicit and validate exported graphs, custom operators, dynamic shapes, quantization, and numerical
quality.

### NVIDIA TensorRT-LLM

Official sources:

- [support matrix](https://nvidia.github.io/TensorRT-LLM/reference/support-matrix.html)
- [supported models](https://nvidia.github.io/TensorRT-LLM/latest/models/supported-models.html)
- [quantization](https://nvidia.github.io/TensorRT-LLM/latest/features/quantization.html)
- [paged attention and in-flight batching](https://nvidia.github.io/TensorRT-LLM/latest/features/paged-attention-ifb-scheduler.html)
- [quick start and `trtllm-serve`](https://nvidia.github.io/TensorRT-LLM/latest/quick-start-guide.html)

`DOC-VERIFIED`: TensorRT-LLM targets supported NVIDIA architectures on documented Linux platforms and exposes
NVIDIA-specific quantization and batching paths. The support matrix ties precision features to GPU architecture; FP8/FP4
labels must never be assumed portable across G/P generations.

Recommended role: advanced optimization after the vLLM/SGLang baseline exposes a throughput, latency, or memory need.
Account for engine/build time, conversion validation, version coupling, and narrower hardware portability.

### SGLang

Official sources:

- [SGLang documentation](https://docs.sglang.io/)
- [hardware platforms](https://docs.sglang.io/docs/hardware-platforms/overview)
- [supported generative models](https://docs.sglang.io/docs/supported-models/generative_models)
- [quantization matrix](https://docs.sglang.io/docs/advanced_features/quantization)
- [installation/server start](https://docs.sglang.io/docs/get-started/install)

`DOC-VERIFIED`: SGLang documents NVIDIA, AMD, CPU, Intel XPU, TPU, and other platform paths, along with
OpenAI-compatible serving and multiple quantization backends. Platform breadth does not mean identical behavior or
maturity.

Recommended role: benchmark alongside vLLM for the selected checkpoint, especially where prefix reuse, structured
generation, or a model supported earlier/better by SGLang matters. Pin a tested release because its compatibility
surface changes quickly.

## Runtime qualification protocol

For each candidate, record:

1. immutable model and tokenizer revision, license, architecture, adapter state, and chat template;
2. source and target weight format, quantizer and calibration dataset, conversion command, and artifact hash;
3. EC2/SageMaker instance type, Region and Availability Zone, accelerator count, driver, CUDA/Neuron/ROCm, container
   digest, runtime version, and command line;
4. maximum model/context limits accepted by the runtime and the tested limits;
5. cold model-load time, peak host/GPU memory, time to first token, prefill and decode throughput, inter-token latency,
   request throughput, queue time, error rate, and cost inputs;
6. fixed prompt/output token distributions at concurrency 1 and representative concurrency;
7. output-quality regression against the unconverted reference;
8. health, readiness, graceful shutdown, backpressure, timeout, cancellation, metrics, logs, authentication boundary,
   and rolling rollback behavior.

Do not compare vendor benchmark numbers across different models, precisions, context lengths, batching, accelerators,
software versions, or latency constraints.

## Price-verification procedure

No volatile dollar figure is copied into this survey.

For each experiment or architecture decision:

1. Record the verification timestamp in UTC and workload Region.
2. Record exact service, purchase model, instance type, operating system/tenancy where applicable, and unit
   (instance-second/hour, GB-month, requests, tokens, data transfer, provisioned throughput, and so on).
3. Check the service pricing page:
   - [EC2 On-Demand pricing](https://aws.amazon.com/ec2/pricing/on-demand/)
   - [EC2 Spot pricing and history](https://aws.amazon.com/ec2/spot/pricing/)
   - [SageMaker AI pricing](https://aws.amazon.com/sagemaker/ai/pricing/)
   - [Bedrock pricing](https://aws.amazon.com/bedrock/pricing/)
   - [S3 pricing](https://aws.amazon.com/s3/pricing/)
   - [EBS pricing](https://aws.amazon.com/ebs/pricing/)
   - [ECR pricing](https://aws.amazon.com/ecr/pricing/)
   - [ECS pricing](https://aws.amazon.com/ecs/pricing/)
   - [EKS pricing](https://aws.amazon.com/eks/pricing/)
4. Reproduce non-Spot catalog values with the
   [AWS Price List Query API](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/price-changes.html) or
   [AWS Pricing Calculator](https://calculator.aws/). The Price List API does not include EC2 Spot prices and does not
   include Savings Plans prices; use the applicable official source.
5. Capture the returned SKU/rate code or exported calculator estimate with the run record. If a pricing page conflicts
   with a catalog file, AWS states that the service pricing page governs.
6. Include attached EBS, snapshots, S3 requests/storage, data transfer, public IPv4, NAT Gateway, load balancer/API
   Gateway, CloudWatch logs/metrics, registry, support, idle endpoint time, failed/interrupted work, and taxes/credits
   where applicable.
7. For SageMaker, compare its managed-service rate and all attached resources against the EC2 architecture; never
   substitute an EC2 rate for a SageMaker `ml.*` rate.
8. Multiply measured wall time and measured usage by the recorded unit rates. Present a sensitivity range for
   utilization, Spot interruptions, model load/cold start, and traffic.

### Required dated price observations

A cost comparison is incomplete until it records: the EC2 training On-Demand rate; observed Spot rate and interruption
window; exact SageMaker `ml.*` training rate; EC2 and SageMaker inference rates with measured utilization and scaling
bounds; any Bedrock model/customization/import billing unit; S3, EBS, or FSx storage and input/output charges; network,
NAT, load-balancer, API, and public IPv4 charges; and observability ingestion and retention. Each observation must name
Region, date, service configuration, unit, and official source. These values are intentionally absent because no
deployment tuple has been selected and volatile prices must not be fabricated or silently treated as zero.

## Recommended staged AWS architecture

### Stage 1 — one-domain research prototype

- S3, IAM, KMS, VPC, CloudWatch, CloudTrail.
- One EC2 G5/G6/G6e candidate chosen by measured memory need and current availability.
- Encrypted EBS plus S3 checkpoints.
- ECR if containerized.
- vLLM and SGLang GPU benchmarks; llama.cpp for the GGUF/CPU path.
- IaC for networking, role, bucket, key, registry, host, alarms, and automatic shutdown.

### Stage 2 — repeatable managed experiments

- Add SageMaker training jobs and Managed Spot where checkpoint/resume is proven.
- Add Step Functions only for durable multi-step pipelines.
- Add a model registry/release manifest and approval gates.
- Compare a SageMaker real-time endpoint with self-managed EC2/ECS using identical model/runtime artifacts where
  possible.

### Stage 3 — measured production need

- ECS service or SageMaker endpoint autoscaling, health checks, staged deployment, and rollback.
- API Gateway only if its API-management features and streaming constraints fit.
- Secrets Manager for actual runtime secrets.
- TensorRT-LLM if NVIDIA-specific optimization produces a reproduced advantage.
- EKS, FSx for Lustre, multi-node training, Trainium/Inferentia, or Bedrock Custom Model Import only after a dedicated
  compatibility and total-cost experiment.

## Verification gates before implementation claims

- [ ] Exact instance type is listed in the target Region and launch capacity/quota is confirmed.
- [ ] Exact AMI/container and accelerator software versions are pinned.
- [ ] Exact model, tokenizer, adapter, quantization, and runtime combination passes a smoke test.
- [ ] Training resume was tested after forced termination, not merely configured.
- [ ] Artifacts are hashed, encrypted, licensed, scanned, and traceable to data/code/config.
- [ ] Runtime quality is compared with the reference model.
- [ ] Performance is measured under a declared workload and concurrency.
- [ ] Dated prices and every material attached resource are recorded.
- [ ] Least-privilege, network, logging, retention, and deletion controls are reviewed.
- [ ] Deployment rollback and automatic GPU cleanup are tested.
- [ ] No runtime retrieval path was introduced into the primary architecture.

## Survey conclusion

AWS provides several viable SLM paths, but “AWS support” is too broad to be a compatibility claim. For this project, the
defensible small-scale baseline is S3 plus security/observability controls and one right-sized NVIDIA EC2 instance, with
SageMaker evaluated for managed training and hosting rather than adopted automatically. vLLM and SGLang are the current
GPU-serving priorities; llama.cpp is the required GGUF/CPU/cloud-edge comparison; TensorRT-LLM and ONNX Runtime are
targeted optimization/portability experiments. TGI should be retained only for a demonstrated compatibility need because
its vendor now labels it maintenance mode.

Bedrock is useful where its exact customization/import matrix matches the selected model and operational goal. It does
not replace the open-runtime experiments or remove the need to verify Region, architecture, feature, quota, and price.
P-class/Blackwell clusters, Kubernetes, distributed filesystems, and custom accelerator paths are available, but are
likely unnecessary until measured evidence shows that a single-GPU SLM workflow is insufficient.
