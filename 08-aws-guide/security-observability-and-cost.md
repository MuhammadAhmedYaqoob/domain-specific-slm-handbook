# Lesson 4: Secure, observe, and cost the AWS workload

## Learning objectives

You will design least-privilege roles and private paths, distinguish audit from telemetry, create actionable alarms,
obtain current AWS prices without copying stale numbers, calculate an all-in hypothetical cost, and clean up safely.

## 1. Security starts with separated responsibilities

Human users should authenticate through federation with MFA. Workloads should use temporary credentials from roles.
Never place access keys in code, images, user data, notebooks, Terraform outputs, or model files.

| Role | Allowed responsibility | Explicitly excluded |
| --- | --- | --- |
| data preparer | write quarantined and accepted dataset prefixes | train or release |
| trainer | read approved input versions; write one run prefix | mutate source or release objects |
| evaluator | read candidate; write evaluation report | overwrite candidate |
| release approver | approve exact hashes | build artifacts |
| publisher | publish approved hashes and invoke KMS signing | alter approval evidence |
| serving runtime | read one approved release; emit telemetry | read datasets or write releases |
| infrastructure role | apply scoped reviewed plans | invoke, sign, or inspect payloads |

A service role has two relevant policy surfaces: its trust policy and permissions. S3 bucket policies, ECR policies,
KMS key policies, VPC endpoint policies, service control policies, and permission boundaries may add further
restrictions. `AccessDenied` is therefore a policy-intersection problem, not proof that IAM “is broken.”

## 2. Worked least-privilege policy

