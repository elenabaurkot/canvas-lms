# Data Model: Clone Assignment Integration IDs and Data

## Entities

### Assignment (existing — no schema changes)

| Field | Type | Description |
|-------|------|-------------|
| `integration_id` | String | Identifier used by external systems (SIS, LTI) to correlate this assignment with their records. Currently cleared during course copy. |
| `integration_data` | Hash (serialized) | Free-form metadata stored by external integrations. Currently cleared during course copy. |

No new columns or migrations required. The fields already exist on
`abstract_assignments` (via `AbstractAssignment`).

### ContentMigration (existing — no schema changes)

| Field | Type | Description |
|-------|------|-------------|
| `migration_settings` | Hash (serialized YAML) | Stores all copy configuration. Two new keys will be used within this existing hash. |

**New migration_settings keys**:

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `copy_integration_ids` | Boolean | `false` | When true, copy `integration_id` from source assignments to destination assignments. |
| `copy_integration_data` | Boolean | `false` | When true, copy `integration_data` from source assignments to destination assignments. |

No database migration needed — these are new keys in an existing
serialized hash column.

## Relationships

```
Course (source) --[1:N]--> Assignment (with integration_id, integration_data)
    |
    v
ContentMigration (migration_settings includes copy_integration_ids, copy_integration_data)
    |
    v
Course (destination) --[1:N]--> Assignment (integration fields conditionally copied)
```

## State Transitions

No new state transitions. The ContentMigration lifecycle remains
unchanged: `created → pre_processing → exporting → exported → importing → imported`.

## Validation Rules

- `copy_integration_ids` and `copy_integration_data` are independent
  boolean flags — enabling one does not require enabling the other.
- When the flag is false (default), the field is not copied (preserving
  current behavior where the field is nil on the destination).
- When the flag is true, the field value is copied as-is from the
  source assignment, including null values.
