<a id="glossary"></a>

# Glossary and Terminology Guide

Use this glossary when a chapter introduces an unfamiliar abbreviation or when two similar ideas need separation. Each
definition is intentionally short; follow the linked index or chapter for mechanisms and examples. The binding
definitions live in the [terminology standard](../../00-governance/terminology-standard.md).

- **Adapter:** Trained parameter module attached to a compatible base model.
- **ANE:** Apple Neural Engine, normally targeted through supported Apple frameworks.
- **Base model:** Pretrained model before instruction alignment.
- **Benchmark:** Frozen tasks, inputs, scoring rules, and execution conditions used for comparison; not merely a set of
  prompts.
- **Calibration (quantization):** Representative-data process used to determine quantization parameters; distinct from
  probabilistic calibration.
- **Calibration (probabilistic):** Agreement between predicted confidence and observed correctness under a declared
  protocol.
- **Continued pretraining (CPT):** Additional next-token training on a selected corpus.
- **Decode:** Autoregressive generation after prefill, commonly one token per active sequence step.
- **Distillation:** Training a student using teacher responses, distributions, representations, or attention.
- **Domain capability:** Measurable behavior associated with a specialist domain; not proof of exact factual storage.
- **DAPT:** Domain-adaptive pretraining, a form of continued pretraining on domain-relevant text.
- **Edge inference:** Inference executed on a constrained local target.
- **Evidence label:** Controlled term describing the trace behind a claim.
- **Full fine-tuning:** Updating all selected model parameters.
- **GGUF:** Model/container format commonly used with llama.cpp-family tooling; format availability does not prove
  target support.
- **Hard gate:** Requirement that cannot be compensated by better performance elsewhere.
- **Hallucination:** Generated content that is unsupported or false in the applicable evaluation context; fluent wording
  does not reduce the error.
- **Instruct model:** Model post-trained to follow instructions or dialogue conventions.
- **KV cache:** Stored attention keys and values reused during autoregressive decode.
- **LoRA:** Low-rank additive parameterization of selected weight updates.
- **NPU:** Neural processing unit; operator and precision support are device/toolchain specific.
- **On-device:** Required inference executes locally after artifact installation.
- **Operator fallback:** Execution of an unsupported graph operation on a different backend, potentially changing
  memory, latency, or power.
- **Pareto frontier:** Candidates not dominated across declared objectives after mandatory gates.
- **PEFT:** Parameter-efficient fine-tuning; updates a restricted parameterization.
- **Perplexity:** Exponentiated mean token negative log-likelihood under a fixed tokenizer/corpus/masking protocol.
- **Prefill:** Prompt processing that initializes model state and KV cache before the first generated token.
- **PTQ:** Post-training quantization.
- **QAT:** Quantization-aware training.
- **QLoRA:** LoRA training over a quantized frozen base using a declared compute path.
- **Runtime RAG:** Retrieval from an external corpus during inference followed by prompt/context augmentation.
- **Safety margin:** Deliberate headroom between a measured resource peak and the allowed limit.
- **SLM:** Small language model; “small” is contextual and must be accompanied by parameters and deployment constraints.
- **SFT:** Supervised fine-tuning on input/target demonstrations.
- **Structured pruning:** Removal of complete deployable structures such as channels, heads, or layers.
- **Time to first token (TTFT):** User-visible latency from the declared request boundary to the first generated token.
- **Tokenizer:** Rules and assets mapping text to token IDs and back; part of the model contract.
- **Unstructured pruning:** Setting arbitrary scalar weights to zero without necessarily shrinking dense shapes.
- **Weight-only quantization:** Reduced-precision weight storage/compute path while activations use another precision.

## Common distinctions

- **Artifact size is not peak memory:** files omit many live allocations.
- **Documented support is not executed compatibility:** a runtime feature still needs an exact-target test.
- **Base model is not instruct model:** next-token capability does not guarantee dialogue behavior.
- **CPT is not SFT:** CPT learns from token streams; SFT learns from demonstrations and target masks.
- **Quantization is not pruning:** one reduces numeric precision; the other removes or zeros structures or weights.
- **Offline is not automatically private or safe:** local models can still expose memorized data or produce harmful
  outputs.

## Term maintenance

Propose additions through the governance review process with definition, non-examples, aliases, source, affected
chapters, and migration notes. Chapters should link to the canonical standard rather than silently redefining
base/instruct, CPT/SFT, PTQ/QAT, storage/loaded/peak memory, or TTFT/decode throughput.

See also [notation and conventions](../front-matter/notation-and-conventions.md) and the
[detailed index](index-plan.md).
