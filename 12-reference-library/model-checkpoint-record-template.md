# Model checkpoint record tutorial

A family name such as “Qwen” is not enough to reproduce a result. This record teaches how to identify one immutable
checkpoint and keep documented claims separate from local tests.

## Worked example: Qwen2.5-0.5B-Instruct

This is an **illustrative review record**. Recheck the mutable model card, license, files, and revision before an
experiment or release.

### Identity and architecture

- **Record ID:** `qwen-qwen2.5-0.5b-instruct-revision-to-freeze`
- **State:** candidate for research
- **Exact checkpoint:** `Qwen/Qwen2.5-0.5B-Instruct`
- **Immutable revision:** not yet frozen; obtain the full Hugging Face commit hash before download
- **Variant:** instruction-tuned text-generation checkpoint
- **Official card:** https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct
- **Local hashes:** not downloaded; hash every selected file with SHA-256 after freezing the revision
- **Architecture:** decoder-only dense Transformer; read exact layer, width, attention, vocabulary, and context values
  from the pinned `config.json`.
- **Context:** use the model card's advertised value only as documentation evidence; record the longest context that
  actually passes the project's memory and quality protocol.

### Tokenizer and conversation contract

- **Tokenizer source:** freeze the tokenizer files at the same repository revision as the weights.
- **Chat template:** copy from the pinned tokenizer configuration and hash it.
- **Special tokens:** inspect tokenizer files programmatically; do not infer IDs from another Qwen checkpoint.
- **Golden test:** encode a fixed two-message conversation and retain the rendered text and token IDs before training
  or conversion.

### Training, rights, and safety

- **Developer:** Qwen Team.
- **Training facts:** copy only facts documented by the pinned model card or technical report.
- **License:** inspect the exact repository `LICENSE` file at the frozen revision and record its version and URL.
- **Intended use and limitations:** preserve the card's language and safety qualifications; add project-specific
  domain and hallucination tests.
- **Remote code and serialization:** inspect file types and loading configuration before allowing code execution.
- **Evidence boundary:** availability of weights does not prove commercial permission, reproducibility, mobile
  compatibility, or domain accuracy.

### Compatibility and derived artifacts

| Operation | State | Evidence | Required next action |
| --- | --- | --- | --- |
| Full fine-tuning | documented by ecosystem tooling | `DOC-VERIFIED` only after checking exact versions | Execute a smoke test |
| LoRA or QLoRA | planned | `PLANNED EXPERIMENT` | Pin PEFT, Transformers, CUDA, and target modules |
| GGUF conversion | planned | `PLANNED EXPERIMENT` | Pin llama.cpp commit and compare logits |
| Android inference | not tested | No execution claim | Run on a named device and capture fallback, memory, and speed |

For each derived artifact, record its SHA-256, converter commit, command, source revision, quantization algorithm,
format, effective bits per weight, quality result, and runtime tuple.

### Decision

- **Candidate role:** low-cost pipeline smoke test and low-memory baseline.
- **Advantages supported so far:** an official model card and small nominal parameter scale.
- **Blockers:** immutable revision, exact terms review, hashes, conversion parity, domain quality, and physical-device
  evidence are still required.
- **Approval:** research candidate only; release approval is not implied.

## Field-by-field instructions

1. **Identity:** record owner, repository, full commit, variant, access date, manifest, and hashes. Create separate
   records for base, instruct, quantized, and converted variants.
2. **Architecture:** inspect the pinned configuration. Record total and active parameters, layer dimensions, attention
   form, position method, vocabulary, operators, and context. Explain how each value was obtained.
3. **Tokenizer:** pin all files and the chat template. Golden tests detect accidental tokenizer or template changes.
4. **Training and use:** distinguish documented facts from inference. Describe data provenance quality, intended use,
   limitations, languages, and benchmark conditions.
5. **Rights and supply chain:** review weight, tokenizer, and code terms independently. Record gating, notices,
   serialization risks, hashes, and source trust.
6. **Compatibility:** use staged states: documented, converted, loads, executes, numerically validated, quality
   validated, and production qualified.
7. **Performance:** never copy a headline result into project evidence. Link controlled runs for artifact bytes,
   loaded memory, peak memory, TTFT, prefill, decode, energy, and thermal behavior.
8. **Unknowns:** name the file, command, test, or reviewer that can resolve the value; do not leave blanks.

## Reusable copy

### Record identity

- **Record ID:** `organization-checkpoint-full-commit-prefix`
- **Lifecycle:** candidate requiring an immutable freeze
- **Family and variant:** ExampleLM family; 1B instruction-tuned text checkpoint
- **Exact repository:** `organization/examplelm-1b-instruct`
- **Revision:** Full 40-character commit copied from the host
- **Parent:** `organization/examplelm-1b-base` at its full commit
- **Card and access date:** Direct model-card URL; 2026-11-24
- **Manifest:** `artifacts/examplelm-1b-instruct/manifest.json` containing SHA-256 values

### Architecture and tokenizer

- **Configuration:** Decoder-only dense Transformer; values read from pinned `config.json`
- **Dimensions:** 24 layers, 2,048 hidden width, and 5,632 FFN width; replace with inspected values
- **Attention:** GQA with 16 query heads and 4 KV heads; replace with inspected values
- **Position and normalization:** RoPE and RMSNorm with exact parameters from configuration
- **Parameters:** Exact calculated count plus calculation script revision
- **Context:** Advertised 32,768 tokens; project testing planned at 2,048, 8,192, and 32,768
- **Tokenizer:** Repository revision, file hashes, algorithm, normalization, special IDs, and template hash
- **Golden test:** `runs/tokenizer-golden-examplelm-20261124`

### Training, rights, and supply chain

- **Developer and stages:** Facts copied from the pinned card and technical report
- **Data and languages:** Documented categories only; verification action for omitted details
- **Intended use and limits:** Card summary plus project domain and safety constraints
- **Benchmark conditions:** Harness, prompt format, precision, and shot count copied from the source
- **Terms:** Exact weight-license name/version/URL, tokenizer license, code license, and acceptable-use policy
- **Distribution review:** Named reviewer confirms notices and mobile redistribution requirements
- **Security:** Safetensors inventory, remote-code decision, hash validation, and advisory review date

### Compatibility and decision

- **Training:** Planned LoRA smoke test with pinned tool versions and target modules
- **Derived artifact:** `examplelm-1b-q4-project-v1`, source hash, converter commit, command, format, and artifact hash
- **Runtime tuple:** Runtime version, backend, named device, OS, context, and evidence state
- **Quality and performance:** Evaluation and benchmark run IDs, never copied vendor numbers
- **Candidate role:** Edge baseline under a declared 2 GiB peak-memory budget
- **Blockers:** Exact license approval, tokenizer parity, conversion parity, quality floor, and sustained device test
- **Decision:** Candidate until all blockers have evidence
- **Recheck:** On model-card, runtime, converter, OS, or license change

### Completion checks

- [ ] Model, tokenizer, source files, and derived artifacts have immutable revisions and hashes.
- [ ] Family claims were not substituted for checkpoint evidence.
- [ ] Exact terms, architecture, template, and special tokens were inspected.
- [ ] Conversion, load, execution, parity, quality, and production approval remain distinct.
- [ ] Runtime evidence names version, backend, device, context, and fallback.
- [ ] Every unknown names a verification action.

Cross-links: [source record](source-record-template.md) · [runtime record](runtime-record-template.md) ·
[definition register](definition-register.md)
