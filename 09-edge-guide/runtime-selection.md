# Runtime, format, tokenizer, and KV-cache selection

## 1. Separate the layers

GGUF is a model container; GGML supplies tensor primitives; llama.cpp is a generation runtime. ONNX and Core ML
packages describe graphs but do not automatically provide chat templates, sampling, token streaming, or session
policy. A runtime is suitable only when it supports the exact architecture, operators, tokenizer, cache layout,
precision, platform, and lifecycle requirements.

| Runtime | Typical artifact | Strong first use | Important constraint |
| --- | --- | --- | --- |
| llama.cpp | GGUF | portable CPU, Metal/Vulkan experiments | support changes by architecture and commit |
| ExecuTorch | backend-specialized `.pte` | PyTorch-owned Android/iOS export | inspect partition fallback |
| LiteRT-LM | `.litertlm` | supported Google edge generation path | exact model/backend support matters |
| ONNX Runtime GenAI | ONNX plus generation config | an existing ONNX estate | core ORT alone is not a chat engine |
| Core ML | `.mlpackage` | Apple packaging and eligible ANE execution | app may own generation loop |
| MLX Swift LM | MLX weights | Apple CPU/GPU integration | not an ANE path |
| MLC/WebLLM | compiled library and converted weights | mobile GPU and WebGPU | compiler/runtime artifacts must match |
| Qualcomm Genie | target context binaries | controlled Snapdragon cohort | tightly coupled to SDK/SoC/firmware |

## 2. Tokenizer and chat semantics

The tokenizer converts bytes to token IDs. Normalization, byte fallback, added tokens, and special-token IDs must travel
with the model. The chat template renders roles and separators exactly as used during training. A syntactically valid
but different template changes the model's input distribution and stopping behavior.

Generation has two stages. **Prefill** processes all prompt tokens in parallel and creates KV entries. **Decode**
generates one token at a time while reading prior KV. Context growth therefore increases cache memory and attention
work. GQA and MQA reduce \(H_{kv}\); lower cache precision reduces bytes per element but requires compatible kernels
and quality evaluation.

## 3. Worked conversion example

Assume an immutable local Hugging Face model at `C:\models\mechanic-slm`, a pinned llama.cpp checkout, and an output
directory `C:\artifacts\mechanic-v1`. Run the quantizer with a format printed by that build; `Q4_K_M` below is an
illustrative commonly available choice and must be confirmed with `llama-quantize --help`.

```powershell
$Source = 'C:\models\mechanic-slm'
$Out = 'C:\artifacts\mechanic-v1'
New-Item -ItemType Directory -Force $Out | Out-Null
python .\convert_hf_to_gguf.py $Source --outfile "$Out\model-f16.gguf" --outtype f16
.\build\bin\Release\llama-quantize.exe `
  "$Out\model-f16.gguf" "$Out\model-q4_k_m.gguf" Q4_K_M
Get-FileHash "$Out\model-q4_k_m.gguf" -Algorithm SHA256
```

The first command maps supported model tensors and tokenizer metadata into GGUF. The second creates a quantized artifact.
The hash gives immutable byte identity; it does not prove semantic parity. Expected output is an FP16 intermediate, a
smaller quantized file, and a SHA-256 digest.

## 4. Step-by-step selection and integration

1. Write hard gates: model architecture, context, OS, memory, latency, quality, offline behavior, and licenses.
2. Check the pinned runtime's model and operator support, not a generic marketing matrix.
3. Record weight, activation, accumulator, and KV precision and quantization grouping.
4. Convert one backend artifact at a time with pinned source and tool revisions.
5. Compare source and target token IDs for spaces, Unicode, emoji, code, units, and special tokens.
6. Compare rendered prompt bytes and BOS/EOS/EOT behavior.
7. Compare logits within declared tolerance or greedy token sequences at short and long prompts.
8. Test cache append, reset, context boundary, cancellation, and two-session isolation.
9. Run paired domain, general-retention, refusal, and safety evaluation.
10. Measure CPU; add an accelerator only when a partition trace and benchmark show a product benefit.

## 5. Accelerator proof

Selecting Metal, Vulkan, Core ML, or QNN is not proof of execution. Capture operator partitions, fallback boundaries,
compiled-program creation, transfer time, and utilization. Compare identical prompts and output limits against the CPU
artifact. Reject an accelerator that improves burst throughput but violates memory, quality, energy, or sustained gates.

## 6. Common failures

| Symptom | Likely cause | Validation |
| --- | --- | --- |
| Converter rejects model | unsupported architecture or tensor | inspect pinned source support |
| Fluent nonsense | tokenizer, RoPE, template, tensor, or cache mismatch | token/template/logit goldens |
| Wrong stopping | EOS/EOT or template mismatch | exact special-token tests |
| OOM near context limit | KV, static shapes, or prefill workspace | phase allocation and context sweep |
| Accelerator is slower | fallback, copies, compilation, decode shape | partition and cold/warm trace |
| Broken Unicode | decoding each token as a complete string | incremental byte-buffer tests |

## 7. Validation gate

A candidate passes only when source-target semantics, quality, maximum-context memory, cancellation/reset, cold and warm
latency, placement, sustained behavior, license, and update compatibility pass together. Preserve commands, dependency
locks, artifact hashes, and raw reports.

## Sources

- [GGUF specification](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md)
- [ExecuTorch backends](https://docs.pytorch.org/executorch/stable/backends-overview.html)
- [LiteRT-LM](https://developers.google.com/edge/litert-lm/overview)
- [ONNX Runtime GenAI](https://onnxruntime.ai/docs/genai/)
- [MLC model compilation](https://llm.mlc.ai/docs/compilation/compile_models.html)
