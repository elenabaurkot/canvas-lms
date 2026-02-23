# Research: Clone Assignment Integration IDs and Data

## R-001: How do copy options flow from UI to importer?

**Decision**: Follow the existing `settings` parameter pattern used by
`import_quizzes_next` and `import_blueprint_settings`.

**Rationale**: This is the established pattern in Canvas for boolean
migration settings. The flow is:
1. React checkbox → `data.settings.key = value` in submit handler
2. Controller → `migration_settings[:key] = true if params.dig(:settings, :key)`
3. Model accessor → `def key?; Canvas::Plugin.value_to_boolean(migration_settings[:key]); end`
4. Importer → `if migration.key? then item.field = hash[:field] end`

**Alternatives considered**:
- Using `copy_options` hash: Rejected because `copy_options` controls
  *which objects* to copy (e.g., all_assignments), not *how* to copy them.
- Adding to `date_shift_options`: Rejected because unrelated to dates.

## R-002: Should integration_id and integration_data use one setting or two?

**Decision**: Two separate settings: `copy_integration_ids` and
`copy_integration_data`.

**Rationale**: The spec explicitly requires two independent options. This
matches the user's request and allows fine-grained control. A user may
want to copy the identifier without the metadata, or vice versa.

**Alternatives considered**:
- Single `import_integration_fields` setting: Rejected because the spec
  requires independent control of each field.

## R-003: Where exactly do integration fields need to be set during import?

**Decision**: In `AssignmentImporter.import_from_migration` method,
conditionally set `integration_id` and `integration_data` from the
export hash based on migration settings.

**Rationale**: The export hash already contains these fields from the
source assignment serialization. The importer currently does not set them
on the destination assignment, so they remain nil. Adding conditional
logic here follows the same pattern used for `post_to_sis`.

**Alternatives considered**:
- Modifying `AbstractAssignment#duplicate`: Rejected because course copy
  uses the importer path, not `duplicate`. The `duplicate` method is used
  for in-course duplication.

## R-004: Which API endpoint(s) need modification?

**Decision**: Modify `ContentMigrationsController#create` (the primary
API) and `CoursesController#copy_course` (the UI-driven flow). The
deprecated `ContentImportsController#copy_course_content` does not need
modification.

**Rationale**: `ContentMigrationsController#create` is the canonical API
endpoint and already accepts `settings` parameters. `CoursesController#copy_course`
is used by the UI form submission. The deprecated controller is not used
by new features.

**Alternatives considered**:
- Also modifying the deprecated endpoint: Rejected because it's
  deprecated and does not accept settings parameters.

## R-005: What UI component pattern to follow?

**Decision**: Add two new boolean props to `CommonMigratorControls`
(`canCopyIntegrationIds` and `canCopyIntegrationData`), with
corresponding state, checkboxes in the options array, and submit handler
entries.

**Rationale**: This exactly mirrors how `canImportAsNewQuizzes`,
`canOverwriteAssessmentContent`, and `canImportBPSettings` work.

**Alternatives considered**:
- A single combined checkbox: Rejected per spec requirement for
  independent options.
- Adding to a separate "Advanced" section: Rejected because existing
  options are all rendered in the same options list.

## R-006: What test patterns to follow?

**Decision**: Use the existing `course_copy_helper.rb` shared context for
backend tests. Use the existing `CommonMigratorControls.test.tsx` render
pattern for frontend tests.

**Rationale**: These are the established test patterns in Canvas.
Backend tests set `migration_settings` on the ContentMigration, call
`run_course_copy`, then verify assignment fields. Frontend tests render
the component, interact with checkboxes, and assert `onSubmit` is called
with the expected settings.

**Alternatives considered**: None — these are the only appropriate
test patterns.
