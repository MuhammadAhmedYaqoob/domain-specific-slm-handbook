<a id="part-02"></a>

# Part II — Training and Domain Adaptation

This part separates four learning stages that are often mislabeled as “fine-tuning.” Pretraining learns broad token
statistics; continued pretraining shifts domain representation; supervised fine-tuning teaches task and dialogue
behavior; preference optimization adjusts ranked behavior. Their data, losses, risks, and evaluation baselines differ.

## Conceptual map

Think of adaptation as four different questions. Pretraining asks, “what token patterns should the model represent?”
CPT asks, “which domain distribution needs more representation?” SFT asks, “what response behavior should follow an
instruction?” Preference optimization asks, “which of several plausible behaviors should be favored?” Choosing data
before choosing the question commonly produces the wrong objective.

## Learning outcomes and dependencies

Readers should be able to select an objective, construct leakage-resistant splits, estimate training resources, detect
forgetting and alignment regressions, and explain where domain capability enters the parameters. Prerequisites are
[Part I](../part-01-language-model-foundations/README.md); read
[Part VI](../part-06-data-acquisition-and-dataset-engineering/README.md) concurrently before building data. This part
feeds PEFT, evaluation, and AWS training.

## Chapters

1. [CH-04 — Pretraining Fundamentals](ch-04-pretraining-fundamentals.md): token objectives, optimization, scaling
   variables, checkpoints, and data/compute trade-offs.
2. [CH-05 — Continued Pretraining and Domain Adaptation](ch-05-continued-pretraining-and-domain-adaptation.md):
   domain-adaptive pretraining, replay/mixing, catastrophic forgetting, and evaluation.
3. [CH-06 — Supervised Fine-Tuning](ch-06-supervised-fine-tuning.md): instruction schemas, masking, packing, chat
   templates, and supervised behavior learning.
4. [CH-07 — Preference Optimization and Post-Training](ch-07-preference-optimization-and-post-training.md): preference
   data, reward or direct objectives, safety alignment, and post-training evaluation.

## Architectural boundary

Training corpora and synthetic teacher outputs may be used offline when rights, provenance, and validation permit. The
resulting primary inference path may use only model weights and selected trained adapters—not runtime document retrieval
or remote-teacher fallback.

## Decision checkpoint

Declare the baseline artifact, objective sequence, data mixtures, tokenizer/chat template, trainable parameters, seeds,
stop criteria, safety tier, and frozen evaluation suites before training. A lower loss alone is not an acceptance
criterion.

### Practice before continuing

Classify three records: a raw manual paragraph, a user/assistant demonstration, and a chosen/rejected response pair.
Assign each to CPT, SFT, or preference training and explain why using one record unchanged for all three would be
incorrect.

## Cross-links

[Part I](../part-01-language-model-foundations/README.md) ·
[Part III](../part-03-parameter-efficient-fine-tuning/README.md) ·
[Experiment checklist](../back-matter/appendix-experiment-reporting-checklist.md)
