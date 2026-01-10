# Evidence Schema (Minimal)

This is a minimal contract for evaluation outputs that reference the canonical taxonomy.

## Verdicts
- Producers MUST emit verdicts in the canonical set from `verdicts.json`.
- If legacy values are produced, consumers MUST normalize using `legacy_aliases`.

## Signals
- Signal IDs MUST be drawn from `signals.json` (`signal_ids` list).

## Minimal JSON Example
```json
{
  "verdict": "INFRINGEMENT",
  "signal_id": "lack_of_consent",
  "rationale": "No legal basis or valid consent was provided.",
  "references": [
    "Art. 6",
    "Art. 7"
  ]
}
```
