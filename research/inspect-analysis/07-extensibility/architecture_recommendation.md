Recommendation: **PARTIAL / BUILD ON INSPECT**.

- Inspect provides the core orchestration we need: task abstraction, solver chaining/agents, rich scoring, logging, multi-provider support (including OpenRouter), batching, and concurrency controls. It can replace most bespoke eval runners.
- Custom work required: compliance scorers (GDPR/ISO rubrics), VerifyWise import/export adapters, and any bespoke governance dashboards. Hooks/entry points make these straightforward without forking core.
- Data/model portability is strong: datasets accept arbitrary schemas; model roles and eval-sets make cross-model testing simple. Logs already contain token usage for cost tracking.
- Production fit: mature CLI/API, retries, sandboxed tools, approval policies, S3/Azure logging, and condensing for large runs. Still need to harden for our SLAs (observability via hooks, CI gating, rate-limit tuning per provider).
- Path forward: adopt Inspect as the evaluation backbone; build a thin AIGov layer for compliance scorers, VerifyWise translators, and any additional monitors; keep custom orchestration only where enterprise controls or UI requirements demand it.
