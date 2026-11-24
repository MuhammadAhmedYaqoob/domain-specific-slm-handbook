<a id="part-09"></a>

# Part IX — AWS Training, Optimization, and Deployment

This part maps the governed SLM lifecycle onto AWS while keeping cloud services, prices, quotas, and compatibility
explicitly versioned. It covers isolated infrastructure, training, scalable serving, and signed delivery to edge fleets.

## Conceptual map

Identity and network boundaries protect data and control actions; object and image stores hold immutable inputs;
ephemeral compute trains or serves exact artifacts; registries and manifests connect lineage; observability records
behavior and cost; staged delivery promotes or rolls back cloud and edge releases. Managed services change operational
responsibility, not the need for evidence.

## Learning outcomes and dependencies

After Parts II–VIII, readers should be able to design least-privilege infrastructure, choose a training execution
pattern, preserve immutable artifacts, deploy measured serving paths, and deliver updates with staged rollout and
rollback. AWS documentation can establish service behavior; only project execution can establish this system’s
compatibility or cost.

## Chapters

1. [CH-33 — AWS Infrastructure for SLM Development](ch-33-aws-infrastructure-for-slm-development.md): accounts, IAM,
   networking, storage, registries, encryption, budgets, and audit.
2. [CH-34 — Training SLMs on AWS](ch-34-training-slms-on-aws.md): compute selection, containers, data access,
   distributed training, checkpoints, and teardown.
3. [CH-35 — AWS Inference Serving](ch-35-aws-inference-serving.md): serving patterns, autoscaling, observability,
   canaries, and rollback.
4. [CH-36 — AWS-to-Edge Model Delivery](ch-36-aws-to-edge-model-delivery.md): manifests, signing, distribution, staged
   updates, revocation, and device reporting.

## Decision checkpoint

Pin region, service/API versions, images and dependencies, model/tokenizer hashes, IAM boundaries, encryption keys,
quotas, and cost assumptions. Never deploy mutable tags or unsigned artifacts; never present an estimate as a bill.

### Practice before continuing

Trace one checkpoint from source data hash through training configuration, model registry entry, serving image, endpoint,
and rollback artifact. Every arrow needs an identity, authorization boundary, immutable version, and owner.

## Cross-links

[Part V](../part-05-edge-ai-and-mobile-inference/README.md) ·
[Part XII](../part-12-production-engineering-safety-and-maintenance/README.md) ·
[Evidence labels](../front-matter/evidence-labels.md)
