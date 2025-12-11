- Learned:
  - Scenario/test case schema uses JSON arrays with `id`, `category`, `prompt`, `expected_output`, optional `expected_keywords`, `difficulty`, `retrieval_context`; conversational scenarios add `scenario` + ordered `turns` and generate per-turn sample ids.
  - Evaluation config wires model/judge providers, taskType-driven metric bundles (correctness/coherence/tonality/safety/bias/toxicity/relevancy/faithfulness/RAG recall & precision/agent tool correctness), and per-metric thresholds.
  - Framework catalogs are TS data structures: ISO 27001 clauses/annex controls with evidence + key questions + cross_mappings; ISO 42001 clauses/annex categories with applicability flags and auditor_feedback; GDPR references limited to data minimization/lawful basis inside ISO 42001 annex guidance.
  - Reports/exports pull from file records (id, filename, project_id, source, uploader) and risk/compliance models (likelihood, severity, mitigation_status, owners) with evidence links; table exports support CSV/XLSX/PDF.
  - Architecture is PERN + Redis with separate FastAPI eval services and a Python fairness engine; docker-compose brings up Postgres/Redis/backend/worker/frontend/bias-fairness.
- Value to AIGov:
  - Clear JSON schema for eval datasets and conversation scenarios we can mirror without copying code.
  - Metric bundle logic shows how task types drive scoring dimensions; helpful for designing risk-weighted eval plans.
  - Framework data models demonstrate how to store control text, evidence examples, cross-framework mapping, and applicability flags.
  - Evidence schema (file manager) provides a simple linking model for findings → files → framework items.
- Gaps:
  - No explicit scenario→GDPR/ISO mapping; safety/jailbreak categories exist only narratively.
  - DeepEval engine code absent; mapping of metrics to pass/fail thresholds beyond config is inferred, not documented.
  - Onetrust/Vanta export schemas not present; created placeholder field sets only.
  - Eval dataset presets referenced (chatbot/rag/agent/safety) but actual JSON files are missing.
- Recommendations:
  1) Adopt the dataset/test-case schema and conversational scenario pattern as-is; keep taxonomy (category/difficulty/expected_keywords/retrieval_context).
  2) Recreate taskType→metric bundles to align scoring with use case (chatbot vs RAG vs agent vs safety), but define AIGov-specific thresholds and weights.
  3) Build explicit scenario→framework mappings (GDPR/ISO/AI Act) since VerifyWise does not supply them; leverage their control structures as a template.
  4) Mirror evidence linkage model (file metadata + parent/sub/meta ids) to tie findings to controls and reports; extend with audit trail status.
