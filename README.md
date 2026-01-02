# AIGov - AI Governance Audit System

**Master Repository for Architecture, Specifications, and Planning**

## 🎯 Quick Start

**Current Phase**: Phase 0 - Foundation (Report Templates, IntakeAgent, Dashboard, Mock Target)

**Quick Links**:
- [📋 Master Plan v3](docs/planning/Master-Plan-v3.md) - Complete project plan
- [🚀 Phase 0 Details](docs/planning/Phase-0-Detailed.md) - Current phase deliverables
- [📝 Decision Log](docs/planning/Decision-Log.md) - Chronological decision history
- [Eval harness contract (transcript-first)](docs/specs/eval-harness-contract-v0.1.md) - Interface between specs and eval
- [Contracts + links (transcript-first)](docs/contracts/_index.md) - Scenario, target, and evidence contracts
- [📊 GitHub Projects Board](https://github.com/users/Standivarius/projects) - Task tracking
- [🔬 Research Index](https://drive.google.com/drive/folders/1CHKXcmgKRpieDUDC2TmKchHCvPgSBgeCc) - Deep research docs

## 📂 Repository Structure

```
Aigov-specs/
├── README.md ⭐ (YOU ARE HERE - Master entry point)
├── docs/
│   ├── planning/          # Master plan, phase details, decisions
│   ├── architecture/      # C4 diagrams, system overview
│   ├── adr/              # Architectural Decision Records
│   └── specs/            # Schemas, report templates, prompts
└── projects/             # Sub-project organization
    ├── intake-agent/     # Onboarding questionnaire + doc extraction
    ├── scenario-forge/   # Scenario pipeline creation
    ├── judge/           # Transcript → violation mapper
    ├── report-gen/      # L1/L2/L3 report generation
    ├── eval-app/        # Evaluation & testing (separate repo)
    ├── dashboard/       # Central control panel
    ├── akg/            # Autonomous Knowledge Graph
    └── rag/            # Retrieval-Augmented Generation corpus
```

## 🏗️ Sub-Projects

| Project | Status | Description |
|---------|--------|-------------|
| [IntakeAgent](projects/intake-agent/) | 🔴 Not Started | AI-dynamic questionnaire + document extraction |
| [ScenarioForge](projects/scenario-forge/) | 🔴 Not Started | Scenario pipeline with framework taxonomy |
| [Judge](projects/judge/) | 🔴 Not Started | Multilingual transcript analysis |
| [ReportGen](projects/report-gen/) | 🟡 In Progress | L1/L2/L3 + annexes + recommendations |
| [Eval-app](https://github.com/Standivarius/Aigov-eval) | 🔴 Not Started | Systematic testing framework |
| [Dashboard](projects/dashboard/) | 🟡 In Progress | Static mockup → functional control panel |
| [AKG](projects/akg/) | 🟢 Existing | Knowledge graph (Codex-Petri) |
| [RAG](projects/rag/) | 🟢 Existing | Legal corpus (CC-Petri) |

## 📊 Current Status

**Phase 0 Goals**:
- ✅ Repository structure initialized
- 🔄 Report template suite (L1/L2/L3 + annexes + recommendations)
- 🔄 Dashboard static mockup (HTML + Tailwind)
- ⏳ IntakeAgent (questionnaire + doc extraction)
- ⏳ Mock Target LLM (synthetic testing)

**Next Milestone**: Complete Phase 0 deliverables → First Tracer Bullet (Phase 1)

## 🔗 Related Repositories

- **[Aigov-eval](https://github.com/Standivarius/Aigov-eval)** - Evaluation system (Eval-app)
- **aigov-codex-petri** - AKG implementation (local)
- **aigov-cc-petri** - RAG implementation (local)

## 📚 Documentation Navigation

**For Business Context**: See [Notion AIGov-INDEX](https://www.notion.so/) (strategic overview, revenue targets, partnerships)

**For Engineering Details**: This repo (GitHub is source of truth)

**For Deep Research**: See [Google Drive Research Work](https://drive.google.com/drive/folders/1CHKXcmgKRpieDUDC2TmKchHCvPgSBgeCc)

## 🤝 Contributing

This is a solo founder project with occasional partner input (Ally - Nokia CISO). Structure optimized for clarity and least friction.



[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/Standivarius/AiGov-specs)
---

**Last Updated**: 2025-12-11  
**Phase**: 0 - Foundation  
**Next Review**: After Phase 0 completion
