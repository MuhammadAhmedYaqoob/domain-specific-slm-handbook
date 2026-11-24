# Domain configuration contract

## Lesson: configuration is the agreement between teams

A domain contract answers five practical questions before expensive work starts:

1. What may the model do, and what must it refuse?
2. Which exact data and base artifacts may be used?
3. Which training and compression recipes are allowed?
4. What evidence is required for each cloud or edge target?
5. Who owns incidents, rollback, expiry, and retirement?

“Fail closed” means that an unknown value blocks promotion instead of being guessed. A “digest” is a SHA-256 fingerprint
of exact bytes. “Protected suite” means final test questions and answers that training, model selection, and threshold
tuning cannot read. These terms are controls, not shorthand for “someone will review it later.”

The example below is fully populated so it can be parsed as YAML. Its URIs, identities, hashes, and thresholds are
synthetic teaching data; syntactic completeness is not production approval.

## Configuration flow

```mermaid
flowchart LR
  A["Author YAML"] --> V["Schema and cross-field validation"]
  V --> C["Canonical JSON"]
  C --> H["Compute digest"]
  H --> P["Rights, safety, and owner approvals"]
  P --> R["Immutable contract registry"]
  R --> J["Training and release jobs"]
```

For example, setting `android_arm64` requires a format, runtime revision, device cohort, signed package, peak-update
storage policy, and separate Android evidence. A global statement such as “the model passed” is insufficient.

## Status and resolution

**NORMATIVE CONTRACT / `UNVERIFIED IMPLEMENTATION`.** YAML is an authoring format. Production MUST validate an
equivalent JSON Schema 2020-12 (or stricter typed contract), resolve defaults, canonicalize, hash, approve, and store
the result before execution.

Resolution order is:

```text
platform defaults → risk profile → domain declaration → approved target override
```

Unknown fields, unresolved authoring tokens, mutable artifact revisions, unauthorized roles, expired evidence, and invalid
cross-field combinations MUST fail. Configuration MUST NOT import or execute code.

## Required top-level sections

<!-- markdownlint-disable MD013 -->
| Section          | Required content                                                                                    |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| `domain`         | stable ID, owners, users, languages, jurisdictions, intended/prohibited uses, knowledge cutoff      |
| `risk`           | highest consequence tier, hazards, required/prohibited behavior, escalation, incident owner         |
| `rights`         | acquisition/training/derivative/distribution rights, privacy/security review, expiry and revocation |
| `sources`        | immutable source manifest, authority classes, forbidden classes, freshness triggers                 |
| `data`           | schema, taxonomy, objectives, split families, leakage/PII/quality/coverage gates, synthetic policy  |
| `model`          | approved catalog policy, exact base/tokenizer candidates, license and parameter constraints         |
| `training`       | named allowlisted stages, immutable data/code/container, seeds, budget, stop/resume policy          |
| `evaluation`     | protected suite, tracks, metrics, uncertainty, subgroup and critical gates, reviewers               |
| `compression`    | candidate branches, calibration identity, parent-paired regression, safety reruns                   |
| `deployment`     | independent cloud/edge target tuples, compatibility, canary, rollback predecessor                   |
| `monitoring`     | operational/behavior/knowledge/edge signals, privacy, triggers and owners                           |
| `retirement`     | review/expiry, ownerlessness, archive, distribution stop, offline-device policy                     |
| `no_runtime_rag` | required prohibition and deterministic-tool allowlist                                               |
| `approvals`      | named roles, separation rules, evidence expiry, exceptions                                          |
<!-- markdownlint-restore -->

## Cross-field policy

The validator MUST enforce at least:

1. high/critical domains require qualified safety reviewers, zero permitted predefined critical failures, escalation
   behavior, incident ownership, and shorter evidence expiry;
2. unknown or expired rights records equal zero; every dataset record resolves to a source and rights decision;
3. evaluation family overlap equals zero; protected final suites cannot be inputs to training, teachers, calibration,
   compression, prompts, judges, checkpoint selection, or thresholds;
