# RAG - Task Checklist

## Verification Phase (CC-Petri)
- [ ] Access: SSH into VPS, locate CC-Petri
- [ ] Verify: Vector database running (Pinecone/Weaviate/Qdrant)
- [ ] Query: Document counts per source (EDPB, CJEU, ENISA, etc.)
- [ ] Test: Sample queries (email leak → EDPB cases)
- [ ] Measure: Retrieval latency (target <500ms)
- [ ] Measure: Top-5 relevance quality (manual review of 10 queries)
- [ ] Document: Actual architecture (export to `docs/architecture/rag-architecture.md`)

## Extension Phase (Phase 1)
- [ ] Add: Romanian DPA (ANSPDCP) decisions
  - [ ] Scrape: Recent enforcement cases (2020-2025)
  - [ ] Parse: Extract case details, violations, fines
  - [ ] Ingest: Add to RAG corpus
- [ ] Validate: 10 test queries (GDPR + RO cases)

## Future Extensions (Phase 4-5)
- [ ] Add: ISO 27001 audit guidance (BSI, ENISA)
- [ ] Add: AI Act commentary (legal analysis)
- [ ] Add: Industry-specific guidance (healthcare, finance)

---
**Status**: Verification pending  
**Next**: Access VPS, verify CC-Petri state