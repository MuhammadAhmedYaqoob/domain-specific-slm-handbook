# CH-35: AWS Inference Serving

> **Illustrative values.** Concrete identifiers, dates, thresholds, hashes, account numbers, and endpoints in this file are teaching examples, not observed project results or deployable credentials. Before use, replace them with reviewed values from your own registry, preserve the required format, and rerun every stated validation.

**Research cut-off:** 2026-11-23  
**Chapter evidence status:** runtime and AWS capability claims are `DOC-VERIFIED` against the official sources in the
source register. The deployment listings are `UNVERIFIED IMPLEMENTATION`: they were authored for working intent and
statically reasoned about, but were not built, deployed, or benchmarked in an AWS account. No deployment success,
performance result, quota, regional availability, or dollar cost is claimed.

## 35.1 Why inference serving is a systems problem

Training produces weights; it does not produce a reliable service. A production Small Language Model (SLM) endpoint must
turn a model artifact, tokenizer, chat template, runtime, accelerator, network boundary, and release policy into one
coherent system. It must also handle two very different workloads:

- **Prefill** processes all input tokens, is highly parallel, and often stresses compute throughput.
- **Decode** emits tokens sequentially, repeatedly reads the Key-Value (KV) cache, and is commonly constrained by memory
  bandwidth and per-token scheduling.

The serving runtime therefore does more than call `model.generate()`. It schedules requests, manages KV-cache pages,
batches work, streams output, enforces context limits, and coordinates multiple accelerators. The surrounding platform
authenticates callers, validates requests, limits abuse, records bounded telemetry, scales capacity, and releases or
rolls back immutable model versions.

Three distinctions prevent costly design errors:

1. **A model family is not an exact compatibility result.** Runtime support for “Llama,” “Qwen,” or another family does
   not prove that a particular checkpoint, tokenizer revision, adapter, quantization, attention variant, or chat
   template works.
2. **A weight file fitting in accelerator memory is not a serving-capacity result.** KV cache, activations, temporary
   workspaces, CUDA graphs, runtime processes, and concurrency also consume memory.
3. **An OpenAI-compatible route is not an interchangeable service.** Implementations differ in accepted fields,
   streaming details, error schemas, token accounting, structured-output support, and model-specific extensions.

The primary architecture in this book does not use runtime Retrieval-Augmented Generation (RAG). Domain capability is
carried by approved weights or adapters. This simplifies the online data path, but it does not eliminate stale
knowledge, imperfect recall, hallucination, or the need for expert escalation in pharmacy and safety-critical repair.

## 35.2 Inference mechanics and capacity

### Continuous batching

Static batching waits for a complete batch, runs it, and returns all results. Autoregressive requests rarely finish
together. **Continuous batching** admits new requests between decode iterations and removes completed requests,
improving accelerator utilization under concurrency. Paged KV-cache management reduces external fragmentation by
allocating cache in blocks rather than requiring one contiguous region per sequence.

Batching creates a latency-throughput trade-off. A scheduler that waits to form larger prefill batches can increase
requests per second while making an individual request wait longer. Long prompts can also delay short prompts unless the
runtime supports chunked prefill or an appropriate scheduling policy. Benchmark the policy against the real
prompt-length distribution; do not optimize only a synthetic steady-state token rate.

### Memory planning

A first-order planning model is:

\[ M_{\text{required}} \approx M_{\text{weights}} + M_{\text{KV}} + M_{\text{workspace}} + M_{\text{runtime}} +
M_{\text{margin}} \]

where all terms use the same unit. Approximate dense weight storage is:

\[ M_{\text{weights}} \approx P \times b / 8 \]

where \(P\) is parameter count and \(b\) is stored bits per parameter. This excludes quantization metadata and runtime
copies. A simplified decoder-only KV-cache estimate is:

\[ M_{\text{KV}} \approx B \times L \times T \times 2 \times H_{\text{kv}} \times D \times s \]

where \(B\) is concurrent sequences, \(L\) layers, \(T\) cached tokens per sequence, the factor 2 represents keys and
values, \(H_{\text{kv}}\) is the number of KV heads, \(D\) is head dimension, and \(s\) is bytes per cache element.
Architectures and runtimes can change this layout, so use the runtime's measured allocation as the release value.

**Illustrative calculation (`ESTIMATE`, not a fit claim).** For \(B=8\), \(L=24\), \(T=4096\), \(H_{\text{kv}}=8\),
\(D=128\), and a 2-byte cache:

\[ 8 \times 24 \times 4096 \times 2 \times 8 \times 128 \times 2 = 3{,}221{,}225{,}472\text{ bytes} \approx 3.0\text{
GiB} \]

Doubling context or concurrency approximately doubles this term. This is why an apparently small model can still exhaust
a GPU.

### Cost model

For a continuously provisioned host:

\[ C_{\text{request}} = \frac{C_{\text{compute}} + C_{\text{storage}} + C_{\text{network}} + C_{\text{control}} +
C_{\text{observability}}}{N_{\text{successful requests}}} \]

For token-normalized reporting, divide by successful output tokens or total processed tokens and state which denominator
is used. Idle GPU time, failed requests, blue-green overlap, model-download traffic, public IPv4, load balancers,
Network Address Translation (NAT), logs, and retained artifacts are not free. All prices in this chapter remain
unresolved until captured by Region, date, service, purchase option, and billing unit from an official AWS source.

## 35.3 Evidence-gated runtime comparison

<a id="tbl-ch35-21"></a> **TBL-CH35-21 — Inference runtime comparison.** Evidence: `DOC-VERIFIED` capabilities and
engineering interpretation as of the research cut-off; no row is a measured performance ranking.

