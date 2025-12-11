# RAG - Legal Corpus Retrieval

## Overview
Vector search system over legal cases, guidelines, and regulatory commentary for supporting evidence retrieval.

## Purpose
- Store: EDPB guidelines, CJEU rulings, ENISA reports, national DPA decisions
- Query: "Find supporting cases for email leak violations"
- Returns: Relevant citations for report annexes
- Provides: Real-world evidence and legal precedent

## Status
🟢 **Existing** (CC-Petri built locally)

## Architecture

### Document Sources
- **EDPB**: Guidelines, recommendations, enforcement cases
- **CJEU**: European Court of Justice rulings
- **ENISA**: European cybersecurity agency reports
- **National DPAs**: ANSPDCP (Romania), CNIL (France), ICO (UK), etc.
- **Academic**: Legal journals, compliance frameworks

### Vector Search
- Embedding Model: TBD (verify CC-Petri config)
- Database: Pinecone / Weaviate / Qdrant (verify locally)
- Top-K: 5 most relevant documents
- Re-ranking: Optional (LLM-based relevance scoring)

### Example Query
```python
query = "GDPR Article 32 security breach email disclosure"
results = rag.search(query, top_k=5, filters={"source": "EDPB"})
# Returns: [{doc_id, title, excerpt, url, relevance_score}]
```

## Current State
- **Implementation**: CC-Petri (local VPS)
- **Vector DB**: Unknown (needs verification)
- **Documents Loaded**: GDPR text, EDPB cases (parse complete), other sources TBD
- **Verification Needed**: Document counts, search quality, retrieval latency

## Key Decisions
- **Canonical EN**: All documents stored in EN (or EN + original language)
- **Dual System**: RAG (corpus) + AKG (graph) = hybrid approach
- **Judge Uses Both**: AKG for violation confirmation, RAG for supporting evidence

## Links
- [TASKS.md](TASKS.md) - Verification & extension checklist
- [STATUS.md](STATUS.md) - Current state