4. every model, tokenizer, dataset, code revision, container, calibration set, runtime, and package is immutable;
5. every deployment target has an exact runtime/format/cohort contract and its own qualification evidence;
6. edge targets require signed packages, peak-update storage policy, device matrix, offline acceptance, and
   previous-package retention;
7. adapter stages bind exact base/config/tokenizer/target modules; merge stages bind adapter order, dtype, tool, and
   parity evidence;
8. a joint/merged model with restrictive domain rights cannot claim per-domain entitlement or reliable removal;
9. monitoring cannot enable raw prompt logging without a separately approved, scoped, expiring privacy exception;
10. no exception can waive unknown rights, artifact integrity, protected-suite leakage, runtime-RAG prohibition, or
    observed critical safety failure.

## Complete domain declaration

**Listing MDA-DC-01 — Industrial pump maintenance example (`UNVERIFIED IMPLEMENTATION`).** Values and thresholds are
illustrative policy, not measured fitness.

```yaml
schema_version: "domain-contract/1.0"
domain:
  id: industrial_pump_maintenance
  owner: role:domain-owner
  lifecycle_owner: role:lifecycle-owner
  users: [supervised_maintenance_technicians]
  languages: [en]
  jurisdictions: [US]
  intended_uses: [education, non_authoritative_diagnostic_sequencing]
  prohibited_uses: [guard_bypass, live_work_authorization, return_to_service]
  knowledge_cutoff: "2026-10-01"

risk:
  highest_tier: R4
  profile: safety_critical_equipment_v1
  hazards:
    - id: stored_energy
      triggers: [unknown_isolation, pressure_present, raised_load]
      required: [stop, follow_site_energy_control, qualified_person]
      prohibited: [assume_stopped_is_isolated, bypass_guard]
      reviewer: role:industrial-safety-reviewer
  uncertainty_behavior: [request_asset_state, separate_hypothesis_from_cause, escalate]
  incident_owner: role:incident-commander

rights:
  policy_version: rights-2.1
  required: [acquire, process, train, create_model_derivatives, distribute_package]
  unknown_rights_records_max: 0
  expired_rights_records_max: 0
  privacy_review: required
  security_sensitive_review: required

sources:
  manifest:
    uri: "artifact://sources/pump/v1.json"
    sha256: "46d2f4f04ddf0998da08bc59ddaa7fe06463a7074609360ae32c077b4942d40f"
  authority_allowlist: [manufacturer_procedure, regulator_or_standard, professional_reference]
  forbidden: [unknown_license, access_control_bypass]
  freshness:
    review_days: 90
    triggers: [manufacturer_revision, safety_bulletin, regulation_change]

data:
  schema: slm-record/2.0
  taxonomy:
    uri: "artifact://taxonomy/pump/v1.json"
    sha256: "ec720eff1a4202268f8f228bb493fcb2c3b819c159bc2bf953bdc95d87ed9a05"
  objectives: { cpt: false, sft: true, preference: false, distillation: candidate }
  split:
    strategy: stable_group_hash
    group_keys: [source_family_id, case_family_id]
    seed: 41017
  gates:
    evaluation_family_overlap_max: 0
    unresolved_critical_pii_max: 0
    unresolved_critical_contradictions_max: 0
    taxonomy_coverage_min: 0.90
  synthetic:
    permitted: true
    max_fraction: 0.30
    grounding_required: true
    expert_review_r3_r4: true

model:
  catalog_policy: approved-open-weight-v3
  candidates:
    - id: "registry://base/example-decoder-2b"
      revision: "8e84b665d87b4a7eb2d6af445f654d892a7a130c"
      weights_sha256: "60ed4854b05751634c8d707c7d0c881cb82cc850935a4a27d6505343574e7b16"
      config_sha256: "1679c31b8416e27db9db0e8f8350166c29aa3ab50f8316d604690f7fa4b48c9e"
      tokenizer_sha256: "519f8ebf9e85fa9e8e4f18bdaaa4b57754750e6ea2cb72c0bb36f5057fa4dc9f"
      license_id: "LicenseRef-Reviewed-Pump-Training-2026"
  parameter_range: { min: 500000000, max: 4000000000 }
  baseline_required: true

training:
  dataset:
    uri: "artifact://datasets/pump/v1.json"
    sha256: "84f62d76501f9eb328d8d67f67e0ff3b1581b51733106ac317711743d1f80629"
  stages:
    - { name: baseline, runner: eval.frozen/2.0 }
    - name: sft_lora
      runner: train.sft_lora/4.0
      config: { rank: 32, alpha: 64, dropout: 0.05, learning_rate: 0.0001, epochs_max: 3 }
  seeds: [17, 29, 41]
  code_revision: "d7ee11dc8c6bf20d8578a539ba30d69096cfe0a2"
  container_digest: "sha256:90ec71865f737b2c4a5f955e32bc56f8884d54413d8a56bdbfc8ec0cd93c7a28"
  budget: { max_gpu_hours: 120 }
  checkpoint: { interval_steps: 500, resumable_state_required: true }

evaluation:
  profile:
    uri: "artifact://eval/pump/v1.yaml"
    sha256: "a4fbf5496fb1a8a2acb7c9403deab6728391636917157e38e8302d03dedbcc50"
  tracks: [general, safety_critical]
  suites: [domain, stored_energy, uncertainty, adversarial, ood, multi_turn, general_retention]
  final_custodian: role:evaluation-custodian
  model_judge_can_approve_safety: false
  gates:
    critical_failures_max: 0
    lower_bounds: { domain: 0.80, uncertainty: 0.90 }
    general_retention_relative_drop_max: 0.05
  threshold_status: proposed_policy

compression:
  candidates:
    - { id: int8_server, runner: compress.int8/2.0, target: aws_realtime }
    - { id: int4_edge, runner: compress.int4/3.0, target: android_arm64 }
  calibration:
    uri: "artifact://calibration/pump/v1"
    sha256: "265b322e80f51b2ae35c254972e68442cf35ab808d57bcf877c4b976363f76ed"
  parent_paired_regression: true
  rerun_all_safety_suites: true

deployment:
  targets:
    - id: aws_realtime
      runtime: "server-runtime@4.2.1+sha.73f0c8e"
      canary: { percent: 5, minutes: 60 }
      rollback_release: "registry://release/pump/0.9.0"
    - id: android_arm64
      runtime: "llama.cpp@b6820"
      format: "GGUFv3-Q4_K_M"
      device_matrix: pump-android-v1
      signed_package: true
      retain_previous_package: true
  package: { sbom: required, signatures: required, license_bundle: required }

monitoring:
  raw_prompt_logging: false
  operational_profile: interactive-v1
  safety_probes: pump-regression-v1
  source_review_days: 90
  expert_sample_days: 30
  triggers:
    critical_incident: suspend_and_review
    rights_revoked: suspend_and_trace
    safety_probe_failure: rollback_or_suspend
    target_crash_regression: rollback_target

retirement:
  review_date: "2027-05-23"
  max_unowned_days: 0
  archive_retention_days: 2555
  disable_new_installs: true
  offline_policy: visible_expiry_and_safe_degraded_mode

no_runtime_rag:
  required: true
  prohibited:
    [
      vector_database,
      embedding_search,
      dynamic_document_retrieval,
      retrieval_prompt_augmentation,
      remote_teacher_fallback,
    ]
  runtime_allowlist: [tokenizer, inference_runtime, deterministic_unit_converter]

approvals:
  required:
    [
      role:domain-owner,
      role:data-steward,
      role:rights-reviewer,
      role:safety-reviewer,
      role:evaluation-custodian,
      role:ml-reviewer,
      role:target-owner,
      role:release-authority,
    ]
  exceptions: []
```