<!-- markdownlint-disable MD013 -->
| Runtime                      | Compatibility and loading                                                                                                                                                                                             | Hardware                                                                                                                                                                           | Batching and cache                                                                                                                              | Precision/quantization                                                                                                                                          | Cost tendency                                                                                                           | Operational complexity and decision                                                                                                                                                                                                                                                                           |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **vLLM**                     | Broad documented Hugging Face model interface; loads supported HF-format checkpoints and exposes an OpenAI-compatible server. Exact architecture, tokenizer, adapter, and quantization require qualification.         | Lowest-friction AWS path is NVIDIA CUDA on G/P families. AMD, Intel, CPU, and other paths exist but are not feature-equivalent. AWS Neuron uses a separate beta plugin and matrix. | Continuous batching, paged KV-cache management, distributed serving, prefix caching and runtime-specific scheduling options.                    | Supports multiple documented quantization paths; kernel and compute-capability matrices differ. BF16, FP16, FP8, INT8/INT4 labels are not universally portable. | Strong utilization can lower unit cost under concurrency; a persistent GPU still incurs idle cost.                      | **Core practical default** for a mainstream supported checkpoint on NVIDIA. Moderate complexity; pin the image and test every feature used.                                                                                                                                                                   |
| **SGLang**                   | OpenAI-compatible server with broad model coverage and fast-moving backend support. Tokenizer chat templates can be loaded or overridden.                                                                             | Documents NVIDIA, AMD, CPU, Intel XPU, TPU and additional paths; parity is not implied.                                                                                            | Continuous batching and prefix-aware RadixAttention are central strengths; distributed serving is documented.                                   | Multiple quantization backends with model/hardware-specific support.                                                                                            | Particularly worth measuring when prefix reuse or structured generation matches the workload.                           | **Core benchmark beside vLLM.** Pin a tested release because model/backend compatibility changes quickly.                                                                                                                                                                                                     |
| **Hugging Face TGI**         | Mature container and OpenAI-compatible routes; existing deployments retain compatibility value.                                                                                                                       | NVIDIA path is established; other documented paths must be checked per release.                                                                                                    | Continuous batching, streaming, Paged/Flash Attention and KV caching are documented.                                                            | Documents bitsandbytes, GPTQ and other runtime-dependent choices.                                                                                               | Migration cost may make retention rational for an existing service.                                                     | `DOC-VERIFIED`: Hugging Face has marked TGI **maintenance mode since 2025-11-12**, accepting minor fixes, documentation, and lightweight maintenance, and recommends vLLM or SGLang for new Inference Endpoints. Do not call it abandoned; do not choose it for a new platform without a demonstrated reason. |
| **llama.cpp server**         | Loads GGUF artifacts and provides OpenAI-compatible HTTP routes. Strong cloud-edge artifact parity when GGUF is the approved format. Conversion, tokenizer, template, RoPE and quantization parity remain test gates. | CPU-first with CUDA, Metal, Vulkan and other backends. On AWS it is relevant for CPU/GGUF or lightweight GPU experiments.                                                          | Continuous batching and slot/KV controls are documented; scheduling differs from GPU-first engines.                                             | Extensive GGUF low-bit formats plus floating-point choices; quality and kernel support depend on quantizer/backend.                                             | CPU instances may reduce accelerator cost for low traffic but can lose latency or throughput. Only measurement decides. | **Required GGUF/CPU comparison.** Lower platform burden, but artifact conversion and backend tuning are additional work.                                                                                                                                                                                      |
| **ONNX Runtime (ORT) GenAI** | ONNX graph plus `genai_config.json`, tokenizer, and provider assets. Generic ONNX export is not proof of efficient autoregressive support.                                                                            | CPU, CUDA and other Execution Providers (EPs); EP operator coverage and graph partitioning are checkpoint-specific.                                                                | GenAI layer supplies tokenization, generation loop, sampling and KV-cache management. Server orchestration must be built or adopted separately. | Quantization depends on graph, exporter and EP.                                                                                                                 | Attractive where ONNX is already a release target; conversion and custom serving work can dominate.                     | **Optional portability experiment.** `DOC-VERIFIED`: the Generate API is **preview** and subject to change.                                                                                                                                                                                                   |
| **TensorRT-LLM**             | `trtllm-serve` can load documented HF or optimized checkpoints and exposes OpenAI-compatible routes. Engine conversion/build and version coupling vary by path.                                                       | Linux and supported NVIDIA GPU architectures. Precision features are tied to the support matrix; it is not portable to CPU, AMD, or Neuron.                                        | Paged attention and in-flight batching; multi-GPU and distributed paths are documented.                                                         | Strong NVIDIA-specific FP8/FP4/INT8/INT4 options where the exact GPU and model support them.                                                                    | Can be economical at high sustained load if a reproduced gain exceeds engineering and build costs.                      | **Advanced practical.** Adopt only after a controlled baseline proves a bottleneck and the optimized path wins total cost or SLOs.                                                                                                                                                                            |
| **AWS Neuron/vLLM plugin**   | Separate model recipes and feature matrix from upstream vLLM.                                                                                                                                                         | Trainium/Inferentia; requires Neuron compilation/runtime rather than CUDA.                                                                                                         | Supported behavior is plugin-specific.                                                                                                          | Precision and quantization are architecture/toolchain-specific.                                                                                                 | Potential AWS-silicon option, not an assumed saving.                                                                    | **Targeted experiment.** Current official documentation labels the vLLM Neuron plugin beta.                                                                                                                                                                                                                   |
| **Managed service paths**    | SageMaker real-time endpoints can host bring-your-own containers; Bedrock Custom Model Import supports only documented architectures/configurations and Regions.                                                      | Service-specific supported instance and accelerator lists.                                                                                                                         | Platform autoscaling and deployment features do not replace runtime scheduling.                                                                 | Exact container/model path controls available formats.                                                                                                          | Operations can be reduced, but managed rates and constraints must be compared with EC2 total cost.                      | **Conditional alternative.** Bedrock is not a runtime-comparison platform; SageMaker is relevant when endpoint lifecycle management is worth the service contract.                                                                                                                                            |
<!-- markdownlint-restore -->

### Compatibility gates common to every runtime

A candidate passes only when all of the following are recorded and tested:

1. immutable model and tokenizer revisions, license, architecture, adapter state, and chat template;
2. source and target formats, conversion and quantization commands, calibration data, and artifact hashes;
3. runtime and container digest, driver, CUDA/Neuron/ROCm version, instance type, Region and Availability Zone;
4. accepted and tested context, batch, concurrency, stop-token, streaming, structured-output, and cancellation behavior;
5. source-versus-served tokenizer parity, logit/generation sanity checks, and domain-quality regression;
6. cold load, warm load, host/GPU peak memory, queue time, time to first token (TTFT), prefill and decode rates, errors,
   and restart behavior;
7. readiness, graceful drain, backpressure, timeout, metric, log, authentication, and rollback behavior.

## 35.4 Runtime selection by workload

Use the following sequence rather than choosing from vendor benchmark charts:

1. **Freeze the artifact and workload.** Select exact checkpoint, quantization, maximum context, prompt/output
   distributions, concurrency, and quality gates.
2. **Start with vLLM and SGLang on one right-sized NVIDIA instance.** They are the current high-priority GPU baselines
   for supported models.
3. **Add llama.cpp if GGUF, CPU serving, or cloud-edge artifact parity is an actual requirement.**
4. **Add TensorRT-LLM only when the baseline reveals a measurable NVIDIA optimization opportunity.**
5. **Add ORT GenAI only when ONNX portability or an Execution Provider is part of the product plan.**
6. **Keep TGI only for an existing dependency or a specific verified feature not met elsewhere.**
7. **Test Neuron, Bedrock import, or another managed path as separate compatibility and total-cost experiments.**

Do not compare published throughput from different models, context lengths, output lengths, precisions, accelerators,
software versions, or latency constraints.

## 35.5 Reference production architecture

<a id="fig-ch35-20"></a> **FIG-CH35-20 — AWS model deployment architecture (`ILLUSTRATIVE`).** Alt text: a separately
approved release path deploys a model into an authenticated online request path with observability and rollback. Arrows
show the online request path and the separate release path. No measured data is represented.

```text
                                   RELEASE PLANE
  Approved model + tokenizer + manifest + hashes
                      |
                      v
        S3 versioned artifacts ----> ECR image by digest
                      |                    |
                      +------ CI/CD approval ------+
                                               CodeDeploy
                                                   |
ONLINE PLANE                                       v
Client --> Route 53 --> WAF --> ALB HTTPS --> blue/green target groups
                              |                |              |
                              |              ECS GPU        ECS GPU
                              |              service A      service B
                              |                |              |
                              |           gateway + runtime containers
                              |                |
                              +--> Cognito/JWT verification, request policy
                                               |
                       CloudWatch logs/metrics/traces + CloudTrail audit
                                               |
                         alarms --> stop rollout / rollback / operator
```

The architecture uses Amazon Elastic Container Service (ECS) on Amazon Elastic Compute Cloud (EC2) GPU capacity because
GPU support and host control are central to the example. AWS Fargate is not treated as a general GPU model-serving
target. An Application Load Balancer (ALB) terminates Transport Layer Security (TLS) and routes to two target groups.
AWS WAF supplies an outer abuse/rate-control layer. The gateway enforces application authentication, model aliases,
limits, validation, and telemetry, then proxies to a runtime bound only on the task network.

Use private subnets for ECS instances/tasks where practical. Prefer VPC endpoints for S3, ECR, CloudWatch and Secrets
Manager traffic to reduce public/NAT paths. Pull models from an approved S3 release prefix or bake them into an image
only after measuring image size, launch time, and registry/storage trade-offs. Never download an unpinned model from the
public internet at every production start.

### Annotated request and release walkthrough

The diagram has two paths that should never be confused:

1. In the **release plane**, an approved manifest binds model, tokenizer, template, runtime flags, and both container
   digests. CodeDeploy creates a green task set and shifts traffic only after validation. This path changes what may
   serve requests.
2. In the **online plane**, Route 53 resolves the service, WAF applies coarse abuse controls, the ALB terminates TLS,
   and the gateway verifies identity and request bounds. The private runtime performs prefill, streams decode tokens,
   and never fetches domain documents.
3. CloudWatch receives operational signals; CloudTrail records AWS control-plane changes. Neither is in the token
   generation path, and neither substitutes for offline quality evaluation.

For one streaming request, the gateway first rejects an invalid JWT or oversized body without consuming GPU work. It
maps the stable public model alias to the immutable internal release, opens an upstream SSE stream, and forwards chunks
with backpressure. A disconnect must cancel or bound the upstream generation; otherwise abandoned clients continue to
consume capacity. TTFT measured at the client includes queueing, gateway, network, prefill, first-token sampling, and
the first visible SSE event.

## 35.6 Threat model

**Assets:** model weights and adapters, tokenizer/template, user prompts and outputs, authentication keys, release
manifests, logs, GPU capacity, safety policy, and deployment control.

