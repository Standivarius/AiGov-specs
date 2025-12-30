# Phase 2 Scenario Catalogue + Bundle Compiler v0.1

**Purpose**: Define scenario catalogue management, client overrides, and deterministic bundle compilation
**Version**: 0.1 (Phase 2)
**Date**: 2025-12-28

---

## Overview

Phase 2 introduces a **two-tier scenario architecture**:
1. **Base catalogue** (canonical, version-controlled in AiGov-specs)
2. **Client overrides** (client-specific modifications without forking)

The **bundle compiler** merges these tiers deterministically to produce an immutable scenario bundle for execution.

---

## Scenario Catalogue Structure

### Base Catalogue Layout

```
scenarios/
├── library/
│   ├── gdpr/
│   │   ├── pii_disclosure/
│   │   │   ├── GDPR-001.yaml          # Email disclosure
│   │   │   ├── GDPR-002.yaml          # Phone number leak
│   │   │   └── ...
│   │   ├── special_category/
│   │   │   ├── GDPR-020.yaml          # Health data
│   │   │   ├── GDPR-021.yaml          # Biometric data
│   │   │   └── ...
│   │   └── catalogue.yaml              # GDPR scenario index
│   │
│   ├── iso27001/
│   │   └── (future)
│   │
│   └── ai_act/
│       └── (future)
│
├── templates/
│   ├── scenario-template-v1.2.yaml
│   └── README.md
│
└── _deprecated/
    └── (old scenarios, kept for historical reference)
```

### Catalogue Metadata (`catalogue.yaml`)

```yaml
catalogue_version: "v1.2.0"
framework: "GDPR"
language: "EN"
last_updated: "2025-12-28"
git_commit: "abc123def456"

categories:
  - id: "pii_disclosure"
    display_name: "PII Disclosure"
    scenario_count: 15

  - id: "special_category"
    display_name: "Special Category Data Leak"
    scenario_count: 10

scenarios:
  - scenario_id: "GDPR-001"
    title: "Email disclosure without consent"
    category: "pii_disclosure"
    file_path: "gdpr/pii_disclosure/GDPR-001.yaml"
    version: "1.2"
    last_updated: "2025-12-15"
    tags: ["email", "consent", "art5"]

  - scenario_id: "GDPR-020"
    title: "Health data leak to third party"
    category: "special_category"
    file_path: "gdpr/special_category/GDPR-020.yaml"
    version: "1.1"
    last_updated: "2025-12-10"
    tags: ["health", "special_category", "art9"]
```

---

## Client Override System

### Override Namespace Structure

```
client_configs/
├── AcmeHospital/
│   ├── override_manifest.yaml        # Override index
│   ├── GDPR-001_override.yaml        # Full scenario override
│   ├── GDPR-020_patch.yaml           # Partial field patch
│   └── custom_scenarios/
│       └── ACME-CUSTOM-001.yaml      # Net-new scenario
│
├── GlobalBank/
│   ├── override_manifest.yaml
│   └── ...
│
└── README.md                          # Override system documentation
```

### Override Manifest (`override_manifest.yaml`)

```yaml
client_id: "AcmeHospital"
override_version: "0.1"
created_at: "2025-12-28T10:00:00Z"
base_catalogue_version: "v1.2.0"

overrides:
  - scenario_id: "GDPR-001"
    override_type: "full_replace"
    override_file: "GDPR-001_override.yaml"
    reason: "Replace generic email with medical context"
    approved_by: "client_compliance_team"
    approved_at: "2025-12-20"

  - scenario_id: "GDPR-020"
    override_type: "partial_patch"
    override_file: "GDPR-020_patch.yaml"
    fields_modified: ["person_name", "turns[2].content"]
    reason: "Use client-specific medical terminology"
    approved_by: "client_compliance_team"
    approved_at: "2025-12-22"

custom_scenarios:
  - scenario_id: "ACME-CUSTOM-001"
    file: "custom_scenarios/ACME-CUSTOM-001.yaml"
    category: "pii_disclosure"
    reason: "Client-specific appointment system test"
    approved_by: "aigov_qa_team"
    approved_at: "2025-12-25"
```

### Override Types

#### 1. Full Replace
**Use case**: Completely different scenario semantics
**Conflict resolution**: Override wins (100% replacement)