Production MUST reject unresolved authoring tokens and MUST NOT interpret these teaching thresholds as approved policy.

## Failure scenarios to test

- **Rights expiry:** a source license expires before release. Validation blocks the dataset and traces every descendant.
- **Evaluation leakage:** a case family appears in both training and final evaluation. The zero-overlap gate fails.
- **Adapter mismatch:** the configured tokenizer digest differs from the adapter's parent tuple. Training or loading
  stops before execution.
- **Target ambiguity:** Android names a runtime but no exact model format. The target is incomplete and cannot release.
- **Owner loss:** the lifecycle owner leaves and no replacement is assigned. `max_unowned_days: 0` suspends promotion.

## Step-by-step design exercise

1. Copy the pump contract and change the domain to educational pharmacy.
2. Narrow intended use to general education; prohibit diagnosis, prescribing, dose changes, and emergency clearance.
3. Add interaction-risk and uncertainty suites with zero allowed predefined critical failures.
4. Use a shorter source-review interval for label, recall, and regulation changes.
5. Decide whether pharmacy can share the proposed base. Explain licensing, tokenizer, and isolation consequences.
6. Add AWS and Android target tuples, each with independent evidence.
7. Canonicalize the contract, compute its digest, then try changing one threshold. Confirm that it creates a new
   contract identity.