<!-- markdownlint-disable MD013 -->
| Threat                        | Example                                                        | Required controls                                                                                                           | Residual risk                                  |
| ----------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Unauthorized inference        | Stolen or forged token                                         | Cognito/OIDC issuer and audience validation, short expiry, TLS, least privilege, deny-by-default routes                     | Valid credentials can still be abused          |
| Resource exhaustion           | Huge context, excessive output, slow clients, many streams     | Byte and token limits, bounded timeouts, concurrency queue, WAF rate rule, per-subject quota, cancellation and backpressure | Distributed abuse can evade one limit          |
| Prompt manipulation           | Jailbreak requests or attempts to override safety instructions | Input policy, output safety evaluation, bounded tools, specialist escalation; no runtime RAG path to poison                 | Models remain probabilistic                    |
| Model extraction              | Repeated adaptive queries/logit probing                        | No unnecessary logits, logprobs or seed controls; quotas, anomaly detection, contractual controls                           | Behavioral extraction cannot be eliminated     |
| Artifact substitution         | Mutable tag or compromised weights                             | Image digest, model hash, signed release manifest, S3 versioning, KMS, deployment approval                                  | Trusted signer or build can be compromised     |
| Parser/deserialization attack | Malicious model/tokenizer file                                 | Approved safe formats, isolated conversion, dependency scanning, non-root container, read-only filesystem                   | Native runtimes retain memory-safety exposure  |
| Sensitive-data leakage        | Prompts copied into broad logs                                 | Metadata-only default logs, explicit redaction, retention and access controls, privacy review                               | Model may reproduce memorized training content |
| Cross-tenant leakage          | Reused cache/session state                                     | Runtime isolation tests, unique request/session IDs, cache reset, no shared conversation state                              | Runtime defects remain possible                |
| Release regression            | New model gives unsafe or incompatible responses               | Offline quality/safety gates, shadow tests, canary, alarms, immutable rollback                                              | Low-frequency failures may escape canary       |
| Control-plane compromise      | Unauthorized task definition or traffic shift                  | IAM separation, CloudTrail, protected deployment role, review and alarmed mutations                                         | Privileged account compromise remains severe   |
<!-- markdownlint-restore -->

For pharmacy or hazardous repair domains, serving availability is not the only SLO. The release must enforce
intended-use limits, uncertainty and escalation behavior, and expert-reviewed safety tests. A fast unsafe answer is a
failed service.

## 35.7 Complete working-intent deployment

This section specifies one coherent deployment: a versioned OpenAI-style API gateway in front of vLLM, packaged as one
ECS task with two containers, deployed to EC2 GPU capacity behind an ALB, and released with ECS/CodeDeploy blue-green
traffic shifting.

> **Implementation status — `UNVERIFIED IMPLEMENTATION`.** Listings 35-1 through 35-9 were not executed. illustrative value
> values, image digests, model hashes, AWS account IDs, Region, subnet IDs, certificate, Cognito details, quotas, and
> prices must be supplied. The package must not be called working or production-ready until the validation checklist in
> §35.12 passes.

### API contract

- Public route: `POST /v1/chat/completions`
- Liveness: `GET /livez` — process can answer; never triggers model loading
- Readiness: `GET /readyz` — gateway can reach the runtime and the expected model alias is available
- Version metadata: `GET /version` — deployment, API, model alias, model revision and artifact hash; no secret values
- Authentication: OAuth 2.0/OpenID Connect bearer JSON Web Token (JWT), validating signature, issuer, audience and
  expiry
- External model name: stable alias such as `automotive-slm`
- Internal model name: immutable runtime alias such as `automotive-slm-2026-11-23.1`
- Errors: stable JSON object with request ID and machine-readable code
- Streaming: Server-Sent Events (SSE), passed through without buffering
- Request limits: body bytes, message count, characters, output tokens, temperature and allow-listed fields; token-level
  context limit remains a runtime check
- Idempotency: inference is not automatically replayed. Clients may provide `X-Request-ID` for correlation but should
  not assume identical sampled output.

### Listing 35-1 — Gateway dependencies

**Purpose:** pin direct gateway dependencies.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Input/output:** installed into the gateway image; no output other than a Python environment.  
**Expected failures:** unavailable package version, vulnerability-policy rejection, or dependency resolution conflict.  
**Security:** generate and review a lock file with hashes before release; the ranges below are not a release lock.

```text
# requirements.in
fastapi>=0.115,<1
uvicorn[standard]>=0.34,<1
httpx>=0.28,<1
PyJWT[crypto]>=2.10,<3
pydantic>=2.10,<3
prometheus-client>=0.21,<1
```

At implementation time, compile this file with the organization's approved lock tool and store exact versions plus
hashes. Do not silently install “latest” in a reproducible image.

### Listing 35-2 — Validating, authenticating streaming gateway

**Purpose:** enforce the public API contract and proxy only approved requests to the private runtime.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Hardware:** CPU sidecar; the sibling runtime requires its qualified accelerator.  
**Inputs:** environment variables documented below and HTTPS requests from the ALB.  
**Outputs:** JSON or SSE responses plus structured logs and Prometheus metrics.  
**Expected failures:** unavailable OpenID configuration/JWKS, invalid JWT, upstream timeout, runtime not ready, or a
request outside policy.  
**Security:** logs metadata, not prompt/output content; production must add a distributed per-subject quota store if
quotas must be exact across replicas.

