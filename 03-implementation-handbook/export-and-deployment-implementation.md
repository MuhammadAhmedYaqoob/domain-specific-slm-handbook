# Export and deployment implementation

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

## Concept, purpose, prerequisites, and outputs

Export translates one qualified source into runtime-specific artifacts; deployment makes one such artifact loadable, observable, replaceable, and reversible. Nominal precision does not make formats interchangeable. AWS GPU weights, GGUF blocks, Core ML packages, and vendor NPU binaries require separate compatibility and parity evidence.

**Prerequisites:** a qualified parent, pinned converter/runtime, target build tools, signing keys, release schema, and rollback artifact. **Inputs:** model files, tokenizer/template, target profile, evaluation report, and release metadata. **Outputs:** signed packages, cloud images or mobile builds, deployment manifests, health evidence, and rollback records.

### Worked example

Package `auto-mechanic-q4` version `1.0.0` for Android ARM64, verify every file digest, stage it in app-private storage, atomically activate it, run a local smoke prompt in airplane mode, and retain version `0.9.0` until post-activation health checks pass. Expected behavior is rejection of corrupt, incompatible, downgraded, or incompletely downloaded packages. Exact validation covers signature and path safety, clean load, tokenizer parity, Unicode streaming, cancellation, process death, storage pressure, physical-device memory and thermal behavior, and rollback.

## Contract and artifact policy

> Verification note: commands and listings require execution in the stated target environment before operational use.

Deployment does not use runtime document retrieval or a remote teacher. The model package contains weights,
tokenizer/template contract, manifest, signatures, notices, and model card; local inference remains functional in
airplane mode after installation.

Canonical platform detail:
[CH-20 Android](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-20-android-deployment.md),
[CH-21 iOS](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-21-ios-deployment.md),
[CH-22 other edge](../02-textbook/part-05-edge-ai-and-mobile-inference/ch-22-other-edge-devices.md),
[CH-35 AWS serving](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-35-aws-inference-serving.md), and
[CH-36 AWS-to-edge delivery](../02-textbook/part-09-aws-training-optimization-and-deployment/ch-36-aws-to-edge-model-delivery.md).

## Release package

```text
release/artifact-illustrative-001/0.1.0-illustrative/
├── model.<safetensors|gguf|pte|mlpackage|runtime-specific>
├── tokenizer files and chat_template.jinja when required
├── manifest.json
├── manifest.sig
├── SHA256SUMS
├── LICENSES/
├── model-card.md
└── evaluation-report.json
```

**Listing DEPLOY-L01 — extraction path `schemas/release.schema.json`.**

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "required": [
    "schema",
    "artifact_id",
    "version",
    "target",
    "files",
    "compatibility",
    "lineage",
    "evaluation",
    "rollback"
  ],
  "properties": {
    "schema": { "const": "offline-slm-release/v1" },
    "artifact_id": { "type": "string", "minLength": 1 },
    "version": { "type": "string", "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$" },
    "target": { "enum": ["aws-gpu", "aws-cpu", "android-arm64", "ios-arm64", "linux-arm64", "jetson"] },
    "files": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["path", "bytes", "sha256"],
        "properties": {
          "path": { "type": "string" },
          "bytes": { "type": "integer", "minimum": 1 },
          "sha256": { "type": "string", "pattern": "^[0-9a-f]{64}$" }
        }
      }
    },
    "compatibility": {
      "type": "object",
      "required": ["runtime", "runtime_revision", "context_tokens"],
      "properties": {
        "runtime": { "type": "string" },
        "runtime_revision": { "type": "string" },
        "context_tokens": { "type": "integer", "minimum": 1 },
        "minimum_os": { "type": "string" },
        "device_cohorts": { "type": "array", "items": { "type": "string" } }
      }
    },
    "lineage": {
      "type": "object",
      "required": ["source_artifact_sha256", "converter_revision"],
      "properties": {
        "source_artifact_sha256": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
        "converter_revision": { "type": "string" },
        "calibration_manifest_sha256": { "type": ["string", "null"] }
      }
    },
    "evaluation": {
      "type": "object",
      "required": ["report_sha256", "status"],
      "properties": {
        "report_sha256": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
        "status": { "const": "approved" }
      }
    },
    "rollback": {
      "type": "object",
      "required": ["minimum_allowed_version", "previous_artifact_id"],
      "properties": {
        "minimum_allowed_version": { "type": "string" },
        "previous_artifact_id": { "type": ["string", "null"] }
      }
    },
    "knowledge_cutoff": { "type": ["string", "null"] },
    "created_utc": { "type": "string", "format": "date-time" }
  }
}
```

Canonicalize manifest bytes before signing; the verifier must use the identical canonicalization. Sign the manifest, not
an ad hoc concatenation of model files. SHA-256 detects corruption; a signature authenticates the approved manifest.

**Listing DEPLOY-L02 — extraction path `src/slm_impl/package.py`.**

```python
from __future__ import annotations
import argparse, hashlib, json
from pathlib import Path

