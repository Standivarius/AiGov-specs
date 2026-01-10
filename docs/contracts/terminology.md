# Terminology Lock (PE vs EP) — Canonical definitions and boundary rules

**Canonical file:** `AiGov-specs/docs/contracts/terminology.md`  
**Applies to:** all repos (`AiGov-specs`, `aigov-mvp`, `aigov-evals`)

---

## PE vs EP (locked)

### EP — Evals-as-Product / Evaluation Product
**Definition:** The client-run evaluation system we deliver. EP performs: intake → scenario compilation → transcript capture → offline judgement → evidence packs → reporting/exports.

**Is**
- Runtime pipeline executed for a client audit engagement.
- Stage A + Stage B + Stage C pipeline (see Phase2 specs: `AiGov-specs/docs/specs/phase2-index.md`).
- Includes target adapters (scripted/mock/http/live), offline judge, manifests, and report generation.

**Is not**
- The internal regression harness used to test EP correctness/stability.

**Repo placement**
- EP **must live in `aigov-mvp`**.

---

### PE — Product Evaluations (evaluate the EP/MVP)
**Definition:** The evaluation suites and infrastructure that test/regress the EP runtime (QA/benchmarking). PE measures correctness, repeatability, stability, and contract compliance of EP outputs.

**Is**
- Calibration case sets, regression suites, CI eval runs, analysis notebooks, datasets used to validate EP, evaluation infrastructure for validating the MVP.
- Examples (current repo reality):
  - `aigov-evals/cases/calibration/*.json`
  - `aigov-evals/tests/**`

**Is not**
- The client-deliverable evaluation pipeline or report generator.

**Repo placement**
- PE **must live in `aigov-evals`**.

---

### Target System (customer system under audit)
**Definition:** The system being evaluated (chatbot/agent/RAG service).  
**Naming rule:** Do **not** call the customer system “PE” (reserved for Product Evaluations). Use **Target System** or **Customer Target**.

Examples:
- TargetLab sandbox service: `aigov-mvp/services/targetlab_rag/app.py`
- HTTP adapter currently in evals repo (to be moved to EP): `aigov-evals/aigov_eval/targets/http_target.py`

---

## Boundary rule (repo system-of-record)

- **AiGov-specs**: canonical definitions, schemas, taxonomy, traceability contracts  
  Examples:
  - `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json`
  - `AiGov-specs/docs/specs/phase2-artefact-model-v0.1.md`
  - Canonical taxonomy contracts live at `AiGov-specs/docs/contracts/taxonomy/`
- **aigov-mvp**: EP runtime product implementation (client-run)
- **aigov-evals**: PE regression suites, datasets, CI eval harness

**Boundary rule:** “Required for a client run?” ⇒ EP ⇒ `aigov-mvp`; otherwise ⇒ PE ⇒ `aigov-evals`.

---

## Pipeline terms (locked)

### Stage A — Execution / Transcript capture
- Runs scenarios against target adapters and captures transcripts and raw run artefacts.
- **No scoring, no judge calls** (offline judgement is separate).

### Stage B — Offline judgement
- Consumes Stage A artefacts and produces schema-valid findings.
- Fail-closed: schema invalid → judgement fails.

### Stage C — Aggregation / Reporting / Exports
- Generates L1/L2/L3 outputs, plus any GRC exports.

---

## Verdict labels (canonical + mapping)

### Canonical rating enum (TARGET BEHAVIOR)
**Canonical:** `INFRINGEMENT | COMPLIANT | UNDECIDED`

### Required migration rule
- `VIOLATION` or `VIOLATED` → `INFRINGEMENT`
- `NO_VIOLATION` or `COMPLIANT` → `COMPLIANT`
- `UNCLEAR` or `UNDECIDED` → `UNDECIDED`

### Schema versioning requirement
Because `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json` may currently use `VIOLATED`, migration must be explicit:
- Either update the enum and bump schema version, OR
- Introduce `behaviour_json_v1_phase0.schema.json` with canonical enum and mark v0 as deprecated.

---

## Stable IDs
- Requirements: `REQ-###`
- Evals: `EVAL-###`
- Evidence artifacts: `EVID-###`

---

## Evidence vs telemetry (two-lane model)
- **Evidence lane**: artefacts needed to defend the audit result (transcripts, manifests, judge outputs, checksums).
- **Telemetry lane**: optional runtime traces for debugging (token usage, retrieval traces).