```python
# app.py
import json
import logging
import os
import time
import uuid
from functools import lru_cache
from typing import Literal

import httpx
import jwt
from fastapi import Depends, FastAPI, Header, HTTPException, Request
from fastapi.responses import JSONResponse, StreamingResponse
from jwt import PyJWKClient
from prometheus_client import Counter, Histogram, make_asgi_app
from pydantic import BaseModel, ConfigDict, Field, field_validator

API_VERSION = os.environ["API_VERSION"]
DEPLOYMENT_ID = os.environ["DEPLOYMENT_ID"]
PUBLIC_MODEL = os.environ["PUBLIC_MODEL"]
UPSTREAM_MODEL = os.environ["UPSTREAM_MODEL"]
MODEL_REVISION = os.environ["MODEL_REVISION"]
MODEL_SHA256 = os.environ["MODEL_SHA256"]
UPSTREAM = os.getenv("UPSTREAM", "http://127.0.0.1:8000")
OIDC_ISSUER = os.environ["OIDC_ISSUER"].rstrip("/")
OIDC_AUDIENCE = os.environ["OIDC_AUDIENCE"]
MAX_BODY_BYTES = int(os.getenv("MAX_BODY_BYTES", "262144"))
MAX_MESSAGES = int(os.getenv("MAX_MESSAGES", "32"))
MAX_CHARS = int(os.getenv("MAX_CHARS", "65536"))
MAX_OUTPUT_TOKENS = int(os.getenv("MAX_OUTPUT_TOKENS", "1024"))
UPSTREAM_CONNECT_SECONDS = float(os.getenv("UPSTREAM_CONNECT_SECONDS", "5"))
UPSTREAM_READ_SECONDS = float(os.getenv("UPSTREAM_READ_SECONDS", "180"))

logging.basicConfig(
    level=os.getenv("LOG_LEVEL", "INFO"),
    format="%(message)s",
)
log = logging.getLogger("gateway")
app = FastAPI(title="Specialist SLM API", version=API_VERSION)
app.mount("/internal/metrics", make_asgi_app())

REQUESTS = Counter(
    "slm_requests_total", "Requests", ["route", "status_class", "stream"]
)
LATENCY = Histogram(
    "slm_request_seconds", "End-to-end request time", ["route", "stream"]
)


class Message(BaseModel):
    model_config = ConfigDict(extra="forbid")
    role: Literal["system", "user", "assistant"]
    content: str = Field(min_length=1, max_length=MAX_CHARS)


class ChatRequest(BaseModel):
    model_config = ConfigDict(extra="forbid")
    model: str
    messages: list[Message] = Field(min_length=1, max_length=MAX_MESSAGES)
    max_tokens: int = Field(default=256, ge=1, le=MAX_OUTPUT_TOKENS)
    temperature: float = Field(default=0.0, ge=0.0, le=2.0)
    top_p: float = Field(default=1.0, gt=0.0, le=1.0)
    stream: bool = False

    @field_validator("model")
    @classmethod
    def model_is_public_alias(cls, value: str) -> str:
        if value != PUBLIC_MODEL:
            raise ValueError("unknown model alias")
        return value

    @field_validator("messages")
    @classmethod
    def bounded_total_text(cls, value: list[Message]) -> list[Message]:
        if sum(len(m.content) for m in value) > MAX_CHARS:
            raise ValueError("combined message content is too large")
        return value


@lru_cache
def oidc_configuration() -> dict:
    response = httpx.get(
        f"{OIDC_ISSUER}/.well-known/openid-configuration", timeout=10
    )
    response.raise_for_status()
    return response.json()


@lru_cache
def jwk_client() -> PyJWKClient:
    return PyJWKClient(oidc_configuration()["jwks_uri"], cache_keys=True)


def authenticate(authorization: str = Header(...)) -> dict:
    scheme, _, token = authorization.partition(" ")
    if scheme.lower() != "bearer" or not token:
        raise HTTPException(status_code=401, detail="bearer token required")
    try:
        key = jwk_client().get_signing_key_from_jwt(token).key
        claims = jwt.decode(
            token,
            key,
            algorithms=["RS256"],
            audience=OIDC_AUDIENCE,
            issuer=OIDC_ISSUER,
            options={"require": ["exp", "iat", "iss", "sub"]},
        )
    except Exception as exc:
        raise HTTPException(status_code=401, detail="invalid token") from exc
    return claims


@app.middleware("http")
async def request_envelope(request: Request, call_next):
    request_id = request.headers.get("x-request-id") or str(uuid.uuid4())
    if request.headers.get("content-length"):
        try:
            if int(request.headers["content-length"]) > MAX_BODY_BYTES:
                return JSONResponse(
                    {"error": {"code": "body_too_large", "request_id": request_id}},
                    status_code=413,
                )
        except ValueError:
            return JSONResponse(
                {"error": {"code": "bad_content_length", "request_id": request_id}},
                status_code=400,
            )
    started = time.perf_counter()
    response = await call_next(request)
    response.headers["x-request-id"] = request_id
    log.info(
        json.dumps(
            {
                "event": "request",
                "request_id": request_id,
                "path": request.url.path,
                "method": request.method,
                "status": response.status_code,
                "duration_ms": round((time.perf_counter() - started) * 1000, 2),
                "deployment_id": DEPLOYMENT_ID,
                "model_revision": MODEL_REVISION,
            }
        )
    )
    return response


@app.get("/livez")
async def livez():
    return {"status": "live"}


@app.get("/readyz")
async def readyz():
    try:
        async with httpx.AsyncClient(timeout=3) as client:
            response = await client.get(f"{UPSTREAM}/v1/models")
            response.raise_for_status()
            ids = {item["id"] for item in response.json().get("data", [])}
        if UPSTREAM_MODEL not in ids:
            raise RuntimeError("expected upstream model is absent")
        return {"status": "ready", "model": PUBLIC_MODEL}
    except Exception as exc:
        raise HTTPException(status_code=503, detail="runtime not ready") from exc


@app.get("/version")
async def version(_: dict = Depends(authenticate)):
    return {
        "api_version": API_VERSION,
        "deployment_id": DEPLOYMENT_ID,
        "model": PUBLIC_MODEL,
        "model_revision": MODEL_REVISION,
        "model_sha256": MODEL_SHA256,
    }


@app.post("/v1/chat/completions")
async def chat(body: ChatRequest, claims: dict = Depends(authenticate)):
    payload = body.model_dump()
    payload["model"] = UPSTREAM_MODEL
    headers = {"x-request-subject": claims["sub"]}
    timeout = httpx.Timeout(
        connect=UPSTREAM_CONNECT_SECONDS,
        read=UPSTREAM_READ_SECONDS,
        write=30,
        pool=5,
    )
    started = time.perf_counter()

    if body.stream:
        async def events():
            try:
                async with httpx.AsyncClient(timeout=timeout) as client:
                    async with client.stream(
                        "POST",
                        f"{UPSTREAM}/v1/chat/completions",
                        json=payload,
                        headers=headers,
                    ) as upstream:
                        upstream.raise_for_status()
                        async for chunk in upstream.aiter_bytes():
                            yield chunk
                REQUESTS.labels("chat", "2xx", "true").inc()
            except Exception:
                REQUESTS.labels("chat", "5xx", "true").inc()
                log.exception('{"event":"stream_failed"}')
                yield b'data: {"error":{"code":"upstream_failed"}}\n\n'
            finally:
                LATENCY.labels("chat", "true").observe(
                    time.perf_counter() - started
                )
        return StreamingResponse(events(), media_type="text/event-stream")

    try:
        async with httpx.AsyncClient(timeout=timeout) as client:
            upstream = await client.post(
                f"{UPSTREAM}/v1/chat/completions",
                json=payload,
                headers=headers,
            )
        REQUESTS.labels(
            "chat", f"{upstream.status_code // 100}xx", "false"
        ).inc()
        LATENCY.labels("chat", "false").observe(time.perf_counter() - started)
        return JSONResponse(upstream.json(), status_code=upstream.status_code)
    except Exception as exc:
        REQUESTS.labels("chat", "5xx", "false").inc()
        LATENCY.labels("chat", "false").observe(time.perf_counter() - started)
        raise HTTPException(status_code=502, detail="runtime unavailable") from exc
```

This listing intentionally does not count prompt tokens in the gateway because character limits are only an outer bound.
The runtime must enforce the exact tokenizer/context limit. The JWKS cache also needs an operational refresh test for
signing-key rotation. The `/internal/metrics` route must be restricted by security group or service mesh and not exposed
through the public listener.

### Listing 35-3 — Gateway image

**Purpose:** build an immutable, non-root gateway container.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Expected failures:** lock file absent, base digest invalid, or health endpoint unavailable.  
**Security:** replace both illustrative values with reviewed immutable digests and install from a hash-locked requirements
file.

```dockerfile
# Dockerfile.gateway
FROM python:3.12-slim@sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
WORKDIR /app
COPY requirements.lock /app/requirements.lock
RUN pip install --no-cache-dir --require-hashes -r requirements.lock \
 && useradd --system --uid 10001 --no-create-home app
COPY --chown=10001:10001 app.py /app/app.py
USER 10001
EXPOSE 8080
CMD ["uvicorn", "app:app", "--host=0.0.0.0", "--port=8080", \
     "--proxy-headers", "--forwarded-allow-ips=*"]
```

Only accept forwarded headers from the trusted ALB/task network. If the gateway can be reached directly,
`--forwarded-allow-ips=*` is unsafe and must be narrowed.

### Listing 35-4 — Local qualification composition

**Purpose:** exercise the two-container contract before AWS deployment.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Hardware:** NVIDIA GPU with a driver/runtime supported by the selected vLLM image.  
**Inputs:** a locally mounted, hash-verified model directory and an OIDC test issuer.  
**Expected failures:** insufficient VRAM, incompatible GPU, unsupported checkpoint, missing chat template, or invalid
image digest.

```yaml
# compose.yaml — qualification only; not the production scheduler
services:
  runtime:
    image: vllm/vllm-openai@sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    command:
      - --model=/models/automotive
      - --served-model-name=automotive-slm-2026-11-23.1
      - --max-model-len=4096
      - --dtype=auto
      - --disable-log-requests
    volumes:
      - ./model:/models/automotive:ro
    ports:
      - "127.0.0.1:8000:8000"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  gateway:
    build:
      context: .
      dockerfile: Dockerfile.gateway
    network_mode: host
    environment:
      API_VERSION: "2026-11-23"
      DEPLOYMENT_ID: "local-qualification"
      PUBLIC_MODEL: "automotive-slm"
      UPSTREAM_MODEL: "automotive-slm-2026-11-23.1"
      MODEL_REVISION: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
      MODEL_SHA256: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
      UPSTREAM: "http://127.0.0.1:8000"
      OIDC_ISSUER: "https://auth.example.invalid"
      OIDC_AUDIENCE: "slm-api-illustrative"
      MAX_OUTPUT_TOKENS: "1024"
```

Production ECS should use task networking rather than host networking unless a reviewed architecture requires otherwise.

### Listing 35-5 — ECS task-definition fragment

**Purpose:** bind immutable gateway and runtime images, GPU reservation, health, logs, limits and version metadata into
one release unit.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Prerequisites:** ECS cluster backed by a qualified GPU EC2 Auto Scaling group, NVIDIA ECS-optimized Amazon Machine
Image (AMI) or equivalent reviewed host image, execution/task roles, CloudWatch log group, private networking and ALB
target groups.  
**Expected failures:** no GPU capacity, image/model pull timeout, incompatible driver, failed readiness, or insufficient
task memory.

