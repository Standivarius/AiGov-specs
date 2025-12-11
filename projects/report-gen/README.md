# ReportGen - Report Generation Engine

## Overview
Generates complete audit reports from Judge findings: L1/L2/L3, framework annexes, recommendations, GRC exports.

## Purpose
**Reports are the starting point** for client discovery calls (not end product).

## Report Structure

### Core Reports
- **L1 (Board/Executive)**: 5 pages - Business impact, priority recommendations, budget
- **L2 (Compliance/Legal)**: 15-20 pages - Detailed violations, article analysis, remediation plans
- **L3 (Technical/CISO)**: 40-60 pages - Technical deep-dive, code fixes, implementation roadmap

### Framework Annexes
- **GDPR Annex**: Article-by-article compliance table
- **ISO 27001 Annex**: Control gap summary (Annex A)
- **ISO 42001 Annex**: AI-specific control assessment
- **EU AI Act Annex**: Risk classification + requirements

### Audit Preparation Docs
- Required Documents List (30-40 items)
- Evidence Collection Timeline (Gantt)
- Interview Scripts (CISO, DPO, DevOps)

### GRC Exports
- **OneTrust**: Risk register CSV format
- **Vanta**: Control evidence JSON
- **VeriifyWise**: Assessment results API sync

## Status
🟡 **In Progress** (Template research)

## Key Decisions
- **Template Tool**: Research Jinja2, Docxtpl, Carbone (TBD)
- **Languages**: EN + RO initially
- **Format**: PDF + DOCX exports

## Links
- [TASKS.md](TASKS.md) - Implementation checklist
- [RESEARCH.md](RESEARCH.md) - Template tool comparison
- [STATUS.md](STATUS.md) - Current state