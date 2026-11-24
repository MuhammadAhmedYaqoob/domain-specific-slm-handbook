# Lesson 1: Select the smallest useful AWS architecture

## Learning objectives

After this lesson, you can explain what each foundational AWS service contributes, choose EC2 or SageMaker for a
specific workload, select a GPU from memory and runtime evidence, and reject unnecessary services.

## 1. Begin with the workload, not the service catalog

Write down the exact model revision, training method, sequence length, precision, batch, serving context, concurrency,
availability target, data class, and release frequency. A service is justified only when it satisfies one of these
requirements more safely or cheaply than the simpler option.

For the worked example, `garage-slm` fine-tunes a 3-billion-parameter model with QLoRA, serves fewer than ten concurrent
interactive users, and publishes a 4-bit edge artifact. This does not justify Kubernetes or multi-node training.

## 2. Understand the security and artifact foundation

### IAM and service roles

IAM policies authorize API calls. A **trust policy** says who may assume a role; a **permissions policy** says what an
assumed session may do. Use separate roles for infrastructure deployment, training, evaluation, release publication,
and serving. The SageMaker execution role is a service role trusted by SageMaker; an EC2 instance profile delivers a
role to an instance; an ECS task role delivers a role to one task. Do not confuse these with the role used by an
operator to create the resource.

### VPC

A VPC contains subnets, route tables, security groups, and service endpoints. Put GPU hosts in private subnets without
public IP addresses. Security groups are stateful network filters, not user authorization. An S3 gateway endpoint keeps
S3 traffic off the public internet, but IAM and bucket policies must still allow the operation.

### KMS

KMS keys authorize cryptographic operations and create audit events. S3, EBS, ECR, and CloudWatch can use a
customer-managed key. Separate encryption keys from asymmetric release-signing keys. A KMS key policy can block an
otherwise valid IAM policy, so diagnose both.

### S3 and ECR

S3 stores datasets, checkpoints, evaluation evidence, and releases. Enable versioning and record both object Version ID
and SHA-256. ECR stores the training and serving container. A tag such as `latest` can move; an image digest cannot.
The container lifecycle is:

1. build from a pinned base image;
2. install locked dependencies;
3. scan and test;
4. push to ECR;
5. record the returned digest;
6. deploy that digest;
7. retain active and rollback images;
8. expire unreferenced images under policy.

## 3. EC2 versus SageMaker AI

EC2 gives you a virtual machine. You choose the AMI, driver, container runtime, filesystem, process supervisor,
checkpoint uploader, and shutdown behavior. This is useful for kernel experiments and direct debugging, but you own the
host lifecycle.

SageMaker Training creates temporary compute for a submitted job. You supply a container, input channels, output paths,
execution role, network settings, and limits. The service provisions and releases the instance. You still own complete
application checkpoints and reproducible inputs.

| Decision | Prefer EC2 | Prefer SageMaker Training |
| --- | --- | --- |
| host/driver/kernel control | required | standard container contract is enough |
| interactive diagnosis | frequent | job logs and artifacts are enough |
| lifecycle engineering | team can patch, stop, and recover hosts | managed job provisioning is valuable |
| Spot restart | custom automation is acceptable | managed restart plus checkpoint sync is valuable |
| cost comparison | include instance and attached resources | include managed job and attached resources |

For serving, compare one EC2 host, ECS on EC2, and a SageMaker real-time endpoint. ECS adds task replacement and
deployment control; SageMaker adds a managed endpoint lifecycle. Neither makes an incompatible model/runtime work.

## 4. Choose a GPU by constraints

Do not select a GPU family because it is newer. Estimate memory, eliminate incompatible software paths, then benchmark.

For training, include weights, quantized base if used, gradients, trainable parameters, optimizer state, activations,
CUDA workspaces, and fragmentation. For serving, include weights, runtime allocation, KV cache, batching, and safety
margin. A rough KV-cache estimate is:

$$

M_{KV}=B \times L \times T \times 2 \times H_{kv} \times D \times s

$$

where $B$ is concurrent sequences, $L$ layers, $T$ cached tokens, $H_{kv}$ key/value heads, $D$ head
dimension, and $s$ bytes per cache element. Measure the actual runtime allocation.

AWS currently documents G-family GPU instances for graphics/inference/general accelerated workloads and P-family
instances for larger training. Candidate GPUs can include A10G, L4, and L40S depending on the family and Region.
Trainium and Inferentia require a separately qualified Neuron software path.

Selection procedure:

1. calculate a pessimistic memory envelope;
2. verify the exact runtime, quantization, compute capability, driver, and architecture;
3. check [instance types by Region](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-regions.html);
4. inspect the relevant Service Quota;
5. request quota early, understanding that quota does not guarantee capacity;
6. capture the current official rate;
7. run a short representative workload;
8. compare quality, memory, wall time, utilization, interruption, and all-in cost.

