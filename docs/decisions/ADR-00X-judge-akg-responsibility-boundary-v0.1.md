# ADR-00X: Judge–AKG Responsibility Boundary (v0.1)

## Status
Proposed (v0.1)

## Context
AiGov performs offline compliance judgement over immutable run artefacts (scenario bundle, transcripts, target-side evidence).
The offline Judge may query an AKG/RAG layer to retrieve legal references and canonical mappings. This introduces drift risk and audit ambiguity unless responsibilities are explicitly separated.

Key drivers:
- audit defensibility (traceability, reproducibility, clear accountability)
- drift management (LLM behaviour + knowledge retrieval instability)
- pipeline-friendly artefacts (machine-readable core, human-friendly edges)

## Decision
We split responsibilities as follows:

### Judge responsibilities (non-delegable)
The Judge MUST:
1. Determine verdicts (e.g., compliant/non-compliant) and severity/risk ratings.
2. Select applicable violation patterns/signals from the scenario criteria.
3. Cite concrete evidence spans from run artefacts (transcript/evidence pack).
4. Provide decision rationale:
   - why this pattern applies
   - what assumptions were made
   - known limitations of the judgement
5. Emit output strictly conforming to the locked judge_output schema.

The Judge MUST NOT claim that the AKG “decided” a violation.

### AKG responsibilities (bounded grounding service)
The AKG MUST:
1. Return mappings from known signals/patterns to legal references (GDPR articles/recitals, ISO controls, etc.).
2. Provide canonical phrasing / identifiers / crosswalks that are stable and versioned.
3. Return confidence and provenance metadata for its retrieved grounding.

The AKG MUST NOT:
- decide compliance outcomes
- assign severity/risk
- override the Judge’s verdict
- introduce new evaluation criteria beyond what is in the scenario bundle

### Failure mode: AKG incomplete or unavailable
If AKG grounding fails or is low confidence:
- The Judge still produces a judgement if sufficient evidence exists.
- The judge_output MUST:
  - set legal_grounding.confidence to "low" (or numeric threshold)
  - include limitations noting degraded grounding
- The pipeline remains fail-closed only for schema integrity, not for grounding completeness.

## Consequences
### Positive
- Clear accountability: verdicts are owned by the Judge.
- Audit clarity: legal grounding is explicitly a referenced support artefact.
- Drift containment: AKG changes do not silently change verdict logic.
- Easier replay: manifests can pin judge prompts and grounding contract versions.

### Negative / Trade-offs
- Slightly more verbose output schema (rationale + grounding separation).
- Requires an explicit AKG response contract and versioning discipline.
- Some “legal correctness” is limited by the AKG corpus and retrieval quality; this is disclosed in limitations.

## Interfaces
### Judge input (minimum)
- scenario bundle (evaluation criteria + pattern list)
- transcript artefact (with stable segment ids)
- evidence pack (tool traces, retrieval, policies, logs)
- AKG endpoint (optional) returning AKG_GROUNDING_RESPONSE_V0.1

### Judge output (locked)
judge_output.json MUST include two separate sections:
- decision_rationale (Judge-owned)
- legal_grounding (AKG-supplied, confidence-scoped)

Additionally, result_manifest pins:
- judge_output schema version + hash
- judge prompt bundle version + hash
- akg response schema version + hash

## Validation
- Unit: schema validation of judge_output against locked schema.
- Unit: AKG response validation against locked AKG response schema.
- Integration: run a fixed transcript through Judge with AKG ON vs OFF:
  - verdict must be stable unless explicitly allowed by scenario rules
  - limitations must reflect grounding mode
- Drift: store diff if verdict changes across pinned params/prompts.

## Alternatives considered
1) AKG decides legality: rejected (audit ambiguity, accountability blur, drift risk).
2) Judge without AKG: acceptable for MVP but weaker cross-framework mapping and citations; keep as fallback.
3) Hard-coded mappings only: low flexibility; acceptable for first GDPR slice but blocks expansion.

## Implementation notes
- Add `legal_grounding` and `decision_rationale` sections to judge_output schema.
- Create AKG response schema and pin it in specs.
- Update runner manifests to include schema/prompt hashes.
