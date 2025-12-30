# Phase 2 Artefact Model & Run Manifest v0.1

**Purpose**: Define immutable artefact bundles and provenance tracking for reproducible evaluations
**Version**: 0.1 (Phase 2)
**Date**: 2025-12-28

---

## Overview

Phase 2 introduces **artefact bundles** as the atomic unit of evaluation evidence. Every evaluation stage produces a versioned, immutable bundle with full provenance tracking.

### Design Goals
- **Immutability**: Artefacts never modified after creation (append-only)
- **Reproducibility**: Bundle manifest hash uniquely identifies scenario selection
- **Traceability**: Full provenance chain from scenario to report
- **Portability**: Artefacts self-describe dependencies (no external context needed)

---

## Artefact Types

### 1. Bundle Artefact
**Purpose**: Compiled scenario selection for a specific evaluation run
**Stage**: Scenario preparation
**Format**: Directory with manifest + scenarios

```
bundle_<client_id>_<timestamp>_<hash>/
├── bundle_manifest.json          # Provenance + scenario list
├── scenarios/
│   ├── GDPR-001.yaml             # Compiled scenario (base + overrides)
│   ├── GDPR-007.yaml
│   └── ...
└── checksums.sha256              # File integrity verification
```

### 2. Run Artefact
**Purpose**: Transcript evidence from target system execution
**Stage**: Evaluation run
**Format**: Directory with run manifest + transcripts (NO scores)

```
run_<bundle_hash>_<run_id>/
├── run_manifest.json             # Run metadata + target config
├── transcripts/
│   ├── GDPR-001_transcript.json  # Full conversation (no truncation)
│   ├── GDPR-007_transcript.json
│   └── ...
├── target_metadata.json          # Target system info (model, version, etc.)
└── checksums.sha256
```

### 3. Judgement Artefact
**Purpose**: Offline scoring results with locked judge
**Stage**: Offline judgement
**Format**: Directory with judge manifest + scores

```
judgement_<run_id>_<judge_version>/
├── judge_manifest.json           # Judge model + version lock
├── scores/
│   ├── GDPR-001_score.json       # behaviour_json_v1 format
│   ├── GDPR-007_score.json
│   └── ...
├── judge_provenance.json         # Judge model metadata + config
└── checksums.sha256
```

### 4. Translation Artefact
**Purpose**: Language transformation with source preservation
**Stage**: Pre-run or post-judgement (depending on workflow)
**Format**: Directory with translation manifest + bilingual scenarios

```
translation_<bundle_hash>_<lang_pair>/
├── translation_manifest.json     # Translation model + timestamp
├── scenarios_<target_lang>/
│   ├── GDPR-001_RO.yaml          # Translated scenario
│   ├── GDPR-001_EN.yaml          # Original (for provenance)
│   └── ...
└── checksums.sha256
```

---

## Run Manifest Schema

### Purpose
The **run manifest** is the single source of truth for an evaluation run's configuration and provenance.

### Schema (JSON)

```json
{
  "manifest_version": "0.1",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "created_at": "2025-12-28T14:30:00Z",

  "bundle": {
    "bundle_hash": "a3f5e8c2d1b4...",
    "bundle_manifest_path": "../bundle_AcmeHospital_20251228_a3f5e8c2/bundle_manifest.json",
    "scenario_count": 10,
    "client_id": "AcmeHospital",
    "catalogue_version": "v1.2.0",
    "git_commit": "abc123def456"
  },

  "target": {
    "type": "http",
    "base_url": "https://acme-hospital.example.com",
    "chat_path": "/api/v1/chat",
    "config_hash": "7b9d2e1f...",
    "config_file": "target_configs/acme_hospital_prod.json",
    "model_identifier": "gpt-4o-mini",
    "leak_mode": "strict"
  },

  "execution": {
    "executor_version": "aigov-eval-0.2.0",
    "started_at": "2025-12-28T14:30:00Z",
    "completed_at": "2025-12-28T14:45:00Z",
    "duration_seconds": 900,
    "scenarios_executed": 10,
    "scenarios_failed": 0,
    "transcript_format": "inspect_ai_v1"
  },

  "translation": {
    "applied": true,
    "source_lang": "EN",
    "target_lang": "RO",
    "translation_manifest_path": "../translation_a3f5e8c2_EN-RO/translation_manifest.json",
    "translation_model": "gpt-4o-2024-11-20",
    "translation_timestamp": "2025-12-28T14:00:00Z"
  },

  "provenance": {
    "scenario_repo": "https://github.com/Standivarius/AiGov-specs",
    "scenario_commit": "abc123def456",
    "eval_repo": "https://github.com/Standivarius/Aigov-eval",
    "eval_commit": "789fed654cba",
    "operator": "claude@aigov-runner",
    "environment": "production"
  },

  "artefacts": {
    "transcripts_dir": "./transcripts/",
    "target_metadata_file": "./target_metadata.json",
    "checksums_file": "./checksums.sha256"
  }
}
```

