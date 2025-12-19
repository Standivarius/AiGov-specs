# Decision Log

**Purpose**: Chronological record of key architectural and strategic decisions.

---

## 2025-12-11

### Repository Structure: Aigov-specs + Aigov-eval
**Decision**: Separate repos for planning/specs vs evaluation system  
**Rationale**: Eval system is standalone tool, reusable for other compliance products  
**Impact**: Clear separation of concerns, easier open-sourcing of eval system  

### Reports Priority: Example Report FIRST
**Decision**: Create L2 report with fake data in Week 1 (before pipeline)  
**Rationale**: Reports needed for client discovery calls immediately, pipeline proves technical feasibility later  
**Impact**: Week 1 focus on report format/flow, Week 3 on real pipeline reports  

### Translation Architecture (ADR-0001)
**Decision**: Canonical English pipeline, translation at I/O edges only, single multilingual LLM (not separate translator)  
**Rationale**: Avoid context loss from handoff to separate translation LLM, Judge handles RO↔EN internally  
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

## 2025-12-14

### Scenario Storage: File-based (ADR-0004)
**Decision**: Store scenarios as YAML files in `/scenarios/gdpr/`, NOT in AKG knowledge graph  
**Rationale**: Scenarios = test definitions, AKG = legal knowledge (separation of concerns)  
**Impact**: Simpler Phase 0, AKG validates scenarios but doesn't store them  
**References**: `/docs/planning/Master-Plan-v3.md`

### Judge as Inspect Scorer (ADR-0005)
**Decision**: Judge runs DURING evaluation as Inspect scorer, not as post-processor  
**Rationale**: Inspect's native architecture, cleaner integration, real-time verdict generation  
**Alternatives Considered**: Post-processing logs (rejected - breaks Inspect patterns, harder debugging)  
**Impact**: Judge implementation follows Inspect scorer API  

---

## 2025-12-15

### Inline Legal Text (ADR-0006)
**Decision**: Embed GDPR articles + EDPB guidance in scenario_card `legal_context` field  
**Rationale**: Phase 0 speed - no AKG/RAG dependency, self-contained scenario_cards, portable tests  
**Alternatives Considered**:  
  - Wait for AKG/RAG (rejected - delays Phase 0 by 2+ weeks)  
  - External legal text files (rejected - breaks self-containment)  
**Implementation**: `legal_context.gdpr_articles[]` + `legal_context.edpb_guidance[]`  
**Phase 1 Transition**: Keep inline text, use AKG/RAG for **additional** context (national law, case law)  
**Impact**: Judge reads inline text, scenario_cards fully self-contained, no database queries Phase 0  
**References**: `/schemas/scenario_card/scenario-card-schema-v1.2.md` v1.2

### MOCK_LOG Format (ADR-0007)
**Decision**: Test_target outputs structured log of intentional violations for accuracy measurement  
**Purpose**: Compare test_target's self-reported violations vs Judge's detected violations → precision/recall  
**Format**: JSON with scenario_id, turn, violation_intent, gdpr_article, pii_type, context  
**Implementation**: System prompt instructs GPT-4o-mini to output MOCK_LOG alongside responses  
**Rationale**: Enables **objective** Judge validation (80%+ detection rate), not subjective review  
**Impact**: TEST-J03 (accuracy) becomes measurable, defensible accuracy claims for client discovery  
**References**: `/docs/planning/Phase-0-Detailed.md` Deliverable 4

### Minimal Eval in Phase 0 (ADR-0008)
**Decision**: Implement TEST-J01 (consistency), TEST-J02 (schema), TEST-J03 (accuracy) in Week 2  
**Rationale**:  
  - Phase 0 success criteria already requires "80%+ Judge accuracy" → can't measure without tests  
  - Ally review becomes technical ("87% detection") vs subjective ("looks good")  
  - Foundation for Phase 1 expanded testing (AKG, RAG, reports, council)  
  - Only 2-3 days effort, doesn't blow up timeline  
  - Core to AIGov value prop: "We validate our audit tool systematically"  
**Alternatives Considered**:  
  - Full eval infrastructure now (rejected - 2+ weeks, competes with core deliverables)  
  - Defer all eval to Phase 1 (rejected - subjective Ally review, no defensibility proof)  
**Implementation**: Simple pytest scripts, no dashboard/council yet  
**Success Criteria**: TEST-J01 (95%+), TEST-J02 (100%), TEST-J03 (80%+)  
**Impact**: Timeline extends to 21-24 days (add 2-3 days Week 2), defensible results for discovery calls  
**References**: `/docs/planning/Phase-0-Detailed.md` Deliverable 7, `Aigov-eval/tests/judge/`

### Simplified behaviour_json Schema (ADR-0009)
**Decision**: Create behaviour_json_v0_phase0 without confidence scoring, AKG/RAG fields, redaction  
**Rationale**: Phase 0 complexity reduction - focus on core: rating, reasoning, violations, evidence  
**Removed Fields**: confidence, akg_context, rag_citations, metadata.redaction_applied, pii_type in evidence  
**Retained Fields**: audit_id, run_id, finding_id, scenario_id, framework, rating, reasoning, violations, inspect_provenance  
**Phase 1 Migration**: Upgrade to behaviour_json_v1 with full fields when AKG/RAG implemented  
**Impact**: Simpler Judge implementation, faster Phase 0 completion, clear upgrade path  
**References**: `/schemas/behaviour_json_v0_phase0.schema.json`

### Real-World GDPR Violations as Source (ADR-0010)
**Decision**: Base scenario_cards on EDPB enforcement decisions + supervisory authority fines, NOT Petri's 111 mapping  
**Rationale**:  
  - Real audit violations → realistic scenarios  
  - Documented evidence (EDPB case numbers, fines) → scenario traceability  
  - LLM-adapted from real cases → credible test coverage claims  
**Research Approach**: 1h Gemini/Perplexity search → curated list → infer scenario seeds  
**Sources**: EDPB enforcement decisions (2020-2024), CNIL/ICO/ANSPDCP fines, chatbot-specific cases  
**Impact**: Scenario_cards cite real enforcement actions (e.g. "Based on EDPB case 2023-FR-00847")  
**References**: `/docs/planning/Phase-0-Detailed.md` Deliverable 1

---

## 2025-12-19

### Eval Harness Contract in Specs (ADR-0011)
**Decision**: Maintain the eval harness contract in AiGov-specs and link out to Aigov-eval runbooks/reports  
**Rationale**: Avoid duplicated runbooks across repos; keep a single source of truth for the interface  
**Impact**: Specs define the contract, eval repo owns implementation and execution details  
**References**: `/docs/specs/eval-harness-contract-v0.1.md`

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
- Add entries chronologically (newest at bottom per date section)
- Link to specs/docs for detailed context
- Keep entries concise (2-5 sentences per field)
- Reference source (conversation, research, partner feedback)

**Last Updated**: 2025-12-19
