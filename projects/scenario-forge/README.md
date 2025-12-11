# ScenarioForge - Scenario Pipeline Creation

## Overview
Pipeline for creating audit scenarios from EDPB enforcement cases, mapping to framework violations, and generating test definitions.

## Purpose
- Convert real-world violations (EDPB cases) into executable Petri scenarios
- Attach framework-specific legal texts (GDPR, ISO, AI Act)
- Store as structured folders (NOT in AKG)

## Status
🔴 **Not Started**

## Architecture

### Scenario Storage Structure
```
scenarios/
├── scenario-001-email-leak/
│   ├── scenario.json (executable definition)
│   ├── scenario_interpretation.md (human analysis)
│   ├── gdpr-articles.md (relevant GDPR text)
│   ├── iso27001-controls.md
│   ├── iso42001-controls.md
│   ├── ai-act-articles.md
│   ├── ro-law-190.md (national overlay)
│   └── test-transcripts/ (validation examples)
└── scenario-002-rtbf-failure/
    └── ...
```

### Pipeline Steps
1. **Taxonomy First**: Build framework infringement groups (know what we're looking for)
2. **Manual Creation**: Create 1-2 scenarios by hand (understand workflow)
3. **Tool Selection**: Research pipeline frameworks (Prefect, Dagster, n8n, or bash scripts)
4. **Automation**: Build create_scenario.py tool
5. **Validation**: validate_scenario.py (test against sandbox LLM)

## Key Decisions
- **Scenarios NOT in AKG**: Separate file structure (ADR-0004)
- **Taxonomy BEFORE Pipeline**: Know what to look for before automating
- **Manual First**: Build real scenarios by hand, then automate
- **Synthetic Scenarios**: Cover edge cases (e.g., discrimination - rare in real cases)

## Links
- [TASKS.md](TASKS.md) - Implementation checklist
- [RESEARCH.md](RESEARCH.md) - Pipeline tool comparison
- [STATUS.md](STATUS.md) - Current state