### Field Definitions

#### manifest_version
- **Type**: String
- **Format**: Semantic versioning (e.g., "0.1", "1.0")
- **Purpose**: Enables backward-compatible schema evolution

#### run_id
- **Type**: UUID v4
- **Purpose**: Unique identifier for this evaluation run
- **Uniqueness**: Generated at run start, never reused

#### bundle Section
- `bundle_hash`: SHA-256 hash of bundle_manifest.json (first 32 hex chars for readability)
- `bundle_manifest_path`: Relative path to bundle artefact
- `scenario_count`: Total scenarios in bundle (for quick validation)
- `client_id`: Client identifier (matches override namespace)
- `catalogue_version`: Scenario catalogue version tag
- `git_commit`: Git commit hash of scenario catalogue at bundle time

#### target Section
- `type`: Target type (http | mock | scripted)
- `config_hash`: SHA-256 of target config file (for reproducibility)
- `model_identifier`: Target LLM model name (if applicable)
- `leak_mode`: Target configuration (strict | leaky)

#### execution Section
- `executor_version`: Aigov-eval package version
- `started_at` / `completed_at`: ISO 8601 timestamps
- `scenarios_executed` / `scenarios_failed`: Execution stats
- `transcript_format`: Format version for transcript files

#### translation Section
- `applied`: Boolean (true if translation used)
- `source_lang` / `target_lang`: ISO 639-1 language codes
- `translation_manifest_path`: Relative path to translation artefact
- `translation_model`: LLM used for translation
- `translation_timestamp`: When translation was performed

#### provenance Section
- `scenario_repo` / `eval_repo`: Git repository URLs
- `scenario_commit` / `eval_commit`: Git commit hashes
- `operator`: User/service that initiated run
- `environment`: Execution environment (dev | staging | production)

---

## Bundle Manifest Schema

### Purpose
Defines the compiled scenario selection with provenance chain.

### Schema (JSON)

```json
{
  "manifest_version": "0.1",
  "bundle_hash": "a3f5e8c2d1b4a7f9e2c5d8b1f4a7e9c2",
  "created_at": "2025-12-28T13:00:00Z",

  "client": {
    "client_id": "AcmeHospital",
    "override_namespace": "client_configs/AcmeHospital/"
  },

  "catalogue": {
    "base_path": "scenarios/library/",
    "version": "v1.2.0",
    "git_commit": "abc123def456",
    "git_repo": "https://github.com/Standivarius/AiGov-specs"
  },

  "scenarios": [
    {
      "scenario_id": "GDPR-001",
      "source": "base",
      "file_path": "scenarios/GDPR-001.yaml",
      "checksum": "d4f2a8e1..."
    },
    {
      "scenario_id": "GDPR-007",
      "source": "override",
      "base_file": "scenarios/GDPR-007.yaml",
      "override_file": "client_configs/AcmeHospital/GDPR-007_override.yaml",
      "merged_checksum": "b3e9d1c4...",
      "override_reason": "Client-specific medical terminology"
    }
  ],

  "compiler": {
    "version": "bundle-compiler-0.1.0",
    "rules": {
      "conflict_resolution": "client_wins",
      "validation": "strict",
      "schema_version": "scenario_card_v1.2"
    }
  },

  "checksums": {
    "manifest_file": "checksums.sha256",
    "algorithm": "SHA-256"
  }
}
```

### Bundle Hash Calculation

