# ScenarioForge - Task Checklist

## Phase 1: Framework Taxonomy
- [ ] Build: GDPR infringement taxonomy (groups, subgroups)
  - [ ] GROUP 1: Lawfulness & Transparency (Art 5, 6, 13, 14)
  - [ ] GROUP 2: Data Subject Rights (Art 15-22)
  - [ ] GROUP 3: Security & Breach (Art 32, 33, 34)
  - [ ] GROUP 4: Special Categories (Art 9, 10)
  - [ ] GROUP 5: Third-Party & Transfer (Art 44-49, 28)
- [ ] Document: `frameworks/gdpr/taxonomy.md`
- [ ] Create: Machine-readable `infringement-map.json`
- [ ] Validate: With Ally (CISO partner)

## Phase 2: Manual Scenario Creation
- [ ] Select: 2 EDPB enforcement cases (email leak, RTBF)
- [ ] Create: scenario-001-email-leak/ folder structure
  - [ ] Write: scenario.json (executable definition)
  - [ ] Write: scenario_interpretation.md (analysis)
  - [ ] Extract: gdpr-articles.md (relevant text)
  - [ ] Create: test-transcripts/ (validation examples)
- [ ] Create: scenario-002-rtbf-failure/ (same process)
- [ ] Document: Workflow observations (what was hard? what to automate?)

## Phase 3: Pipeline Tool Research
- [ ] Research: Prefect (workflow orchestration)
- [ ] Research: Dagster (data pipeline)
- [ ] Research: Airflow (mature, heavy)
- [ ] Research: Kestra (modern, YAML)
- [ ] Research: n8n (low-code)
- [ ] Test: Build toy pipeline (EDPB case → AKG query → scenario generation)
- [ ] Decide: Framework or bash scripts? (document in RESEARCH.md)

## Phase 4: Automation
- [ ] Build: create_scenario.py
  - Input: EDPB case URL
  - Output: scenario-XXX/ folder with all files
- [ ] Build: validate_scenario.py
  - Test scenario against sandbox LLM
  - Confirm expected violation triggered
- [ ] Test: Create 3 scenarios via automation
- [ ] Iterate: Based on quality

---
**Status**: Not started  
**Next**: Build GDPR taxonomy