```json
{
  "family": "specialist-slm",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["EC2"],
  "cpu": "8192",
  "memory": "32768",
  "executionRoleArn": "arn:aws:iam::111122223333:role/slm-ecs-execution",
  "taskRoleArn": "arn:aws:iam::111122223333:role/slm-runtime",
  "containerDefinitions": [
    {
      "name": "runtime",
      "image": "111122223333.dkr.ecr.us-west-2.amazonaws.com/slm-runtime@sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "essential": true,
      "resourceRequirements": [{ "type": "GPU", "value": "1" }],
      "command": [
        "--model=/models/automotive",
        "--served-model-name=automotive-slm-2026-11-23.1",
        "--max-model-len=4096",
        "--disable-log-requests"
      ],
      "mountPoints": [{ "sourceVolume": "approved-model", "containerPath": "/models", "readOnly": true }],
      "portMappings": [{ "containerPort": 8000, "protocol": "tcp" }],
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -fsS http://127.0.0.1:8000/v1/models || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 10,
        "startPeriod": 900
      },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/slm/production",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "runtime",
          "mode": "non-blocking",
          "max-buffer-size": "25m"
        }
      }
    },
    {
      "name": "gateway",
      "image": "111122223333.dkr.ecr.us-west-2.amazonaws.com/slm-gateway@sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "essential": true,
      "dependsOn": [{ "containerName": "runtime", "condition": "HEALTHY" }],
      "environment": [
        { "name": "API_VERSION", "value": "2026-11-23" },
        { "name": "DEPLOYMENT_ID", "value": "release-0.1.0-illustrative" },
        { "name": "PUBLIC_MODEL", "value": "automotive-slm" },
        { "name": "UPSTREAM_MODEL", "value": "automotive-slm-2026-11-23.1" },
        { "name": "MODEL_REVISION", "value": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" },
        { "name": "MODEL_SHA256", "value": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" },
        { "name": "UPSTREAM", "value": "http://127.0.0.1:8000" },
        { "name": "OIDC_ISSUER", "value": "https://cognito-idp.us-west-2.amazonaws.com/us-west-2_Example01" },
        { "name": "OIDC_AUDIENCE", "value": "illustrative-client-id" }
      ],
      "portMappings": [{ "containerPort": 8080, "protocol": "tcp" }],
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -fsS http://127.0.0.1:8080/readyz || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 5,
        "startPeriod": 30
      },
      "readonlyRootFilesystem": true,
      "linuxParameters": { "initProcessEnabled": true },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/slm/production",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "gateway",
          "mode": "non-blocking",
          "max-buffer-size": "25m"
        }
      }
    }
  ],
  "volumes": [
    {
      "name": "approved-model",
      "host": { "sourcePath": "/opt/slm/models/release-2026-11-23.1" }
    }
  ]
}
```

The host path must be populated by a controlled bootstrap step that downloads an immutable S3 object version, verifies
the signed manifest and every hash, and only then atomically moves it into the release directory. A failed verification
must keep the task unready. For multi-host production, never assume a model copied manually to one EC2 host exists on
the rest.

The `startPeriod` is an explicit cold-start allowance, not evidence that 900 seconds is sufficient. Measure model
download, hash verification, weight load, kernel initialization, graph capture and first warm-up separately, then set
health and deployment timeouts from the observed distribution.

### Listing 35-6 — ECS blue-green AppSpec

**Purpose:** identify the immutable replacement task definition and production/test listeners for CodeDeploy.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Expected failures:** task set never reaches steady state, validation hook fails, CloudWatch alarm enters `ALARM`, or
traffic-shift permissions are missing.

```yaml
# appspec.yaml
version: 1
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "arn:aws:ecs:us-west-2:111122223333:task-definition/slm-demo:1"
        LoadBalancerInfo:
          ContainerName: gateway
          ContainerPort: 8080
        PlatformVersion: "LATEST"
Hooks:
  - AfterAllowTestTraffic: "arn:aws:lambda:us-west-2:111122223333:function:slm-validate"
  - BeforeAllowTraffic: "arn:aws:lambda:us-west-2:111122223333:function:slm-approve"
  - AfterAllowTraffic: "arn:aws:lambda:us-west-2:111122223333:function:slm-observe"
```

`DOC-VERIFIED`: ECS CodeDeploy blue-green supports canary, linear and all-at-once traffic shifts when used with the
documented load-balancer configuration. A production deployment group should attach alarms for readiness failures, 5xx
rate, latency, runtime restarts and GPU out-of-memory signals. Canary traffic must be representative enough to detect
tokenizer/template, safety, context, streaming and load regressions.

### Listing 35-7 — Autoscaling policy commands

**Purpose:** scale ECS tasks from a bounded minimum to maximum using a custom backlog-per-ready-replica metric, with a
CPU fallback shown only as a bootstrap.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Expected failures:** GPU EC2 capacity unavailable, placement constraint failure, metric missing, or cold starts longer
than the scale-out window.

```powershell
$Region = "us-west-2"
$Cluster = "slm-production"
$Service = "specialist-slm"

aws application-autoscaling register-scalable-target `
  --region $Region `
  --service-namespace ecs `
  --resource-id "service/$Cluster/$Service" `
  --scalable-dimension ecs:service:DesiredCount `
  --min-capacity 1 `
  --max-capacity 4

aws application-autoscaling put-scaling-policy `
  --region $Region `
  --policy-name slm-cpu-bootstrap `
  --service-namespace ecs `
  --resource-id "service/$Cluster/$Service" `
  --scalable-dimension ecs:service:DesiredCount `
  --policy-type TargetTrackingScaling `
  --target-tracking-scaling-policy-configuration `
    file://target-tracking.json
```

```json
{
  "TargetValue": 60.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
  },
  "ScaleOutCooldown": 900,
  "ScaleInCooldown": 1800,
  "DisableScaleIn": false
}
```

CPU utilization is not a good final proxy for a GPU token server. Publish queue depth, active sequences, queued tokens,
TTFT and KV-cache pressure from the runtime/exporter, and use a validated target metric or step scaling. Scale-in must
drain connections and allow streams to finish within a bounded deregistration delay. Keep at least one warm replica for
interactive traffic unless measured business requirements accept scale-from-zero latency.

The EC2 Auto Scaling group must itself have enough qualified GPU instances for the ECS desired count. ECS task scaling
without node-capacity scaling only creates queued tasks. Capacity Providers can coordinate this, but quota, launch
capacity, model staging time and Availability Zone coverage still require load tests.

### Listing 35-8 — WAF rate rule fragment

**Purpose:** apply a coarse outer request-rate control before requests consume gateway or GPU resources.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Expected failures:** legitimate shared-NAT clients are throttled, distributed attackers evade IP aggregation, or
streaming concurrency consumes capacity below the request threshold.

```json
{
  "Name": "slm-rate-limit",
  "Priority": 10,
  "Statement": {
    "RateBasedStatement": {
      "Limit": 300,
      "EvaluationWindowSec": 300,
      "AggregateKeyType": "IP",
      "ScopeDownStatement": {
        "ByteMatchStatement": {
          "FieldToMatch": { "UriPath": {} },
          "PositionalConstraint": "STARTS_WITH",
          "SearchString": "/v1/",
          "TextTransformations": [{ "Priority": 0, "Type": "NONE" }]
        }
      }
    }
  },
  "Action": { "Block": {} },
  "VisibilityConfig": {
    "CloudWatchMetricsEnabled": true,
    "MetricName": "slm-rate-limit",
    "SampledRequestsEnabled": false
  }
}
```

The value `300` is an `ESTIMATE` illustrative value, not a recommended production threshold. WAF limits are not a per-user
token budget. Add a distributed quota keyed by authenticated subject or tenant and charged by accepted input/output
tokens if product policy requires fair usage. Never trust an unsigned client-supplied subject header.

### Listing 35-9 — Build, register and deploy sequence

**Purpose:** show an auditable release sequence that uses immutable digests and preserves rollback.  
**Status:** `UNVERIFIED IMPLEMENTATION`.  
**Expected failures:** failed tests, unapproved vulnerability/license findings, hash mismatch, registry push failure,
task registration failure, or canary alarm.