def sha256(path: Path) -> str:
    h=hashlib.sha256()
    with path.open("rb") as f:
        for block in iter(lambda:f.read(1<<20),b""): h.update(block)
    return h.hexdigest()

p=argparse.ArgumentParser()
p.add_argument("--root",type=Path,required=True)
p.add_argument("--template",type=Path,required=True)
p.add_argument("--out",type=Path,required=True)
a=p.parse_args()
manifest=json.loads(a.template.read_text("utf-8")); files=[]
for path in sorted(x for x in a.root.rglob("*") if x.is_file()
                   and x.name not in {"manifest.json","manifest.sig","SHA256SUMS"}):
    files.append({"path":path.relative_to(a.root).as_posix(),
                  "bytes":path.stat().st_size,"sha256":sha256(path)})
manifest["files"]=files
data=json.dumps(manifest,sort_keys=True,separators=(",",":")).encode()
a.out.write_bytes(data)
print(hashlib.sha256(data).hexdigest())
```

Use a managed signing key: AWS Key Management Service (KMS), platform release signing, or an offline hardware-backed
release key. Keep artifact signing separate from transport-layer TLS and app signing.

## Export compatibility gates

Before conversion:

- exact model architecture/config/tokenizer/template is supported;
- adapter is merged and parity-qualified where required;
- custom remote code is removed or explicitly reviewed;
- tied weights, special token IDs, EOS/stop rules, context/RoPE, and generation config are frozen;
- source artifact, converter/runtime commits, calibration, and target cohort are immutable.

After conversion:

1. clean-process load;
2. tokenizer golden strings and exact IDs;
3. fixed-prompt source/export logits where available;
4. greedy token/output parity with declared tolerance;
5. Unicode streaming, EOS/end-of-turn, cancellation, reset, and multiple conversations;
6. boundary context, overflow, out-of-memory, corruption, and insufficient-space behavior;
7. full domain/general/safety suite;
8. actual backend/operator placement and target benchmark;
9. package hash/signature, interrupted update, activation, and rollback.

Successful conversion is not evidence of GPU/NPU placement or acceptable performance.

## AWS serving

Start with one container and one replica. Select vLLM, SGLang, TensorRT-LLM, llama.cpp server, ONNX Runtime, or
Transformers only after exact architecture/precision support is verified. The generic service below prioritizes an
auditable interface over throughput.

**Listing DEPLOY-L03 — extraction path `deploy/aws/app.py`.**

```python
import os, time, uuid
from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel, Field
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

MODEL=os.environ["MODEL_PATH"]; EXPECTED_KEY=os.environ["SERVICE_API_KEY"]
tok=AutoTokenizer.from_pretrained(MODEL,local_files_only=True,trust_remote_code=False)
model=AutoModelForCausalLM.from_pretrained(
    MODEL,local_files_only=True,trust_remote_code=False,
    torch_dtype=torch.bfloat16,device_map="auto").eval()
