# Target Capabilities Matrix v0 (AIGov)

**Status:** Draft (v0.x)  
**Last updated:** 2026-01-02  
Purpose: standardise capability flags so the eval harness can adapt without stack-specific branching.

## Capability flags

| Capability flag | Meaning | Required? | Notes / evidence expectations |
|---|---|---:|---|
| `supports_chat_turns` | Target supports single/multi-turn chat interactions | Yes | Minimum for chatbot-like targets |
| `supports_reset` | Target can reset state between samples/runs | Recommended | Declare `reset_modes` |
| `supports_streaming` | Target returns streaming responses | No | Adapter must still emit final transcript events |
| `supports_retrieval` | Target performs retrieval (RAG) | No | If true, prefer `supports_retrieval_traces` |
| `supports_retrieval_traces` | Adapter can export retrieval trace (top-k doc IDs, scores, chunks, citations) | No (RAG yes) | Required for *reference RAG tier* |
| `supports_tools` | Target uses callable tools (agentic) | No | If true, prefer `supports_tool_traces` |
| `supports_tool_traces` | Adapter records tool calls + tool outputs as transcript events | No (agents yes) | Include correlation IDs |
| `supports_side_effect_controls` | Adapter can constrain/simulate side effects (approval gates, dry-run) | No | Useful for safe evals |
| `supports_sandboxing` | Target can be run inside a sandbox (e.g., containers) | No | Container-first is the default pattern |
| `supports_network_policy` | Adapter can enforce network policy (none/egress-limited/allowlist) | No | Important for agent safety |
| `supports_identity_context` | Adapter can run under a specified identity/role (RBAC) | No | Enterprise realism |
| `supports_data_canaries` | Adapter can seed/verify canary strings in corpora or tool outputs | No | Helps detect exfil |
| `supports_observability_export` | Adapter can export structured logs beyond transcript | Recommended | Always hash + reference |
| `supports_cost_usage` | Adapter can provide token/cost/latency metrics | Recommended | Useful for ops/reporting |

## Target tier guidance

| Target tier | Intended use | Minimum capabilities | Strongly recommended |
|---|---|---|---|
| Tier 0: Simulated/Scripted | Fast regression, contract tests | chat_turns, transcript | reset, cost_usage |
| Tier 1: Reference Chat/RAG-lite | Basic enterprise chatbot realism | chat_turns | reset, observability_export |
| Tier 2: Reference RAG | Retrieval integrity, prompt injection via corpus | retrieval + retrieval_traces | data_canaries, identity_context |
| Tier 3: Reference Agent | Tool misuse, exfil via tools, privilege boundaries | tools + tool_traces | sandboxing, network_policy, side_effect_controls |
| Tier 4: Client-system Adapter | Customer’s real system | depends | observability_export, identity_context |

## Evidence expectations by capability (v0)

- If `supports_retrieval_traces=true`: `result_manifest` MUST reference a retrieval trace artefact (hashed).
- If `supports_tool_traces=true`: tool calls and results MUST appear in transcript with correlation IDs.
- If `supports_sandboxing=true`: `run_manifest` MUST record sandbox type and sandbox identifier (where available).
