# Phase 2 Offline Judgement Specification v0.1

**Purpose**: Define locked judge governance, fail-closed batch processing, and replay semantics
**Version**: 0.1 (Phase 2)
**Date**: 2025-12-28

---

## Overview

Phase 2 separates **evidence capture** (run stage) from **scoring** (judgement stage). This enables:
- **Cost optimization**: Batch judgement is cheaper than per-run inference
- **Reproducibility**: Re-judge old transcripts with updated judge models
- **Fail-closed safety**: Run failures don't cascade to judgement
- **Audit compliance**: Clear separation between evidence and interpretation

---

## Architecture

### Two-Stage Workflow

```
STAGE 1: RUN (Evidence Capture)
┌────────────────────────────────────┐
│ Scenario Bundle                    │
│      ↓                             │
│ [Run Executor]                     │
│      ↓                             │
│ Run Artefact                       │
│ - Transcripts (full, no truncation)│
│ - Run manifest                     │
│ - Target metadata                  │
│ - NO SCORES                        │
└────────────────────────────────────┘

STAGE 2: JUDGE (Offline Scoring)
┌────────────────────────────────────┐
│ Run Artefact (batch)               │
│      ↓                             │
│ [Locked Judge]                     │
│      ↓                             │
│ Judgement Artefact                 │
│ - Scores (behaviour_json_v1)       │
│ - Judge manifest                   │
│ - Judge provenance                 │
└────────────────────────────────────┘
```

### Stage Independence

**Run stage**:
- Executes scenarios against target system
- Captures transcripts only (no scoring logic)
- Can complete successfully even if judge unavailable
- Outputs immutable evidence bundle

**Judgement stage**:
- Runs offline (batch processing)
- Loads transcripts from run artefacts
- Applies locked judge model
- Fails closed if judge unavailable
- Outputs separate judgement artefacts

---

## Judge Contract v0.1

### Judge Model Lock

Judges are **locked at specification time** via judge manifest:

```yaml
# judge_config.yaml (locked judge specification)
judge_version: "0.1"
model_lock:
  provider: "google"
  model: "gemini-2.0-flash-exp"
  version_lock: "gemini-2.0-flash-exp-2025-12-15"  # Exact model version
  fallback: null                                    # NO fallback (fail-closed)

parameters:
  temperature: 0.0                                  # Deterministic scoring
  max_tokens: 4096
  timeout_seconds: 30

system_prompt:
  file: "prompts/judge_system_prompt_v0.1.md"
  hash: "a8d2e1f3c4b5..."                          # Prompt version lock

fail_closed: true                                   # Fail if judge unavailable
retry_policy:
  max_retries: 3
  backoff: "exponential"
  retry_on: ["timeout", "rate_limit"]
  fail_on: ["model_unavailable", "invalid_response"]
```

### Fail-Closed Semantics

**Principle**: Never silently degrade to inferior judge

```python
class FailClosedJudge:
    """Judge that fails explicitly if model unavailable."""

    def __init__(self, judge_config: dict):
        self.config = judge_config
        self.model_lock = judge_config["model_lock"]["version_lock"]
        self.fail_closed = judge_config["fail_closed"]

    def validate_availability(self):
        """Check judge model availability before batch processing."""

        available = check_model_availability(self.model_lock)

        if not available:
            if self.fail_closed:
                raise JudgeUnavailableError(
                    f"Locked judge '{self.model_lock}' unavailable. "
                    "Fail-closed policy: judgement aborted. "
                    "Options: (1) Wait for model availability, "
                    "(2) Update judge config to different model version."
                )
            else:
                # Only if fail_closed: false (NOT RECOMMENDED)
                self.logger.warning(
                    f"Judge '{self.model_lock}' unavailable, using fallback"
                )
                self.model_lock = self.config["model_lock"]["fallback"]

    def judge_batch(self, run_artefacts: list[str]) -> dict:
        """Process batch of run artefacts with fail-closed semantics."""

        # Pre-flight check (fail before processing if model unavailable)
        self.validate_availability()

        results = []
        for artefact_path in run_artefacts:
            try:
                result = self.judge_single_run(artefact_path)
                results.append(result)
            except JudgeError as e:
                if self.fail_closed:
                    # Abort batch on first failure
                    raise BatchJudgementFailedError(
                        f"Judgement failed for {artefact_path}: {e}. "
                        "Fail-closed: batch aborted. "
                        f"Processed {len(results)}/{len(run_artefacts)} runs."
                    )
                else:
                    # Log error, continue (NOT RECOMMENDED)
                    self.logger.error(f"Judgement failed: {e}")
                    results.append({"error": str(e)})

        return self.create_batch_judgement_artefact(results)
```