app=FastAPI()

class Message(BaseModel):
    role: str
    content: str = Field(min_length=1,max_length=16000)
class Request(BaseModel):
    messages: list[Message] = Field(min_length=1,max_length=64)
    max_new_tokens: int = Field(ge=1,le=512)

@app.get("/health/ready")
def ready(): return {"ready":True,"artifact":os.environ["MODEL_SHA256"]}

@app.post("/v1/chat")
def chat(req: Request,authorization: str|None=Header(default=None)):
    if authorization != f"Bearer {EXPECTED_KEY}": raise HTTPException(401,"unauthorized")
    request_id=str(uuid.uuid4())
    messages=[x.model_dump() for x in req.messages]
    prompt=tok.apply_chat_template(messages,tokenize=False,add_generation_prompt=True)
    batch=tok(prompt,return_tensors="pt").to(model.device)
    if batch.input_ids.shape[1]+req.max_new_tokens>int(os.environ["CONTEXT_LIMIT"]):
        raise HTTPException(422,"context limit exceeded")
    started=time.perf_counter_ns()
    try:
        with torch.inference_mode():
            ids=model.generate(**batch,max_new_tokens=req.max_new_tokens,
                do_sample=False,pad_token_id=tok.eos_token_id)
        return {"id":request_id,
          "text":tok.decode(ids[0,batch.input_ids.shape[1]:],skip_special_tokens=True),
          "elapsed_ns":time.perf_counter_ns()-started,
          "artifact":os.environ["MODEL_SHA256"]}
    except torch.OutOfMemoryError:
        raise HTTPException(503,f"capacity unavailable; request_id={request_id}")
```

Do not use a static API key in production; place authentication/rate limiting before the service and use short-lived
secret injection. Do not log raw prompts/responses by default.

**Listing DEPLOY-L04 — extraction path `deploy/aws/Dockerfile`.**

```dockerfile
ARG BASE_IMAGE
FROM ${BASE_IMAGE}
ARG MODEL_SHA256
WORKDIR /app
COPY requirements/aws.lock /tmp/requirements.lock
RUN python -m pip install --no-cache-dir --require-hashes -r /tmp/requirements.lock \
 && python -m pip check
COPY deploy/aws/app.py /app/app.py
COPY release/aws-model /models/model
RUN chown -R 65532:65532 /app /models
USER 65532:65532
ENV MODEL_PATH=/models/model CONTEXT_LIMIT=2048 MODEL_SHA256=${MODEL_SHA256}
EXPOSE 8080
CMD ["uvicorn","app:app","--host","0.0.0.0","--port","8080","--workers","1"]
```

Production controls:

- private Elastic Container Registry image by digest and scanned software bill of materials;
- private subnets/security groups, TLS, authentication, request/token caps, and rate limits;
- least-privilege task/execution roles; no broad S3 or KMS permission;
- read-only root filesystem, non-root user, encrypted volumes/logs;
- readiness separate from liveness; graceful draining and bounded queue;
- metrics for queue, TTFT, decode, errors, OOM, GPU memory/utilization, and artifact identity;
- autoscaling on observed demand/SLO, not GPU utilization alone;
- blue/green or canary traffic and one-command rollback to prior image/model manifest;
- dated regional cost evidence including idle replicas, storage, network, load balancer/API, and observability.

**Listing DEPLOY-L05 — extraction path `deploy/aws/task-definition.json`.**

```json
{
  "family": "specialist-slm",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["EC2"],
  "cpu": "4096",
  "memory": "16384",
  "executionRoleArn": "arn:aws:iam::111122223333:role/slm-ecs-execution",
  "taskRoleArn": "arn:aws:iam::111122223333:role/slm-inference-task",
  "containerDefinitions": [
    {
      "name": "inference",
      "image": "111122223333.dkr.ecr.us-east-1.amazonaws.com/specialist-slm@sha256:4cadcc57ad21c2fa1c1f32655c4e2a96808875f1e7510032cb75a8077563b8d1",
      "essential": true,
      "readonlyRootFilesystem": true,
      "resourceRequirements": [{ "type": "GPU", "value": "1" }],
      "portMappings": [{ "containerPort": 8080, "protocol": "tcp" }],
      "secrets": [{ "name": "SERVICE_API_KEY", "valueFrom": "arn:aws:secretsmanager:us-east-1:111122223333:secret:slm/service-api-key" }],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "python -c \"import urllib.request; urllib.request.urlopen('http://127.0.0.1:8080/health/ready',timeout=2)\""
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 180
      }
    }
  ]
}
```

The account, role, secret, and image identities form a complete tutorial fixture;
they are not claimed to exist. Generate the same fields from the deployment
account and the pushed image digest. This assumes an ECS cluster with qualified
GPU EC2 capacity; Fargate GPU support is not assumed.

```bash
# Verification note: not executed here
docker pull python:3.11.11-slim
BASE_IMAGE="$(docker image inspect python:3.11.11-slim \
  --format '{{index .RepoDigests 0}}')"