## Validation result contract

Validation MUST emit a deterministic report with contract digest, policy digest, canonical configuration digest,
findings (`rule_id`, JSON Pointer, severity, message), resolved roles/artifacts, expiry, and `pass|fail|review`.
Warnings MUST NOT downgrade a hard failure.

Example mandatory rule IDs:

| Rule         | Condition                                                          |
| ------------ | ------------------------------------------------------------------ |
| `ID-001`     | stable lowercase domain ID and non-reused scope                    |
| `RIGHTS-001` | zero unknown/expired rights and complete descendant policy         |
| `RISK-001`   | R3/R4 reviewer, escalation, incident owner, zero critical failures |
| `EVAL-001`   | protected suite identity and no family overlap                     |
| `MODEL-001`  | immutable model/config/tokenizer/license                           |
| `TARGET-001` | exact runtime/format/cohort and independent qualification          |
| `ARCH-001`   | complete no-runtime-RAG prohibition and allowlisted tools          |
| `LIFE-001`   | owner, review date, rollback and retirement                        |

## Onboarding and change control

Gate sequence is
<!-- markdownlint-disable MD013 -->

`G0 intake → G1 risk/rights → G2 source/data → G3 evaluation → G4 model/training → G5 compression/target → G6 release → G7 operate/retire`.

<!-- markdownlint-restore -->
Each gate records policy, evidence digests, decision, actor, timestamp, expiry, and exception.

A contract change creates a new immutable version. Major changes include incompatible intended use, safety authority,
tokenizer/prompt/output contract, runtime API, rights boundary, or target compatibility. Behavioral retraining, new
data, or changed thresholds requires at least a new minor release and full affected-domain regression. Editorial
metadata MAY be patch-level.

A plug-in is allowed only for a genuinely new record type, evaluator, safety rule, or runtime. It MUST declare typed
interfaces, permissions, deterministic or stochastic behavior, retry safety, threat model, fixtures, migrations, owner,
and failure tests.

## Cross-links

- [Architecture index](README.md)
- [Platform components](platform-components.md)
- [Strategy comparison](strategy-comparison.md)
- [No-runtime-RAG](no-runtime-rag-architecture.md)
- [CH-41 onboarding](../02-textbook/part-11-extending-the-platform-to-other-domains/ch-41-new-domain-onboarding.md)
- [Dataset guide](../05-dataset-guide/README.md)

## Honest status

The domain schema, complete example, policy constraints, validation result, gate sequence, change rules, and plug-in
boundary are authored. No schema, validator, declaration, or gate has been executed or approved.
