# ScenarioForge - Research Findings

## Framework Taxonomy Research
**Status**: Pending  
**Date**: TBD

### GDPR Infringement Groups
[TBD: Structure to be documented after creation]

---

## Pipeline Tool Research
**Status**: Pending

### Evaluation Criteria
- Workflow orchestration (DAG visualization)
- LLM integration support (API calls to Claude/GPT)
- Retry logic (if AKG query fails)
- Self-hostable (open source)
- Complexity (overkill for solo founder?)

### Candidates

#### Prefect
- **Pros**: [TBD]
- **Cons**: [TBD]

#### Dagster
- **Pros**: [TBD]
- **Cons**: [TBD]

#### Bash Scripts + Make
- **Pros**: Simple, no overhead
- **Cons**: No DAG viz, manual orchestration

### Decision
**Winner**: TBD after manual creation reveals needs

---

## scenario_interpretation Research
**Status**: Pending

### Purpose
Bridge between legal analysis (EDPB case) and technical test (Petri scenario).

### Structure (Proposed)
```markdown
# Scenario Interpretation: Email Leak

## Incident Summary
[What happened in EDPB case]

## Violation Analysis
[Why it's a GDPR breach]

## LLM Behavior Mapping
[How to replicate in test]

## Expected Outcome
[What Judge should detect]

## Edge Cases
[Ambiguous situations]
```

### Test Plan
Create 1 example scenario with scenario_interpretation.md, evaluate if it aids clarity.

---

**Links to GDrive**: [Will add after Gemini research]