```python
import hashlib
import json

def calculate_bundle_hash(manifest: dict) -> str:
    """Generate deterministic hash from bundle manifest."""

    # Extract deterministic fields (exclude timestamps, paths)
    stable_data = {
        "client_id": manifest["client"]["client_id"],
        "catalogue_commit": manifest["catalogue"]["git_commit"],
        "scenarios": [
            {
                "scenario_id": s["scenario_id"],
                "checksum": s.get("merged_checksum") or s["checksum"]
            }
            for s in sorted(manifest["scenarios"], key=lambda x: x["scenario_id"])
        ]
    }

    # Canonical JSON (sorted keys, no whitespace)
    canonical = json.dumps(stable_data, sort_keys=True, separators=(',', ':'))

    # SHA-256 hash (truncate to 32 chars for readability)
    return hashlib.sha256(canonical.encode()).hexdigest()[:32]
```

---

## Translation Manifest Schema

### Purpose
Preserve translation provenance and prevent double translation.

### Schema (JSON)

```json
{
  "manifest_version": "0.1",
  "translation_id": "550e8400-e29b-41d4-a716-446655440001",
  "created_at": "2025-12-28T14:00:00Z",

  "source": {
    "bundle_hash": "a3f5e8c2d1b4...",
    "language": "EN"
  },

  "target": {
    "language": "RO",
    "scenarios_dir": "./scenarios_RO/"
  },

  "translation_engine": {
    "model": "gpt-4o-2024-11-20",
    "temperature": 0.3,
    "system_prompt_hash": "f8e2d1c4..."
  },

  "translations": [
    {
      "scenario_id": "GDPR-001",
      "source_file": "../bundle_AcmeHospital_20251228_a3f5e8c2/scenarios/GDPR-001.yaml",
      "source_checksum": "d4f2a8e1...",
      "translated_file": "./scenarios_RO/GDPR-001_RO.yaml",
      "translated_checksum": "c3e8d2f1...",
      "translated_at": "2025-12-28T14:01:30Z",
      "word_count": 150,
      "already_translated": false
    }
  ],

  "validation": {
    "schema_check": "passed",
    "field_preservation": "passed",
    "placeholder_check": "passed"
  }
}
```

### No Double Translation Rule

```python
def check_double_translation(scenario: dict) -> bool:
    """
    Prevent re-translating already translated content.

    Returns True if scenario contains translation metadata.
    """
    if "translation_metadata" in scenario:
        return True

    # Check for common translation artifacts
    if "original_language" in scenario.get("metadata", {}):
        return True

    return False

def translate_scenario(scenario_path: str, target_lang: str) -> dict:
    """Translate scenario with source preservation."""

    scenario = load_scenario(scenario_path)

    if check_double_translation(scenario):
        raise ValueError(
            f"Scenario {scenario['scenario_id']} already translated. "
            "Use original source from translation manifest."
        )

    # Perform translation...
    translated = translate_content(scenario, target_lang)

    # Preserve source
    translated["translation_metadata"] = {
        "original_language": scenario.get("language", "EN"),
        "original_file": scenario_path,
        "translated_language": target_lang,
        "translation_model": "gpt-4o-2024-11-20",
        "translation_timestamp": datetime.utcnow().isoformat()
    }

    return translated
```

---

## Judge Manifest Schema

### Purpose
Lock judge model version for reproducible scoring.

### Schema (JSON)

```json
{
  "manifest_version": "0.1",
  "judgement_id": "550e8400-e29b-41d4-a716-446655440002",
  "created_at": "2025-12-28T15:00:00Z",

  "run": {
    "run_id": "550e8400-e29b-41d4-a716-446655440000",
    "run_manifest_path": "../run_a3f5e8c2_550e8400/run_manifest.json"
  },

  "judge": {
    "contract_version": "judge_contract_v0.1",
    "model": "google/gemini-2.0-flash-exp",
    "model_version_lock": "gemini-2.0-flash-exp-2025-12-15",
    "temperature": 0.0,
    "system_prompt_hash": "a8d2e1f3..."
  },

  "execution": {
    "mode": "offline_batch",
    "started_at": "2025-12-28T15:00:00Z",
    "completed_at": "2025-12-28T15:10:00Z",
    "scenarios_judged": 10,
    "failures": 0,
    "fail_closed": true
  },

  "scores": [
    {
      "scenario_id": "GDPR-001",
      "transcript_file": "../run_a3f5e8c2_550e8400/transcripts/GDPR-001_transcript.json",
      "score_file": "./scores/GDPR-001_score.json",
      "judged_at": "2025-12-28T15:02:00Z",
      "rating": "VIOLATED",
      "confidence": 0.95
    }
  ]
}
```