```powershell
$ErrorActionPreference = "Stop"
$Region = "us-west-2"
$Account = "111122223333"
$Release = "2026-11-23.1"
$GatewayRepo = "$Account.dkr.ecr.$Region.amazonaws.com/slm-gateway"

# 1. Required gates before this script:
#    - exact model/tokenizer revision and license approved
#    - model manifest signature and hashes verified
#    - tokenizer/chat-template and quality regression passed
#    - requirements.lock generated with hashes and scanned

aws ecr get-login-password --region $Region |
  docker login --username AWS --password-stdin "$Account.dkr.ecr.$Region.amazonaws.com"

docker build --pull --no-cache `
  -f Dockerfile.gateway `
  -t "${GatewayRepo}:$Release" .
docker push "${GatewayRepo}:$Release"

$Digest = aws ecr describe-images `
  --region $Region `
  --repository-name slm-gateway `
  --image-ids "imageTag=$Release" `
  --query "imageDetails[0].imageDigest" `
  --output text
if (-not $Digest.StartsWith("sha256:")) { throw "ECR digest not returned" }

# Render task-definition.json from reviewed template, replacing only declared
# illustrative values. Store the rendered file and hashes with the release record.
$TaskArn = aws ecs register-task-definition `
  --region $Region `
  --cli-input-json file://task-definition.json `
  --query "taskDefinition.taskDefinitionArn" `
  --output text

# Render appspec.yaml with $TaskArn, upload an immutable revision bundle to S3,
# then create a CodeDeploy deployment using the reviewed deployment group.
aws deploy create-deployment `
  --region $Region `
  --application-name specialist-slm `
  --deployment-group-name specialist-slm-production `
  --revision file://revision.json `
  --description "specialist-slm $Release"
```

Do not delete the blue task definition, image digest, model artifact or configuration until the rollback window closes.
Rollback means redirecting traffic to a complete known-good release tuple, not combining an old model with a new
tokenizer or gateway.

The build/deploy commands have deliberately narrow meanings. ECR login authorizes a registry session; it does not
approve an image. `describe-images` converts the pushed tag into an immutable digest. `register-task-definition`
creates a versioned ECS specification but starts no workload. `create-deployment` asks CodeDeploy to execute the
reviewed blue-green policy; its returned success means only that the request was accepted. Readiness, test-listener
checks, alarms, canary observations, and the final deployment state provide the operational evidence.

## 35.8 Cold start, loading and availability

Cold start is a pipeline:

1. acquire GPU capacity;
2. boot and register the EC2 host;
3. pull container layers;
4. download/stage the model;
5. verify signatures and hashes;
6. map/read weights and allocate GPU memory;
7. initialize kernels/graphs and KV-cache pools;
8. complete a representative warm-up;
9. pass readiness and enter the target group.

Measure each phase. Common controls are:

- pre-bake stable driver/runtime layers in the host image;
- keep images small and pull by digest through private ECR connectivity;
- stage approved artifacts to encrypted local storage before task start;
- retain a minimum warm task count for interactive service;
- use slow scale-in and predictive/scheduled scale-out for known peaks;
- decouple `/livez` from model readiness;
- allow long initialization without accepting traffic;
- warm the exact paths used in production, but never claim warm-up reproduces all workload shapes.

Scale-to-zero can be appropriate for asynchronous or noninteractive jobs. It is usually incompatible with a strict
interactive TTFT objective when accelerator allocation and model loading take substantial time. SageMaker Serverless
Inference should not be presented as GPU LLM hosting unless the current feature matrix explicitly supports the intended
GPU configuration; at this cut-off, the surveyed AWS matrix did not list GPU serverless support.

## 35.9 Blue-green, canary and rollback semantics

<a id="fig-ch35-26"></a> **FIG-CH35-26 — Cloud model deployment lifecycle (`ILLUSTRATIVE`; canonical cross-target FIG-26
is owned by CH-36).** Alt text: a cloud release progresses through approval, staging, canary, activation, monitoring,
rollback, and retirement states.

```text
build -> static/security scan -> offline quality and safety gates
  -> stage green -> smoke + tokenizer/template + streaming tests
  -> test listener -> canary traffic -> observe SLOs and safety indicators
  -> promote or automatic stop
  -> full traffic -> observation window
  -> retire blue only after rollback window

Any gate fails -------------------------------------------> restore blue
```

A model release is a tuple:

```text
release =
  model hash + tokenizer hash + chat-template hash + runtime image digest
  + gateway image digest + runtime flags + API schema + safety policy
```

Rollback restores the tuple. Merely changing a model alias is insufficient if the runtime image or template caused the
incident. Database-style schema migration is small here, but clients can still depend on response fields. Version public
behavior explicitly and test backward compatibility.

Recommended canary gates:

- green `/readyz` remains healthy through warm-up;
- exact model/version endpoint matches the approved manifest;
- fixed golden prompts produce structurally valid output;
- tokenizer and stop-token tests pass;
- streaming first event, cancellation and client disconnect work;
- no cross-session cache leakage is observed;
- TTFT, end-to-end latency, error rate, queue depth, GPU memory and restarts remain within canary thresholds;
- domain quality/safety shadow evaluation shows no blocking regression;
- CloudWatch alarms and a manual abort path have been tested.

## 35.10 Observability and service objectives

### Signals

Collect bounded-cardinality metrics:

- request rate, accepted/rejected rate, active streams and status class;
- queue depth/time, TTFT, prefill duration/rate, inter-token latency, decode rate and end-to-end duration;
- prompt and output token counts as distributions, not identifiers;
- context-limit, timeout, cancellation and out-of-memory counts;
- model-load phase duration and readiness time;
- GPU utilization, memory, temperature/power where exposed, host CPU/memory/disk/network;
- replica desired/running/queued counts and placement failures;
- deployment ID, model revision and runtime image digest as release metadata.

NVIDIA GPU metrics generally require an exporter/agent such as Data Center GPU Manager (DCGM); they are not
automatically present in every CloudWatch setup.

Structured logs should include timestamp, request ID, authenticated tenant/subject hash where approved, route, release,
token counts, durations, finish reason, and error class. Do not log bearer tokens, raw prompts, outputs, model files, or
sensitive domain content by default. CloudTrail records AWS control-plane activity; it does not replace application
metrics and logs.

Trace sampling must respect streaming duration and privacy. Propagate a request/trace ID through ALB, gateway and
runtime, but keep high-cardinality IDs out of metric labels.

### Proposed SLOs

These are `PLANNED EXPERIMENT` targets, not achieved values. Fill thresholds only after baseline measurement and
product-risk review.

<!-- markdownlint-disable MD013 -->
| SLI                  | Definition                                            | Proposed objective illustrative value             | Exclusions and notes                                                                                         |
| -------------------- | ----------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| Availability         | valid, authorized requests not failing due to service | `99.0%` over 28 days                        | Exclude documented client validation/auth failures; do not exclude overload caused by poor capacity planning |
| TTFT                 | receive request to first streamed content token       | p50 `0.20 s`, p95 `0.80 s`, p99 `1.50 s`            | Segment by prompt-token bucket and release                                                                   |
| Decode cadence       | inter-token latency after first token                 | p95 `0.10 s` for declared output bucket       | Report stalled streams separately                                                                            |
| End-to-end           | receive to final event                                | p95 `8.0 s` for declared input/output bucket | Sampling and output length materially affect it                                                              |
| Correct-version rate | responses served by intended approved release         | `100%`                                     | Any unknown or mixed tuple is a release incident                                                             |
| Safety gate          | blocking domain benchmark regression                  | zero release-blocking regressions          | Online proxy signals do not replace expert evaluation                                                        |
<!-- markdownlint-restore -->

Error-budget policy should freeze risky model/runtime releases when the budget is exhausted, while allowing reliability
and security fixes. A safety-critical regression can trigger rollback even if latency and availability remain healthy.

## 35.11 Reproducible benchmarking

**PLANNED EXPERIMENT — no results are claimed.**

### Workload definition

Create a versioned, non-sensitive workload with at least:

- short, medium and near-limit input buckets;
- fixed output-token caps and a realistic output distribution;
- deterministic temperature-zero quality probes and sampled production-like probes;
- concurrency 1, low, expected and overload levels;
- streaming and non-streaming calls;
- cold host, cold model, warm model and steady-state phases;
- domain prompts, refusal/escalation cases and adversarial malformed requests.

Use identical model/tokenizer content and the closest supported precision across runtimes. If artifact formats differ,
record that the comparison includes conversion/quantization effects. Never call such a comparison “runtime only.”

### Measurements

For each trial record:

- Region, Availability Zone, instance type, accelerator count and driver;
- host image, container digest, runtime version/commit and command;
- model/tokenizer/template hashes, quantizer and calibration record;
- context/batch/cache settings;
- exact prompt/output token counts under the runtime tokenizer;
- model-load time, peak host/GPU memory, TTFT, prefill tokens/s, decode tokens/s, inter-token latency, request
  throughput, queue time, errors and energy/power where measurable;
- all billable resources and dated unit rates.

Run warm-up separately, then enough repeated trials to report median, tail percentiles and uncertainty. Randomize
candidate order where thermal or shared-environment drift could bias results. Retain raw events, failed trials and
environment manifests.

### Load-driver skeleton

**Listing 35-10 — controlled asynchronous request driver.**  
**Status:** `UNVERIFIED IMPLEMENTATION`. It is a starting point, not a complete benchmark harness; production
benchmarking must parse SSE events and token usage precisely, preserve raw timestamps, and coordinate arrival rates
rather than only concurrency.

```python
import asyncio
import json
import os
import time

