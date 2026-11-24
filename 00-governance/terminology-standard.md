# Terminology Standard

## Why precise words matter

Many SLM engineering mistakes begin with two different quantities sharing one casual name. A “4-bit model” may describe
weight storage while its activations use 16-bit arithmetic. A model that “fits on a phone” may fit in storage but exceed
runtime memory. Precise terms make the engineering decision testable.

The [definition register](../12-reference-library/definition-register.md) owns normative meanings, the
[glossary](../12-reference-library/glossary.md) teaches those meanings, and
[abbreviations](../12-reference-library/abbreviations.md) owns expansions. They MUST agree; the definition register
controls until a conflict is reviewed.

Each material term needs an immutable ID such as `TERM-TIME-TO-FIRST-TOKEN`, its canonical name, definition, scope,
approved abbreviation and synonyms, deprecated forms, related distinctions, source IDs, owner, review date, and state.

## Authoring rules

- Use the canonical term in headings, captions, schemas, IDs, and first substantive use.
- Expand an abbreviation at first substantive use in each chapter or standalone guide.
- Use an approved synonym only after establishing the canonical term and without changing meaning.
- Preserve exact product capitalization, checkpoint IDs, and runtime identifiers.
- Qualify a term that has multiple accepted meanings.
- Register and review a new term before release.
- Mark deprecated language and link its replacement.

### Good example

> Time to first token (TTFT) was measured from request submission until the first decoded token became available.

The metric boundary is clear.

### Anti-example

> Latency was 400 ms.

The reader cannot tell whether this means model load, prefill, first token, one decode step, or the whole response.

## Distinctions the book must preserve

- A **small language model (SLM)** is a project category, not a universal parameter cutoff. Name the checkpoint or range.
- A **base model** is not an **instruction-tuned model** or **chat model**.
- **Continued pretraining (CPT)**, **domain-adaptive pretraining (DAPT)**, **supervised fine-tuning (SFT)**, and
  **preference optimization** use different objectives and data.
- **Full fine-tuning**, **parameter-efficient fine-tuning (PEFT)**, **LoRA**, and **QLoRA** are not interchangeable.
- **Post-training quantization (PTQ)** differs from **quantization-aware training (QAT)**. State weight, activation,
  compute, storage, and key-value (KV) cache precision separately.
- **Parameter count**, **download size**, **installed size**, **memory after load**, **peak runtime memory**, and
  **KV-cache memory** are different measurements.
- **TTFT**, **prefill throughput**, **decode throughput**, **inter-token latency**, and **end-to-end latency** answer
  different performance questions.
- **Execution**, **reproduction**, **static review**, **documentation verification**, and **external reporting** use the
  meanings in the [evidence standard](evidence-and-citation-standard.md).
- **Offline inference** means answer generation has no network dependency. Installation and updates may still use a
  network.
- **Open-weight** does not mean **open source**. Describe licenses for weights, code, and data separately.
- **Support**, **compatibility**, **exportability**, and **performance** are separate claims. A converter existing does
  not prove that a particular model converts or runs acceptably.
- **Training**, **validation**, **test**, **calibration**, and **synthetic** data have separate roles and leakage
  boundaries.
- **Accuracy**, **factuality**, **calibration**, **safety**, and **helpfulness** must not disappear into one undefined
  “quality” score.

## Quantities and names

Use SI units where practical. Distinguish binary `MiB`/`GiB` from decimal `MB`/`GB`. Explain whether `M` and `B`
parameter counts are rounded. Rates need numerator, denominator, and workload. Prices need currency, region, billing
unit, and date.

Do not use “real time,” “mobile-ready,” “production-ready,” “state of the art,” or “best” without criteria and evidence.
For example, “mobile-ready” could mean a 1.8 GB artifact installs successfully, or it could mean sustained interactive
generation under a thermal limit; those are not the same claim.

When relevant, identify a model by organization, exact repository/checkpoint ID, revision, variant, parameter class, and
quantization or export state. A family name alone is insufficient for compatibility or benchmark claims.

## Review workflow

1. Expand abbreviations and link unfamiliar terms.
2. Compare usage with the definition register.
3. Check units, metric boundaries, capitalization, and exact model names.
4. Search for vague superlatives and overloaded words such as “size,” “latency,” and “quality.”
5. Record conflicts and approved exceptions in the
   [cross-link and terminology ledger](../15-audit-ledgers/cross-link-and-terminology-ledger.md).
6. Obtain relevant domain review for safety-critical terminology.

A change to an approved definition must name affected files, schemas, and claims; explain migration impact; and record
the reviewer, date, and superseded term. This policy is authoritative, while individual glossary and register entries
retain their evidence-supported review state.