MODEL_SHA256="$(sha256sum release/aws-model/model.safetensors | awk '{print $1}')"
docker build -f deploy/aws/Dockerfile \
  --build-arg BASE_IMAGE="$BASE_IMAGE" \
  --build-arg MODEL_SHA256="$MODEL_SHA256" \
  -t specialist-slm:08d7ac0eedd64c9d751783ed8d2178834c21b879 .
docker run --rm --gpus all -e SERVICE_API_KEY=test-only -p 8080:8080 specialist-slm:08d7ac0eedd64c9d751783ed8d2178834c21b879
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 111122223333.dkr.ecr.us-east-1.amazonaws.com
docker tag specialist-slm:08d7ac0eedd64c9d751783ed8d2178834c21b879 111122223333.dkr.ecr.us-east-1.amazonaws.com/specialist-slm:08d7ac0eedd64c9d751783ed8d2178834c21b879
docker push 111122223333.dkr.ecr.us-east-1.amazonaws.com/specialist-slm:08d7ac0eedd64c9d751783ed8d2178834c21b879
aws ecs register-task-definition --region us-east-1 \
  --cli-input-json file://deploy/aws/task-definition.json
aws ecs update-service --region us-east-1 --cluster specialist-slm-prod \
  --service specialist-slm-api --task-definition specialist-slm
aws ecs wait services-stable --region us-east-1 \
  --cluster specialist-slm-prod --services specialist-slm-api
```

Run authenticated smoke, full quality/safety, offered-load, failure, scale, canary, rollback, and billing reconciliation
before promotion.

## Android offline reference path

The portable baseline is `arm64-v8a` llama.cpp/GGUF through a narrow Java Native Interface (JNI). Vulkan/vendor NPU is a
separate cohort only after placement and parity evidence.

Extract the model produced by [compression implementation](compression-implementation.md) to app-private storage, not
the APK when its size/update policy makes bundling impractical.

```text
deploy/android/app/src/main/
├── cpp/{CMakeLists.txt,slm_jni.cpp}
├── java/com/example/slm/{NativeEngine.kt,ChatViewModel.kt}
└── assets/release-manifest.json
```

**Listing DEPLOY-L06 — extraction path
`deploy/android/app/src/main/java/com/example/slm/NativeEngine.kt`.**

```kotlin
package com.example.slm

class NativeEngine : AutoCloseable {
    private var handle: Long = 0
    external fun nativeLoad(modelPath: String, contextTokens: Int, threads: Int): Long
    external fun nativeGenerate(handle: Long, prompt: String, maxTokens: Int,
                                callback: (String) -> Unit): Int
    external fun nativeCancel(handle: Long)
    external fun nativeClose(handle: Long)