If the run fails with CUDA OOM, preserve the allocator report and configuration. Try a smaller microbatch, activation
checkpointing, shorter sequence, PEFT, or a larger-memory candidate. Do not silently change several variables at once.

## 5. Decide which higher-level services are justified

| Service | Use when | Usually defer when |
| --- | --- | --- |
| ECR | containers are deployed | no container exists |
| ECS on EC2 | task replacement, desired count, or blue/green is needed | one learning host is sufficient |
| EKS | an existing Kubernetes platform or measured scheduling need exists | one team and one model |
| Step Functions | multi-stage retries and durable state transitions matter | one job script is clear enough |
| Lambda | small control-plane validation or notifications | GPU inference is the workload |
| API Gateway | its auth, quota, payload, timeout, and streaming contract fit | ALB plus private gateway is simpler |
| Secrets Manager | a real non-IAM secret needs controlled retrieval/rotation | workload roles solve access |
| FSx for Lustre/EFS | measured shared or multi-node I/O requires it | one node stages immutable data from S3 |
| Bedrock | exact import/customization support and product contract fit | low-level runtime control is required |

## 6. Worked foundation configuration

The following Terraform creates a learning bucket and ECR repository. It intentionally demonstrates substitution
without token-like illustrative values. Save it as `foundation.tf`, then change the default account-derived bucket suffix and
Region for your environment.

```hcl
terraform {
  required_version = ">= 1.7.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {
  type        = string
  description = "Region verified for the required GPU and services."
  default     = "us-west-2"
}

variable "artifact_bucket_name" {
  type        = string
  description = "Globally unique name; replace 111122223333 with your account ID."
  default     = "garage-slm-111122223333-us-west-2"
}

resource "aws_kms_key" "artifacts" {
  description             = "garage-slm-learning-artifacts"
  enable_key_rotation     = true
  deletion_window_in_days = 30
}

resource "aws_s3_bucket" "artifacts" {
  bucket = var.artifact_bucket_name
  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "artifacts" {
  bucket = aws_s3_bucket.artifacts.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "artifacts" {
  bucket = aws_s3_bucket.artifacts.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.artifacts.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "artifacts" {
  bucket                  = aws_s3_bucket.artifacts.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_ecr_repository" "training" {
  name                 = "garage-slm-training"
  image_tag_mutability = "IMMUTABLE"
  encryption_configuration {
    encryption_type = "KMS"
    kms_key          = aws_kms_key.artifacts.arn
  }
  image_scanning_configuration {
    scan_on_push = true
  }
}
```

Commands:

```bash
terraform fmt -check
terraform init
terraform validate
terraform plan -out=foundation.tfplan
terraform show foundation.tfplan
```

`fmt` checks canonical formatting. `init` installs the locked provider. `validate` checks Terraform structure, not AWS
permissions. `plan` calculates proposed changes; `show` supports human review. Apply only in an authorized learning
account with `terraform apply foundation.tfplan`.

Expected behavior: one protected bucket, one KMS key, and one immutable ECR repository are planned. A bucket-name
collision means the name is not globally unique. `AccessDenied` usually means the deployment role or organizational
policy blocks KMS, S3, or ECR. Do not weaken public-access controls to make the plan pass.

This example omits production key/bucket policies, CloudTrail, VPC, roles, remote state, retention, and budget alarms.
Add those through reviewed modules before shared or production use.

## 7. Architecture decision record

Replace bare checklists with a short decision record:

```yaml
decision: "Use SageMaker Training and ECS on EC2 for garage-slm learning"
region: "us-west-2"
trainingReason: "Managed job lifecycle and checkpoint synchronization"
servingReason: "Need container replacement and controlled blue/green"
rejected:
  EKS: "No existing Kubernetes platform or multi-service scheduling requirement"
  multiNode: "The measured workload fits one validated GPU"
verification:
  quotaOwner: "ml-platform@example.org"
  runtimeTest: "tests/runtime-compatibility-2026-11-24.json"
  priceEvidence: "cost/aws-price-evidence-2026-11-24.yaml"
cleanupOwner: "ml-platform@example.org"
expiresUtc: "2026-12-08T18:00:00Z"
```

Substitute real organizational identities and evidence paths. The expiry date is a control for the learning
environment, not a promise that AWS stops resources automatically.

## 8. Safe cleanup

Do not delete the bucket or KMS key while retained artifacts depend on them. First terminate compute and endpoints,
remove temporary volumes and networking, classify bucket versions and ECR digests, then apply reviewed lifecycle rules.
KMS key deletion has a waiting period and can make retained ciphertext unreadable.

## Official sources

- [EC2 accelerated computing](https://docs.aws.amazon.com/ec2/latest/instancetypes/ac.html)
- [SageMaker training](https://docs.aws.amazon.com/sagemaker/latest/dg/how-it-works-training.html)
- [SageMaker Managed Spot Training](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training.html)
- [Choosing an AWS container service](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/choosing-aws-container-service.html)
- [IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [ECR image immutability](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)
