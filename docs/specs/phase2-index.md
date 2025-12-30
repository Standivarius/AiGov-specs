# Phase 2 Evaluation System Specifications

**Version**: 0.1 (Draft)
**Status**: Under Development
**Last Updated**: 2025-12-28

---

## Overview

Phase 2 specifications define the production-grade evaluation architecture for systematic GDPR compliance testing. This phase transitions from MVP (Phase 0/1) to a scalable, reproducible platform with:

- **Artefact-driven evaluation**: Immutable evidence bundles with full provenance tracking
- **Locked judge governance**: Fail-closed offline judgement with version control
- **Client-specific customisation**: Override scenarios without forking the catalogue
- **Multi-language support**: Translation with provenance and bilingual evidence
- **GRC integration**: Automated exports to OneTrust, Vanta, and other platforms

---

## Core Architecture Specifications

### 1. System Overview
**[phase2-system-overview-v0.1.md](./phase2-system-overview-v0.1.md)**

Entry point for Phase 2 architecture. Covers:
- Architecture principles (fail-closed, provenance chain, client isolation)
- Component responsibilities (scenario catalogue, bundle compiler, translation engine, etc.)
- Data flow diagram
- Key design decisions and migration path from Phase 0/1

**Start here** if you're new to Phase 2 architecture.

### 2. Artefact Model + Run Manifest
**[phase2-artefact-model-v0.1.md](./phase2-artefact-model-v0.1.md)**

Defines immutable artefact bundles and provenance tracking:
- Artefact types (bundle, run, judgement, translation)
- Run manifest schema (full provenance chain)
- Bundle manifest schema (deterministic hash calculation)
- Translation manifest (no double translation enforcement)
- Judge manifest (locked judge governance)
- Validation requirements and checksum verification

**Critical for**: Understanding artefact lifecycle and reproducibility guarantees.

### 3. Scenario Catalogue + Bundle Compiler
**[phase2-scenario-bundle-v0.1.md](./phase2-scenario-bundle-v0.1.md)**

Client override system and deterministic bundle compilation:
- Scenario catalogue structure (base + client overrides)
- Override types (full replace, partial patch, custom scenarios)
- Bundle compiler process and merge algorithm
- Validation rules and conflict detection
- Reproducibility guarantees (deterministic hash)

**Critical for**: Client customisation and scenario management.

### 4. Offline Judgement
**[phase2-offline-judgement-v0.1.md](./phase2-offline-judgement-v0.1.md)**

Two-stage evaluation with locked judge governance:
- Run stage (evidence capture only, NO scoring)
- Judgement stage (offline batch processing)
- Judge contract v0.1 with version lock
- Fail-closed semantics and error handling
- Batch processing with parallel API calls
- Replay workflow (re-judge with upgraded models)

**Critical for**: Cost optimization, reproducibility, and audit compliance.

### 5. Translation & Localisation
**[phase2-translation-v0.1.md](./phase2-translation-v0.1.md)**

Multi-language testing with provenance tracking:
- **No double translation** principle (source preservation)
- **Evidence anchoring** (bilingual findings with snippet_translated + snippet_source)
- Translation workflow and validation rules
- Quality review process
- Supported language pairs (EN→RO, EN→DE, EN→FR)

**Critical for**: Multi-language compliance testing with audit trail.

### 6. Reporting & Exports
**[phase2-reporting-exports-v0.1.md](./phase2-reporting-exports-v0.1.md)**

L2 reports with full provenance and GRC integration:
- L2 report structure with provenance annexes
- GRC platform exports (OneTrust CSV, Vanta JSON)
- L3 JSON schema (full artefact dump for third-party audit)
- Report generation pipeline
- Client branding and customisation

**Critical for**: Client deliverables and GRC platform integration.

---

## Supporting Documentation

### Architectural Decision Records (ADRs)

**[ADR-TEMPLATE.md](../decisions/ADR-TEMPLATE.md)**
Template for documenting architectural decisions, especially **one-way doors** (irreversible decisions).

**[ADR-001-EXAMPLE-offline-judgement-one-way-door.md](../decisions/ADR-001-EXAMPLE-offline-judgement-one-way-door.md)**
Example ADR demonstrating the offline judgement decision (one-way door with full justification).