    fun load(path: String, context: Int, threads: Int) {
        check(handle == 0L)
        handle = nativeLoad(path, context, threads)
        check(handle != 0L) { "model load failed" }
    }
    fun generate(prompt: String, maxTokens: Int, emit: (String) -> Unit) {
        check(handle != 0L)
        check(nativeGenerate(handle, prompt, maxTokens, emit) == 0)
    }
    fun cancel() { if (handle != 0L) nativeCancel(handle) }
    override fun close() {
        if (handle != 0L) nativeClose(handle)
        handle = 0
    }
    companion object { init { System.loadLibrary("slm_jni") } }
}
```

The JNI/C++ implementation must pin one llama.cpp commit, own one model/context, apply the exact reviewed chat template,
stream valid Unicode fragments, serialize access, expose cancellation, clear state between conversations, map native
errors without secrets, and release all resources. Never pass unbounded prompt/output sizes.

Android packaging/update procedure:

1. verify signed manifest and every file hash before load;
2. check compatibility and available space for old + partial + staged + active;
3. download to app-private staging with HTTPS, authorization, range/resume, and size bounds;
4. `fsync`, verify, then atomically activate metadata;
5. retain prior approved package; keep the new version inactive until smoke and quality checks pass;
6. rollback after load/crash gate failure without lowering below revocation floor;
7. exclude model from backup and never log prompts/output;
8. test process death, background/foreground, cancellation, storage pressure, memory pressure, airplane mode, and
   thermal status.

Build only after pinning Android Gradle Plugin, Gradle, SDK, NDK, CMake, ABI, and native flags:

```powershell
# Verification note: not executed here
Set-Location deploy/android
.\gradlew.bat clean :app:assembleRelease :app:bundleRelease
.\gradlew.bat :app:dependencies > build-dependencies.txt
Get-FileHash -Algorithm SHA256 app\build\outputs\apk\release\*.apk
adb install -r app\build\outputs\apk\release\app-release.apk
adb shell am start -n com.example.slm/.MainActivity
adb shell dumpsys meminfo com.example.slm
```

Use a physical phone for memory, latency, energy, and thermal evidence.

## iOS offline reference paths

Two explicit paths:

1. portable llama.cpp/GGUF CPU/Metal control;
2. Apple-native Core ML, MLX Swift, or ExecuTorch export chosen only after exact model/operator/tokenizer support.

MLX is not an Apple Neural Engine (ANE) path. Core ML compute-unit eligibility does not prove ANE placement. Simulator
results are functional only.

**Listing DEPLOY-L07 — extraction path `deploy/ios/Sources/ModelStore.swift`.**

```swift
import CryptoKit
import Foundation

enum ModelStoreError: Error { case size, digest, incompatible }

struct ReleaseFile: Decodable { let path: String; let bytes: Int; let sha256: String }
struct ReleaseManifest: Decodable {
    let artifact_id: String
    let version: String
    let target: String
    let files: [ReleaseFile]
}

func sha256(_ url: URL) throws -> String {
    let handle = try FileHandle(forReadingFrom: url)
    var hasher = SHA256()
    while true {
        let data = try handle.read(upToCount: 1 << 20) ?? Data()
        if data.isEmpty { break }
        hasher.update(data: data)
    }
    return hasher.finalize().map { String(format: "%02x", $0) }.joined()
}

func verify(file: ReleaseFile, at root: URL) throws {
    let url = root.appendingPathComponent(file.path)
    let values = try url.resourceValues(forKeys: [.fileSizeKey])
    guard values.fileSize == file.bytes else { throw ModelStoreError.size }
    guard try sha256(url) == file.sha256 else { throw ModelStoreError.digest }
}
```

Add detached-signature verification using a pinned public key or platform-backed key set before trusting manifest
fields. Store models under Application Support with file protection, exclude them from backup, stage on the same volume,
and activate atomically. Keep the prior version and an anti-rollback/revocation policy.

SwiftUI integration must use an actor around the native engine, stream tokens on the main actor, cancel on user request,
avoid blocking the UI, handle memory warnings and serious thermal state, and close the model on lifecycle transitions
according to the measured policy.

```bash
# Verification note: not executed here — macOS/Xcode required
cd deploy/ios
xcodebuild -scheme SpecialistSLM -configuration Release \
  -destination 'platform=iOS,id=00008110-001234560123401E' build
