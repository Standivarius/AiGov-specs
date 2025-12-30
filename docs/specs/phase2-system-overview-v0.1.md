# Phase 2 Evaluation System Overview v0.1

**Purpose**: Define the production-grade evaluation architecture for systematic GDPR compliance testing
**Version**: 0.1 (Phase 2)
**Date**: 2025-12-28

---

## Executive Summary

Phase 2 transitions the AIGov evaluation system from MVP (Phase 0-1) to a production-ready platform with:
- **Artefact-driven evaluation**: All runs produce immutable evidence bundles
- **Locked judge governance**: Fail-closed offline judgement with audit trail
- **Translation-first workflow**: Multi-language testing with provenance tracking
- **Client-specific customisation**: Override scenarios without forking the catalogue
- **Reproducible builds**: Bundle compiler ensures deterministic scenario selection

---

## Architecture Principles

### 1. Fail-Closed by Default
- Judge models are **locked at specification time** via configuration manifest
- Evaluation runs fail if judge dependencies unavailable (no silent degradation)
- All judgement happens **offline** after transcript capture (no live model calls during eval)

### 2. Provenance Chain
Every artefact includes:
- **Git commit hash** of scenario catalogue
- **Bundle manifest hash** for scenario selection
- **Translation manifest** for language transformations
- **Judge contract version** for reproducibility

### 3. Client Data Isolation
- Client-specific overrides live in separate namespace (`client_configs/`)
- Base scenario catalogue remains canonical and version-controlled
- Bundle compiler merges base + overrides deterministically

### 4. Evidence-First Workflow
```
Scenario Bundle → Run (transcript only) → Judgement (offline) → Report
```
- **Run stage**: Execute scenarios, capture transcripts, no scoring
- **Judge stage**: Apply locked judge to all transcripts in batch
- **Report stage**: Aggregate findings with translations + context

---

## Component Responsibilities

### A. Scenario Catalogue Service
**Location**: `scenarios/library/`
**Contract**: Scenario card schema v1.2+
**Responsibilities**:
- Maintain canonical GDPR scenario definitions
- Version scenarios with semantic versioning
- Provide base templates for client customisation

### B. Client Override Manager
**Location**: `client_configs/{client_id}/`
**Contract**: Override manifest v0.1
**Responsibilities**:
- Store client-specific scenario modifications
- Validate overrides against base schema
- Track override history for audit purposes

### C. Bundle Compiler
**Location**: `aigov-eval` (implementation)
**Contract**: Bundle manifest v0.1
**Responsibilities**:
- Merge base scenarios + client overrides
- Generate deterministic bundle manifest (SHA-256 hash)
- Resolve conflicts (client overrides take precedence)
- Output compiled scenario pack for execution

### D. Translation Engine
**Location**: `aigov-eval` (implementation)
**Contract**: Translation manifest v0.1
**Responsibilities**:
- Translate scenarios to target language (EN → RO, etc.)
- Preserve original text in metadata (no double translation)
- Anchor evidence to source language for traceability
- Generate translation manifest with model version + timestamps

### E. Run Executor
**Location**: `aigov-eval` (implementation)
**Contract**: Run manifest v0.1
**Responsibilities**:
- Execute compiled scenario bundle against target system
- Capture full transcripts (no truncation)
- Record run metadata (target config, timestamps, git commit)
- Output run artefact bundle (transcript + metadata, **no scores**)

### F. Offline Judge
**Location**: `aigov-eval` (implementation)
**Contract**: Judge contract v0.1
**Responsibilities**:
- Load locked judge model from manifest
- Process batch of run artefacts offline
- Apply scoring logic with fail-closed error handling
- Output judgement artefact (scores + evidence + provenance)

### G. Report Generator
**Location**: `projects/report-gen/`
**Contract**: Report input v0.1 (from data-contracts-v0.1.md)
**Responsibilities**:
- Aggregate judgement artefacts for client audit
- Render L1/L2/L3 reports with translations
- Export to GRC platforms (OneTrust, Vanta, etc.)
- Include full provenance chain in annexes

---

## Data Flow

```
┌──────────────────────────────────────────────────────────────────┐
│ 1. SCENARIO PREPARATION                                          │
└──────────────────────────────────────────────────────────────────┘
  Scenario Catalogue (base)
         +
  Client Overrides
         ↓
  [Bundle Compiler]
         ↓
  Compiled Bundle Manifest (SHA-256)
         ↓
  [Translation Engine] (if multilingual)
         ↓
  Translated Scenario Pack + Translation Manifest

┌──────────────────────────────────────────────────────────────────┐
│ 2. EXECUTION (No Scoring)                                        │
└──────────────────────────────────────────────────────────────────┘
  Scenario Pack → [Run Executor] → Run Artefact Bundle
    (transcripts + run manifest, NO scores)

┌──────────────────────────────────────────────────────────────────┐
│ 3. OFFLINE JUDGEMENT                                             │
└──────────────────────────────────────────────────────────────────┘
  Run Artefacts (batch) → [Locked Judge] → Judgement Artefacts
    (scores + evidence + judge manifest)

┌──────────────────────────────────────────────────────────────────┐
│ 4. REPORTING                                                     │
└──────────────────────────────────────────────────────────────────┘
  Judgement Artefacts → [Report Generator] → L2 PDF + GRC Exports
```