This policy allows a training role to read approved data and base models, write one run, and use one KMS key:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListApprovedInputs",
      "Effect": "Allow",
      "Action": ["s3:ListBucket", "s3:GetBucketVersioning"],
      "Resource": "arn:aws:s3:::garage-slm-111122223333-us-west-2",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["datasets/approved/*", "models/base/*"]
        }
      }
    },
    {
      "Sid": "ReadApprovedVersions",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:GetObjectVersion"],
      "Resource": [
        "arn:aws:s3:::garage-slm-111122223333-us-west-2/datasets/approved/*",
        "arn:aws:s3:::garage-slm-111122223333-us-west-2/models/base/*"
      ]
    },
    {
      "Sid": "WriteThisRun",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:AbortMultipartUpload"],
      "Resource": "arn:aws:s3:::garage-slm-111122223333-us-west-2/runs/sft-20261124-001/*"
    },
    {
      "Sid": "UseArtifactKey",
      "Effect": "Allow",
      "Action": ["kms:Decrypt", "kms:Encrypt", "kms:GenerateDataKey", "kms:DescribeKey"],
      "Resource": "arn:aws:kms:us-west-2:111122223333:key/12345678-1234-1234-1234-123456789012"
    }
  ]
}
```

Substitute the bucket, run ID, account, Region, and actual key UUID. The role cannot delete objects, change IAM, manage
the key, sign a release, or publish to the release prefix. The example still needs matching bucket/key policies,
encryption conditions, organization controls, and tests using the actual role.

Validate positive and negative cases. A trainer should read an approved object version and write its run; it should be
denied when reading quarantine data, deleting a checkpoint, or writing a release.

## 3. Network and encryption model

Use private subnets without public IP addresses for training and serving. Security groups should permit only named
flows, such as load-balancer-to-gateway and gateway-to-runtime. Use Systems Manager instead of public SSH. Add an S3
gateway endpoint and interface endpoints for actual dependencies when their security and cost trade-offs are justified.
An endpoint creates a path; endpoint, identity, resource, and KMS policies still control access.

Map every architecture arrow:

| Source | Destination | Protocol | Identity | Data | Failure |
| --- | --- | --- | --- | --- | --- |
| training task | S3 endpoint | HTTPS 443 | training role | approved dataset/checkpoints | job stops; checkpoint retry |
| gateway | runtime | HTTP in private task/SG | security-group path | validated request | 503/timeout |
| runtime | CloudWatch Logs endpoint | HTTPS 443 | task role | metadata-only logs | buffer/drop under policy |

Encrypt S3, EBS, snapshots, ECR, logs, CloudTrail, and Terraform state according to classification. Separate key
administration from use. TLS protects data in transit; verify distributed-training encryption rather than assuming it.
Use Secrets Manager only for real non-IAM secrets.

## 4. Supply-chain and release security

- pin model/data revisions and use safe serialization such as `safetensors`;
- disable unreviewed remote code and reject pickle from untrusted sources;
- lock dependencies, scan images, generate an SBOM, sign/attest, and deploy by digest;
- isolate conversion tools and quarantine output until quality, safety, license, and hash gates pass;
- sign a release manifest that binds model, tokenizer, template, runtime flags, evaluation, and rollback;
- retain the prior complete tuple until the rollback window closes.

Encryption does not prove that an artifact is approved. The signed manifest supplies an offline-verifiable approval
statement.

## 5. CloudTrail versus CloudWatch

**CloudTrail** records AWS API and configuration activity: role changes, key operations, bucket policy changes, and
deployment calls. Configure protected centralized storage, KMS encryption, validation, retention, and alerts. Consider
S3 data events for sensitive prefixes after evaluating volume and cost.

**CloudWatch** receives application logs, metrics, dashboards, and alarms. It answers questions such as “Is TTFT
degrading?” and “Did checkpoint age exceed the recovery objective?” It is not an approval ledger.

### Training telemetry

Collect loss, learning rate, token count, step/data/communication time, GPU/host utilization and memory, disk, network,
checkpoint age and verification, Spot interruption, resume source, OOM, NaN, and retry count.

### Serving telemetry

Collect accepted/rejected requests, errors, active streams, queue and TTFT, prefill/decode/inter-token latency, token
distributions, context rejection, cancellation, GPU/KV pressure, OOM, desired/running/queued replicas, placement
failure, and exact release identity.

Do not log raw prompts, responses, credentials, model bytes, or full signed URLs by default. Keep request IDs in logs,
not metric dimensions. Apply explicit retention by log class.

## 6. Turn signals into runbooks

An alarm needs an owner, threshold rationale, notification, containment, recovery, and test.

| Alarm | Example signal | Immediate action | Recovery |
| --- | --- | --- | --- |
| stale checkpoint | no accepted checkpoint within lost-work objective | avoid risky interruption; inspect writer | repair and verify path |
| serving regression | p95 TTFT/error exceeds baseline threshold | stop canary; limit admission | roll back complete tuple |
| GPU OOM | restart/OOM count above zero for canary | remove canary | reduce limits or restore blue |
| idle accelerator | no work and low GPU use beyond expiry window | confirm owner; drain | terminate nonproduction resource |
| high-risk audit event | key/policy/public-access mutation | contain principal; freeze release | restore reviewed state and investigate |
| cost anomaly | service/tag spend departs from forecast | identify resource and actor | stop expired/unapproved capacity |
| artifact verification failure | signature or hash mismatch | quarantine and stop promotion | restore exact approved version |

Test alarms in nonproduction. A configured alarm is not evidence that notification and rollback work.

## 7. Obtain current prices

Never treat a price copied into a lesson as current. For each estimate:

1. record UTC timestamp, Region, service, exact SKU/instance, purchase model, OS/tenancy, and unit;
2. open the official pricing page or AWS Pricing Calculator;
3. reproduce non-Spot catalog rates with the Price List Query API when practical;
4. obtain Spot history from the Spot API/source because catalog pricing is not Spot pricing;
5. preserve the calculator export, SKU/rate code, or dated screenshot/record;
6. measure actual usage, including failed and idle time;
7. include storage, requests, snapshots, ECR, logs, metrics, KMS, network, NAT, public IPv4, load balancers, WAF,
   orchestration, blue/green overlap, and support/tax assumptions;
8. reverify before approval.

Example Price List query setup:

```bash
aws pricing get-products \
  --region us-east-1 \
  --service-code AmazonEC2 \
  --filters \
    Type=TERM_MATCH,Field=instanceType,Value=g5.2xlarge \
    Type=TERM_MATCH,Field=location,Value="US West (Oregon)" \
    Type=TERM_MATCH,Field=operatingSystem,Value=Linux \
    Type=TERM_MATCH,Field=tenancy,Value=Shared \
    Type=TERM_MATCH,Field=preInstalledSw,Value=NA \
    Type=TERM_MATCH,Field=capacitystatus,Value=Used
