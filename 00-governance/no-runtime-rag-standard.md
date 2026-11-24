# No-Runtime-RAG Standard

## The design goal

The primary specialist model must carry its domain capability in released model parameters or packaged trained
adapters. After approved artifacts are installed, answer generation MUST work with the network disabled.

This is a deliberate engineering constraint, not a claim that retrieval-augmented generation (RAG) is always inferior.
It lets the project study what a genuinely trained specialist SLM can do under offline, privacy, latency, and device
constraints. The corresponding system design is in the
[no-runtime-RAG architecture](../10-multi-domain-architecture/no-runtime-rag-architecture.md).

## What is not allowed during answer generation

The primary inference path MUST NOT:

- search a vector database, web index, document store, knowledge graph, or external corpus;
- embed a query to select passages, dynamically chunk documents, or insert selected prose into the prompt;
- call a remote teacher, larger model, human answer service, or answer API;
- hide retrieval behind a plugin, tool, agent, cache, “memory,” grounding service, or router; or
- describe itself as offline when any of those dependencies are required to answer.

A locally installed manual still counts as runtime retrieval if software searches it and inserts matching passages into
the model context. Changing the name of the component does not change its function.

### Noncompliant example

A mechanic asks about diagnostic code P0301. The app searches a bundled manual, selects three passages, adds them to the
prompt, and asks the SLM to compose an answer. The network is offline, but this is still runtime RAG.

### Compliant example

During dataset preparation, engineers extract licensed repair material, verify it, and use it for continued pretraining
and supervised fine-tuning. At runtime, the installed model answers from its parameters without searching that material.

## What is allowed

The project MAY use external information before or after primary inference for:

- lawful acquisition, extraction, cleaning, labeling, synthesis, deduplication, and expert validation of training data;
- continued pretraining, supervised fine-tuning, preference optimization, distillation, and quantization calibration;
- evaluation ground truth that is hidden from the model being tested;
- deterministic arithmetic, unit conversion, schema validation, or local sensor input;
- a user-requested exact-identifier lookup that does not select prose to build an answer;
- model update checks and downloads outside answer generation; and
- a separately named RAG research comparator that is never presented as the compliant primary product.

Every runtime tool MUST declare its input, output, data source, network behavior, and purpose. A tool that returns domain
facts is presumed noncompliant until an architecture review shows otherwise.

## How to review a borderline tool

Ask these questions:

1. Does the tool select domain information in response to the user's content?
2. Does it return prose or facts that are inserted into the generation prompt?
3. Would the answer fail or materially change if the tool or network were unavailable?
4. Is a remote model or human effectively answering for the local model?

If any answer is yes, treat the tool as retrieval until a reviewer demonstrates a narrower deterministic purpose.

## Release evidence

Each release candidate MUST include:

1. a data-flow diagram from tokenization to output;
2. every process, store, call, tool, cache, adapter, router, and update channel;
3. a component classification: `TRAINING_ONLY`, `EVALUATION_ONLY`, `UPDATE_PLANE`, or `INFERENCE_PATH`;
4. allowed and denied outbound calls during inference;
5. an offline test protocol and retained result;
6. an explanation of any exact-identifier lookup; and
7. a named reviewer and dated decision.

The primary runtime must then pass network-disabled end-to-end inference, static dependency inspection, observation of
network/filesystem/tool calls under normal and adversarial prompts, prompt inspection, and failure tests showing that
telemetry and update services are optional.

Record this evidence under the [evidence standard](evidence-and-citation-standard.md) and in the
[implementation ledger](../15-audit-ledgers/implementation-verification-ledger.md). A diagram or design statement alone
does not prove runtime behavior.

## The limitation learners must remember

Training does not store a perfect, searchable copy of every fact. A no-runtime-RAG model can have stale knowledge,
forget details, combine contradictory examples, hallucinate, or answer outside its domain. Offline operation is not
proof of factuality, privacy, or safety.

User-facing material MUST explain those limits, the update cadence, uncertainty behavior, and domain boundary. For
safety-critical questions, follow the [safety standard](safety-and-risk-standard.md): require independent verification,
qualified escalation, or refusal as appropriate. Never add covert retrieval to avoid admitting that the model lacks
current information.

There is no waiver for runtime RAG in the primary architecture. A retrieval-enabled companion must be separately named,
evaluated, and reported. This policy is authoritative; implementation compliance remains unverified until the runtime
gates are executed.
