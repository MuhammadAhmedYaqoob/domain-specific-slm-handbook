# Domain-Specific Small Language Models

This repository is a book and engineering guide for building specialist small language models (SLMs). It follows the
whole path: understand the model, prepare lawful domain data, adapt and compress the model, evaluate it honestly, and
deploy it either on Amazon Web Services (AWS) or on an offline edge device.

The intended reader already knows Python and basic machine learning. You do not need prior experience with language
model training, mobile inference, or model governance. New ideas are introduced before the book asks you to use them.

## Choose a learning path

**Learn the subject from the beginning.** Start with the [textbook](02-textbook/README.md). Read Parts I–III in order,
then choose the cloud, edge, data, or evaluation chapters that match your goal.

**Build while you learn.** Use the [implementation handbook](03-implementation-handbook/README.md) beside the textbook.
The handbook turns concepts into commands, configurations, checks, and troubleshooting steps.

**Follow one complete project.** The [automotive case study](07-automotive-case-study/README.md) applies the method from
scope definition through model selection, training, compression, evaluation, and deployment.

**Extend the platform.** After the case study, use the
[multi-domain architecture](10-multi-domain-architecture/README.md) and
[new-domain material](02-textbook/part-11-extending-the-platform-to-other-domains/README.md) to add another specialty.

The [bookmap](BOOKMAP.md) shows every major resource and suggests routes for different goals.

## What you will learn

By the end, you should be able to:

1. explain how a decoder-only language model turns input tokens into generated tokens;
2. decide among continued pretraining, supervised fine-tuning, LoRA, and QLoRA;
3. build separate training, calibration, and evaluation datasets without leaking test answers;
4. compare quantization, pruning, and distillation using quality, memory, latency, and cost;
5. package and benchmark a model for AWS, Android, iOS, or another edge target;
6. preserve evidence so another engineer can understand what ran and reproduce it; and
7. set safe limits for specialist models, especially in pharmacy, repair, and animal-care domains.

## A practical way to use each chapter

1. Read the problem and intuition first.
2. Work through the numerical or architectural example.
3. Run a listing only after checking its execution label and prerequisites.
4. Complete the review questions or exercise.
5. Record your own result using the experiment framework instead of copying a reported number.

This approach matters because a command that looks plausible is not automatically compatible with your model, runtime,
or hardware.

## What has and has not been validated

The explanatory material, implementation specifications, experiment protocols, deployment guides, and assessments have
been written. The repository does **not** yet contain proof that its model-training experiments ran, that AWS
infrastructure was deployed, or that mobile builds passed on physical devices. Domain-expert, safety, data-rights, and
release reviews are also still required.

Treat code marked `UNVERIFIED IMPLEMENTATION` as a worked implementation that still needs the listed validation steps.
Treat `PLANNED EXPERIMENT` as a recipe for collecting evidence, not as a missing chapter and not as a result. Claims
from papers or vendors describe their conditions; they are not measurements from this project.

For example, a chapter may explain exactly how to benchmark a 4-bit model on Android. Until someone runs that protocol
and preserves the device details and raw output, the book can teach **how to validate** the model but cannot claim a
tokens-per-second result.

## The no-runtime-RAG boundary

The primary product deliberately excludes retrieval-augmented generation (RAG) at answer time. Domain capability must
come from released model parameters or packaged trained adapters. Once installed, the model must be able to answer
offline without searching documents, querying a vector database, or asking a remote teacher model.

Offline inference does not guarantee correct or current answers. A trained model can forget, combine conflicting
examples, or hallucinate. High-consequence advice therefore needs uncertainty handling, independent verification, and
qualified human escalation. The [no-runtime-RAG standard](00-governance/no-runtime-rag-standard.md) defines the exact
boundary, and the [safety standard](00-governance/safety-and-risk-standard.md) defines the required protections.

## Before publishing or deploying

The [governance guide](00-governance/README.md) explains how to label evidence, validate code, preserve experiment
records, write safe limitations, and decide whether an artifact is ready. These rules are not paperwork added after
engineering; they prevent a planned benchmark, an illustrative number, or an untested command from being mistaken for
real-world proof.
