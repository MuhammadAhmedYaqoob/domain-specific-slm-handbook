# Part III — Parameter-Efficient Fine-Tuning

<a id="part-03"></a>

This part asks which parameters need to change to acquire domain behavior. Full fine-tuning provides a reference point;
PEFT reduces trainable state and enables modular domain artifacts, but can introduce rank, target-module, merge, and
serving constraints.

## Conceptual map

Full fine-tuning changes the selected backbone weights directly. PEFT keeps most backbone weights fixed and learns a
smaller intervention. LoRA represents selected weight updates through two low-rank matrices; QLoRA changes how the
frozen base is stored during training, not the basic meaning of the LoRA update. Saving trainable memory, storing small
domain artifacts, and serving them efficiently are related but distinct goals.

## Learning outcomes and dependencies

After [Part II](../part-02-training-and-domain-adaptation/README.md), readers should be able to compare full tuning with
adapters, derive LoRA’s low-rank update, choose rank/scaling/targets, distinguish LoRA from QLoRA, and evaluate modern
variants without assuming novelty means superiority. The choices feed Parts IV, VII, and IX.

## Chapters

1. [CH-08 — Full Fine-Tuning Versus PEFT](ch-08-full-fine-tuning-versus-peft.md) compares memory, optimization, quality,
   storage, isolation, and serving implications.
2. [CH-09 — LoRA and Its Evolution](ch-09-lora-and-its-evolution.md) covers LoRA, QLoRA, AdaLoRA, DoRA, rsLoRA, LoRA+,
   PiSSA, LoftQ, OLoRA, EVA, and CorDA.

## Decision checkpoint

Record the base hash, trainable modules, rank, scaling, dropout, quantization state, merge policy, adapter license, and
evaluated artifact form. Benchmark merged and unmerged forms separately when the production runtime differs.

### Practice before continuing

For one \(4096\times4096\) projection, calculate LoRA trainable elements at ranks 8, 16, and 32. Then list what the
calculation omits: number of targeted projections, optimizer state, activations, frozen-base storage, and runtime merge
behavior.

## Cross-links

[Mathematical reference](../back-matter/appendix-mathematical-reference.md) ·
[Part IV](../part-04-model-compression-and-optimization/README.md) ·
[Part VII](../part-07-multi-domain-slm-engineering/README.md)
