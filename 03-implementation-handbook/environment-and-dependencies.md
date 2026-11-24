# Environment and dependencies

## Concept, purpose, and outcome

An environment is part of an experiment, not incidental setup. Dependency versions, operating system, accelerator driver, compiler, and runtime decide whether code imports, numerical kernels execute, and artifacts remain reproducible. This procedure creates isolated data, training, evaluation, conversion, and mobile lanes instead of allowing incompatible toolchains to contaminate one another.

**Prerequisites:** Python 3.11, Git, sufficient local storage, and the platform tools for the chosen lane. Docker, CUDA, Android Studio, Xcode, and AWS credentials are required only for their respective procedures. **Inputs:** reviewed requirement files, platform identity, and immutable source revisions. **Outputs:** hash-locked dependency files, environment probes, container identity, and smoke-test evidence.

### Worked example

For the `train` lane, create `.venv-train`, resolve `requirements/train.in`, install `requirements/train.lock` with hashes, run `pip check`, and save the package freeze. Expected behavior is a repeatable clean install whose probe records Python, PyTorch, CUDA availability, package versions, Git commit, and GPU identity. A resolver conflict is a failed environment—not permission to remove pins. Validate by rebuilding in a fresh directory, comparing lock hashes, importing every direct dependency, running one forward/backward step, and saving/reloading a tiny checkpoint.

## Status and strategy

> Verification note: commands and listings require execution in the stated target environment before operational use.

Use five isolated lanes:

<!-- markdownlint-disable MD013 -->
| Lane        | Purpose                                        | Host                                                      |
| ----------- | ---------------------------------------------- | --------------------------------------------------------- |
| `data`      | acquisition, parsing, validation, snapshots    | Windows/Linux CPU, 16+ GiB RAM; OCR may need more         |
| `train`     | PyTorch, Transformers, PEFT, TRL, bitsandbytes | Linux x86-64; NVIDIA GPU for common QLoRA path            |
| `eval`      | deterministic scoring/statistics/reports       | CPU; inference hardware is artifact-specific              |
| `llama-cpp` | GGUF conversion and CPU/Metal/Vulkan builds    | Linux/macOS/Windows; target cross-build separately        |
| `mobile`    | Android/iOS app and native runtime builds      | Android Studio on supported desktop; Xcode requires macOS |
<!-- markdownlint-restore -->

Named pruning, quantization, and distillation repositories get a container or virtual environment pinned to an immutable
commit. Do not force SparseGPT, Wanda, AWQ, GPTQ, Core ML, and ExecuTorch into the training lock.

## Hardware prerequisites

- **Data/evaluation CPU:** 8 logical cores, 16 GiB RAM, and enough storage for raw bytes, derived snapshots, and
  duplicate temporary artifacts. This is a planning floor, not a measured requirement.
- **Training:** CUDA-capable NVIDIA GPU for the candidate bitsandbytes QLoRA path. VRAM is determined by model, sequence
  length, microbatch, activations, optimizer, and precision; perform a one-batch pilot. Full fine-tuning can require
  sharding or multiple accelerators.
- **GGUF conversion:** host RAM and temporary disk can exceed final artifact size because source and converted tensors
  coexist.
- **Android:** 64-bit ARM physical device, Android Studio, SDK/NDK/CMake, USB debugging, and enough private storage for
  old + staged + new + rollback artifacts.
- **iOS:** Apple Developer toolchain, physical arm64 iPhone/iPad, signing identity, Xcode on macOS, and storage for
  packaged artifacts. Windows cannot build/sign an iOS application.
- **AWS:** an account, explicit region, least-privilege Identity and Access Management (IAM), quota for the selected
  instance, encrypted artifact storage, and budget alarms. No instance or price is assumed here.

## Candidate Python inputs

**Listing ENV-L01 — extraction path `requirements/train.in`.**

```text
accelerate==1.4.0
bitsandbytes==0.45.3
datasets==3.3.2
evaluate==0.4.3
huggingface-hub==0.29.2
jsonschema==4.23.0
numpy==2.2.3
peft==0.14.0
pydantic==2.10.6
PyYAML==6.0.2
safetensors==0.5.3
sentencepiece==0.2.0
torch==2.6.0
transformers==4.49.0
trl==0.15.2
```

**Listing ENV-L02 — extraction path `requirements/data.in`.**

