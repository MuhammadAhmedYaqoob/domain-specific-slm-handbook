# Evidence and maturity ranking

**Research freeze:** 2026-11-23  
**Status:** authored prioritization based on the surveys in this directory. It is not a performance leaderboard and
contains no reproduced experiment.

## Two dimensions that must remain separate

**Evidence status** describes what supports a claim:

- `REPRODUCED`: this project independently reran a referenced result under declared conditions.
- `EXECUTED`: this project executed the stated implementation or measurement; not necessarily an external reproduction.
- `PAPER-REPORTED`: attributed primary-paper result.
- `VENDOR-REPORTED`: attributed developer/vendor result.
- `DOC-VERIFIED`: inspected official documentation, model card, repository, license, or API.
- `STATICALLY REVIEWED`: reasoned review without execution.
- `UNVERIFIED IMPLEMENTATION`: authored or available implementation not run here.
- `PLANNED EXPERIMENT`: protocol not yet executed.
- `ILLUSTRATIVE`: educational example/diagram.
- `ESTIMATE`: assumption-dependent planning calculation.

**Project maturity** describes implementation priority:

- **Historical:** useful for lineage, not a current implementation default.
- **Foundational:** necessary to understand or to serve as a control.
- **Core practical:** belongs in the first experimental funnel.
- **Advanced practical:** test after baseline or for a measured target constraint.
- **Experimental:** credible research hypothesis with higher evidence/tooling/deployment risk.

An established paper can still be experimental for this project; a documented API can still lack runtime evidence.

## Lesson: evidence strength and implementation priority answer different questions

The problem with a single “maturity” label is that it can mix scientific support, software maintenance, compatibility,
and project value. These dimensions diverge. A classic method may be well established but irrelevant to a mobile
runtime; a new documented API may be useful yet untested. The project therefore separates claim evidence from the order
in which engineering work should occur.

For example, a pruning paper can report retained quality (`PAPER-REPORTED`) while the method remains experimental for
this project because no Android sparse kernel is selected. Conversely, runtime-native 4-bit GGUF quantization is a core
edge experiment because current tooling makes it directly testable, even though no result is yet `EXECUTED`. This
prevents publication prestige or API documentation from masquerading as local performance evidence.

Use the ranking to stage experiments and to word claims accurately. Avoid treating “core practical” as “proven best,”
or “experimental” as “invalid.” A classification changes only when evidence for the exact claim and tuple changes.

## Ranking criteria

Each classification considers:

1. primary-source quality and publication status;
2. independent replication breadth (when known; not assumed);
3. official implementation and current maintenance;
4. exact checkpoint/runtime/backend coverage;
5. reproducibility burden and number of coupled variables;
6. artifact lineage, license, and rollback clarity;
7. target deployment value, not nominal compression;
8. training/conversion/device-test cost;
9. safety and quality failure surface; and
10. whether a simpler baseline can answer the same question.

No aggregate numerical score is used because unsupported values would create false precision.

## Model and adaptation ranking

