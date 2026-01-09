# Repo Coherence Report (duplicates, contradictions, stale docs, missing links)

## Inventory (current repo reality)
- AiGov-specs: canonical specs/contracts/schemas
  - Phase2 specs: `AiGov-specs/docs/specs/phase2-*.md`
  - Canonical schema: `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json`
  - Templates: `AiGov-specs/scenarios/templates/scenario-template-v1.2.yaml`
- aigov-evals: PE harness but currently holds EP runtime code
  - EP-like runtime: `aigov-evals/aigov_eval/*`
  - PE tests: `aigov-evals/tests/**`
  - Cases: `aigov-evals/cases/calibration/*.json`
- aigov-mvp: target repo for EP runtime; currently contains TargetLab target system
  - `aigov-mvp/services/targetlab_rag/app.py`
  - `aigov-mvp/docs/eval-status.md` (deprecated)

---

## Duplicate / conflicting docs

| Paths | Conflict | Canonical | Consolidation edit |
|---|---|---|---|
| `AiGov-specs/docs/contracts/terminology.md` vs `aigov-evals/docs/GLOSSARY.md` | glossary uses old names/labels | `AiGov-specs/docs/contracts/terminology.md` | add “DEPRECATED” banner to `aigov-evals/docs/GLOSSARY.md` pointing to canonical terminology |
| `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json` vs `aigov-evals/schemas/behaviour_json_v0_phase0.schema-*.json` | schema duplication drift risk | specs schema | delete eval copies OR vendor with hash-sync rule |
| `AiGov-specs/docs/specs/phase2-reporting-exports-v0.1.md` vs `aigov-evals/docs/REPORTING_SPEC.md` | “reporting” means EP client reports vs PE batch reports | keep both but scope | rename eval doc to PE scope or add banner + cross-link |

---

## Stale/legacy docs

| Path | Why stale | Deprecation mechanism |
|---|---|---|
| `aigov-mvp/docs/eval-status.md` | explicitly deprecated and out of date | keep, add banner + link to Program Pack |
| `aigov-evals/docs/Refactor_summary-eval.md` | status claims conflict with ongoing refactor | add banner linking to Action Plan |

---

## Missing cross-links
- Add links from `aigov-evals/README_MINIMAL_LOOP.md` to:
  - `AiGov-specs/schemas/behaviour_json_v0_phase0.schema.json`
  - `AiGov-specs/docs/specs/phase2-index.md`

---

## Repo move/merge/deprecate map (workspace-relative paths)

| FROM_PATH | TO_PATH | Reason |
|---|---|---|
| `aigov-evals/aigov_eval/targets/*` | `aigov-mvp/aigov_ep/targets/*` | required for client runs (EP) |
| `aigov-evals/aigov_eval/runner.py` | `aigov-mvp/aigov_ep/execute/runner.py` | Stage A is EP |
| `aigov-evals/aigov_eval/judge.py` | `aigov-mvp/aigov_ep/judge/judge.py` | Stage B judge is EP |
| `aigov-evals/aigov_eval/judge_output_mapper.py` | `aigov-mvp/aigov_ep/judge/mapper.py` | EP outputs must validate against canonical schema |
| `aigov-evals/aigov_eval/evidence.py` | `aigov-mvp/aigov_ep/artifacts/evidence.py` | evidence packs are EP deliverables |

---

## Discrepancy log (desired behavior vs repo reality)
1) EP runtime currently lives in `aigov-evals/aigov_eval/*` but must be migrated to `aigov-mvp/`.
2) Stage A currently scores inline (must become transcript-only).
3) Verdict label mismatch across code/schema must be standardized per terminology lock.
4) Scenario library in specs is currently empty; runnable examples are in evals repo.
