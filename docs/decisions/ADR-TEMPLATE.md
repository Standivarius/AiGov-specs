# ADR-XXX: [Decision Title]

**Date**: YYYY-MM-DD
**Status**: [Proposed | Accepted | Rejected | Deprecated | Superseded]
**Owner**: [Team/Person]
**Decision Type**: [One-Way Door | Two-Way Door]

---

## Context

What is the issue we're facing? What forces are at play?

- Describe the technical, organizational, or business context
- Explain why a decision is needed now
- Reference related ADRs if applicable

---

## Decision

What is the change we're proposing and/or doing?

- State the decision clearly and concisely
- If applicable, describe the chosen option among alternatives
- For one-way doors: explicitly call out irreversibility

**One-Way Door Notice** (if applicable):
> This is a **one-way door** decision. Once implemented, reverting will be costly or impossible due to:
> - Data migration complexity
> - Breaking API changes
> - Dependency on external systems
> - [Other reason specific to this decision]

---

## Consequences

What becomes easier or harder as a result of this decision?

### Positive Consequences
- Benefit 1
- Benefit 2
- ...

### Negative Consequences / Tradeoffs
- Tradeoff 1
- Tradeoff 2
- ...

### Risks
- Risk 1 (and mitigation)
- Risk 2 (and mitigation)
- ...

---

## Alternatives Considered

What other options did we evaluate?

### Option A: [Alternative Name]
**Pros**:
- Pro 1
- Pro 2

**Cons**:
- Con 1
- Con 2

**Rejected because**: [Reason]

### Option B: [Alternative Name]
**Pros**:
- Pro 1

**Cons**:
- Con 1

**Rejected because**: [Reason]

---

## Implementation Plan

How will this decision be implemented?

1. Step 1
2. Step 2
3. ...

**Timeline**: [Estimated timeframe]
**Dependencies**: [What needs to happen first]
**Rollback Plan** (if two-way door): [How to revert if needed]

---

## Validation Criteria

How will we know this decision was correct?

- Success metric 1
- Success metric 2
- Review date: [When to re-evaluate]

---

## References

- Link to related issue/ticket
- Link to design doc
- Link to related ADRs
- External references

---

**Notes**:
- For trivial/reversible decisions, consider if ADR is needed (prefer lightweight documentation)
- For one-way doors, require review by 2+ team members before acceptance
- Update status to "Superseded" if a later ADR replaces this one
