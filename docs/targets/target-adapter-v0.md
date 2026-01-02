# Target Adapter v0 (AIGov)

**Status:** Draft (v0.x)  
**Last updated:** 2026-01-02  
**Audience:** AIGov implementers (eval harness + MVP), target integrators  
**Purpose:** Define a *capability-driven* adapter interface for Targets (chatbots, RAG systems, agents) that is independent of any specific stack.

## 1. Definitions

- **Target**: the system under test (SUT). Examples: a hosted chatbot API, a deployed RAG service, an agent runtime with tools.
- **Target Adapter**: code that makes a Target usable by the AIGov evaluation harness in a consistent way.
- **Transcript**: the ordered record of interaction events (messages, tool calls, tool outputs, errors).
- **Eval log vs trace log**:
  - **Eval log** is the auditable record used for analysis and evidence (structured, per-run/per-sample).
  - **Trace log** is runtime diagnostics (JSONL) useful for debugging issues not visible in eval logs.

## 2. Non-goals

- Not a full schema for transcripts or tool events (v0 introduces required fields only).
- Not defining any specific “reference targets” (RAG stack / agent platform selection is separate).
- Not defining judge behaviour or verdict schemas (Judge remains canonical elsewhere).

## 3. Design principles

1. **Capability-driven**: adapters declare what they can provide (retrieval traces, tool logs, reset, sandboxing).
2. **Run-level provenance**: every run records a stable **target fingerprint** (config + environment) to support reproducibility.
3. **Observability first**: adapters must expose enough telemetry for evidence packs (transcripts, errors, optional traces).
4. **Fail closed**: if required evidence cannot be obtained, the run is non-auditable and must fail.
5. **Target neutrality**: no coupling to a particular RAG/agent framework.

## 4. Required adapter surface (v0)

The adapter is a *runtime object* created per run (or per sample) that supports:

### 4.1 `describe() -> TargetDescription` (required)

Returns stable identifiers and capability flags.

**Minimum fields**
- `adapter_id` (e.g., `aigov.targets.http_chat`)
- `adapter_version` (semver or git SHA)
- `target_kind` (enum): `chat`, `rag`, `agent`, `other`
- `deployment_mode` (enum): `local`, `client_cloud`, `vendor_cloud`, `on_prem`, `unknown`
- `capabilities` (see capability matrix)
- `target_fingerprint` (hashable structure; see 4.5)

### 4.2 `open(run_context) -> None` (required)

Prepare the target for invocation (auth, connections, warmup). Must be idempotent per run.

### 4.3 `invoke(request: TargetRequest) -> TargetResponse` (required)

Executes one step (single-turn or one agent “act”) and returns:
- `response` (assistant content and/or tool call directives as applicable)
- `events` (see 4.4)
- `usage` (tokens/cost if known)
- `errors` (if any; structured)

### 4.4 `get_transcript() -> Transcript` (required)

Returns the ordered set of interaction events to be written into eval logs.

**Transcript event minimum (v0)**
- `event_id`
- `timestamp`
- `event_type` (enum): `user_message`, `assistant_message`, `tool_call`, `tool_result`, `system`, `error`
- `content` (string or structured)
- `correlation_id` (links tool call ↔ tool result; optional but strongly recommended)

### 4.5 `fingerprint() -> TargetFingerprint` (required)

Returns a canonicalised structure used to compute a **fingerprint hash** recorded in `run_manifest`.

**Must include**
- `target_endpoint` (if remote) OR `deployment_ref` (if local)
- `model_id` / `model_version` (if applicable)
- `rag_index_id` / `corpus_id` (if applicable)
- `tooling_profile_id` (if agent/tools applicable)
- `config_hash` (canonicalised adapter config)
- `runtime_env` (best-effort): container image digest, git SHA, or build version

### 4.6 `reset(reset_mode) -> None` (optional but recommended)

Resets target state for reproducibility.
- `reset_mode`: `stateless`, `soft`, `hard`
- If unsupported, adapter must declare `supports_reset=false`.

### 4.7 `close() -> None` (required)

Clean shutdown; flush buffers; ensure transcript is complete.

## 5. Optional evidence hooks (capability-gated)

### 5.1 Retrieval traces (RAG)

If `supports_retrieval_traces=true`, adapter SHOULD export:
- top-k document identifiers, scores, chunk IDs
- citation spans (if available)
- (optional) embedding model ID

### 5.2 Tool call traces (agents)

If `supports_tool_traces=true`, adapter MUST record tool calls and tool outputs as transcript events.

### 5.3 Sandboxing / containment

If `supports_sandboxing=true`, adapter MUST expose:
- `sandbox_type` (none/docker/other)
- `sandbox_id` (per run, where available)
- `network_policy` (none/egress-limited/allowlist)

## 6. Manifest integration (v0)

### 6.1 run_manifest (intent)

Record:
- `target.adapter_id`, `target.adapter_version`
- `target.target_kind`, `target.deployment_mode`
- `target.capabilities`
- `target.fingerprint_hash` (+ selected non-sensitive fingerprint fields)
- `target.reset_mode` used (if any)

### 6.2 result_manifest (outputs)

Record hashed artefact refs for:
- transcript export(s)
- retrieval traces export(s) (if any)
- tool traces export(s) (if any)
- trace logs (if any)
- target-side logs (optional)

If any required artefact cannot be produced, the run is **non-auditable**.

## 7. One-way doors (avoid)

- Baking target-specific semantics into the adapter contract (e.g., “must be LangChain”).
- Requiring a heavyweight agent platform before defining the telemetry you need.
- Treating trace logs as canonical evidence (they are diagnostics; transcripts/manifests are evidence).
- Allowing adapters to mutate judgement artefacts (adapter produces target-side evidence only).

## 8. Minimal conformance tests (for adapters)

Any adapter claiming conformance MUST pass:
- Smoke invoke: open → invoke → transcript exists → close
- Fingerprint stability: same config yields same fingerprint hash
- Fail-closed evidence: missing transcript triggers run failure
- Capability truthfulness: capability-flagged artefacts exist and are referenced (hashed) in result_manifest