### Contribution Guidelines

**[CONTRIBUTING.md](../../CONTRIBUTING.md)**
Development workflow and PR checklist with critical reminder:
> **When updating code/config in Aigov-eval, ALWAYS update corresponding specs in AiGov-specs.**

Includes:
- Specs-first development principle
- PR checklist for consistency
- Synchronization guidelines
- Spec writing standards (versioning, breaking changes)

---

## Locked decisions (Phase 2)

Architectural decisions that define Phase 2 governance boundaries:

- **[ADR-002: Judge–AKG Responsibility Boundary](../decisions/ADR-002-judge-akg-responsibility-boundary-v0.1.md)** - Defines verdict ownership (Judge owns verdict, AKG provides grounding context), establishes audit accountability, and sets limits on knowledge graph responsibility

---

## Phase 2 Design Principles

### 1. Fail-Closed by Default
Judge models are **locked at specification time** via configuration manifest. Evaluation runs fail if judge dependencies unavailable (**no silent degradation**).

### 2. Provenance Chain
Every artefact includes:
- Git commit hash of scenario catalogue
- Bundle manifest hash for scenario selection
- Translation manifest for language transformations
- Judge contract version for reproducibility

### 3. Client Data Isolation
- Client-specific overrides in separate namespace (`client_configs/`)
- Base scenario catalogue remains canonical
- Bundle compiler merges deterministically

### 4. Evidence-First Workflow
```
Scenario Bundle → Run (transcript only) → Judgement (offline) → Report
```
Clear separation between evidence capture and interpretation.

---

## Key Contracts (Locked)

### Judge Output Contract
**Canonical source**: [data-contracts-v0.1.md](./data-contracts-v0.1.md)

All Phase 2 specs reference `behaviour_json_v1` schema from data-contracts-v0.1.md. This is the **locked judge output format** used by:
- Offline judgement stage
- Report generator
- GRC exports

**Do NOT duplicate this schema** in other specs. Always reference data-contracts-v0.1.md.

### Translation Contract
**Principles** (enforced across all translation specs):
1. **No double translation**: Use `translation_metadata` to detect already-translated content
2. **Evidence anchoring**: All findings include `snippet_translated` + `snippet_source` fields
3. **Provenance tracking**: Every translation includes model version, timestamp, and source file reference

### Scenario Bundle Contract
**Separation of concerns**:
- Scenario bundles define **test cases** (what to test)
- Judge output defines **findings** (test results)
- These are **separate contracts** with no overlap

---

## Implementation Status

| Component | Specification | Implementation | Status |
|-----------|---------------|----------------|--------|
| System Overview | ✅ v0.1 | Aigov-eval | 🔴 Not Started |
| Artefact Model | ✅ v0.1 | Aigov-eval | 🔴 Not Started |
| Bundle Compiler | ✅ v0.1 | Aigov-eval | 🔴 Not Started |
| Offline Judge | ✅ v0.1 | Aigov-eval | 🔴 Not Started |
| Translation Engine | ✅ v0.1 | Aigov-eval | 🔴 Not Started |
| Report Generator (Phase 2) | ✅ v0.1 | report-gen | 🔴 Not Started |

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
2. Bundle compiler as optional layer (gradual rollout)
3. Migrate inline judgement to offline (per-client opt-in)
4. Add translation manifests to multilingual runs (additive)

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

## Related Documentation

### Phase 0/1 Specs (Foundation)
- [Data contracts v0.1](./data-contracts-v0.1.md) - Canonical judge output schema (behaviour_json_v1)
- [Eval harness contract v0.1](./eval-harness-contract-v0.1.md) - Phase 0/1 contract (superseded by Phase 2)
- [Scenario card schema v1.2](../../schemas/scenario_card/scenario-card-schema-v1.2.md)

### Master Index
- [AiGov Specs Index](../_index.md) - All specs organized by phase

---

**Document Status**: v0.1 Landing Page
**Last Updated**: 2025-12-28
**Next Review**: After Phase 2 implementation begins