```text
beautifulsoup4==4.13.3
jsonschema==4.23.0
lxml==5.3.1
numpy==2.2.3
pydantic==2.10.6
pypdf==5.3.0
PyYAML==6.0.2
pyarrow==19.0.1
```

**Listing ENV-L03 — extraction path `requirements/eval.in`.**

```text
jsonschema==4.23.0
numpy==2.2.3
pandas==2.2.3
psutil==7.0.0
pyarrow==19.0.1
scipy==1.15.2
```

These candidate pins have not been resolved together here. A security or compatibility fix must produce a new reviewed
lock and environment identifier; do not silently float one package.

## Resolve reproducibly

**Listing ENV-L04 — extraction path `scripts/resolve-environment.ps1`.**

```powershell
param(
  [ValidateSet("data","train","eval")] [string]$Lane = "train"
)
$ErrorActionPreference = "Stop"
py -3.11 -m venv ".venv-$Lane"
& ".\.venv-$Lane\Scripts\python.exe" -m pip install `
  "pip==25.0.1" "pip-tools==7.4.1"
& ".\.venv-$Lane\Scripts\pip-compile.exe" --resolver=backtracking `
  --generate-hashes --output-file "requirements/$Lane.lock" `
  "requirements/$Lane.in"
& ".\.venv-$Lane\Scripts\python.exe" -m pip install `
  --require-hashes -r "requirements/$Lane.lock"
& ".\.venv-$Lane\Scripts\python.exe" -m pip check
& ".\.venv-$Lane\Scripts\python.exe" -m pip freeze --all |
  Set-Content "results/validation/$Lane.freeze.txt"
Get-FileHash -Algorithm SHA256 `
  "requirements/$Lane.in","requirements/$Lane.lock",
  "results/validation/$Lane.freeze.txt" |
  ConvertTo-Json | Set-Content "results/validation/$Lane.hashes.json"
```

Linux uses the same procedure with `python3.11 -m venv`, `. .venv-$LANE/bin/activate`, and `sha256sum`. The resulting
lock is platform-specific when wheels differ; record operating system and architecture in its manifest.

## Training image

**Listing ENV-L05 — extraction path `containers/train/Dockerfile`.**

```dockerfile
ARG BASE_IMAGE
FROM ${BASE_IMAGE}
ARG LOCK_SHA256
WORKDIR /workspace
COPY requirements/train.lock /tmp/train.lock
RUN echo "${LOCK_SHA256}  /tmp/train.lock" | sha256sum -c - \
 && python -m pip install --no-cache-dir --require-hashes -r /tmp/train.lock \
 && python -m pip check
RUN useradd --create-home --uid 10001 runner
USER 10001
ENV PYTHONHASHSEED=0 TOKENIZERS_PARALLELISM=false
CMD ["python","-c","import torch,transformers,peft,trl; print(torch.__version__)"]
```

Resolve the tag to the digest that the registry actually returned, then pass that
digest to the build. This avoids publishing a fabricated or stale digest in the
handbook:

```bash
# Verification note: not executed here
docker pull python:3.11.11-slim
BASE_IMAGE="$(docker image inspect python:3.11.11-slim \
  --format '{{index .RepoDigests 0}}')"
LOCK_SHA256="$(sha256sum requirements/train.lock | awk '{print $1}')"
docker build -f containers/train/Dockerfile \
  --build-arg BASE_IMAGE="$BASE_IMAGE" \
  --build-arg LOCK_SHA256="$LOCK_SHA256" \
  -t slm-train:08d7ac0eedd64c9d751783ed8d2178834c21b879 .
docker image inspect slm-train:08d7ac0eedd64c9d751783ed8d2178834c21b879 \
  --format '{{json .RepoDigests}}'
```

Do not place cloud credentials or model tokens in an image layer. Mount short-lived credentials at runtime using the
platform's secret mechanism.

## Environment probe

**Listing ENV-L06 — extraction path `scripts/probe_environment.py`.**

```python
from __future__ import annotations
import importlib.metadata as md
import json, os, platform, subprocess, sys
from pathlib import Path

PACKAGES = ["torch","transformers","datasets","accelerate","peft","trl",
            "bitsandbytes","safetensors","numpy"]

