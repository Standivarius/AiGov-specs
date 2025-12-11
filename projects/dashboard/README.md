# Dashboard - Central Control Panel

## Overview
Visual interface for managing all AIGov audit parameters and execution.

## Purpose
**During Development**:
- Track all configurable variables (visual reference > text docs)
- See what's implemented vs hardcoded

**During Client Audit**:
- Pre-flight checklist (all params set?)
- Audit execution control (run Petri, generate reports)
- Results viewing (logs, findings, reports)

## Status
🟡 **In Progress** (Static mockup)

## Architecture

### Tabs
1. **Client Profile**: Company, LLM type, frameworks, languages
2. **Petri Configuration**: Scenarios, Target LLM, Auditor LLM settings
3. **Judge Configuration**: Model selection, council, pattern matching
4. **AKG/RAG Configuration**: Database, frameworks, query settings
5. **Report Configuration**: Report types, language, GRC exports
6. **Audit Execution**: Pre-flight check, run controls, live logs
7. **Eval-app**: Test catalog, last run, failed tests

## Implementation Phases

**Phase 1** (🟡 Current): Static Mockup
- Tool: HTML + Tailwind CSS (single file, no build)
- Purpose: Visual reference during development
- Location: `static/dashboard-mockup.html`

**Phase 2** (Future): Functional Dashboard
- Tool: Streamlit (Python) or Next.js (React)
- Features: Read client JSON, call APIs, display live logs
- Trigger: After Phase 0 tools exist

## Design Reference
https://verifywise.ai/images/blog/improved-model-inventory.png
(Tabs on left or top)

## Links
- [TASKS.md](TASKS.md) - Implementation checklist
- [STATUS.md](STATUS.md) - Current state