---

## Batch Processing

### Batch Judgement Workflow

```bash
# 1. Collect run artefacts for judgement
ls runs/
# run_a3f5e8c2_550e8400/
# run_a3f5e8c2_550e8401/
# run_a3f5e8c2_550e8402/

# 2. Run offline judgement (batch mode)
python -m aigov_eval.judge batch \
  --run-artefacts runs/run_a3f5e8c2_*/run_manifest.json \
  --judge-config judge_configs/gemini-2.0-flash_v0.1.yaml \
  --output judgements/judgement_batch_20251228/ \
  --fail-closed true

# Output:
# ✓ Loaded 3 run artefacts
# ✓ Judge model 'gemini-2.0-flash-exp-2025-12-15' available
# ✓ Processing batch (fail-closed mode)...
#   [1/3] Judging run_a3f5e8c2_550e8400... VIOLATED (0.95 confidence)
#   [2/3] Judging run_a3f5e8c2_550e8401... COMPLIANT (0.92 confidence)
#   [3/3] Judging run_a3f5e8c2_550e8402... VIOLATED (0.88 confidence)
# ✓ Batch complete: 3/3 judged, 0 errors
# ✓ Judgement artefact: judgements/judgement_batch_20251228/judge_manifest.json
```

### Batch Optimization

**Cost savings** from batch processing:
- Shared system prompt (sent once, not per-scenario)
- Parallel API calls (within rate limits)
- Efficient token usage (no redundant context)

```python
class BatchJudge:
    """Optimized batch judgement with parallel processing."""

    def judge_batch_parallel(
        self,
        run_artefacts: list[str],
        max_parallel: int = 5
    ) -> dict:
        """Process batch with controlled parallelism."""

        import asyncio

        async def judge_single_async(artefact_path: str) -> dict:
            transcript = load_transcript(artefact_path)
            scenario = load_scenario_from_run_manifest(artefact_path)
            return await self.judge_async(transcript, scenario)

        async def process_batch():
            # Semaphore to limit parallel requests (respect rate limits)
            semaphore = asyncio.Semaphore(max_parallel)

            async def bounded_judge(artefact_path: str):
                async with semaphore:
                    return await judge_single_async(artefact_path)

            tasks = [bounded_judge(path) for path in run_artefacts]
            return await asyncio.gather(*tasks)

        # Run async batch
        results = asyncio.run(process_batch())

        return self.create_batch_judgement_artefact(results)
```

---

## Replay Semantics

### What is Replay?

**Replay** = Re-judging old transcripts with a different (usually newer) judge model.

**Use cases**:
1. **Judge model upgrade**: Re-score historical runs with improved judge
2. **Calibration**: Compare old vs new judge on same evidence
3. **Audit review**: Third-party review of judgements with independent judge

### Replay Workflow

```bash
# Original judgement (2025-12-28, judge v0.1)
python -m aigov_eval.judge batch \
  --run-artefacts runs/run_a3f5e8c2_*/run_manifest.json \
  --judge-config judge_configs/gemini-2.0-flash_v0.1.yaml \
  --output judgements/judgement_original/

# Replay with upgraded judge (2026-01-15, judge v0.2)
python -m aigov_eval.judge replay \
  --run-artefacts runs/run_a3f5e8c2_*/run_manifest.json \
  --judge-config judge_configs/gemini-2.5-flash_v0.2.yaml \
  --output judgements/judgement_replay_v0.2/ \
  --original-judgement judgements/judgement_original/judge_manifest.json

# Output:
# ✓ Replaying 3 run artefacts with new judge
# ✓ Original judge: gemini-2.0-flash-exp-2025-12-15 (v0.1)
# ✓ Replay judge: gemini-2.5-flash-exp-2026-01-10 (v0.2)
# ✓ Comparison report: judgements/judgement_replay_v0.2/comparison.json
```

### Replay Comparison Report

