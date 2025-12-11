# Judge - Transcript Analysis Engine

## Overview
Multilingual LLM that analyzes Petri transcripts, queries AKG/RAG, and produces structured violation findings.

## Purpose
- Receive RO/EN transcripts from Petri
- Think internally in EN (canonical pipeline)
- Query AKG (graph) + RAG (corpus) for legal validation
- Output: `behaviour_json_v1` (structured findings)
- Translate reports to RO if needed

## Status
🔴 **Not Started**

## Architecture
```
Input: Petri Transcript (RO or EN)
  ↓
Judge LLM (multilingual, thinks in EN)
  ├→ Pattern Detection (LLM behavior analysis)
  ├→ AKG Query ("Does X violate Art Y?")
  ├→ RAG Query ("Find supporting cases")
  └→ Council Validation (if ambiguous)
  ↓
Output: behaviour_json_v1
  ├→ violations: [{article, behavior, severity, evidence}]
  ├→ recommendations: {immediate, short_term, long_term}
  └→ ambiguous_flags: [{reason, human_review_needed}]
```

## Key Decisions
- **Single LLM**: No separate translation service (context preservation)
- **Canonical EN**: All reasoning in EN, translate at I/O edges only
- **Model Candidates**: Gemini 2.0 Flash Thinking (1M context, multilingual), Mistral Large 3, Claude Sonnet 4.5, GPT-4.5.1
- **Council Pattern**: 3/4 consensus for ambiguous cases

## Testing Protocol
**LLM Selection** (Phase 0):
- Same RO transcript → 5 candidate models
- Measure: Translation accuracy, pattern detection, article precision, cost, latency
- Winner becomes Judge LLM

**Consistency Testing** (Phase 2):
- 100 runs same input → measure variance (Eval-app TEST-J01)
- Target: 95%+ stable output

## Links
- [TASKS.md](TASKS.md) - Implementation checklist
- [STATUS.md](STATUS.md) - Current state