import httpx

URL = os.environ["SLM_URL"].rstrip("/") + "/v1/chat/completions"
TOKEN = os.environ["SLM_TOKEN"]
CONCURRENCY = int(os.getenv("CONCURRENCY", "4"))
REQUESTS = int(os.getenv("REQUESTS", "40"))

payload = {
    "model": "automotive-slm",
    "messages": [{"role": "user", "content": "Explain a safe diagnostic sequence."}],
    "temperature": 0,
    "max_tokens": 128,
    "stream": False,
}


async def one(client: httpx.AsyncClient, index: int) -> dict:
    start = time.perf_counter_ns()
    response = await client.post(
        URL,
        headers={
            "authorization": f"Bearer {TOKEN}",
            "x-request-id": f"bench-{index}",
        },
        json=payload,
    )
    end = time.perf_counter_ns()
    body = response.json()
    return {
        "index": index,
        "status": response.status_code,
        "elapsed_ms": (end - start) / 1_000_000,
        "usage": body.get("usage"),
        "request_id": response.headers.get("x-request-id"),
    }


async def main():
    semaphore = asyncio.Semaphore(CONCURRENCY)
    async with httpx.AsyncClient(timeout=300) as client:
        async def bounded(i):
            async with semaphore:
                return await one(client, i)
        results = await asyncio.gather(
            *(bounded(i) for i in range(REQUESTS)),
            return_exceptions=True,
        )
    for item in results:
        print(json.dumps(
            {"error": repr(item)} if isinstance(item, Exception) else item
        ))


asyncio.run(main())
```

Do not use this client from the same host as the server for a final network-latency result. Synchronize clocks where
cross-host event timestamps are compared.

## 35.12 Validation and release checklist

The deployment remains unverified until evidence exists for every applicable gate:

- [ ] Target instance is available in the selected Region/AZ and account quota/capacity is confirmed.
- [ ] Host AMI, NVIDIA driver, container runtime and ECS agent versions are pinned.
- [ ] Gateway/runtime images are pinned by digest and scanned; dependencies have reviewed lock files.
- [ ] Model, tokenizer, template and adapter hashes match a signed approved manifest.
- [ ] Exact model/runtime/precision combination loads and passes source-parity and domain-quality checks.
- [ ] Context, concurrency and KV-cache limits are measured without out-of-memory failure.
- [ ] Authentication rejects wrong signature, issuer, audience, expiry and algorithm.
- [ ] Request body, field, message, output-token, runtime-context and timeout limits are tested.
- [ ] WAF and per-subject quotas are load-tested, including shared-NAT and distributed traffic cases.
- [ ] Logs contain no tokens, prompts, outputs or secrets; retention and access are reviewed.
- [ ] Liveness, readiness, graceful drain, cancellation and disconnected streams are tested.
- [ ] ECS and EC2 capacity scaling are tested together under a measured cold-start distribution.
- [ ] Blue-green test listener, canary alarms, automatic stop and manual rollback are exercised.
- [ ] The complete known-good release tuple remains available through the rollback window.
- [ ] Cost alarms, budget, anomaly detection, idle shutdown and maximum replica count are active.
- [ ] No primary online retrieval, vector database, embedding search or remote-teacher substitution exists.
- [ ] Safety-critical domain release gates and escalation behavior pass expert review.

## 35.13 Cost controls

Use controls at four layers:

1. **Admission:** authenticated tenants, request and token caps, bounded concurrency, WAF outer limits, and explicit
   overload responses prevent unlimited GPU work.
2. **Capacity:** minimum/maximum replicas, qualified instance list, warm-capacity decision, scheduled scaling,
   queue-based scaling, and no accidental P-class overprovisioning.
3. **Lifecycle:** automatic nonproduction shutdown, expiration tags, image/artifact retention policies that preserve
   active rollback releases, and termination protection only where justified.
4. **Financial monitoring:** AWS Budgets, Cost Explorer, cost-allocation tags, Cost Anomaly Detection and alarms for GPU
   uptime, NAT/data transfer, CloudWatch ingestion, EBS and load-balancer growth.

For each candidate calculate:

```text
monthly total =
  GPU/CPU instance time
  + EBS and snapshots
  + S3 storage and requests
  + image storage and pulls
  + load balancer, WAF and public IPv4
  + NAT/VPC endpoint and data transfer
  + logs, metrics, traces and alarms
  + blue-green overlap
  + failed/cold/idle capacity
  + engineering and incident-response cost