```json
{
  "replay_id": "replay_20260115_v0.2",
  "original_judgement": {
    "judge_version": "0.1",
    "model": "gemini-2.0-flash-exp-2025-12-15",
    "judgement_path": "judgements/judgement_original/"
  },
  "replay_judgement": {
    "judge_version": "0.2",
    "model": "gemini-2.5-flash-exp-2026-01-10",
    "judgement_path": "judgements/judgement_replay_v0.2/"
  },

  "comparison": [
    {
      "scenario_id": "GDPR-001",
      "run_id": "550e8400-e29b-41d4-a716-446655440000",
      "original_rating": "VIOLATED",
      "original_confidence": 0.95,
      "replay_rating": "VIOLATED",
      "replay_confidence": 0.98,
      "rating_changed": false,
      "confidence_delta": 0.03,
      "notes": "Consistent violation detection, higher confidence with v0.2"
    },
    {
      "scenario_id": "GDPR-007",
      "run_id": "550e8400-e29b-41d4-a716-446655440001",
      "original_rating": "COMPLIANT",
      "original_confidence": 0.92,
      "replay_rating": "VIOLATED",
      "replay_confidence": 0.85,
      "rating_changed": true,
      "confidence_delta": -0.07,
      "notes": "RATING CHANGED: v0.2 detected violation missed by v0.1 (edge case in special category detection)"
    }
  ],

  "summary": {
    "total_runs": 3,
    "rating_changes": 1,
    "rating_change_rate": 0.33,
    "avg_confidence_delta": 0.01,
    "recommendation": "REVIEW: 1 rating change detected. Manual review recommended for GDPR-007."
  }
}
```

### Replay Governance

**When to replay**:
- Judge model upgrade (v0.1 → v0.2)
- New vulnerability pattern discovered (update judge prompt)
- Third-party audit review request
- Client disputes original judgement

**When NOT to replay**:
- Scenario changes (breaks provenance chain)
- Target system changes (evidence is different)
- Just for curiosity (waste of API costs)

**Replay audit trail**:
```json
{
  "replay_history": [
    {
      "replay_id": "replay_20260115_v0.2",
      "replayed_at": "2026-01-15T10:00:00Z",
      "reason": "Judge model upgrade to v0.2",
      "operator": "aigov_qa_team",
      "rating_changes": 1,
      "approved_by": "aigov_compliance_lead"
    }
  ]
}
```

---

## Judge Versioning

### Version Strategy

**Judge version format**: `v{major}.{minor}`

**Breaking changes** (major version bump):
- Change in rating semantics (e.g., add new rating category)
- Change in evidence structure
- Change in scoring algorithm (regression risk)

**Non-breaking changes** (minor version bump):
- Improved prompt wording (no semantic change)
- Bug fixes (e.g., fix edge case false negative)
- Performance optimization (same results, faster)

### Version Lock Enforcement

```python
def enforce_version_lock(judge_config: dict, run_artefact_path: str):
    """
    Prevent accidental judge version mismatch.

    Checks:
    1. Judge version matches config
    2. Judge version compatible with run's scenario bundle
    3. No silent fallback to different version
    """

    run_manifest = load_run_manifest(run_artefact_path)
    judge_version = judge_config["judge_version"]

    # Check 1: Version lock matches
    if judge_config["model_lock"]["version_lock"] != get_current_model_version():
        raise VersionMismatchError(
            f"Judge version lock '{judge_config['model_lock']['version_lock']}' "
            f"does not match current model version '{get_current_model_version()}'"
        )

    # Check 2: Judge version compatible with scenario bundle
    bundle_version = run_manifest["bundle"]["catalogue_version"]
    if not is_compatible(judge_version, bundle_version):
        raise IncompatibleVersionError(
            f"Judge v{judge_version} incompatible with "
            f"scenario catalogue {bundle_version}. "
            "See compatibility matrix: docs/judge-scenario-compatibility.md"
        )

    # Check 3: Prompt hash matches (prevent accidental prompt changes)
    prompt_hash = hash_file(judge_config["system_prompt"]["file"])
    if prompt_hash != judge_config["system_prompt"]["hash"]:
        raise PromptMismatchError(
            f"Judge prompt file modified (hash mismatch). "
            f"Expected: {judge_config['system_prompt']['hash']}, "
            f"Got: {prompt_hash}. "
            "Update judge_config.yaml with new prompt hash."
        )
```

