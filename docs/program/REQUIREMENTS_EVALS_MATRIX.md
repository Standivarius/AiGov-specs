# Requirements / Evals / Evidence Matrix

Columns are exactly: (1) Capability, (2) Acceptance, (3) Evals + Evidence.  
IDs: REQ-###, EVAL-###, EVID-###.  
Machine export: `AiGov-specs/docs/program/REQUIREMENTS_EVALS_MATRIX.json`

| Capability | Acceptance | Evals + Evidence |
|---|---|---|
| **REQ-001 — Repo boundary enforcement (EP vs PE)** | EP runtime code lives in `aigov-mvp`; PE suites live in `aigov-evals`; canonical schemas/docs live in `AiGov-specs`. | **EVAL-001** (planned): CI lint for move-map. **EVID-001**: `AiGov-specs/docs/program/REPO_COHERENCE_REPORT.md`. |
| **REQ-010 — Intake config validation** | EP validates client_intake and fails fast on missing required fields. | **EVAL-010** (planned): intake unit tests. **EVID-010**: validated intake artifact (planned). |
| **REQ-011 — Deterministic bundle compiler** | Same inputs → same bundle hash; conflicts are surfaced. | **EVAL-011** (planned): determinism test. **EVID-201**: bundle_manifest.json (planned). |
| **REQ-030 — Stage A transcript-only** | Stage A writes transcript + run manifest; no scoring/judge. | **EVAL-030** (planned): artefact set check. **EVID-301/302**: transcript + run_manifest. |
| **REQ-031 — Target adapters (scripted/mock/http)** | Adapters exist and contract tests pass. | Existing tests: `aigov-evals/tests/minimal_loop/*`. Evidence: `aigov-evals/aigov_eval/targets/*`, `aigov-mvp/services/targetlab_rag/app.py`. |
| **REQ-040 — Offline judgement schema-valid & fail-closed** | Offline judge emits behaviour_json that validates against canonical schema; invalid outputs hard-fail. | Upgrade existing judge schema test: `aigov-evals/tests/judge/test_j02_schema.py`. Evidence: canonical schema `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json`. |
| **REQ-050 — Provenance chain & manifests** | Bundle/run/judge/report manifests include hashes and references. | **EVAL-050** (planned): manifest completeness tests. Spec evidence: `AiGov-specs/docs/specs/phase2-artefact-model-v0.1.md`. |
| **REQ-060 — L1/L2/L3 reporting exports** | EP produces L1/L2/L3 outputs per spec. | **EVAL-060** (planned): report generation tests. Spec evidence: `AiGov-specs/docs/specs/phase2-reporting-exports-v0.1.md`. |
| **REQ-070 — PE batch calibration harness** | PE runs calibration suites and emits batch report artifacts. | Existing: `aigov-evals/aigov_eval/batch_runner.py`, tests under `aigov-evals/tests/batch_calibration/*`. |
| **REQ-080 — Verdict label standardization** | Canonical labels enforced across schema, judge output, reporting; legacy mapped. | **EVAL-080** (planned): mapping + schema tests. Canonical terms: `AiGov-specs/docs/contracts/terminology.md`. |
