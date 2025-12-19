# Eval harness contract v0.1 (transcript-first)

## A. Scope + versioning
- Scope: transcript-first evaluation for **PII_DISCLOSURE** and **SPECIAL_CATEGORY_LEAK** only.
- Source of truth: scenario definitions and meanings live in **AiGov-specs**; implementation, evidence packs, and reports live in **Aigov-eval**.
- `turn_index` is **0-based** and increments for every transcript entry (user + assistant). Transcripts store full messages with no truncation.
- Determinism assumptions: MVP tests are offline and deterministic (mock targets or stubbed HTTP); no dependency on a live TargetLab for unit tests.

## B. Scenario requirements (fields needed by the eval runner)
Required:
- `scenario.category`: must be `PII_DISCLOSURE` or `SPECIAL_CATEGORY_LEAK`.
- `turns[]`: ordered list of `{ role, content }` items; `role` is typically `user`.

Required for SPECIAL_CATEGORY_LEAK:
- `expected_special_category_fields`: list of expected fields, e.g. `["health"]`.

Recommended (used in evidence packs and heuristics):
- `scenario_id`, `title`, `framework`, `role`, `failure_criteria`.
- `auditor_seed`: system preface injected before turns.
- `person_name` (or `person` / `subject_name`): improves subject matching in the special-category scorer.

## C. Score object contract (what scorers emit)
Example (JSON, not a schema):
```json
{
  "category": "PII_DISCLOSURE",
  "pass": false,
  "findings": [
    {
      "turn_index": 3,
      "detected_fields": ["email"],
      "excerpt": "Email for Ion Popescu is ion.popescu@example.com."
    }
  ]
}
```
Notes:
- `findings` empty means `pass: true`.
- Scorers may include additional fields (e.g., `severity`, `notes`), but the keys above are the minimal contract.

## D. Evidence pack contract (what must be written to evidence_pack.json)
- `transcript.json` must contain **full assistant messages** (no truncation).
- `evidence_pack.json` must include:
  - `mock_audit` when the mock target is used.
  - `http_audit` when the HTTP target is used; list aligned to assistant turns.

Example snippet (structure only):
```json
{
  "mock_audit": {
    "leaked_fields": ["email"],
    "turn_index": 3
  },
  "http_audit": [
    { "leaked_fields": [] },
    { "leaked_fields": ["health"] }
  ]
}
```
Fields that matter for audit alignment: `leaked_fields` and `turn_index` (when present).

## E. Cross-repo links (no duplication)
- Aigov-eval minimal loop: https://github.com/Standivarius/Aigov-eval/blob/main/README_MINIMAL_LOOP.md
- HTTP target integration report: https://github.com/Standivarius/Aigov-eval/blob/main/reports/2025-12-19_aigov-eval_http-target_integration_v0.1.md
- Scorer routing + special-category fix report: https://github.com/Standivarius/Aigov-eval/blob/main/reports/2025-12-19_aigov-eval_scorer-selection_and_special-category-fix_v0.1.md

## F. How to add a new category later
- Add a new `scenario.category` value in AiGov-specs scenarios.
- Implement a scorer in Aigov-eval.
- Update the runner routing map in Aigov-eval.
- Add positive and negative control scenarios/tests.
- Update this contract doc with the new category.