<!-- markdownlint-disable MD013 -->
| Technology                                | Class                            | Evidence basis                              | Why prioritized or deferred                                                      |
| ----------------------------------------- | -------------------------------- | ------------------------------------------- | -------------------------------------------------------------------------------- |
| Exact base/instruct checkpoint separation | Foundational/core                | Official cards/licenses (`DOC-VERIFIED`)    | Required for causal DAPT/SFT comparisons                                         |
| Common SFT baseline                       | Core practical                   | Established practice and tooling            | Fastest direct domain-behavior baseline                                          |
| Full fine-tuning control                  | Foundational/core where feasible | Established                                 | Measures PEFT constraint cost; expensive                                         |
| LoRA                                      | Core practical                   | ICLR 2022, mature PEFT/tooling              | Reproducible adapter baseline; mergeable                                         |
| QLoRA                                     | Core practical                   | NeurIPS 2023, mature ecosystem              | Lowers training memory; deployment remains separate                              |
| CPT/DAPT with general replay              | Advanced practical               | Established domain adaptation literature    | Potential knowledge/distribution gains; higher compute/forgetting risk           |
| Prompt/prefix/P-Tuning/IA3/BitFit         | Foundational/conditional         | Published methods, current support varies   | Useful controls or special constraints; less default edge value than merged LoRA |
| Preference optimization after SFT         | Advanced practical               | Multiple published methods/tooling          | Use only for measured behavior gap; not knowledge acquisition                    |
| rsLoRA, DoRA, LoRA+                       | Advanced practical               | Published + current PEFT paths              | Focused ablations after stable LoRA                                              |
| AdaLoRA, PiSSA, EVA, LoftQ, CorDA         | Advanced/experimental            | Published, official/current implementations | Added schedules/calibration/provenance; match to measured problem                |
| OLoRA, QA-LoRA, aLoRA                     | Experimental/specialized         | Primary/preprint evidence and narrow paths  | Base mutation, backend coupling, or server-specific semantics                    |
| Astra, MiCA, NoRA-init                    | Experimental/emerging            | Very recent primary sources/current PEFT    | Independent maturity and domain transfer limited                                 |
| Train from scratch                        | Experimental for project         | Foundational methodology                    | Data/compute burden unjustified before adaptation baselines                      |
<!-- markdownlint-restore -->

The adaptation table prioritizes causal, reproducible baselines. Exact base/instruct separation and common Supervised
Fine-Tuning (SFT) come first; Low-Rank Adaptation (LoRA) and Quantized LoRA (QLoRA) then test practical update
constraints. Continued or Domain-Adaptive Pretraining (CPT/DAPT) and advanced adapter variants enter only after the
baseline reveals a knowledge, forgetting, rank, scaling, or initialization problem.

## Compression ranking

<!-- markdownlint-disable MD013 -->
| Technology                                              | Class                         | Evidence basis                            | Decision                                                       |
| ------------------------------------------------------- | ----------------------------- | ----------------------------------------- | -------------------------------------------------------------- |
| FP/BF16/FP16 reference                                  | Foundational                  | Hardware/runtime docs                     | Required parent and numerical control                          |
| Runtime-native INT8                                     | Core practical                | Broad CPU/GPU/NPU support                 | First portable lower-precision baseline                        |
| GGUF mature 4-bit type                                  | Core edge practical           | llama.cpp documentation/ecosystem         | First mobile CPU low-bit experiment                            |
| GPTQ/AWQ                                                | Core cloud practical          | ICLR/MLSys papers and mature CUDA serving | Compare for supported cloud model/runtime                      |
| SmoothQuant W8A8                                        | Advanced practical            | ICML paper, server implementations        | Throughput/NPU hypothesis where W8A8 kernels exist             |
| KV-cache quantization                                   | Advanced practical            | Runtime-specific docs/research            | Valuable at long context/concurrency; independent quality test |
| Target-specific QAT                                     | Advanced practical            | Established principle, exporter-dependent | Only after PTQ fails and exact runtime recipe exists           |
| Wanda/SparseGPT                                         | Foundational sparse baselines | ICLR/ICML primary papers                  | Research quality baselines; no speed assumption                |
| SliceGPT/LLM-Pruner/static layer pruning                | Advanced practical            | Primary papers/official code              | Portable dense reduction hypothesis with recovery              |
| ShortGPT/dynamic skipping                               | Experimental                  | Recent paper/code                         | Generation and runtime-control risks                           |
| Response/sequence distillation                          | Core edge practical           | Established KD/sequence KD                | Practical smaller dense student route with validation          |
| Logit KD                                                | Advanced practical            | Foundational/modern generative evidence   | Rich signal; alignment/storage/teacher cost                    |
| Feature/attention/on-policy KD                          | Experimental/advanced         | Primary research                          | Architecture coupling and high compute                         |
| AQLM/SpQR                                               | Experimental deployment       | ICML/ICLR research/code                   | Extreme compression; narrower kernels                          |
| QuIP/QuIP#                                              | Experimental                  | NeurIPS/ICML; maintenance/license caveats | Research only until supported target exists                    |
| BitNet/native low-bit                                   | Experimental separate branch  | Primary papers/bitnet.cpp                 | Not a drop-in quantizer; checkpoint/training comparison        |
| Arbitrary unstructured sparsity on dense mobile runtime | Not prioritized               | Kernel reality                            | No expected executable reduction without sparse support        |
<!-- markdownlint-restore -->

