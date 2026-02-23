# Implementation Plan: Clone Assignment Integration IDs and Data

**Branch**: `001-clone-integration-data` | **Date**: 2026-02-22 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-clone-integration-data/spec.md`

## Summary

Add two opt-in boolean settings (`copy_integration_ids` and
`copy_integration_data`) to the course copy flow. These settings are
passed via `params[:settings]` through to `ContentMigration#migration_settings`,
and checked in `AssignmentImporter.import_from_migration` to conditionally
preserve `integration_id` and `integration_data` on copied assignments.
The UI exposes these as two independent checkboxes in `CommonMigratorControls`.

## Technical Context

**Language/Version**: Ruby 3.x (Rails) + TypeScript (React)
**Primary Dependencies**: Rails, React, Instructure UI
**Storage**: PostgreSQL with Switchman sharding (no schema changes needed)
**Testing**: RSpec (backend), Vitest (frontend)
**Target Platform**: Web application (Canvas LMS)
**Project Type**: Feature addition to existing web application
**Performance Goals**: No measurable impact — adds two boolean checks
during an already-expensive course copy operation.
**Constraints**: Must follow Canvas contribution guidelines (single
focused commit, test plan, I18n, accessibility, linter-clean).
**Scale/Scope**: Touches 4 backend files, 3 frontend files, 2 test files.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Code Style Compliance | PASS | Will follow Ruby/JS style guides, run linters. |
| II. Internationalization | PASS | Checkbox labels will use `I18n.t()`. |
| III. Testing Discipline | PASS | RSpec + Vitest tests planned. Commit will include Test Plan. |
| IV. Multi-Tenant & Sharding | PASS | No raw SQL. Uses existing `migration_settings` serialized hash. Hash syntax in AR queries. |
| V. Accessibility | PASS | Using Instructure UI `Checkbox` components with proper labels. |
| VI. Performance | PASS | Two boolean checks in an existing importer — negligible cost. |
| VII. Security | PASS | No user input in SQL. Boolean settings only. Existing authorization gates apply. |
| PR Process | PASS | Single focused commit. Developing against master. ICA required. |

All gates pass. No violations to justify.

## Project Structure

### Documentation (this feature)

```text
specs/001-clone-integration-data/
├── plan.md
├── spec.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── content-migrations-api.md
└── checklists/
    └── requirements.md
```

### Source Code (files to modify)

```text
app/
├── controllers/
│   ├── courses_controller.rb              # Add settings extraction
│   └── content_migrations_controller.rb   # Add API docs for new params
├── models/
│   ├── content_migration.rb               # Add accessor methods
│   └── importers/
│       └── assignment_importer.rb         # Add conditional field copy

ui/
├── shared/content-migrations/react/
│   └── CommonMigratorControls/
│       └── CommonMigratorControls.tsx     # Add checkboxes + state
├── features/content_migrations/react/
│   └── components/migrator_forms/
│       └── course_copy.tsx                # Pass new props
└── features/copy_course/react/
    └── components/form/
        └── CopyCourseForm.tsx             # Pass new props

spec/
└── models/content_migration/
    └── course_copy_assignments_spec.rb    # Add integration copy tests

ui/shared/content-migrations/react/
└── CommonMigratorControls/__tests__/
    └── CommonMigratorControls.test.tsx    # Add checkbox tests
```

**Structure Decision**: No new files or directories in the source tree.
All changes are modifications to existing files, following established
patterns in each file.

## Implementation Details

### Backend Changes

#### 1. `courses_controller.rb` — Extract settings (~line 2948)

Add two lines after the existing `import_blueprint_settings` extraction:

```ruby
@content_migration.migration_settings[:copy_integration_ids] = true if params.dig(:settings, :copy_integration_ids)
@content_migration.migration_settings[:copy_integration_data] = true if params.dig(:settings, :copy_integration_data)
```

#### 2. `content_migrations_controller.rb` — API documentation (~line 333)

Add `@argument` annotations for the new settings:

```ruby
# @argument settings[copy_integration_ids] [Boolean]
#   Whether to copy integration_id values from source assignments
#   to destination assignments.
#
# @argument settings[copy_integration_data] [Boolean]
#   Whether to copy integration_data values from source assignments
#   to destination assignments.
```

No code changes needed — `update_migration_settings` already passes
all settings through.

#### 3. `content_migration.rb` — Accessor methods

Add two methods following the `import_quizzes_next?` pattern:

```ruby
def copy_integration_ids?
  Canvas::Plugin.value_to_boolean(migration_settings[:copy_integration_ids])
end

def copy_integration_data?
  Canvas::Plugin.value_to_boolean(migration_settings[:copy_integration_data])
end
```

#### 4. `assignment_importer.rb` — Conditional field copy

In `import_from_migration`, after existing field assignments, add:

```ruby
item.integration_id = hash[:integration_id] if migration.copy_integration_ids?
item.integration_data = hash[:integration_data] if migration.copy_integration_data?
```

#### 5. `lib/cc/assignment_resources.rb` — Export fields

Add `integration_id` to the `atts` array so it is included in the
course export XML. `integration_data` is a hash and needs JSON
serialization (same pattern as `turnitin_settings`):

```ruby
node.tag!(:integration_data, assignment.integration_data.to_json) if assignment.integration_data.present?
```

#### 6. `lib/cc/importer/standard/assignment_converter.rb` — Parse exported fields

Add `integration_id` to the string-type attribute list so it is read
back from the XML. Parse `integration_data` from JSON:

```ruby
integration_data_val = get_node_val(meta_doc, "integration_data")
if integration_data_val.present?
  assignment["integration_data"] = JSON.parse(integration_data_val)
end
```

### Frontend Changes

#### 5. `CommonMigratorControls.tsx` — Checkboxes

- Add props: `canCopyIntegrationIds?: boolean`, `canCopyIntegrationData?: boolean`
- Add state: `copyIntegrationIds` and `copyIntegrationData` (both default `false`)
- Add checkboxes to the `options` array with `I18n.t()` labels
- Add to `handleSubmit`:
  ```typescript
  canCopyIntegrationIds && (data.settings.copy_integration_ids = copyIntegrationIds)
  canCopyIntegrationData && (data.settings.copy_integration_data = copyIntegrationData)
  ```

#### 6. `course_copy.tsx` — Pass props

Pass `canCopyIntegrationIds={true}` and `canCopyIntegrationData={true}` to
`CommonMigratorControls`.

#### 7. `CopyCourseForm.tsx` — Pass props

Same as above.

### Test Changes

#### 8. `course_copy_assignments_spec.rb` — Backend tests

Add tests following the blueprint settings test pattern:

- Test: integration_id is copied when `copy_integration_ids` is true
- Test: integration_data is copied when `copy_integration_data` is true
- Test: both fields copied when both settings are true
- Test: neither field copied when settings are false (default)

#### 9. `CommonMigratorControls.test.tsx` — Frontend tests

Add tests following the `import_quizzes_next` test pattern:

- Test: checkbox renders when `canCopyIntegrationIds` is true
- Test: checkbox renders when `canCopyIntegrationData` is true
- Test: `onSubmit` called with `copy_integration_ids` in settings
- Test: `onSubmit` called with `copy_integration_data` in settings

## Complexity Tracking

No constitution violations. No complexity justifications needed.
