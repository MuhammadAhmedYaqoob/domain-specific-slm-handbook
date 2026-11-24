# Secure offline installation and model updates

## 1. Threat model and contract

After an approved installation, weights, tokenizer, template, runtime, policy, and cache work without networking.
Missing or corrupt assets produce an explicit unavailable state, never hidden cloud inference. The updater defends
against tampering, truncation, path traversal, replay, incompatible artifacts, interrupted writes, and signing-key
compromise. Local encryption raises extraction cost but cannot provide perfect DRM against a device owner.

SHA-256 identifies exact bytes. A detached signature authenticates the manifest that contains those hashes. HTTPS
protects a connection but does not replace package signatures. Use a dedicated asymmetric release key, keep private
keys in controlled signing infrastructure, and embed an allow-list of public-key fingerprints in the app.

## 2. Worked storage example

Assume a 650 MB active package, 650 MB previous release, 300 MB partial download, 650 MB staged package, and 200 MB
margin. Peak update storage is \(650+650+300+650+200=2,450\) MB. This illustrative calculation excludes unpacking; if
the format expands, add the measured unpacked and temporary bytes before admission.

## 3. Signed manifest

Sign deterministic UTF-8 bytes or use a tested canonicalization such as RFC 8785. A concrete teaching manifest:

```json
{
  "schema": "offline-slm-release/v1",
  "releaseId": "mechanic-android-cpu-v1",
  "releaseSequence": 1,
  "channel": "stable",
  "artifactSet": "android-arm64-llamacpp-q4",
  "compatibility": {
    "platform": "android",
    "architectures": ["arm64-v8a"],
    "minimumOs": 28,
    "minimumAppBuild": 100,
    "runtimeRevision": "recorded-build-revision",
    "backend": "cpu",
    "contextTokensMaximum": 2048,
    "requiredRamBytes": 1675037245,
    "requiredFreeStorageBytes": 2450000000
  },
  "artifacts": [{
    "path": "model.gguf",
    "bytes": 650000000,
    "sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  }],
  "knowledgeCutoff": "2026-06-30",
  "licenseIds": ["project-reviewed-license"]
}
```

The digest of repeated `a` characters and revision/license labels are visibly illustrative. A production signer must
reject training values and populate them from actual artifacts, qualification reports, and legal review.

## 4. Build the installer

1. Fetch manifest bytes and detached signature.
2. Verify the signature before parsing trusted fields.
3. Parse with duplicate-key rejection; validate schema, lengths, file counts, and path rules.
4. Enforce channel, entitlement, sequence, app/OS/ABI/cohort, runtime, backend, context, RAM, and free-storage gates.
5. Create private random staging on the same filesystem as releases.
6. Download with a signed byte ceiling and strict HTTP range validation.
7. Reread every completed file and verify exact length and SHA-256.
8. Rename the complete directory into immutable releases and run isolated load/semantic smoke tests.
9. Atomically replace a small activation pointer and retain the predecessor through the health window.
10. Persist the accepted sequence and manifest digest; garbage-collect only unreferenced releases.

## 5. Resumable transfer algorithm

Persist release ID, artifact path/hash/length, immutable remote object identity, strong validator, staging identity, and
durable local length. On resume at byte \(n\), send `Range: bytes=n-` and `If-Range`. Append only when `206` begins at
\(n\) and reports the signed total. If the server returns `200`, truncate and restart. Treat `416` as complete only
when local length equals signed length, then full-rehash. Never treat an ETag as SHA-256.

```text
verify detached manifest signature
validate schema, compatibility, sequence, and storage
create private staging file with signed byte ceiling
repeat until signed length:
  request the exact immutable object from durable local offset
  validate status, Content-Range, validator, and total
  append no more than signed remaining bytes
  flush data and durable resume state
full-reread and verify SHA-256
move complete release within the same filesystem
run smoke tests and atomically replace activation pointer
```

Expected behavior after a disconnect is a safe resume or clean restart; the active model remains untouched.

## 6. Anti-rollback, key rotation, and platform storage

Reject lower release sequences. Permit automatic local rollback only to the exact predecessor named by the accepted
release and within its signed policy. To restore old artifact bytes later, publish a new higher-sequence manifest.
Clock time alone is not a monotonic trust anchor.

Support overlapping old/new public keys during rotation, then remove the old key through the app-signing channel. For
delegated roles or high assurance, adopt [The Update Framework](https://theupdateframework.github.io/) instead of
inventing more metadata.

On Android use private app storage such as `noBackupFilesDir`; on iOS use Application Support. Staging is never
loadable. Downloaded packages contain data, not executable frameworks or native code.

## 7. Common failures and validation

- Hash without signature: an attacker can replace both artifact and unsigned hash.
- Writing active bytes: interruption corrupts the only working release.
- Cross-volume rename: activation may not be atomic.
- Trusting resume metadata: stale offsets can splice two objects.
- Arbitrary rollback: a valid but vulnerable old package can be replayed.
- Logging signed URLs or prompts: credentials and private content leak.

Inject malformed and duplicate-key manifests, wrong keys, traversal paths, symlinks, oversized archives, disconnects at
every range, changed validators, bad `206`, low space, process kill at every journal state, concurrency, downgrade, and
missing predecessor. Finally reboot, deny DNS/routes, expire authorization, cold-load, generate, cancel, reset, and
inspect traffic. Network loss may change update availability, not local inference.

## Sources

- [The Update Framework specification](https://theupdateframework.github.io/specification/latest/)
- [HTTP range requests](https://www.rfc-editor.org/rfc/rfc9110.html#name-range-requests)
- [RFC 8785 JSON canonicalization](https://www.rfc-editor.org/rfc/rfc8785)
- [Android app-specific storage](https://developer.android.com/training/data-storage/app-specific)
- [Apple background transfers](https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background)
