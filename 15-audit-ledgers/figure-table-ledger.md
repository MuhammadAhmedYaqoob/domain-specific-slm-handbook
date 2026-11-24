# Figure and table ledger

## What is checked and why

This ledger checks that every visual and comparison requested by `prompt.txt` has one stable ID, one canonical chapter,
a useful teaching purpose, and an honest evidence boundary. It also checks that diagrams parse and can be understood
without relying only on the image. This matters because duplicate IDs break references, decorative diagrams do not
teach data flow, and an empty result chart must not be mistaken for missing or fabricated evidence.

`ILLUSTRATIVE` means a conceptual teaching aid. `ESTIMATE` means values come from a formula and stated inputs.
`PLANNED EXPERIMENT` means axes and protocol exist but no measured points do. A measured figure requires raw data,
conditions, uncertainty, provenance, and review.

## Inventory result

- `TBL-01`–`TBL-25`: **25/25**, each with one canonical chapter anchor.
- `FIG-01`–`FIG-26`: **26/26**, each with one canonical chapter anchor.
- CH-47 `FIG-27`–`FIG-33`: **7/7** required production views.
- Mermaid: **111/111** blocks parsed with Mermaid CLI 11.9.0 in the recorded audit.
- Required figures include an evidence statement and text alternative.
- Required tables state whether cells are comparisons, estimates, or future measurements.

## Figure learning map

<!-- markdownlint-disable MD013 -->
| IDs | Chapter | What the learner should understand | Evidence status |
| --- | --- | --- | --- |
| FIG-01 | CH-28 | End-to-end specialist-model lifecycle and hand-offs | `ILLUSTRATIVE` |
| FIG-02–06 | CH-02, CH-05, CH-09 | Transformer/token flow, attention, training-stage differences, and low-rank adaptation | `ILLUSTRATIVE` |
| FIG-07–12 | CH-11–15 | Precision memory, quantization error, pruning, distillation, and compression order | `ESTIMATE` where calculated; otherwise `ILLUSTRATIVE` |
| FIG-13–15, FIG-24 | CH-32 | How to compare quality with parameters, latency, memory, and Pareto position | `PLANNED EXPERIMENT`; measured points not yet performed |
| FIG-16–19 | CH-17, CH-19, CH-20 | CPU/GPU/NPU roles, prefill/decode, KV growth, and mobile inference flow | `ILLUSTRATIVE`; KV growth is an `ESTIMATE` |
| FIG-20–23 | CH-24, CH-27, CH-31, CH-33 | AWS, multi-domain, dataset, and benchmark workflows | `ILLUSTRATIVE` |
| FIG-25–26 | CH-20, CH-36 | Offline installation/inference and model update lifecycle | `ILLUSTRATIVE` |
| FIG-27–33 | CH-47 | Training, AWS, Android, iOS, multi-domain, distribution, and approval architectures | `ILLUSTRATIVE` |
<!-- markdownlint-restore -->

## Table learning map

<!-- markdownlint-disable MD013 -->
| IDs | Teaching purpose | Current interpretation |
| --- | --- | --- |
| TBL-01–05 | Compare model scale, checkpoint type, training routes, and LoRA choices | Authored comparisons; source review still applies |
| TBL-06–10 | Compare quantization, pruning, distillation, and combined compression | Authored comparisons; not project benchmark results |
| TBL-11–12 | Compare model families and reason about memory | Candidate facts plus calculated planning values |
| TBL-13 | Teach training-cost calculation | Rates not yet captured; learners must insert dated values |
| TBL-14–20 | Compare runtimes, platforms, hardware, cloud/edge, domain strategy, data, and evaluation | Decision support based on stated evidence boundaries |
| TBL-21–23 | Record inference, energy, and quality-retention results | Intentionally empty because project runs are not yet performed |
| TBL-24–25 | Compare AWS services and production architecture choices | Authored decision guidance, not deployed-system proof |
<!-- markdownlint-restore -->

## Passing and failing evidence

**Pass:** a measured latency plot links a hashed result table and raw timing records, names model/runtime/device,
defines warm-up and prompt/output lengths, shows repeated trials and uncertainty, and labels failures.

**Fail:** points copied from unrelated hardware are plotted together as if conditions were comparable, or planned axes
are filled with invented values. Leaving TBL-21–23 and FIG-13–15/24 without results is currently the truthful choice.

## How to close the gap

Content and ID checks pass. Measurement and publication review are not yet performed. Run the approved benchmark
protocols, retain raw data and failed runs, generate figures from those records, and have a reviewer check calculations
and conclusions. Then test rendered output for contrast, grayscale and print, screen-reader order, text alternatives,
narrow layouts, and minimum font size. Responsibility remains unassigned until named reviewers accept those tasks.
