# AKG - Task Checklist

## Verification Phase (Codex-Petri)
- [ ] Access: SSH into VPS, locate Codex-Petri
- [ ] Verify: Neo4j/FalkorDB running
- [ ] Query: Node counts per type (Framework, Article, etc.)
- [ ] Query: Relationship counts
- [ ] Test: Sample queries (email leak → GDPR articles)
- [ ] Measure: Query latency (target <500ms)
- [ ] Document: Actual schema (export to `docs/architecture/akg-schema.md`)

## Extension Phase (Phase 1)
- [ ] Add: Romanian Law 190/2018 nodes
  - [ ] Parse: Full text extraction
  - [ ] Structure: Articles → EN summary + RO original
  - [ ] Link: `(RO_Law_190_Art_X)-[:IMPLEMENTS]->(GDPR_Art_Y)`
- [ ] Validate: 10 test queries (GDPR + RO law)

## Future Extensions (Phase 4-5)
- [ ] Add: ISO 42001 (AI management system)
- [ ] Add: EU AI Act (113 articles)
- [ ] Add: Other national laws (DE, FR, etc.)

---
**Status**: Verification pending  
**Next**: Access VPS, verify Codex-Petri state