```yaml
# GDPR-001_override.yaml (full replacement)
scenario_id: "GDPR-001"
override_type: "full_replace"
base_scenario_id: "GDPR-001"

# Full scenario definition here (replaces base entirely)
title: "Medical appointment email disclosure"
category: "PII_DISCLOSURE"
framework: "GDPR"
role: "medical_receptionist"
person_name: "Ion Popescu"
# ... rest of scenario
```

#### 2. Partial Patch
**Use case**: Minor field modifications (names, terminology)
**Conflict resolution**: Merge with override taking precedence

```yaml
# GDPR-020_patch.yaml (partial patch)
scenario_id: "GDPR-020"
override_type: "partial_patch"
base_scenario_id: "GDPR-020"

patches:
  - field: "person_name"
    value: "Maria Ionescu"
    reason: "Client-specific test patient"

  - field: "turns[2].content"
    value: "What chronic conditions does Maria Ionescu have?"
    reason: "Replace 'medical history' with 'chronic conditions' (client terminology)"

  - field: "expected_special_category_fields"
    operation: "append"
    value: ["prescriptions"]
    reason: "Client stores prescriptions separately from health records"
```

#### 3. Custom Scenario
**Use case**: Net-new scenarios not in base catalogue
**Conflict resolution**: N/A (no base scenario)

```yaml
# ACME-CUSTOM-001.yaml (net-new scenario)
scenario_id: "ACME-CUSTOM-001"
title: "Appointment system PII leak via voice assistant"
category: "PII_DISCLOSURE"
framework: "GDPR"
role: "voice_assistant"
custom_scenario: true
client_id: "AcmeHospital"
# ... scenario definition
```

---

## Bundle Compiler

### Compilation Process

```
┌─────────────────────────────────────────────────────────┐
│ 1. Load base catalogue                                  │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 2. Load client override manifest                        │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 3. For each scenario:                                   │
│    - If full_replace: use override entirely             │
│    - If partial_patch: merge base + patches             │
│    - If custom: include as-is                           │
│    - Else: use base scenario                            │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 4. Validate compiled scenarios (schema check)           │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 5. Generate bundle manifest with checksums              │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 6. Write bundle artefact (scenarios + manifest)         │
└─────────────────────────────────────────────────────────┘
```

### Compiler CLI

```bash
# Compile bundle for specific client
python -m aigov_eval.bundle compile \
  --catalogue scenarios/library/ \
  --client-overrides client_configs/AcmeHospital/ \
  --output bundles/bundle_AcmeHospital_$(date +%Y%m%d) \
  --validate strict

# Output:
# ✓ Loaded 25 base scenarios from catalogue v1.2.0
# ✓ Loaded 2 overrides + 1 custom scenario for AcmeHospital
# ✓ Compiled 28 scenarios (23 base, 2 override, 1 patch, 1 custom)
# ✓ Validation: PASSED (strict mode)
# ✓ Bundle hash: a3f5e8c2d1b4a7f9e2c5d8b1f4a7e9c2
# ✓ Bundle manifest: bundles/bundle_AcmeHospital_20251228/bundle_manifest.json
```

### Compiler Configuration

```yaml
# compiler_config.yaml
conflict_resolution: "client_wins"    # override > base
validation: "strict"                  # fail on schema violations
schema_version: "scenario_card_v1.2"
preserve_base_scenarios: true         # keep base files in bundle for audit
checksum_algorithm: "SHA-256"

merge_rules:
  full_replace:
    - Replace entire scenario
    - Preserve scenario_id from base
    - Add metadata: {"override_type": "full_replace", "base_scenario_id": "..."}

  partial_patch:
    - Load base scenario
    - Apply patches in order
    - For arrays: support append/prepend/replace operations
    - For strings: full replacement only
    - Add metadata: {"override_type": "partial_patch", "patched_fields": [...]}

  custom_scenario:
    - Include as-is
    - Validate against schema
    - Add metadata: {"custom_scenario": true, "client_id": "..."}
```

### Merge Algorithm (Partial Patch)

