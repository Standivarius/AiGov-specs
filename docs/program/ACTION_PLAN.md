# Action Plan (phased refactor + build)

**Rule:** One step at a time. Each step must include Verification commands and expected success signal.

This plan is designed so an implementation agent can execute it without guessing.
The machine-readable task list is in:
- `AiGov-specs/docs/program/CODEX_EXECUTION_PACK.yaml`

---

## Phase 0 — Canonical docs + contracts lock (AiGov-specs only)
**Goal:** make the plan canonical so later refactors are grounded.

- Update `AiGov-specs/docs/contracts/terminology.md` (done in this step)
- Add Program Pack folder `AiGov-specs/docs/program/*` (done in this step)

Verification (how to run):
- `ls -la AiGov-specs/docs/program` → shows program files
- `grep -n "Boundary rule" AiGov-specs/docs/contracts/terminology.md` → shows locked boundary rule

---

## Phase 1 — Create EP package skeleton in aigov-mvp
(Do not execute in this step; implemented later by Codex)

- Add `aigov_ep/` python package + CLI skeleton in `aigov-mvp/`
- Verification will be:
  - `cd aigov-mvp && python -m venv .venv && . .venv/bin/activate && pip install -e .`
  - `python -c "import aigov_ep; print('ok')"` → prints ok

---

## Phase 2 — Migrate EP runtime modules from aigov-evals to aigov-mvp
(Do not execute in this step; implemented later)

- Move target adapters / runner / evidence / judge / mapper into `aigov-mvp/aigov_ep/*`
- Update PE tests in `aigov-evals/tests/**` to import from EP or call EP CLI
- Verification will be:
  - `cd aigov-evals && pytest -q` → remains green

---

## Phase 3 — Stage A transcript-only + Stage B offline judge schema-valid
(Do not execute in this step; implemented later)

- Refactor runner so Stage A produces transcripts/manifests only (no scoring)
- Offline judge reads Stage A artefacts and emits schema-valid behaviour_json

---

## Phase 4 — Reporting exports (L1/L2/L3)
(Do not execute in this step; implemented later)

- Add L3 export packager, then L1/L2 markdown generators aligned to Phase2 reporting spec

---