### Fail-Closed Judge Lock

```python
class LockedJudge:
    """Fail-closed judge with version lock."""

    def __init__(self, judge_manifest: dict):
        self.manifest = judge_manifest
        self.model_lock = judge_manifest["judge"]["model_version_lock"]

    def validate_model_availability(self):
        """Fail if locked model unavailable."""

        available_models = get_available_models()

        if self.model_lock not in available_models:
            raise JudgeModelUnavailableError(
                f"Locked judge model '{self.model_lock}' not available. "
                f"Available models: {available_models}. "
                "Judgement aborted (fail-closed)."
            )

    def judge_batch(self, run_artefact_path: str) -> dict:
        """Process batch of transcripts with locked judge."""

        # Fail-closed: validate model before starting
        self.validate_model_availability()

        run_manifest = load_run_manifest(run_artefact_path)
        transcripts = load_transcripts(run_manifest)

        scores = []
        for transcript in transcripts:
            try:
                score = self.judge_single(transcript)
                scores.append(score)
            except Exception as e:
                # Fail-closed: don't skip failures
                raise JudgementFailedError(
                    f"Judgement failed for {transcript['scenario_id']}: {e}. "
                    "Batch processing aborted (fail-closed)."
                )

        return self.create_judgement_artefact(scores)
```

---

## Artefact Storage Structure

```
artefacts/
├── bundles/
│   ├── bundle_AcmeHospital_20251228_a3f5e8c2/
│   │   ├── bundle_manifest.json
│   │   ├── scenarios/
│   │   └── checksums.sha256
│   └── ...
│
├── runs/
│   ├── run_a3f5e8c2_550e8400/
│   │   ├── run_manifest.json
│   │   ├── transcripts/
│   │   ├── target_metadata.json
│   │   └── checksums.sha256
│   └── ...
│
├── translations/
│   ├── translation_a3f5e8c2_EN-RO/
│   │   ├── translation_manifest.json
│   │   ├── scenarios_RO/
│   │   └── checksums.sha256
│   └── ...
│
└── judgements/
    ├── judgement_550e8400_gemini2.0/
    │   ├── judge_manifest.json
    │   ├── scores/
    │   ├── judge_provenance.json
    │   └── checksums.sha256
    └── ...
```

---

## Validation Requirements

### Artefact Integrity
All artefacts MUST include `checksums.sha256`:
```
d4f2a8e1...  bundle_manifest.json
c3e8d2f1...  scenarios/GDPR-001.yaml
...
```

### Manifest Chain Validation
```python
def validate_provenance_chain(judgement_artefact_path: str):
    """Verify full provenance chain from scenario to judgement."""

    # Load judgement manifest
    judge_manifest = load_manifest(
        f"{judgement_artefact_path}/judge_manifest.json"
    )

    # Follow chain backwards
    run_manifest = load_manifest(judge_manifest["run"]["run_manifest_path"])
    bundle_manifest = load_manifest(run_manifest["bundle"]["bundle_manifest_path"])

    # Verify checksums at each stage
    verify_checksums(judgement_artefact_path)
    verify_checksums(run_manifest["artefacts"]["transcripts_dir"])
    verify_checksums(bundle_manifest["checksums"]["manifest_file"])

    # Verify hash consistency
    assert run_manifest["bundle"]["bundle_hash"] == bundle_manifest["bundle_hash"]

    return {
        "valid": True,
        "scenario_commit": bundle_manifest["catalogue"]["git_commit"],
        "eval_commit": run_manifest["provenance"]["eval_commit"],
        "judge_version": judge_manifest["judge"]["model_version_lock"]
    }
```

---

## Related Documents

- [Phase 2 system overview](./phase2-system-overview-v0.1.md)
- [Scenario catalogue + bundle compiler spec](./phase2-scenario-bundle-v0.1.md)
- [Offline judgement spec](./phase2-offline-judgement-v0.1.md)
- [Translation spec](./phase2-translation-v0.1.md)
- [Data contracts v0.1](./data-contracts-v0.1.md)

---

**Document Status**: v0.1 Draft Specification
**Last Updated**: 2025-12-28
**Next Review**: After artefact model implementation
