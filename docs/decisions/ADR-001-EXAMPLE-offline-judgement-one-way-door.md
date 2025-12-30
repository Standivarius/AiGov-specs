# ADR-001: Offline Judgement Architecture (One-Way Door)

**Date**: 2025-12-28
**Status**: Accepted
**Owner**: AiGov Architecture Team
**Decision Type**: One-Way Door

---

## Context

Phase 0/1 implemented **inline judgement**: scoring happens during the evaluation run. This approach has limitations:

- **Cost inefficiency**: Every run pays for judge model inference, even if re-testing same scenarios
- **No reproducibility**: Can't re-judge old transcripts with improved judge models
- **Tight coupling**: Run failures cascade to judgement failures
- **No audit separation**: Evidence capture and interpretation mixed in same stage

As we scale to production clients (Phase 2), we need a more robust architecture.

---

## Decision

We will implement **offline judgement** as a separate stage from evaluation runs:

1. **Run stage**: Execute scenarios, capture transcripts, NO scoring
2. **Judgement stage**: Process batch of transcripts offline with locked judge model

This is a **one-way door**: Once we migrate existing clients to offline judgement, reverting to inline judgement would require:
- Re-running all historical evaluations (expensive)
- Breaking artefact provenance chain
- Incompatible report formats (reports reference judgement manifests)

**One-Way Door Notice**:
> This is a **one-way door** decision. Once implemented, reverting will be costly or impossible due to:
> - **Artefact model dependency**: Run artefacts explicitly exclude scores; adding inline scoring would require schema migration
> - **Client contract changes**: L2 reports include judgement manifest as provenance; removing this would break audit compliance
> - **Cost optimization lock-in**: Clients budget for batch judgement pricing; reverting to inline would increase costs 3-5x
> - **Replay workflow dependency**: Clients rely on replay for judge upgrades; inline judgement doesn't support this

---

## Consequences

### Positive Consequences
- **Cost reduction**: Batch judgement ~70% cheaper than inline (shared system prompt, parallel API calls)
- **Reproducibility**: Can re-judge historical transcripts with new judge versions (replay workflow)
- **Fail-closed safety**: Run failures don't cascade to judgement; evidence preserved even if judge unavailable
- **Audit compliance**: Clear separation between evidence capture and interpretation (required by some legal frameworks)
- **Batch optimization**: Process 100+ runs in parallel, respecting rate limits

### Negative Consequences / Tradeoffs
- **Workflow complexity**: Two-stage process instead of one-click evaluation
- **Storage overhead**: Run artefacts stored longer (can't delete after inline judgement)
- **Developer experience**: More CLIs to learn (run, judge, replay)
- **Initial implementation cost**: ~2-3 weeks to build + test offline judge

### Risks
- **Risk**: Clients forget to run judgement stage (incomplete audits)
  - **Mitigation**: Add validation in report generator (fail if judgement missing)

- **Risk**: Judge model unavailable when needed (fail-closed blocks progress)
  - **Mitigation**: Support multiple judge versions in parallel; document upgrade process

- **Risk**: Replay confusion (clients re-judge too often, waste budget)
  - **Mitigation**: Replay approval workflow; log all replays for audit

---

## Alternatives Considered

### Option A: Keep Inline Judgement (Status Quo)
**Pros**:
- Simple one-stage workflow
- Familiar to Phase 0/1 users
- No migration needed

**Cons**:
- High cost (can't batch judge)
- No reproducibility (can't re-judge old runs)
- Tight coupling (run failure = judgement failure)

**Rejected because**: Doesn't scale to production clients (cost prohibitive at 100+ scenarios per client)

### Option B: Hybrid (Inline for Dev, Offline for Prod)
**Pros**:
- Fast feedback in dev (inline)
- Cost optimization in prod (offline)

**Cons**:
- Maintenance burden (two code paths)
- Risk of dev/prod parity issues
- Confusing for users (when to use which?)

**Rejected because**: Added complexity outweighs benefits; better to standardize on one approach

### Option C: Optional Offline (Clients Choose)
**Pros**:
- Flexibility (clients pick workflow)
- Gradual migration path

**Cons**:
- Confusing product (two ways to do same thing)
- Support burden (need to maintain both forever)
- Prevents one-way door optimization (can't assume offline-only)

**Rejected because**: Violates principle of "one obvious way"; better to pick best approach and commit

---

## Implementation Plan

### Phase 2.1: Build Offline Judge
1. Implement `LockedJudge` class with fail-closed semantics
2. Add batch processing with parallel API calls
3. Create judge manifest schema
4. Build judgement artefact validator

**Timeline**: 2 weeks
**Dependencies**: Run artefact model complete (separate ADR)

### Phase 2.2: Migrate Existing Clients
1. Re-run Phase 0/1 scenarios (capture transcripts without scores)
2. Judge offline with locked judge v0.1
3. Update reports to reference judgement manifests
4. Archive old inline-judgement artefacts

**Timeline**: 1 week
**Dependencies**: Phase 2.1 complete + client approval

### Phase 2.3: Implement Replay Workflow
1. Add `replay` CLI command
2. Build replay comparison report
3. Document replay governance policy
4. Add replay audit trail to judge manifest

**Timeline**: 1 week
**Dependencies**: Phase 2.2 complete

**Rollback Plan**: N/A (one-way door; no rollback)

---

## Validation Criteria

This decision will be considered successful if:
- ✅ Batch judgement reduces inference cost by >50% (vs inline baseline)
- ✅ Replay workflow used successfully for at least 1 judge upgrade
- ✅ Zero run failures cascade to judgement failures (fail-closed working)
- ✅ Client L2 reports include complete provenance chain (bundle → run → judge)
- ✅ No client requests to revert to inline judgement (product satisfaction)

**Review date**: 2026-03-01 (after 3 months of production use)

---

## References

- [Phase 2 system overview](../specs/phase2-system-overview-v0.1.md)
- [Offline judgement spec](../specs/phase2-offline-judgement-v0.1.md)
- [Artefact model spec](../specs/phase2-artefact-model-v0.1.md)
- Related issue: #123 (Phase 2 architecture planning)
- External reference: [AWS re:Invent talk on one-way doors](https://www.youtube.com/watch?v=...)

---

## Approval

This is a **one-way door** decision and requires approval from:

- [x] Architecture Lead: @aigov_arch (approved 2025-12-28)
- [x] Engineering Lead: @aigov_eng (approved 2025-12-28)
- [ ] Product Lead: @aigov_product (pending)
- [ ] Client Success: @aigov_cs (pending)

**Final Status**: Accepted (2025-12-28)