```

The Pricing API endpoint is queried in `us-east-1`, while the `location` filter selects Oregon pricing. The output can
contain multiple terms and dimensions; inspect the SKU, unit, begin date, and description rather than selecting the
first number mechanically. Verify filter values against returned product attributes.

## 8. Worked hypothetical cost calculation

The following rates are **hypothetical teaching values, not AWS prices**:

- GPU compute: 6.00 currency units/hour for 10.5 measured hours;
- EBS: 0.10 currency units/GB-month for a 200 GB volume retained 2 days;
- S3: 0.025 currency units/GB-month for 150 GB retained 30 days;
- logs and requests combined: 4.00 currency units for the run;
- NAT and transfer: 3.50 currency units for the run.

Using a 30-day month:

$$

C_{compute}=10.5 \times 6.00=63.00

$$

$$

C_{EBS}=200 \times 0.10 \times \frac{2}{30}=1.33

$$

$$

C_{S3}=150 \times 0.025=3.75

$$

$$

C_{total}=63.00+1.33+3.75+4.00+3.50=75.58

$$

If the run processed 8.4 million accepted training tokens, the illustrative cost per million tokens is
`75.58 / 8.4 = 9.00` currency units, rounded. Replace every hypothetical rate with a dated official rate and every
quantity with measured usage. Match each service's real billing unit and rounding rules.

For serving:

$$

C_{successful\ request}=
\frac{C_{fixed\ period}+C_{variable\ period}}{N_{successful\ requests}}

$$

Report the request/token distribution and latency objective; otherwise two services are not comparable.

## 9. Populate a price evidence record

```yaml
schema: aws-price-evidence/v1
verifiedUtc: "2026-11-24T08:30:00Z"
region: "us-west-2"
workload:
  runId: "sft-20261124-001"
  measuredStartUtc: "2026-11-24T09:00:00Z"
  measuredEndUtc: "2026-11-24T19:30:00Z"
rates:
  - service: "EC2"
    resource: "g5.2xlarge Linux shared tenancy"
    purchaseModel: "On-Demand"
    unit: "Hrs"
    currency: "USD"
    unitRate: 6.00
    rateNote: "HYPOTHETICAL TEACHING VALUE; NOT AN AWS PRICE"
    sourceUrl: "https://aws.amazon.com/ec2/pricing/on-demand/"
usage:
  - resource: "EC2 g5.2xlarge"
    quantity: 10.5
    unit: "Hrs"
attachedCostCategories:
  - "EBS and snapshots"
  - "S3 storage and requests"
  - "NAT, public IPv4, load balancer, WAF, and transfer"
  - "ECR, CloudWatch, KMS, and orchestration"
assumptions:
  creditsExcluded: true
  taxesExcluded: true
  supportExcluded: true
status: "HYPOTHETICAL_EXAMPLE_NOT_FOR_BUDGET_APPROVAL"
```

The example is fully populated so no blank value can be misread as free. Replace `unitRate`, currency, source evidence,
timestamp, and status with a dated official observation before budget approval. Store evidence beside the run manifest.

## 10. Cost controls

Apply project, domain, environment, owner, run/release ID, cost center, and expiry tags. Use AWS Budgets for forecast and
actual alerts, Cost Explorer for analysis, and Cost Anomaly Detection for deviations. These are not hard caps and may
alert after spend occurs.

Use maximum training runtime/wait, minimum and maximum replicas, request/token/concurrency limits, log retention,
checkpoint-aware expiry, idle-GPU alarms, and an independent cleanup path. Buy commitments only after stable measured
demand.

## 11. Incident response

Detect and classify the incident; preserve CloudTrail, logs, manifests, hashes, identity, and deployment evidence;
contain the principal, key, network, capacity, or release without destroying evidence; restore a known-good tuple;
notify required parties; determine blast radius and root cause; rotate credentials/keys with dependency awareness; and
test the corrected controls. Offline edge devices cannot receive instant revocation.

## 12. Safe cleanup and residual billing

After preserving required evidence, enumerate EC2, SageMaker, ECS/EKS, EBS, snapshots, ENIs, load balancers, target
groups, NAT gateways, VPC endpoints, public IPv4, S3 versions and multipart uploads, ECR images, logs, alarms, secrets,
KMS grants, orchestration schedules, DNS, and CDN distributions. Record resource ID, action, actor, UTC time, retained
reason, and remaining billing dimension.

Do not schedule a KMS key for deletion while retained ciphertext needs it. Do not delete the known-good rollback
release before its signed window closes.

## Official sources

- [IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [VPC security best practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
- [CloudTrail security best practices](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/best-practices-security.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [Cost Anomaly Detection](https://docs.aws.amazon.com/cost-management/latest/userguide/manage-ad.html)
- [AWS Price List Query API](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/using-price-list-query-api.html)
- [AWS Pricing Calculator](https://calculator.aws/)