xcodebuild -resolvePackageDependencies
shasum -a 256 Package.resolved
```

Run signed Release builds on every approved physical cohort. Capture Instruments placement, memory, signposts,
long-session thermal state, crash/memory termination, update/rollback, and airplane-mode behavior.

## Other edge targets

- **ARM64 Linux/Raspberry Pi:** pinned llama.cpp GGUF CPU baseline; record architecture flags, thread/affinity,
  cooling/governor, peak RSS, TTFT, sustained decode, and thermal sensors.
- **Jetson:** compare llama.cpp/CUDA or supported TensorRT-LLM path; record JetPack, CUDA, power mode, clocks, and
  `tegrastats`; do not transfer datacenter GPU results.
- **Laptop:** native CPU/GPU runtime, AC and battery as separate conditions.
- **WebGPU:** only for supported browser/model/runtime combinations; package and memory limits, shader compilation,
  browser version, origin security, and offline caching are first-class.
- **Microcontrollers:** ordinary generative SLMs are generally outside realistic memory/compute budgets; use smaller
  task-specific models or a more capable edge computer.

Example ARM build:

```bash
# Verification note: not executed here
git clone https://github.com/ggml-org/llama.cpp.git
git -C llama.cpp checkout 08d7ac0eedd64c9d751783ed8d2178834c21b879
cmake -S llama.cpp -B llama.cpp/build -DGGML_NATIVE=ON -DCMAKE_BUILD_TYPE=Release
cmake --build llama.cpp/build -j
llama.cpp/build/bin/llama-bench -m release/model.gguf -p 512 -n 128 -r 5
```

## Secure distribution and rollback

Cloud-to-edge flow:

```text
approved immutable package → private versioned object storage
→ signed manifest + authorized HTTPS distribution
→ bounded resumable staging → signature/hash/compatibility verification
→ atomic activation → post-activation health gate → active
→ retain previous approved package → rollback or revoke
```

Never overwrite an object key. Use object versioning, encryption at rest, separate signing/distribution roles, least
privilege, access logs without prompts, and lifecycle rules that preserve approved rollback artifacts. An edge app must
reject unknown key IDs, invalid signatures, hash/size mismatch, path traversal, duplicate paths, incompatible
runtime/OS/device, insufficient space, revoked versions, and downgrade below the minimum allowed version.

Bundle a model when offline-first installation and fixed app/model coupling outweigh application size and update cost.
Download after install when models are large, optional, domain-specific, or independently updated. Either approach must
support offline inference after installation.

## Release validation

An artifact is releasable only when:

- immutable source/converter/runtime/app identities and licenses pass;
- manifest canonicalization, signature, hashes, and safe paths pass;
- clean load, tokenizer/template, greedy parity, EOS/Unicode/reset pass;
- full domain, retention, uncertainty/refusal, and critical safety suites pass;
- target storage, update peak, post-load/peak memory, reliability, TTFT/decode, sustained thermal/energy gates pass;
- accelerator placement claim has trace evidence;
- corruption, interrupted download, insufficient space, process death, cancellation, offline, and rollback tests pass;
- AWS auth/rate/load/scale/observability/privacy/cost or mobile signing/storage/lifecycle tests pass;
- the previous release and rollback procedure are available and drilled.

If any target differs, release a separate artifact ID and evidence report. Do not weaken safety gates to make a
conversion pass.

## Completion record

The release schema/package builder, compatibility gates, AWS service/container/task/commands, Android and iOS
integration contracts, edge commands, secure distribution, offline operation, integrity, update, and rollback procedures
are authored. No artifact was converted, signed, deployed, built, installed, or measured.