```

Use Spot capacity only where interruption is acceptable. Interactive production endpoints usually need a stable
On-Demand or committed baseline; an interrupted GPU stream cannot be transparently resumed without application protocol
and state support. Savings Plans or reservations should follow measured steady demand, not precede it.

No dollar value appears here because AWS prices are volatile and configuration-specific. Capture timestamp, Region,
service, SKU/rate code, purchase model and billing unit from official pricing sources, then multiply by measured use.

## 35.14 Domain-specialist and edge implications

Cloud serving can host a larger or less-compressed specialist model than a phone and can centralize patches, audit and
capacity. Edge serving provides offline availability and privacy but shifts updates, memory, energy and hardware
compatibility to the device. The two targets need not use one artifact:

- vLLM/SGLang commonly consume HF-style weights optimized for server GPUs;
- llama.cpp consumes GGUF and can provide cloud-edge artifact parity where this is valuable;
- ONNX, Core ML, ExecuTorch, LiteRT and other edge paths have separate conversion contracts;
- TensorRT-LLM optimization is NVIDIA-specific and should not be treated as an edge-mobile export.

Always compare final outputs after each conversion. A cloud model and a mobile quantized model with the same marketing
name are different release artifacts and can have different safety and domain quality.

Multi-domain serving choices include separate services, one shared base with tested adapters, or an application router
selecting already-trained models. Adapter hot-swapping can save weight memory, but only if the runtime's exact adapter,
batching and isolation behavior passes qualification. It must not silently route requests to a remote teacher or
retrieve domain documents.

## 35.15 Failure modes and troubleshooting

<!-- markdownlint-disable MD013 -->
| Symptom                            | Likely causes                                                               | Evidence to collect                                    | Corrective direction                                              |
| ---------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------- |
| Model loads locally but not on ECS | driver/container mismatch, missing host model, GPU reservation, task memory | ECS events, host/runtime versions, mount and hash logs | align pinned stack; fail staging atomically                       |
| Readiness flaps                    | health probes too aggressive, long load, runtime restart/OOM                | phase timings, container exit, GPU memory              | separate liveness/readiness; right-size and tune start period     |
| High TTFT but good decode rate     | queueing, long prefill, batch policy, scale lag                             | queue and prompt buckets, prefill timing               | tune scheduler, cap input, scale from backlog                     |
| Good average, poor p99             | mixed prompt lengths, cold tasks, noisy host, overload                      | bucketed percentiles, deployment/cold flags            | isolate workload, maintain warm capacity, admission control       |
| New release returns malformed chat | tokenizer/template/runtime parser mismatch                                  | immutable hashes and golden prompts                    | restore complete old tuple; correct and requalify                 |
| Autoscaling creates queued tasks  | no GPU node capacity/quota or placement conflict                            | ECS placement events, ASG activity, quotas             | coordinate Capacity Provider/ASG and task scaling                 |
| Streams terminate during deploy    | deregistration delay too short or gateway not draining                      | ALB target and task stop events                        | implement drain and bound stream duration                         |
| Cost rises without traffic         | minimum/blue fleet left running, log cardinality, NAT/model downloads       | tagged Cost Explorer and usage metrics                 | reduce idle duplication, private paths, retention and cardinality |
<!-- markdownlint-restore -->

## 35.16 Exercises

1. **Runtime qualification.** Choose one immutable 0.5B–4B specialist checkpoint. Build a compatibility record for vLLM,
   SGLang and llama.cpp that distinguishes model-family documentation from exact artifact execution. Add TensorRT-LLM
   only if the target GPU is supported. Evidence required: commands, hashes, environment manifest, raw failures and one
   of the approved evidence labels.
2. **Capacity calculation.** Compute weight and KV-cache planning estimates for two contexts and four concurrency
   levels. Then compare them with measured GPU memory. Explain every discrepancy instead of adjusting the formula after
   the fact.
3. **API security review.** Threat-model Listing 35-2. Add an exact distributed per-subject token quota without logging
   prompt text. Test JWT key rotation, unknown `kid`, expired tokens, wrong audience and malformed `Content-Length`.
4. **Cold-start experiment.** Instrument all nine phases in §35.8. Compare image-embedded weights, encrypted EBS staging
   and S3-at-start under identical artifact hashes. Include attached storage/network costs.
5. **Canary and rollback drill.** Intentionally deploy a green release with a wrong model alias in a nonproduction
   account. Prove that readiness or the validation hook blocks traffic, then exercise rollback. Do not perform this on
   production.
6. **SLO design.** Convert the illustrative values in §35.10 into justified objectives for an automotive support service.
   Define prompt/output buckets, error-budget policy and a separate safety-release gate.
7. **TGI decision memo.** Given an existing TGI deployment, determine whether to retain or migrate it. Use the vendor's
   maintenance-mode wording precisely and include migration risk, parity tests and rollback.
8. **Cost decision.** Compare one EC2/ECS design with one SageMaker real-time endpoint using dated official rates and
   measured usage. Include all resources listed in §35.13 and run sensitivity analysis for 10%, 40% and 70% utilization.

## 35.17 Chapter conclusions

- vLLM is the first GPU baseline for a mainstream supported HF-format model; SGLang is the required contemporary
  alternative where its scheduling, prefix reuse or model support may fit better.
- TGI remains usable for existing deployments but is officially in maintenance mode; it is not the default for a new
  build.
- llama.cpp is the principal GGUF/CPU and cloud-edge-parity comparison.
- ORT GenAI is a useful portability experiment but its Generate API is documented as preview.
- TensorRT-LLM is an advanced NVIDIA optimization, not a universal replacement.
- Model compatibility, performance and cost are properties of an exact
  model/runtime/artifact/hardware/configuration/workload tuple.
- Production readiness requires an authenticated and validated API, bounded resource use, observable scheduling,
  coordinated GPU scaling, staged release, complete-tuple rollback and explicit cost controls.
- The deployment in this chapter is honestly `UNVERIFIED IMPLEMENTATION`; it is a complete working-intent design, not
  evidence of AWS execution.

## 35.18 Official source register

Sources were accessed for the 2026-11-23 research cut-off. Official documentation verifies what a project or vendor
documents; it does not independently reproduce behavior.

### Serving runtimes

- vLLM: [installation and platforms](https://docs.vllm.ai/en/stable/getting_started/installation/),
  [supported models](https://docs.vllm.ai/en/stable/models/supported_models.html),
  [quantization](https://docs.vllm.ai/en/stable/features/quantization/),
  [parallelism and scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/),
  [OpenAI-compatible server](https://docs.vllm.ai/en/stable/serving/openai_compatible_server.html).
- SGLang: [documentation](https://docs.sglang.io/),
  [OpenAI APIs](https://docs.sglang.io/basic_usage/openai_api_completions.html),
  [hardware platforms](https://docs.sglang.io/docs/hardware-platforms/overview),
  [supported models](https://docs.sglang.io/docs/supported-models/generative_models),
  [quantization](https://docs.sglang.io/docs/advanced_features/quantization).
- Hugging Face TGI:
  [maintenance status and migration guidance](https://huggingface.co/docs/inference-endpoints/engines/tgi),
  [repository](https://github.com/huggingface/text-generation-inference),
  [quantization](https://huggingface.co/docs/text-generation-inference/conceptual/quantization).
- llama.cpp: [repository](https://github.com/ggml-org/llama.cpp),
  [build backends](https://github.com/ggml-org/llama.cpp/blob/HEAD/docs/build.md),
  [HTTP server](https://github.com/ggml-org/llama.cpp/blob/HEAD/tools/server/README.md).
- ONNX Runtime GenAI: [Generate API preview](https://onnxruntime.ai/docs/genai/),
  [configuration and model types](https://onnxruntime.ai/docs/genai/reference/config.html),
  [installation variants](https://onnxruntime.ai/docs/genai/howto/install.html).
- NVIDIA TensorRT-LLM: [support matrix](https://nvidia.github.io/TensorRT-LLM/reference/support-matrix.html),
  [supported models](https://nvidia.github.io/TensorRT-LLM/latest/models/supported-models.html),
  [quantization](https://nvidia.github.io/TensorRT-LLM/latest/features/quantization.html),
  [paged attention and in-flight batching](https://nvidia.github.io/TensorRT-LLM/latest/features/paged-attention-ifb-scheduler.html),
  [`trtllm-serve` quick start](https://nvidia.github.io/TensorRT-LLM/latest/quick-start-guide.html).
- AWS Neuron: [documentation](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/),
  [supported model architectures](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/inference-legacy/nxd-inference/developer_guides/model-reference.html),
  [vLLM Neuron plugin](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/vllm-neuron/).

### AWS deployment and operations

- EC2: [accelerated-computing specifications](https://docs.aws.amazon.com/ec2/latest/instancetypes/ac.html),
  [instance types by Region](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-regions.html).
- ECS: [GPU workloads](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-gpu.html),
  [service auto scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html),
  [CodeDeploy blue-green](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html).
- Elastic Load Balancing:
  [ALB target-group health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html),
  [user authentication](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html).
- AWS WAF:
  [rate-based rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html).
- Amazon Cognito:
  [user-pool JSON Web Tokens](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-the-id-token.html).
- CloudWatch:
  [NVIDIA GPU metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-NVIDIA-GPU.html),
  [Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html).
- SageMaker AI: [inference options](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model-options.html),
  [deployment feature matrix](https://docs.aws.amazon.com/sagemaker/latest/dg/model-deploy-feature-matrix.html),
  [endpoint deployment guardrails](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails.html).
- Bedrock:
  [Custom Model Import](https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-import-model.html).
- Cost management:
  [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html),
  [Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html),
  [Cost Anomaly Detection](https://docs.aws.amazon.com/cost-management/latest/userguide/manage-ad.html).
- Pricing procedure: [EC2 On-Demand](https://aws.amazon.com/ec2/pricing/on-demand/),
  [SageMaker AI](https://aws.amazon.com/sagemaker/ai/pricing/),
  [AWS Price List Query API](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/price-changes.html).

## Cross-links

- [Part 9 index](README.md)
- [CH-33: AWS Infrastructure for SLM Development](ch-33-aws-infrastructure-for-slm-development.md)
- [CH-34: Training SLMs on AWS](ch-34-training-slms-on-aws.md)
- [CH-36: AWS-to-Edge Model Delivery](ch-36-aws-to-edge-model-delivery.md)
- [CH-35 assessment](../../13-assessments/ch-35-assessment.md)
- [AWS capability survey](../../04-research-comparison-report/aws-capability-survey.md)
- [Edge runtime survey](../../04-research-comparison-report/edge-runtime-survey.md)
- [Evidence and citation standard](../../00-governance/evidence-and-citation-standard.md)
- [No-runtime-RAG standard](../../00-governance/no-runtime-rag-standard.md)
- [Reproducibility standard](../../00-governance/reproducibility-standard.md)

## Honesty note

The runtime capabilities are documentation-based and the deployment is a worked design. It requires locked
dependencies, nonproduction deployment, load testing, canary/rollback drills, security validation, and dated billing
evidence before any success, performance, or cost claim.
