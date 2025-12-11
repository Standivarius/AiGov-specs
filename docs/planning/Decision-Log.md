# AIGov Decision Log

**Purpose**: Chronological record of major decisions, rationale, and context.

---

## 2025-12-11

### GitHub Structure Initialization
**Decision**: Use GitHub as single source of truth for engineering plans, NOT duplicate to Claude Project Knowledge  
**Rationale**: Avoid dual maintenance, GitHub already version-controlled, Claude can read directly  
**Impact**: Project Knowledge only for conversation summaries, not plan documents  

### Aigov-eval Separate Repository
**Decision**: Create separate `Aigov-eval` repo for evaluation system (not folder in Aigov-specs)  
**Rationale**: Eval-app is standalone product, can evolve independently, reusable for other compliance tools  
**Implementation**: https://github.com/Standivarius/Aigov-eval  
**Impact**: Clean separation of concerns, parallel development possible  

### Dashboard Technology: HTML + Tailwind
**Decision**: Build dashboard mockup as single HTML file with Tailwind CDN (not Figma)  
**Rationale**: Least friction (no install, double-click to open), can evolve to functional later, executable immediately  
**Location**: `projects/dashboard/static/dashboard-mockup.html`  
**Impact**: Faster iteration, no tool lock-in  

### IntakeAgent as Claude Skill Pattern
**Decision**: Build onboarding questionnaire as Claude Skill, NOT deterministic form platform  
**Rationale**: AI-dynamic branching, minimizes client friction (auto-extracts from docs), flexible adaptation  
**Alternative Rejected**: Typeform/Fillout/Tally (no LLM integration, rigid branching)  
**Impact**: More development but better UX, migrate to Agent SDK later if needed  

### Scenarios NOT Stored in AKG (ADR-0004)
**Decision**: Store scenarios as file structure (`scenarios/scenario-001-email-leak/`), NOT as AKG nodes  
**Rationale**: Scenarios = test definitions (what we're testing), AKG = legal knowledge (what law says), separation prevents coupling  
**From**: ChatGPT File 2 discussion  
**Impact**: Cleaner architecture, AKG role is validation only ("Does X violate Y?")  

### Translation Architecture (ADR-0001)
**Decision**: Canonical English pipeline, translation at I/O edges only, single multilingual LLM (not separate translator)  
**Rationale**: Avoid context loss from handoff to separate translation LLM, Judge handles RO→EN→RO internally  
**Impact**: Simpler architecture, preserved context across translation  

### Synthetic Target LLM Approach
**Decision**: Use standard LLM + instructional layer ("You are Target in Petri test") instead of fine-tuned model  
**Rationale**: Fast iteration, flexible violation simulation, no red flag concerns, reasoning output aids scenario development  
**Implementation**: Prompt engineering, not training  
**Impact**: Phase 0 viable without ML infrastructure  

### LLM Council: Latest Models Only
**Decision**: Use GPT-4.5.1 (not 4.0), Gemini 2.0 (not 1.5) for council voting  
**Rationale**: Latest models = best performance, avoid stale baselines  
**Impact**: Test harness needs regular model updates  

### Framework Taxonomy BEFORE Scenario Pipeline
**Decision**: Build GDPR infringement taxonomy (groups, subgroups) before automating scenario creation  
**Rationale**: "Know what we're looking for" prevents scattered scenario development  
**Impact**: 2-3 days taxonomy work precedes pipeline tooling  

### Eval-app Naming
**Decision**: Call evaluation system "Eval-app" (not Verifier, TestSuite, QA, Validator)  
**Rationale**: Marius preference, consistent with other tool names (-Agent, -Forge, -Gen pattern less applicable here)  
**Impact**: All docs reference Eval-app  

---

## Template for Future Entries

```markdown
## YYYY-MM-DD

### [Decision Title]
**Decision**: [What was decided]  
**Rationale**: [Why this decision]  
**Alternatives Considered**: [What was rejected and why]  
**Impact**: [How this affects project]  
**References**: [Links to ADRs, discussions, etc.]  
```

---

**Maintenance Notes**:
- Add entries chronologically (newest at bottom)
- Link to ADRs for architectural decisions
- Keep entries concise (2-4 sentences per field)
- Reference source (ChatGPT discussion, Notion, etc.)