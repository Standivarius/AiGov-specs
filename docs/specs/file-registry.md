# AIGov File Registry

**Purpose**: Central tracking of all project files, dependencies, and ownership  
**Status**: Living document - update when files added/removed/renamed  
**Last Updated**: 2025-12-19

---

## Governance Files

| File Path | Purpose | Owner | Version | Dependencies |
|-----------|---------|-------|---------|-------------|
| `/docs/project-principles.md` | Development philosophy, quality standards | Marius/Claude | v1.0 | None |
| `/docs/project-governance.md` | Living documents, consistency checks | Claude | v1.0 | None |
| `/docs/_index.md` | Specs index and canonical entry points | Claude | v0.1 | None |
| `/docs/specs/variable-registry.md` | Variable governance | Claude | v1.0 | All schemas |
| `/docs/specs/file-registry.md` | File tracking (this file) | Claude | v1.1 | None |
| `/docs/specs/client-intake-variables.md` | Client onboarding inputs | Claude | v1.0 | scenario-card-schema-v1.2 |
| `/docs/specs/eval-harness-contract-v0.1.md` | Eval harness contract (transcript-first) | Claude | v0.1 | Aigov-eval minimal loop |
| `/docs/decisions/2025-12-19_specs_repo_structure_cleanup_v0.1.md` | Repo structure cleanup decision | Claude | v0.1 | docs/_index.md |

---

## Custom Instructions (For userPreferences)

| File Path | Purpose | Location | Status |
|-----------|---------|----------|--------|
| `/docs/custom-instructions/memory.md` | Project context for Memory section | Project Settings | Active |
| `/docs/custom-instructions/instructions.md` | Behaviors for Instructions section | Project Settings | Active |
| `/docs/custom-instructions/files.md` | Quick reference for Files section | Project Settings | Active |

---

## Schema Files

| File Path | Purpose | Owner | Version | Dependencies |
|-----------|---------|-------|---------|-------------|
| `/schemas/scenario_card/scenario-card-schema-v1.2.md` | Scenario test specification | Marius/Claude | v1.2 | variable-registry.md |
| `/schemas/behaviour_json_v0_phase0.schema.json` | Judge output format (Phase 0) | Claude | v0 | scenario-card-schema-v1.2.md |
| `/docs/specs/data-contracts-v0.1.md` | Pipeline data interfaces | Claude | v0.1 | behaviour_json schema |

---

## Evaluation Criteria Files

| File Path | Purpose | Based On | Status |
|-----------|---------|----------|--------|
| `/schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml` | GDPR Judge rubric | Art.24, Art.32, supervisory authority standards | Active |
| `/schemas/evaluation_criteria/iso27001-evaluation-criteria-v1.0.yaml` | ISO 27001 Judge rubric | ISO 27001:2022 controls | Phase 1 |
| `/schemas/evaluation_criteria/iso42001-evaluation-criteria-v1.0.yaml` | ISO 42001 Judge rubric | ISO 42001:2023 AI management | Phase 1 |

---

## Scenario Files

| File Path | Purpose | Framework | Status |
|-----------|---------|-----------|--------|
| `/scenarios/gdpr/scenario_001_third_party_pii.yaml` | Third-party PII disclosure test | GDPR | Pending creation |
| `/scenarios/gdpr/scenario_007_healthcare_email.yaml` | Healthcare email leak test | GDPR | Pending creation |
| `/scenarios/templates/scenario-template-v1.2.yaml` | Template for new scenarios | All | Active |

---

## Research Files

| File Path | Purpose | Format | Updated |
|-----------|---------|--------|--------|
| `/scenarios/research/gdpr-court-violations-research-2025-12-15.md` | Court case research | Markdown | 2025-12-15 |
| `/scenarios/research/gdpr-violations-research-2025-12-15.md` | Violation database research | Markdown | 2025-12-15 |

---

## Planning Files

| File Path | Purpose | Audience | Update Frequency |
|-----------|---------|----------|------------------|
| `/docs/planning/Phase-0-Detailed.md` | Phase 0 execution plan | Marius | Weekly |
| `/docs/planning/Phase-0-Integration-Summary.md` | Eval system integration | Marius | As needed |
| `/docs/planning/Decision-Log.md` | Architectural decisions (ADRs) | Both | Per decision |
| `/docs/planning/Master-Plan-v3.md` | Overall project roadmap | Marius | Monthly |

---

## Test Files (Aigov-eval repo)

| File Path | Purpose | Status |
|-----------|---------|--------|
| `/tests/judge/test_j01_consistency.py` | Judge consistency test | Active |
| `/tests/judge/test_j02_schema.py` | Schema compliance test | Active |
| `/tests/judge/test_j03_accuracy.py` | Accuracy vs MOCK_LOG test | Active |

---

## Naming Conventions

### Scenario Files
**Format**: `scenario_NNN_descriptive_name.yaml`  
**Examples**: `scenario_007_healthcare_email.yaml`, `scenario_015_rtbf_failure.yaml`

### Research Files
**Format**: `topic-research-YYYY-MM-DD.md`  
**Examples**: `gdpr-violations-research-2025-12-15.md`, `iso27001-controls-research-2025-12-20.md`

### Schema Files
**Format**: `schema_name_vX.Y.format`  
**Examples**: `behaviour_json_v0_phase0.schema.json`, `scenario-card-schema-v1.2.md`

### Test Files
**Format**: `test_component_name.py`  
**Examples**: `test_j01_consistency.py`, `test_report_generation.py`

---

## Dependencies Matrix

```
scenario_card.yaml
  -> schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml (Judge reads)
  -> variable-registry.md (field definitions)
  -> behaviour_json_v0_phase0.schema.json (Judge outputs)

behaviour_json_v0_phase0.schema.json
  -> scenario_card.yaml (finding_id references scenario_id)
  -> report templates (L2 report consumes findings)

schemas/evaluation_criteria/gdpr-evaluation-criteria-v1.0.yaml
  -> Official GDPR Art.24, Art.32, supervisory authority standards

variable-registry.md
  -> scenario-card-schema-v1.2.md
  -> behaviour_json schema
  -> client-intake-variables.md

project-principles.md
  -> Project Knowledge (read every new chat)

project-governance.md
  -> project-principles.md
  -> file-registry.md
  -> variable-registry.md
```

---

## File Lifecycle

### Adding New File
1. Check file-registry.md for naming conflicts
2. Follow naming conventions
3. Add entry to appropriate section
4. Update dependencies matrix if applicable
5. Commit with message: "Add [file]: [purpose]"

### Removing File
1. Check dependencies matrix (will other files break?)
2. Remove from file-registry.md
3. Update dependent files if needed
4. Commit with message: "Remove [file]: [reason]"

### Renaming File
1. Update all references in dependent files
2. Update file-registry.md
3. Commit with message: "Rename [old] → [new]: [reason]"

---

## Orphan Detection

**Run at end of each project week**: Search repo for files NOT in file-registry.md

```bash
# Find all YAML files
find . -name "*.yaml" -o -name "*.yml"

# Find all Markdown files
find . -name "*.md"

# Compare against file-registry.md
# Flag orphans for review
```

---

## Change Log

| Date | Version | Changes | Impact |
|------|---------|---------|--------|
| 2025-12-15 | 1.0 | Initial registry created | All files |
| 2025-12-16 | 1.1 | Added governance files, custom instructions, updated dependencies | Schema v1.2 ecosystem |
| 2025-12-19 | 1.2 | Added eval harness contract | Specs/eval interface |

---

**Maintenance**: Update this registry when files are added/removed/renamed  
**Review**: Weekly consistency check (compare registry vs actual files)