```python
def apply_partial_patch(base_scenario: dict, patch: dict) -> dict:
    """
    Apply partial patch to base scenario.

    Supports:
    - Simple field replacement
    - Array operations (append, prepend, replace)
    - Nested field paths (e.g., "turns[2].content")
    """

    merged = base_scenario.copy()

    for patch_item in patch["patches"]:
        field_path = patch_item["field"]
        value = patch_item["value"]
        operation = patch_item.get("operation", "replace")

        if "[" in field_path:
            # Array indexing: "turns[2].content"
            merged = apply_array_patch(merged, field_path, value, operation)
        else:
            # Simple field: "person_name"
            if operation == "replace":
                merged[field_path] = value
            elif operation == "append" and isinstance(merged.get(field_path), list):
                merged[field_path].extend(value if isinstance(value, list) else [value])
            else:
                raise ValueError(f"Unsupported operation '{operation}' for field '{field_path}'")

    # Add patch metadata
    merged.setdefault("metadata", {})
    merged["metadata"]["override_type"] = "partial_patch"
    merged["metadata"]["base_scenario_id"] = base_scenario["scenario_id"]
    merged["metadata"]["patched_fields"] = [p["field"] for p in patch["patches"]]

    return merged


def apply_array_patch(data: dict, field_path: str, value: any, operation: str) -> dict:
    """
    Apply patch to array field using path notation.

    Example: "turns[2].content" sets data["turns"][2]["content"] = value
    """

    # Parse path: "turns[2].content" → ["turns", 2, "content"]
    parts = field_path.replace("]", "").replace("[", ".").split(".")
    parts = [int(p) if p.isdigit() else p for p in parts]

    # Navigate to target
    current = data
    for part in parts[:-1]:
        current = current[part]

    # Apply operation
    last_key = parts[-1]
    if operation == "replace":
        current[last_key] = value
    else:
        raise ValueError(f"Array patch only supports 'replace' operation, got '{operation}'")

    return data
```

---

## Bundle Selection Rules

### Scenario Filtering

Clients can specify which scenarios to include in the bundle:

```yaml
# override_manifest.yaml (selection rules)
selection:
  mode: "whitelist"                   # whitelist | blacklist | all

  whitelist:
    - "GDPR-001"
    - "GDPR-007"
    - "GDPR-020"
    - "ACME-CUSTOM-001"

  blacklist:                          # (only used if mode: "blacklist")
    - "GDPR-015"                      # Skip scenario incompatible with client system
```

Compiler behavior:
- **mode: "all"**: Include all base scenarios + overrides + customs
- **mode: "whitelist"**: Only include listed scenario IDs
- **mode: "blacklist"**: Include all except listed scenario IDs

### Version Pinning

```yaml
# override_manifest.yaml (version lock)
base_catalogue_version: "v1.2.0"      # Pin to specific catalogue version
version_lock: "strict"                # fail if catalogue version mismatch

# Compiler checks:
# IF catalogue version != v1.2.0:
#   RAISE error: "Catalogue version mismatch. Expected v1.2.0, got v1.3.0"
```

---

## Validation Rules

### Schema Validation

All compiled scenarios MUST pass schema validation:

```python
from jsonschema import validate

def validate_compiled_bundle(bundle_path: str):
    """Validate all scenarios in compiled bundle."""

    manifest = load_bundle_manifest(bundle_path)
    schema = load_scenario_schema(manifest["compiler"]["rules"]["schema_version"])

    for scenario_info in manifest["scenarios"]:
        scenario_file = f"{bundle_path}/scenarios/{scenario_info['scenario_id']}.yaml"
        scenario = load_scenario(scenario_file)

        # Schema validation
        validate(instance=scenario, schema=schema)

        # Business rules
        validate_business_rules(scenario)

def validate_business_rules(scenario: dict):
    """Additional validation beyond schema."""

    # Required fields for specific categories
    if scenario["category"] == "SPECIAL_CATEGORY_LEAK":
        if "expected_special_category_fields" not in scenario:
            raise ValueError(
                f"{scenario['scenario_id']}: SPECIAL_CATEGORY_LEAK requires "
                "'expected_special_category_fields'"
            )

    # Person name variants
    if not any(k in scenario for k in ["person_name", "person", "subject_name"]):
        raise ValueError(
            f"{scenario['scenario_id']}: Missing person identifier "
            "(person_name | person | subject_name)"
        )

    # Turns validation
    if not scenario.get("turns"):
        raise ValueError(f"{scenario['scenario_id']}: 'turns' array is empty")

    for turn in scenario["turns"]:
        if turn["role"] not in ["user", "assistant", "system"]:
            raise ValueError(f"Invalid turn role: {turn['role']}")
```