The compression table favors deployable representations before increasingly specialized research. Runtime-native
8-bit and 4-bit paths answer the largest practical question cheaply. Distillation and structured pruning enter when
architecture reduction is needed. Extreme low-bit and arbitrary sparse methods remain experiments until exact kernels,
licenses, and quality gates are available.

## Runtime and AWS ranking

<!-- markdownlint-disable MD013 -->
| Technology                           | Class                                      | Evidence basis                                | Decision                                              |
| ------------------------------------ | ------------------------------------------ | --------------------------------------------- | ----------------------------------------------------- |
| llama.cpp/GGUF CPU                   | Core edge                                  | Official repo/docs/releases                   | Portable correctness and low-bit baseline             |
| ExecuTorch XNNPACK                   | Core edge                                  | Official stable docs                          | Cross-platform graph CPU baseline                     |
| LiteRT-LM on supported Android model | Core/advanced                              | Official docs; language/backend status varies | Android graph/accelerator candidate                   |
| Core ML and MLX Swift LM             | Core Apple comparison                      | First-party docs/repos                        | Compare placement, memory, energy on named iPhones    |
| ORT Mobile/GenAI                     | Advanced                                   | Mature core; GenAI preview                    | Use for explicit ONNX/provider requirement            |
| MLC LLM                              | Advanced                                   | Active compiler/runtime                       | Cross-platform GPU/web use case                       |
| MNN/ncnn                             | Advanced/conditional                       | Active official projects                      | Enter for concrete model/hardware/footprint advantage |
| Direct QNN/Genie                     | Advanced vendor-specific                   | Qualcomm docs                                 | Controlled Snapdragon cohorts only                    |
| WebLLM/OpenVINO GenAI                | Conditional core for browser/Intel targets | Official active docs                          | Only when those targets are product requirements      |
| vLLM and SGLang                      | Core AWS GPU                               | Active official docs                          | First server-runtime comparison                       |
| llama.cpp server                     | Core AWS CPU/GGUF                          | Official docs                                 | Cloud-edge parity/CPU comparison                      |
| TensorRT-LLM                         | Advanced AWS                               | NVIDIA support matrices/docs                  | After baseline exposes measured NVIDIA need           |
| TGI for a new build                  | Historical/compatibility                   | Vendor maintenance-mode notice                | Retain only for existing dependency                   |
| Neuron/vLLM plugin                   | Experimental/advanced                      | AWS beta documentation                        | Dedicated architecture/feature/cost spike             |
| EKS/multi-node/FSx/P-class           | Deferred                                   | AWS documentation                             | Add only after single-node workload evidence          |
<!-- markdownlint-restore -->

The runtime table recommends a portfolio: portable edge CPU, a cross-platform graph route, operating-system-native
Apple paths, and mainstream AWS GPU serving. Vendor accelerators and specialist server compilers are second-stage
choices because their narrower compatibility must purchase a measured benefit.

## Evidence maturity by claim type

