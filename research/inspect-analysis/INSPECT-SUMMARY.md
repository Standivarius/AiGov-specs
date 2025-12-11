**Gap Analysis**
- Inspect covers core orchestration (tasks/solvers/scorers/datasets/logging), multi-provider LLM abstraction (incl. OpenRouter), sandboxed tools, and batch/parallel execution. This replaces most custom runner logic.
- Compliance/reporting gaps: no native GDPR/ISO scorer or VerifyWise exporter; both can be added via custom scorers + log-to-VerifyWise adapter.
- Observability/ops: logs capture transcripts, token usage, and errors; hooks exist for custom telemetry, but production dashboards/alerting require integration.
- Performance/cost: async engine with `max_connections`, batching (`GenerateConfig.batch`), caching, and model_usage accounting; throughput limited by provider rate limits and sandbox/tool overhead.
- Maturity: rich docs, 100 benchmark suites, CI, retries, approval policies, and S3/Azure logging; still requires tuning and security review for production SLAs.

**Critical Answers**
- Replace custom orchestration? **Partial**—use Inspect as backbone, keep thin layers for compliance scorers, exports, and monitoring.
- OpenRouter/multi-provider? **Yes**—native OpenRouter provider; eval-set + model roles support GPT/Claude/Gemini comparisons.
- Custom GDPR/ISO scorers? **Yes**—implement with `@scorer` (model-graded or external services); metrics/reducers available.
- VerifyWise schema for datasets? **Yes** via `FieldSpec`/`record_to_sample`; metadata/files/sandbox map cleanly.
- Export to VerifyWise? **No built-in**, but EvalLog API + CLI make a straightforward adapter feasible.
- Production-ready? **Strong foundation** (retryable eval-sets, sandboxing, approval, logs, provider breadth); add ops layers (monitoring hooks, secrets/governance review, cost dashboards).
- Performance/cost/scale? **Async + batched** requests; tune `max_connections`, batch config, and sandbox limits. Token usage per model is logged for cost tracking.

**AIGov on Inspect (proposed)**
```mermaid
flowchart LR
  Scenarios[VerifyWise / AIGov scenarios<br/>CSV/JSON/HF] -->|FieldSpec/adapter| Tasks[Inspect @task<br/>datasets+metadata]
  Tasks --> Solvers[Inspect solvers/agents<br/>prompting, attacks, tools]
  Solvers --> Providers[LLM providers<br/>OpenAI/Anthropic/Gemini/OpenRouter/local]
  Solvers --> Tools[Tools + sandbox + approvals]
  Tasks --> Scorers[Built-in + custom GDPR/ISO scorers]
  Scorers --> Metrics[Metrics/reducers]
  {Solvers,Scorers} --> Logs[EvalLogs (.eval/.json)]
  Logs --> Exports[Adapters -> VerifyWise reports / BI]
  Logs --> Monitor[Hooks/telemetry/cost dashboards]
```

**What to build**
- Custom scorers for GDPR/ISO/enterprise controls; optional multi-model grading council.
- VerifyWise import/export utilities (dataset ingester + log-to-report transformer).
- Observability: hook events into APM/metrics, add dashboards for success/cost/rate-limit errors.
- Hardening: provider-specific rate-limit tuning, sandbox image review, secrets management, CI gates.

**Recommendation**
- Adopt Inspect as the evaluation engine; layer AIGov-specific scorers, exports, and monitoring on top. Keep minimal custom orchestration only for enterprise governance and UI/reporting needs.
