# Judge - Task Checklist

## Research Phase
- [ ] Research: Inspect AI LLM abstraction layer
- [ ] Design: Platform-agnostic wrapper (OpenRouter + direct APIs)
- [ ] Test: 5 candidate models (Gemini 2.0, Mistral Large 3, Claude, GPT, Llama)
  - [ ] Test 1: Translation fidelity (RO→EN→RO)
  - [ ] Test 2: Pattern detection accuracy (known violations)
  - [ ] Test 3: Article precision (correct GDPR articles cited)
  - [ ] Test 4: Cost analysis (tokens per transcript)
  - [ ] Test 5: Latency measurement (seconds per analysis)
- [ ] Decision: Select Judge LLM (document in RESEARCH.md)

## Design Phase
- [ ] Design: `behaviour_json_v1` schema
  - Fields: violations, recommendations, ambiguous_flags, metadata
- [ ] Design: AKG query protocol (Cypher queries)
- [ ] Design: RAG query protocol (vector search)
- [ ] Design: Council voting logic (3/4 consensus)

## Implementation Phase
- [ ] Implement: LLM wrapper (OpenRouter + Anthropic + OpenAI clients)
- [ ] Implement: Judge prompt (pattern detection instructions)
- [ ] Implement: AKG integration (query AKG, parse response)
- [ ] Implement: RAG integration (query RAG, parse citations)
- [ ] Implement: Council orchestration (parallel queries, vote counting)
- [ ] Implement: Output formatter (behaviour_json_v1 generation)

## Validation Phase
- [ ] Test: Email leak scenario (known violation)
- [ ] Test: RTBF scenario (edge case handling)
- [ ] Test: Ambiguous case (council voting triggers)
- [ ] Measure: Consistency (5 runs same input)
- [ ] Iterate: Prompt refinement based on failures

---
**Status**: Not started  
**Next**: Research LLM abstraction layer