<!-- markdownlint-disable MD013 -->
| Claim                           | Minimum acceptable evidence                                                                                                                   |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| “Paper X reports improvement”   | Primary paper and exact scope; `PAPER-REPORTED`                                                                                               |
| “Runtime documents support”     | Current official support matrix/API; `DOC-VERIFIED`                                                                                           |
| “Exact checkpoint converts”     | Retained command/log/artifact hash; `EXECUTED`                                                                                                |
| “Conversion preserves behavior” | Tokenizer/template, logit/greedy and quality parity; `EXECUTED`                                                                               |
| “Runs on device/instance”       | Named hardware/OS/runtime execution and raw logs; `EXECUTED`                                                                                  |
| “Faster/lower memory/energy”    | Controlled repeated benchmark with uncertainty and phase definitions; `EXECUTED`                                                              |
| “Reproduces paper result”       | Paper-matched protocol and independent run; `REPRODUCED`                                                                                      |
| “Production ready”              | Quality/safety, lifecycle, security, load/soak, rollback, license, and operations gates; multiple evidence records, never documentation alone |
<!-- markdownlint-restore -->

## Prioritized experiment sequence

1. **Artifact gate:** licenses, exact revisions, tokenizer/template, source-framework baseline.
2. **Runtime feasibility:** untouched model conversion/load/parity before expensive adaptation.
3. **Adaptation core:** common SFT with LoRA, QLoRA, and feasible full-FT control.
4. **Objective study:** SFT versus DAPT→SFT on explicit base checkpoint.
5. **Compression core:** high precision, INT8, mature 4-bit target format.
6. **Architecture branch:** native small versus validated response-distilled and structured-pruned/recovered candidate.
7. **Combination:** quantize only qualified high-precision architecture winners.
8. **Runtime/device/cloud:** portable CPU, OS-native/graph, one accelerator-specific path; AWS vLLM/SGLang.
9. **Advanced ablations:** DoRA/rsLoRA/LoRA+, KV quantization, mixed precision, QAT.
10. **Experimental work:** extreme low-bit, joint pruning/KD, dynamic skipping, native low-bit training.

At every stage, successive halving retains only candidates that pass hard domain, general-retention, safety, lineage,
and compatibility gates.

## Project maturity decisions

### Implement now

- exact checkpoint/license manifest;
- tokenizer/template goldens;
- LoRA/QLoRA common SFT;
- runtime-native INT8/4-bit;
- llama.cpp and one graph runtime on edge;
- vLLM/SGLang on one right-sized AWS candidate;
- reproducible quality/safety/system measurement.

### Implement after baseline

- DAPT with replay;
- response distillation;
- structured pruning with recovery;
- KV-cache quantization and mixed precision;
- Core ML/MLX and Android vendor accelerator cohorts;
- target-specific QAT.

### Research only until justified

- sub-3-bit codebook/lattice methods;
- unstructured sparsity without a selected kernel;
- joint pruning/distillation;
- dynamic layer skipping;
- recent LoRA initialization variants without an identified baseline failure;
- multi-node and specialist accelerator infrastructure.

## Reclassification rule

Classification changes only when new evidence is attached:

- documentation change updates compatibility status, not execution status;
- one successful run upgrades only that exact tuple;
- a benchmark upgrades only the declared workload/hardware/configuration;
- a production release requires repeated quality, safety, lifecycle, security, and rollback evidence;
- a newer method does not outrank a simpler method by date.

## Evidence gaps

The directory currently contains no `EXECUTED` or `REPRODUCED` model experiment. Exact package/commit pins, conversion
outcomes, device measurements, cloud load tests, prices, and legal determinations remain unresolved. Model training-data
provenance is incomplete for most families. Physical-device and AWS recommendations are therefore shortlist decisions,
not winners.

## Direct references and cross-links

- [Report index](README.md)
- [Model comparison](model-family-comparison.md)
- [Adaptation comparison](adaptation-and-peft-comparison.md)
- [Compression comparison](compression-methods-comparison.md)
- [Runtime/deployment comparison](runtime-and-deployment-comparison.md)
- [Prioritized experiment roadmap](../11-research-roadmap/prioritized-experiment-roadmap.md)
- [Open risks and evidence gaps](../11-research-roadmap/open-risks-and-evidence-gaps.md)
- [Evidence standard](../00-governance/evidence-and-citation-standard.md)
- [Decision framework](../14-decision-framework/README.md)
