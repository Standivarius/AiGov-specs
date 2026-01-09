# AiGov EP PRD (Petri–AKG / AiGov)

**Purpose:** Consolidated product requirements + buildable architecture for **EP** (Evals-as-Product, client-run runtime) and its relationship to **PE** (Product Evaluations that evaluate EP/MVP).

**Primary desired-behavior truth:** Specs Audit Pack v2 (external doc).  
**Implementation snapshot truth:** current repos in this workspace.  
**Conflict rule:** follow desired behavior; log repo deltas with file paths in `AiGov-specs/docs/program/REPO_COHERENCE_REPORT.md`.

---

## 0) Repo intent & boundaries (must be enforced)

### Canonical repo roles (target state)
- **AiGov-specs** = canonical specs/contracts/schemas/taxonomy (shared truth)
  - Examples:
    - `AiGov-specs/docs/specs/phase2-system-overview-v0.1.md`
    - `AiGov-specs/docs/specs/phase2-offline-judgement-v0.1.md`
    - `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json`
    - `AiGov-specs/schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml`
- **aigov-mvp** = **EP runtime product** (client-run evaluation pipeline)
  - Must contain: intake handling, scenario bundle compilation, Stage A transcript capture, Stage B offline judgement, evidence packs, L1/L2/L3 exports, operator tooling/CLI.
  - Repo reality today: mostly a sandbox target system (TargetLab): `aigov-mvp/services/targetlab_rag/*`
- **aigov-evals** = **PE** (regression/benchmark eval suites that test EP)
  - Must contain: regression suites, benchmarks, CI eval runs, datasets, analysis.
  - Repo reality today: contains substantial EP runtime under `aigov-evals/aigov_eval/*` (must migrate).

### Boundary rule (locked)
“Required for a client run?” ⇒ **EP ⇒ `aigov-mvp`**; otherwise ⇒ **PE ⇒ `aigov-evals`**.

---

## 1) Product definitions

### EP (Evaluation Product)
EP is the system that performs compliance evaluation of a client Target System, producing audit-grade outputs:
- Stage A: transcripts + raw artefacts (no scoring)
- Stage B: offline judgement (schema-valid findings)
- Stage C: aggregation + reporting exports (L1/L2/L3)

Repo reality:
- Stage A runner + target adapters + scorers exist today in `aigov-evals/aigov_eval/*` (mislocated per boundary).
- Target sandbox exists in `aigov-mvp/services/targetlab_rag/app.py`.
- Phase2 contracts/specs exist in `AiGov-specs/docs/specs/phase2-*.md`.

### PE (Product Evaluations)
PE is the harness + datasets that validate EP runtime behavior:
- batch calibration: `aigov-evals/aigov_eval/batch_runner.py`
- cases: `aigov-evals/cases/calibration/*.json`
- judge tests: `aigov-evals/tests/judge/*`
- minimal loop tests: `aigov-evals/tests/minimal_loop/*`

---

## 2) Goals and success criteria

### Primary goals
1) Enforce repo boundary discipline (EP in `aigov-mvp`, PE in `aigov-evals`, canonical contracts in `AiGov-specs`).  
2) Deliver Phase2-aligned EP pipeline:
`Intake → Bundle compile → Stage A (evidence only) → Stage B (offline judge) → Stage C (reports/exports)`.  
3) Eval-first development: requirements are driven by the matrix in `AiGov-specs/docs/program/REQUIREMENTS_EVALS_MATRIX.md`.

---

## 3) System overview (target architecture)

### End-to-end flow (target behavior)
1) Intake (validated config): `AiGov-specs/docs/specs/client-intake-variables.md`  
2) Scenario selection + client tailoring → bundle compilation: `AiGov-specs/docs/specs/phase2-scenario-bundle-v0.1.md`  
3) Stage A execution (no scoring): `AiGov-specs/docs/specs/phase2-offline-judgement-v0.1.md`  
4) Stage B offline judgement (fail-closed): `AiGov-specs/docs/specs/phase2-offline-judgement-v0.1.md`  
5) Stage C reporting + exports: `AiGov-specs/docs/specs/phase2-reporting-exports-v0.1.md`

### Evidence vs telemetry
- Evidence lane: artefacts required to defend audit result (transcript, manifests, judge outputs, bundle composition, checksums).
- Telemetry lane: optional debug traces (latency/token usage, retrieval traces).

---

## 4) Requirements (by capability)
Authoritative list: `AiGov-specs/docs/program/REQUIREMENTS_EVALS_MATRIX.md`.

Highlights:
- REQ-001 boundary enforcement
- REQ-030 Stage A transcript-only
- REQ-040 Offline judge schema-valid + fail-closed
- REQ-060 L1/L2/L3 exports

---

## 5) Data contracts (single sources of truth)
Canonical in AiGov-specs:
- `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json`
- `AiGov-specs/schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml`
- `AiGov-specs/scenarios/templates/scenario-template-v1.2.yaml`
- `AiGov-specs/docs/specs/variable-registry.md`
- `AiGov-specs/docs/specs/file-registry.md`

Known conflicts to resolve (see coherence report):
- `aigov-evals/schemas/behaviour_json_v0_phase0.schema-*.json` duplicates
- `AiGov-specs/schemas/judge_scenario_instruction_v0.1.json` shape divergence vs phase0 schema

---

## 6) Target repo layout (after refactor)
aigov-mvp (EP runtime)
- `aigov_ep/` (new package): intake/, bundle/, execute/, judge/, artifacts/, reporting/, cli.py
- existing: `services/targetlab_rag/`, `targets/compose/`

aigov-evals (PE)
- `cases/`, `tests/`, PE tooling; tests should call EP package/CLI rather than owning EP runtime code.

---

## 7) Deployment constraints
- EP runnable locally as CLI (Phase0).
- Target integration initially via HTTP adapter; TargetLab provides `/chat` and `/health`.
- Fail-closed on missing artefacts or schema invalid outputs.

---

## 8) AKG integration points (future)
Specs exist under `AiGov-specs/projects/akg/*`; runtime integration not present in repos today.
