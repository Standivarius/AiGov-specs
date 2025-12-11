# AKG - Autonomous Knowledge Graph

## Overview
Structured legal knowledge graph for precise violation validation and article retrieval.

## Purpose
- Store: Framework articles, national provisions, violation patterns
- Query: "Does behavior X violate GDPR Art Y?"
- Relationships: IMPLEMENTS, SUPPLEMENTS, VIOLATES, REQUIRES
- Provides: Authoritative compliance validation

## Status
🟢 **Existing** (Codex-Petri built locally)

## Architecture

### Node Types
- **Framework**: GDPR, ISO_27001, ISO_42001, EU_AI_Act
- **Article**: GDPR_Article_32, ISO_Control_A_8_1
- **National_Provision**: RO_Law_190_Art_15
- **Violation_Pattern**: email_leak, rtbf_failure
- **Requirement**: technical_measure, organizational_control

### Relationships
- `(National_Provision)-[:IMPLEMENTS]->(Article)`
- `(Article)-[:REQUIRES]->(Requirement)`
- `(Violation_Pattern)-[:VIOLATES]->(Article)`
- `(Framework)-[:SUPPLEMENTS]->(Framework)`

### Example Query (Cypher)
```cypher
MATCH (pattern:Violation_Pattern {name: 'email_leak'})
      -[:VIOLATES]->(article:Article)
      -[:PART_OF]->(framework:Framework {name: 'GDPR'})
RETURN article.number, article.text, article.title
```

## Current State
- **Implementation**: Codex-Petri (local VPS)
- **Database**: Neo4j or FalkorDB (TBD - verify locally)
- **Nodes Loaded**: GDPR (parse complete), ISO 27001 (parse complete), ISO 42001 (TBD), AI Act (TBD)
- **Verification Needed**: Actual node counts, query latency, schema validation

## Key Decisions
- **Scenarios NOT in AKG** (ADR-0004): Scenarios stored as files, AKG validates only
- **Canonical EN**: All nodes in EN, national provisions have `text_original` field
- **Dual System**: AKG (structured) + RAG (corpus) = hybrid approach

## Links
- [TASKS.md](TASKS.md) - Verification & extension checklist
- [STATUS.md](STATUS.md) - Current state