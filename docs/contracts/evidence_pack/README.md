# Evidence Pack Contract (Canonical)

The evidence pack is the canonical JSON artifact emitted per run as
`evidence_pack.json`. It aggregates scored signals and supporting
evidence items for reporting and downstream validation.

Canonical schema (v0):
- `docs/contracts/evidence_pack/evidence_pack_v0.schema.json`

Vendoring guidance:
- Downstream repos (EP/PE) should vendor the schema file as-is.
- Treat the schema as read-only contract data; do not fork values inline.
- Track updates by syncing this folder and reviewing `CHANGELOG.md`.