---

## Key Design Decisions

### Why Offline Judgement?
- **Cost control**: Batch judgement is cheaper than per-run inference
- **Reproducibility**: Can re-judge old transcripts with new judge versions
- **Fail-closed**: Run failures don't cascade to judgement stage
- **Audit trail**: Clear separation between evidence capture and analysis

### Why Translation Manifest?
- **No double translation**: Original text preserved in metadata
- **Provenance**: Every translation includes model version + timestamp
- **Evidence anchoring**: Findings reference both source and translated text
- **Review workflow**: Human reviewers can verify translation quality

### Why Client Overrides Instead of Forks?
- **Maintenance**: Upstream scenario improvements flow to all clients
- **Audit trail**: Override history tracked separately from base catalogue
- **Reproducibility**: Bundle compiler creates deterministic merge
- **Conflict resolution**: Explicit precedence rules (client wins)

### Why Bundle Manifest Hash?
- **Reproducibility**: Same hash = same scenario selection
- **Change detection**: Any scenario modification changes hash
- **Audit compliance**: Prove which scenarios were tested at time T
- **Rollback safety**: Can reproduce historical audits exactly

---

## Migration from Phase 0/1

### What Changes
| Component | Phase 0/1 | Phase 2 |
|-----------|-----------|---------|
| Scenario selection | Ad-hoc file paths | Bundle manifest (SHA-256) |
| Judgement timing | Inline during run | Offline batch processing |
| Client customisation | Fork scenarios | Override manifests |
| Translation | Manual pre-translation | Translation engine + manifest |
| Evidence format | Individual files | Artefact bundles |
| Reproducibility | Git commit only | Full provenance chain |

### What Stays Same
- Scenario card schema v1.2 (extends, doesn't replace)
- Transcript format (full messages, no truncation)
- Evidence pack structure (adds manifest fields)
- behaviour_json_v1 schema (adds provenance fields)

### Upgrade Path
1. Existing scenarios remain valid (backward compatible)
2. Add bundle compiler as optional layer (gradual rollout)
3. Migrate inline judgement to offline (per-client opt-in)
4. Add translation manifests to multilingual runs (additive)

---

## Phase 2 Deliverables

### Specifications (This Repo)
- [x] System overview (this document)
- [ ] Artefact model + run manifest spec
- [ ] Scenario catalogue + client overrides + bundle compiler
- [ ] Offline judgement stage spec
- [ ] Translation & localisation spec
- [ ] Reporting & exports spec
- [ ] ADR template for architectural decisions

### Implementation (Aigov-eval Repo)
- [ ] Bundle compiler CLI
- [ ] Translation engine + manifest generator
- [ ] Offline judge batch processor
- [ ] Artefact bundle validator
- [ ] Provenance chain verification

### Validation (Both Repos)
- [ ] End-to-end test with client overrides
- [ ] Translation provenance verification
- [ ] Bundle manifest reproducibility test
- [ ] Offline judgement fail-closed test
- [ ] Report generation with full provenance

---

## Success Criteria

Phase 2 is complete when:
1. ✅ All specs published with v0.1 version tag
2. ✅ Bundle compiler produces deterministic manifests
3. ✅ Offline judge processes batches with fail-closed semantics
4. ✅ Translation manifest preserves source text provenance
5. ✅ Client overrides merge without catalogue forks
6. ✅ Full provenance chain validates end-to-end
7. ✅ L2 reports include manifest hashes in annexes

---

## Related Documents

- [Artefact model + run manifest spec](./phase2-artefact-model-v0.1.md)
- [Scenario catalogue + bundle compiler spec](./phase2-scenario-bundle-v0.1.md)
- [Offline judgement spec](./phase2-offline-judgement-v0.1.md)
- [Translation & localisation spec](./phase2-translation-v0.1.md)
- [Reporting & exports spec](./phase2-reporting-v0.1.md)
- [Data contracts v0.1](./data-contracts-v0.1.md) (extends this spec)
- [Eval harness contract v0.1](./eval-harness-contract-v0.1.md) (superseded by Phase 2)

---

**Document Status**: v0.1 Draft Specification
**Last Updated**: 2025-12-28
**Next Review**: After Phase 2 implementation complete