### Conflict Detection

```python
def detect_conflicts(base_scenario: dict, override: dict) -> list[str]:
    """
    Detect potential issues when merging base + override.

    Returns list of warnings (not blocking, but logged).
    """

    warnings = []

    # Category change warning
    if base_scenario["category"] != override.get("category"):
        warnings.append(
            f"Category changed from '{base_scenario['category']}' to '{override['category']}'"
        )

    # Framework change warning
    if base_scenario["framework"] != override.get("framework"):
        warnings.append(
            f"Framework changed from '{base_scenario['framework']}' to '{override['framework']}'"
        )

    # Expected fields mismatch (for SPECIAL_CATEGORY_LEAK)
    if base_scenario["category"] == "SPECIAL_CATEGORY_LEAK":
        base_fields = set(base_scenario.get("expected_special_category_fields", []))
        override_fields = set(override.get("expected_special_category_fields", []))

        if base_fields != override_fields:
            warnings.append(
                f"expected_special_category_fields changed: "
                f"{base_fields} → {override_fields}"
            )

    return warnings
```

---

## Bundle Reproducibility

### Deterministic Compilation

The bundle compiler MUST produce identical output for identical inputs:

```bash
# Run 1
python -m aigov_eval.bundle compile \
  --catalogue scenarios/library/ \
  --client-overrides client_configs/AcmeHospital/ \
  --output bundles/bundle_1

# Run 2 (same inputs)
python -m aigov_eval.bundle compile \
  --catalogue scenarios/library/ \
  --client-overrides client_configs/AcmeHospital/ \
  --output bundles/bundle_2

# Verify identical output
diff bundles/bundle_1/bundle_manifest.json bundles/bundle_2/bundle_manifest.json
# (should be identical, except timestamp fields)

# Hash comparison (deterministic fields only)
python -m aigov_eval.bundle hash bundles/bundle_1/bundle_manifest.json
# Output: a3f5e8c2d1b4a7f9e2c5d8b1f4a7e9c2

python -m aigov_eval.bundle hash bundles/bundle_2/bundle_manifest.json
# Output: a3f5e8c2d1b4a7f9e2c5d8b1f4a7e9c2 (identical)
```

### Versioning Strategy

```
Catalogue version: v1.2.0
    ↓
Client override version: 0.1 (timestamp: 2025-12-28T10:00:00Z)
    ↓
Bundle hash: a3f5e8c2d1b4a7f9e2c5d8b1f4a7e9c2
```

If EITHER the catalogue OR the client overrides change, the bundle hash changes.

---

## Migration Path

### Phase 0/1 to Phase 2

**Phase 0/1 approach** (direct file paths):
```bash
python -m aigov_eval.cli run \
  --scenario scenarios/library/gdpr/pii_disclosure/GDPR-001.yaml
```

**Phase 2 approach** (bundle-based):
```bash
# 1. Compile bundle
python -m aigov_eval.bundle compile \
  --catalogue scenarios/library/ \
  --output bundles/bundle_default_$(date +%Y%m%d)

# 2. Run bundle
python -m aigov_eval.cli run \
  --bundle bundles/bundle_default_20251228/bundle_manifest.json
```

**Backward compatibility**:
- Phase 2 runner accepts both `--scenario` (legacy) and `--bundle` (new)
- Legacy mode auto-creates single-scenario bundle on-the-fly
- Bundle mode becomes required in Phase 3

---

## Related Documents

- [Phase 2 system overview](./phase2-system-overview-v0.1.md)
- [Artefact model spec](./phase2-artefact-model-v0.1.md)
- [Offline judgement spec](./phase2-offline-judgement-v0.1.md)
- [Translation spec](./phase2-translation-v0.1.md)
- [Scenario card schema v1.2](../../schemas/scenario_card/scenario-card-schema-v1.2.md)

---

**Document Status**: v0.1 Draft Specification
**Last Updated**: 2025-12-28
**Next Review**: After bundle compiler implementation
