# Quickstart: Clone Assignment Integration IDs and Data

## What This Feature Does

Adds two opt-in checkboxes to the course copy UI (and corresponding API
parameters) that allow `integration_id` and `integration_data` on
assignments to be carried over during a course copy. By default, these
fields are cleared during copy.

## Files to Modify

### Backend (Ruby)

1. **`app/controllers/courses_controller.rb`** (~line 2948)
   - Extract `copy_integration_ids` and `copy_integration_data` from
     `params[:settings]` and store in `migration_settings`.

2. **`app/controllers/content_migrations_controller.rb`** (~line 333)
   - Add `@argument` documentation for the two new settings parameters.
   - The settings are already passed through via `update_migration_settings`.

3. **`app/models/content_migration.rb`**
   - Add two accessor methods: `copy_integration_ids?` and
     `copy_integration_data?`.

4. **`app/models/importers/assignment_importer.rb`** (~line 359)
   - In `import_from_migration`, conditionally set `integration_id`
     and `integration_data` on the destination assignment based on
     migration settings.

### Frontend (TypeScript/React)

5. **`ui/shared/content-migrations/react/CommonMigratorControls/CommonMigratorControls.tsx`**
   - Add props: `canCopyIntegrationIds`, `canCopyIntegrationData`.
   - Add state for each checkbox.
   - Add checkboxes to the options array.
   - Add settings to submit handler.

6. **`ui/features/content_migrations/react/components/migrator_forms/course_copy.tsx`**
   - Pass the new props to `CommonMigratorControls`.

7. **`ui/features/copy_course/react/components/form/CopyCourseForm.tsx`**
   - Pass the new props to `CommonMigratorControls`.

### Tests

8. **`spec/models/content_migration/course_copy_assignments_spec.rb`**
   - Add tests for copying with integration settings enabled/disabled.

9. **`ui/shared/content-migrations/react/CommonMigratorControls/__tests__/CommonMigratorControls.test.tsx`**
   - Add tests for the new checkboxes and submit data.

## Development Setup

```bash
docker compose up
docker compose run --rm web bash
```

### Run Backend Tests

```bash
bin/rspec spec/models/content_migration/course_copy_assignments_spec.rb
```

### Run Frontend Tests

```bash
yarn test ui/shared/content-migrations/react/CommonMigratorControls/__tests__/CommonMigratorControls.test.tsx
```

### Run Linters

```bash
script/rlint    # Ruby
script/eslint   # JavaScript/TypeScript
```

## Manual Test Plan

1. Create a source course with assignments that have `integration_id`
   and `integration_data` set (via API or Rails console).
2. Navigate to the course copy UI.
3. Verify the two new checkboxes appear.
4. Copy the course with both options enabled.
5. Verify the destination assignments have the integration fields.
6. Repeat with options disabled — verify fields are cleared (default).
7. Test via API with `settings[copy_integration_ids]=true`.
