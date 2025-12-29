# AiGov Specs Index

## Phase 0/1 Specs (Foundation)

Key specs and canonical artifacts:

- Scenario card schema: [schemas/scenario_card/scenario-card-schema-v1.2.md](../schemas/scenario_card/scenario-card-schema-v1.2.md)
- Scenario template: [scenarios/templates/scenario-template-v1.2.yaml](../scenarios/templates/scenario-template-v1.2.yaml)
- GDPR evaluation criteria: [schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml](../schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml)
- Variable registry: [docs/specs/variable-registry.md](./specs/variable-registry.md)
- File registry: [docs/specs/file-registry.md](./specs/file-registry.md)
- Data contracts: [docs/specs/data-contracts-v0.1.md](./specs/data-contracts-v0.1.md)
- Eval harness contract (transcript-first): [docs/specs/eval-harness-contract-v0.1.md](./specs/eval-harness-contract-v0.1.md)
- Contracts + links (transcript-first): [docs/contracts/_index.md](./contracts/_index.md)
- Client intake variables: [docs/specs/client-intake-variables.md](./specs/client-intake-variables.md)

## Phase 2 Specs (Production Architecture)

**👉 [Phase 2 Index & Overview](./specs/phase2-index.md)** - Complete guide to Phase 2 evaluation system

Core specifications:

- **System overview**: [docs/specs/phase2-system-overview-v0.1.md](./specs/phase2-system-overview-v0.1.md) - Phase 2 architecture principles and component responsibilities
- **Artefact model + run manifest**: [docs/specs/phase2-artefact-model-v0.1.md](./specs/phase2-artefact-model-v0.1.md) - Immutable artefact bundles and provenance tracking
- **Scenario catalogue + bundle compiler**: [docs/specs/phase2-scenario-bundle-v0.1.md](./specs/phase2-scenario-bundle-v0.1.md) - Client overrides and deterministic bundle compilation
- **Offline judgement**: [docs/specs/phase2-offline-judgement-v0.1.md](./specs/phase2-offline-judgement-v0.1.md) - Locked judge governance, fail-closed batch processing, and replay
- **Translation & localisation**: [docs/specs/phase2-translation-v0.1.md](./specs/phase2-translation-v0.1.md) - Multi-language testing with provenance and evidence anchoring
- **Reporting & exports**: [docs/specs/phase2-reporting-exports-v0.1.md](./specs/phase2-reporting-exports-v0.1.md) - L2 reports with full provenance and GRC platform integration

## Architectural Decision Records

Templates and examples for documenting one-way door decisions:

- **ADR template**: [docs/decisions/ADR-TEMPLATE.md](./decisions/ADR-TEMPLATE.md) - Template for architectural decision records (one-way vs two-way doors)
- **ADR-001 (example)**: [docs/decisions/ADR-001-EXAMPLE-offline-judgement-one-way-door.md](./decisions/ADR-001-EXAMPLE-offline-judgement-one-way-door.md) - Example ADR for offline judgement architecture
