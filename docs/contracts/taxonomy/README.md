# Taxonomy Contracts (Canonical)

This taxonomy package is the single source of truth for:
- Allowed signal IDs
- Verdict/rating enums (including legacy normalization rules)
- Minimal evidence schema references to those enums

## Ownership and Usage
- **Canonical ownership:** `AiGov-specs/docs/contracts/taxonomy/`
- EP/PE **must** treat this taxonomy as read-only contract data.
- Any runtime or test logic should consume this taxonomy; do not fork or inline values.

## Versioning
- `taxonomy_version` is required in each contract artifact.
- Update `CHANGELOG.md` for every taxonomy change.
- Breaking changes require a new version and explicit migration guidance.