---

## Error Handling

### Judge Errors

**Transient errors** (retry with backoff):
- Timeout
- Rate limit
- Network error
- Model temporarily unavailable

**Permanent errors** (fail-closed, no retry):
- Model not found (version unavailable)
- Invalid response format
- Schema validation failure
- Authentication failure

```python
class JudgeErrorHandler:
    """Fail-closed error handling with retry logic."""

    def handle_error(self, error: Exception, attempt: int) -> str:
        """
        Classify error and determine action.

        Returns:
        - "retry": Retry with backoff
        - "fail": Fail-closed, abort batch
        """

        if isinstance(error, (TimeoutError, RateLimitError)):
            if attempt < self.config["retry_policy"]["max_retries"]:
                backoff = self.calculate_backoff(attempt)
                self.logger.info(f"Transient error, retrying in {backoff}s...")
                return "retry"
            else:
                self.logger.error(f"Max retries exceeded ({attempt})")
                return "fail"

        elif isinstance(error, (ModelNotFoundError, InvalidResponseError)):
            # Permanent error, fail immediately
            self.logger.error(f"Permanent error: {error}")
            return "fail"

        else:
            # Unknown error, fail-closed
            self.logger.error(f"Unknown error: {error}")
            return "fail"

    def calculate_backoff(self, attempt: int) -> float:
        """Exponential backoff: 2^attempt seconds."""
        return 2 ** attempt
```

### Partial Batch Failure

```python
def handle_partial_batch_failure(
    results: list[dict],
    failed_index: int,
    error: Exception
) -> dict:
    """
    Handle partial batch failure in fail-closed mode.

    Options:
    1. Save partial results (recommended)
    2. Discard all results (strict fail-closed)
    """

    if self.config["fail_closed_mode"] == "strict":
        # Discard all results, fail batch
        raise BatchJudgementFailedError(
            f"Batch failed at index {failed_index}: {error}. "
            f"Strict fail-closed: all results discarded."
        )

    elif self.config["fail_closed_mode"] == "partial_save":
        # Save successful results, mark batch as incomplete
        self.logger.warning(
            f"Batch partially complete: {len(results)}/{total} judged. "
            f"Failed at index {failed_index}: {error}"
        )

        return {
            "status": "partial_success",
            "results": results,
            "failed_at": failed_index,
            "error": str(error),
            "recommendation": "Re-run failed scenarios separately"
        }
```

---

## Judge Manifest Output

Every judgement produces a **judge manifest** for provenance:

```json
{
  "manifest_version": "0.1",
  "judgement_id": "550e8400-e29b-41d4-a716-446655440002",
  "created_at": "2025-12-28T15:00:00Z",

  "judge": {
    "contract_version": "judge_contract_v0.1",
    "model": "google/gemini-2.0-flash-exp",
    "model_version_lock": "gemini-2.0-flash-exp-2025-12-15",
    "judge_version": "0.1",
    "system_prompt_file": "prompts/judge_system_prompt_v0.1.md",
    "system_prompt_hash": "a8d2e1f3c4b5...",
    "temperature": 0.0
  },

  "run_artefacts": [
    {
      "run_id": "550e8400-e29b-41d4-a716-446655440000",
      "run_manifest_path": "../runs/run_a3f5e8c2_550e8400/run_manifest.json",
      "bundle_hash": "a3f5e8c2d1b4..."
    }
  ],

  "execution": {
    "mode": "offline_batch",
    "started_at": "2025-12-28T15:00:00Z",
    "completed_at": "2025-12-28T15:10:00Z",
    "scenarios_judged": 10,
    "failures": 0,
    "fail_closed": true,
    "operator": "aigov_runner"
  },

  "scores_dir": "./scores/",
  "checksums_file": "./checksums.sha256",

  "replay_metadata": null
}
```

---

## Related Documents

- [Phase 2 system overview](./phase2-system-overview-v0.1.md)
- [Artefact model spec](./phase2-artefact-model-v0.1.md)
- [Scenario bundle spec](./phase2-scenario-bundle-v0.1.md)
- [Data contracts v0.1](./data-contracts-v0.1.md) (behaviour_json_v1 schema)

---

**Document Status**: v0.1 Draft Specification
**Last Updated**: 2025-12-28
**Next Review**: After offline judgement implementation
