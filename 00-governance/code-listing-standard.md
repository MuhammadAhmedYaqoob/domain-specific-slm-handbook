# Code Listing Standard

## Why a code block needs a status

Readers often assume that polished code has run. That assumption is dangerous when commands can allocate paid cloud
resources, download an incompatible checkpoint, or fail on a phone with limited memory. Every executable-looking block,
configuration, query, command sequence, and build file MUST state what validation it actually received.

Markdown is currently the canonical source for listings. Pseudocode MUST say `PSEUDOCODE` and must not be described as
runnable.

## Identify and explain each listing

Give each substantive listing an immutable ID such as `LST-TRAIN-0017`, an explicit anchor, and an entry in the
[implementation ledger](../15-audit-ledgers/implementation-verification-ledger.md). Before the fence, tell the learner:

- the listing's purpose and what it does not do;
- language, destination path, entry point, and execution order;
- exact dependencies or lockfile/container digest;
- supported targets and the smaller set of targets actually tested;
- required inputs, rights, environment variables, and expected outputs;
- working directory, command, permissions, network, resources, and likely duration;
- cost, safety, destructive effects, cleanup, rollback, and common failures;
- validation assertions and links to run, result, or failure records.

Small blocks may share one record only when they are parts of one clearly ordered execution unit.

## Use one of four implementation labels

| Label | Meaning |
| --- | --- |
| `EXECUTED` | The exact revision ran on every target explicitly claimed; logs, assertions, hashes, and environment are linked |
| `REPRODUCED` | An independent party also reran it under the reproducibility standard |
| `STATICALLY REVIEWED` | A named reviewer inspected the declared aspects, but target runtime behavior is unknown |
| `UNVERIFIED IMPLEMENTATION` | Target execution is absent or incomplete; prerequisites and validation steps are supplied |

Documentation can verify an API exists, but `DOC-VERIFIED` is not a code execution label. Compilation alone does not
prove end-to-end behavior. If a reader must make manual edits before a run, incorporate and hash those edits before
claiming that the listed revision executed.

## A fully populated listing example

<a id="lst-governance-0001"></a>

### LST-GOVERNANCE-0001 — Verify a SHA-256 checksum

- Label: `ILLUSTRATIVE`; the example teaches listing structure and is not a project execution claim.
- Purpose: compare a downloaded file with a published SHA-256 value.
- Target: PowerShell 7 on Windows; no network access required.
- Input: `artifacts/model.gguf`.
- Expected result: command prints `Checksum verified` or exits with an error.
- Safety: read-only; verify the published checksum through a trusted release channel.
- Validation: replace neither path nor digest; this example intentionally uses a complete fictional artifact.

```powershell
$expected = "8f4f0f6f8e9dc7c4671d19568c70cf1b0f2d62782d7f48e9bb4e4e42f2fc7a31"
$actual = (Get-FileHash -Algorithm SHA256 "artifacts/model.gguf").Hash.ToLowerInvariant()
if ($actual -ne $expected) {
    throw "Checksum mismatch: expected $expected, received $actual"
}
Write-Output "Checksum verified"
```

The digest is explicitly fictional, so this is a teaching example. A real release listing would use the release
manifest's digest and link the executed run.

### Anti-example

```text
Run the training command with the right model and the usual options...
```

This omits the model, options, dependencies, cost, and expected result. It is neither executable nor useful pseudocode.

## Extract complete source from Markdown

Extractable listings follow the
[all-Markdown extraction convention](cross-reference-standard.md#all-markdown-extraction-convention). Use matching
start and end markers, one complete primary fenced block, and a recognized language. Shell dialects need separate
listings. Console output uses `console` and is never extracted as source.

Extracted content must be byte-identical to the fence except for a declared newline policy. Complete listings MUST NOT
contain unfilled angle-bracket tokens, omitted imports, ellipses, or “rest of code” comments. If deterministic
configuration generation is required, document and validate that generation step.

## Build safety into the example

Listings MUST pin fast-moving dependencies, validate untrusted input and checksums, avoid embedded secrets and personal
data, prefer safe model formats, use least privilege, and bound resources. They must fail loudly instead of silently
switching the model, device, backend, or precision.

Cloud examples need a cost boundary and cleanup command. Mobile examples need lifecycle, memory, threading, permission,
integrity, and offline assumptions. Safety-critical code also requires the
[expert gate](safety-and-risk-standard.md).

## Validation workflow

1. Parse and format.
2. Lint, type-check, and perform static security review.
3. Run unit tests.
4. Run integration tests.
5. Execute end to end on every claimed target.
6. Ask an independent person to reproduce it when required.

State which levels passed and link their output. Express expected behavior as assertions or bounded properties, not a
made-up console transcript. Any code change invalidates earlier execution evidence until hash equivalence or reexecution
is shown. Changes to dependencies, models, datasets, drivers, runtimes, operating systems, or hardware require a
compatibility review.

Before review, confirm complete files and imports, balanced fences and markers, version pins, command order,
input/output contracts, cost and safety warnings, accurate labels, hashes, and ledger links. This policy is
authoritative; existing listings keep their evidence-supported status until validation promotes them.