def command(argv: list[str]) -> str | None:
    try:
        return subprocess.run(argv, check=True, capture_output=True, text=True,
                              timeout=20).stdout.strip()
    except (OSError, subprocess.SubprocessError):
        return None

record = {
    "status": "execution_requires_review",
    "python": sys.version,
    "platform": platform.platform(),
    "machine": platform.machine(),
    "packages": {},
    "git_commit": command(["git","rev-parse","HEAD"]),
    "nvidia_smi": command(["nvidia-smi","--query-gpu=name,memory.total,driver_version",
                           "--format=csv,noheader"]),
    "environment": {k: os.environ.get(k) for k in
                    ["CUDA_VISIBLE_DEVICES","PYTHONHASHSEED"]},
}
for name in PACKAGES:
    try: record["packages"][name] = md.version(name)
    except md.PackageNotFoundError: record["packages"][name] = None
try:
    import torch
    record["torch"] = {
        "cuda_available": torch.cuda.is_available(),
        "cuda_runtime": torch.version.cuda,
        "bf16_supported": bool(torch.cuda.is_available()
                              and torch.cuda.is_bf16_supported()),
        "device_count": torch.cuda.device_count(),
    }
except Exception as exc:
    record["torch_import_error"] = repr(exc)
Path("results/validation").mkdir(parents=True, exist_ok=True)
Path("results/validation/environment.json").write_text(
    json.dumps(record, indent=2, sort_keys=True), encoding="utf-8")
print(json.dumps(record, indent=2))
```

## Toolchain pins

Record these independently:

The following is a fully populated **illustrative inventory**, not a claim about
the reader's workstation. Replace it by running the probe and the platform
version commands; retain this shape so reviews can compare inventories.

```yaml
# configs/toolchains.yaml
python: "3.11.11"
cuda_driver: "550.54.15"
cuda_runtime: "12.4"
container_image: "python:3.11.11-slim"
llama_cpp_commit: "08d7ac0eedd64c9d751783ed8d2178834c21b879"
android:
  studio: "2024.2.1"
  gradle: "8.10.2"
  agp: "8.8.0"
  compile_sdk: "35"
  min_sdk: "28"
  ndk: "27.2.12479018"
  cmake: "3.31.1"
ios:
  macos: "15.2"
  xcode: "16.2"
  swift: "6.0.3"
  deployment_target: "17.0"
aws:
  cli: "2.22.35"
  region: "us-east-1"
  account_alias_not_id: "slm-lab"
```

Android and iOS runtime dependencies belong in Gradle/Swift Package Manager locks, not Python. Save
`gradlew dependencies`, `Package.resolved`, compiler versions, native build flags, and produced binary hashes.

## Smoke-test gate

An environment is not qualified until all applicable checks succeed:

1. clean install from lock with network disabled after wheel/cache preparation;
2. import and `pip check`;
3. trusted `safetensors` model load with `trust_remote_code=False`;
4. tokenizer/chat-template golden test;
5. one forward pass and one backward/optimizer step for each training mode;
6. checkpoint save in a temporary directory and reload in a clean process;
7. LoRA attach, trainable-parameter audit, merge, and logit comparison;
8. QLoRA kernel and compute-dtype check on the exact GPU;
9. CUDA out-of-memory pilot at intended sequence length;
10. software bill of materials and vulnerability/license review.

Expected failures include missing platform wheels, CUDA driver/runtime mismatch, unsupported BF16, bitsandbytes backend
failure, mutable model files, renamed TRL configuration fields, absent assistant masks, temporary disk exhaustion, and
hash mismatch. Resolve failures by changing one declared component and repeating the entire gate.

## Canonical references

- [CH-06 SFT environment and template requirements](../02-textbook/part-02-training-and-domain-adaptation/ch-06-supervised-fine-tuning.md)
- [CH-09 LoRA/QLoRA compatibility](../02-textbook/part-03-parameter-efficient-fine-tuning/ch-09-lora-and-its-evolution.md)
- [CH-34 AWS training](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-34-training-slms-on-aws.md)
- [Troubleshooting and validation](troubleshooting-and-validation.md)

## Verification note

The documentation and embedded Python syntax were checked in this authoring
environment. Dependency resolution, CUDA kernels, container digests, Android
and iOS toolchains, and AWS resources remain external validations because they
depend on the selected host, account, and target hardware.
