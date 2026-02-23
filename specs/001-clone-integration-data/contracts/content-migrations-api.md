# API Contract: Content Migrations — Integration Copy Settings

## Endpoint

`POST /api/v1/courses/:course_id/content_migrations`

## New Parameters

### settings[copy_integration_ids]

| Property | Value |
|----------|-------|
| Type | Boolean |
| Required | No |
| Default | `false` |
| Description | When true, copies `integration_id` values from source assignments to destination assignments during course copy. |

### settings[copy_integration_data]

| Property | Value |
|----------|-------|
| Type | Boolean |
| Required | No |
| Default | `false` |
| Description | When true, copies `integration_data` values from source assignments to destination assignments during course copy. |

## Request Example

```json
{
  "migration_type": "course_copy_importer",
  "settings": {
    "source_course_id": "123",
    "copy_integration_ids": true,
    "copy_integration_data": true
  }
}
```

## Response

No changes to the response format. The existing `ContentMigration` JSON
response is returned.

## Behavior

- Both parameters only apply when `migration_type` is
  `"course_copy_importer"`.
- When omitted or false, existing behavior is preserved (integration
  fields are cleared on destination assignments).
- When true, the corresponding field is copied from each source
  assignment to the matching destination assignment.
- These parameters are independent — each can be enabled without the
  other.
- Works with both full copy (`everything: true`) and selective copy
  (specific assignments selected).
