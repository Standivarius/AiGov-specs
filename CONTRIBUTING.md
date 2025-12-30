# Contributing to AiGov-specs

Thank you for contributing to the AiGov specifications repository! This document outlines guidelines for maintaining high-quality, synchronized specifications.

---

## Core Principle: Specs-First Development

**Specifications are the source of truth.** All architectural decisions, contracts, and system designs should be documented here before implementation in [Aigov-eval](https://github.com/Standivarius/Aigov-eval) or other repositories.

---

## Contribution Workflow

### 1. Propose Changes

For significant changes (new specs, breaking changes to contracts):
1. Open an issue describing the proposed change
2. If it's an architectural decision (especially a **one-way door**), create an ADR using the [ADR template](docs/decisions/ADR-TEMPLATE.md)
3. Discuss with team/stakeholders before proceeding

For minor changes (typos, clarifications, examples):
1. Open a PR directly with clear description

### 2. Update Specifications

When making changes, ensure:
- **Consistency**: New specs align with existing architecture
- **Completeness**: Include examples, schemas, and validation rules
- **Cross-references**: Link related specs and ADRs
- **Versioning**: Bump version number if breaking changes (v0.1 → v0.2 or v1.0 → v2.0)

### 3. PR Checklist

Before submitting a pull request, verify:

- [ ] **Spec completeness**: All sections filled out (overview, schema, examples, validation)
- [ ] **Cross-repository sync**: If this changes a contract, have you updated implementation in Aigov-eval?
- [ ] **Index updated**: Added new specs to [docs/_index.md](docs/_index.md)
- [ ] **Backward compatibility**: Breaking changes documented in ADR + migration guide
- [ ] **Examples included**: Code examples compile/validate
- [ ] **Version bumped**: Updated version number in spec header (if applicable)
- [ ] **Links verified**: All internal links work (no broken references)

**CRITICAL**: When updating code or config in [Aigov-eval](https://github.com/Standivarius/Aigov-eval), **always update corresponding specs in AiGov-specs**. Specs must stay in sync with implementation.

### 4. Review Process

- Solo founder project with partner input (Ally - Nokia CISO)
- For one-way doors: Require review from 2+ people (architect + product/engineering)
- Merge after approval

---

## Repository Structure

```
AiGov-specs/
├── docs/
│   ├── _index.md                    # Entry point (update when adding specs)
│   ├── contracts/                   # Data contracts between components
│   ├── decisions/                   # Architectural Decision Records (ADRs)
│   │   ├── ADR-TEMPLATE.md         # Template for new ADRs
│   │   └── ADR-001-*.md            # Individual ADRs
│   ├── planning/                    # Master plan, phase details
│   ├── specs/                       # Technical specifications
│   │   ├── phase2-*.md             # Phase 2 specs (v0.1)
│   │   └── data-contracts-v0.1.md  # Data contracts
│   └── workflows/                   # Process documentation
│
├── scenarios/
│   ├── library/                     # Canonical scenario definitions
│   ├── templates/                   # Scenario templates
│   └── _deprecated/                 # Archived scenarios
│
├── schemas/
│   ├── scenario_card/               # Scenario schema
│   └── evaluation_criteria/         # Evaluation criteria
│
├── projects/                        # Sub-project organization
├── research/                        # Research and analysis
└── legal/                           # Legal frameworks (GDPR, etc.)
```

---

## Spec Writing Guidelines

### 1. Use Clear Structure

Every spec should include:
- **Purpose**: One-sentence summary
- **Version**: Semantic version (v0.1, v1.0)
- **Date**: Last updated date
- **Overview**: High-level description
- **Schema/Contract**: Formal definition (JSON schema, YAML example)
- **Examples**: Concrete use cases
- **Validation**: How to verify compliance
- **Related Documents**: Links to related specs/ADRs

### 2. Version Numbering

- **v0.x**: Draft/experimental (Phase 0-2)
- **v1.0**: First stable release (production-ready)
- **v1.x**: Non-breaking additions
- **v2.0**: Breaking changes (requires migration guide)

### 3. Breaking vs Non-Breaking Changes

**Non-breaking** (minor version bump):
- Add optional fields
- Add new enum values
- Clarify ambiguous language
- Add examples

**Breaking** (major version bump):
- Remove required fields
- Change field types
- Rename fields
- Change semantics (e.g., rating scale)
- Requires ADR for one-way doors

### 4. Document One-Way Doors

Use the [ADR template](docs/decisions/ADR-TEMPLATE.md) for:
- Irreversible architectural decisions
- Breaking API changes
- Data model migrations
- External dependency lock-in

Include:
- **Context**: Why this decision?
- **Consequences**: What becomes harder/easier?
- **Alternatives**: What did we reject and why?
- **Approval**: Who signed off (for one-way doors)?

---

## Synchronization with Implementation

### Critical Rule: Specs → Implementation

**Always update specs BEFORE or ALONGSIDE implementation changes.**

**Example workflow**:
1. Need to add new scenario category
2. **First**: Update [phase2-scenario-bundle-v0.1.md](docs/specs/phase2-scenario-bundle-v0.1.md) with new category
3. **Then**: Implement in Aigov-eval
4. **Finally**: Update [docs/_index.md](docs/_index.md) if new spec added

**Anti-pattern** (don't do this):
1. Implement feature in Aigov-eval
2. Forget to update specs
3. Specs drift out of sync → confusion and bugs

### Sync Checklist

When making changes in [Aigov-eval](https://github.com/Standivarius/Aigov-eval):

- [ ] Added new API endpoint → Update relevant contract spec
- [ ] Changed artefact schema → Update [phase2-artefact-model-v0.1.md](docs/specs/phase2-artefact-model-v0.1.md)
- [ ] Modified judge behavior → Update [phase2-offline-judgement-v0.1.md](docs/specs/phase2-offline-judgement-v0.1.md)
- [ ] Changed scenario schema → Update [scenario card schema](../schemas/scenario_card/scenario-card-schema-v1.2.md)
- [ ] New translation feature → Update [phase2-translation-v0.1.md](docs/specs/phase2-translation-v0.1.md)
- [ ] New export format → Update [phase2-reporting-exports-v0.1.md](docs/specs/phase2-reporting-exports-v0.1.md)

---

## Testing Specifications

### 1. Validate Examples

All code examples in specs MUST be valid:

```bash
# Example: Validate Python code in specs
grep -r "```python" docs/specs/*.md | while read file; do
  # Extract code block and validate syntax
  python -m py_compile extracted_code.py
done
```

### 2. Check Cross-References

All internal links MUST resolve:

```bash
# Check for broken links
find docs -name "*.md" -exec markdown-link-check {} \;
```

### 3. Schema Validation

If spec includes JSON schema, validate examples against schema:

```python
from jsonschema import validate

schema = load_schema("docs/specs/phase2-artefact-model-v0.1.md")
example = load_example("docs/specs/phase2-artefact-model-v0.1.md")
validate(instance=example, schema=schema)  # Should pass
```

---

## Style Guide

### Writing Style
- **Concise**: Prefer short sentences and bullet points
- **Technical**: Assume technical audience (engineers, architects)
- **Precise**: Avoid ambiguous terms like "should" or "might" (use MUST, MAY, SHOULD per RFC 2119)
- **English only**: All documentation in English

### Code Examples
- **Realistic**: Use real scenario IDs, field names, etc. (not "foo", "bar")
- **Complete**: Include imports, error handling, etc.
- **Commented**: Explain non-obvious logic

### Formatting
- **Headers**: Use ATX-style (`#`, `##`, `###`)
- **Code blocks**: Always specify language (```python, ```bash, ```json)
- **Lists**: Use `-` for unordered, `1.` for ordered
- **Emphasis**: Use `**bold**` for important terms, `*italic*` for emphasis

---

## Questions or Issues?

- **Spec clarification needed**: Open an issue in this repo
- **Implementation question**: Open an issue in [Aigov-eval](https://github.com/Standivarius/Aigov-eval)
- **Urgent sync issue**: Contact @Standivarius directly

---

**Last Updated**: 2025-12-28
**Version**: 1.0
