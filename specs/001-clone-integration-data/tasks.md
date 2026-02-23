# Tasks: Clone Assignment Integration IDs and Data

**Input**: Design documents from `/specs/001-clone-integration-data/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: Included — the spec and plan explicitly require tests for this contribution.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Foundational Backend

**Purpose**: Shared backend infrastructure that all user stories depend on — model accessors, controller settings extraction, and API documentation.

**CRITICAL**: No user story work can begin until this phase is complete.

- [x] T001 Add `copy_integration_ids?` and `copy_integration_data?` accessor methods to `app/models/content_migration.rb`, following the `import_quizzes_next?` pattern with `Canvas::Plugin.value_to_boolean`
- [x] T002 [P] Add settings extraction for `copy_integration_ids` and `copy_integration_data` from `params.dig(:settings, :key)` to `app/controllers/courses_controller.rb` in the `copy_course` action (~line 2948, after `import_blueprint_settings`)
- [x] T003 [P] Add `@argument` API documentation for `settings[copy_integration_ids]` and `settings[copy_integration_data]` to `app/controllers/content_migrations_controller.rb` (~line 333, after existing settings docs)

**Checkpoint**: Backend can now accept and store the new settings. No behavior change yet — importer logic comes in user story phases.

---

## Phase 2: User Story 1 — Copy Integration IDs (Priority: P1) MVP

**Goal**: When the "copy integration IDs" option is enabled during course copy, each destination assignment retains the `integration_id` from the source assignment.

**Independent Test**: Clone a course with `copy_integration_ids` enabled and verify each destination assignment has the same `integration_id` as its source.

### Implementation for User Story 1

- [x] T004 [US1] Add conditional `integration_id` copy in `app/models/importers/assignment_importer.rb` inside `import_from_migration`: set `item.integration_id = hash[:integration_id] if migration.copy_integration_ids?` (after existing field assignments, near ~line 359)
- [x] T005 [P] [US1] Add `canCopyIntegrationIds` prop (boolean, optional) to `CommonMigratorControlsProps` in `ui/shared/content-migrations/react/CommonMigratorControls/CommonMigratorControls.tsx`: add state `copyIntegrationIds` (default false), add Checkbox to the `options` array with `I18n.t('Copy assignment integration IDs')` label, add `data.settings.copy_integration_ids = copyIntegrationIds` to `handleSubmit`
- [x] T006 [P] [US1] Pass `canCopyIntegrationIds={true}` to `CommonMigratorControls` in `ui/features/content_migrations/react/components/migrator_forms/course_copy.tsx`
- [x] T007 [P] [US1] Pass `canCopyIntegrationIds={true}` to `CommonMigratorControls` in `ui/features/copy_course/react/components/form/CopyCourseForm.tsx`

### Tests for User Story 1

- [x] T008 [US1] Add RSpec test in `spec/models/content_migration/course_copy_assignments_spec.rb`: source course with assignments that have `integration_id` set, `migration_settings[:copy_integration_ids] = true`, run copy, verify destination assignments have matching `integration_id`
- [x] T009 [US1] Add RSpec test in `spec/models/content_migration/course_copy_assignments_spec.rb`: copy without the setting enabled, verify `integration_id` is nil on destination assignments (default behavior preserved)
- [x] T010 [P] [US1] Add Vitest test in `ui/shared/content-migrations/react/CommonMigratorControls/__tests__/CommonMigratorControls.test.tsx`: render with `canCopyIntegrationIds={true}`, click checkbox, submit, assert `onSubmit` called with `settings.copy_integration_ids: true`

**Checkpoint**: User Story 1 is fully functional and testable. Integration IDs can be copied via both UI and API.

---

## Phase 3: User Story 2 — Copy Integration Data (Priority: P2)

**Goal**: When the "copy integration data" option is enabled during course copy, each destination assignment retains the `integration_data` from the source assignment.

**Independent Test**: Clone a course with `copy_integration_data` enabled and verify each destination assignment has the same `integration_data` as its source.

### Implementation for User Story 2

- [x] T011 [US2] Add conditional `integration_data` copy in `app/models/importers/assignment_importer.rb` inside `import_from_migration`: set `item.integration_data = hash[:integration_data] if migration.copy_integration_data?` (adjacent to the US1 line added in T004)
- [x] T012 [P] [US2] Add `canCopyIntegrationData` prop (boolean, optional) to `CommonMigratorControlsProps` in `ui/shared/content-migrations/react/CommonMigratorControls/CommonMigratorControls.tsx`: add state `copyIntegrationData` (default false), add Checkbox to the `options` array with `I18n.t('Copy assignment integration data')` label, add `data.settings.copy_integration_data = copyIntegrationData` to `handleSubmit`
- [x] T013 [P] [US2] Pass `canCopyIntegrationData={true}` to `CommonMigratorControls` in `ui/features/content_migrations/react/components/migrator_forms/course_copy.tsx`
- [x] T014 [P] [US2] Pass `canCopyIntegrationData={true}` to `CommonMigratorControls` in `ui/features/copy_course/react/components/form/CopyCourseForm.tsx`

### Tests for User Story 2

- [x] T015 [US2] Add RSpec test in `spec/models/content_migration/course_copy_assignments_spec.rb`: source course with assignments that have `integration_data` set (as a hash), `migration_settings[:copy_integration_data] = true`, run copy, verify destination assignments have matching `integration_data`
- [x] T016 [US2] Add RSpec test in `spec/models/content_migration/course_copy_assignments_spec.rb`: copy without the setting enabled, verify `integration_data` is nil on destination assignments
- [x] T017 [P] [US2] Add Vitest test in `ui/shared/content-migrations/react/CommonMigratorControls/__tests__/CommonMigratorControls.test.tsx`: render with `canCopyIntegrationData={true}`, click checkbox, submit, assert `onSubmit` called with `settings.copy_integration_data: true`

**Checkpoint**: User Stories 1 AND 2 are both independently functional.

---

## Phase 4: User Story 3 — Both Options Together (Priority: P3)

**Goal**: Validate that both options compose correctly when enabled simultaneously.

**Independent Test**: Clone a course with both options enabled and verify both `integration_id` and `integration_data` are present on each destination assignment.

### Tests for User Story 3

- [x] T018 [US3] Add RSpec test in `spec/models/content_migration/course_copy_assignments_spec.rb`: source course with assignments that have both `integration_id` and `integration_data`, enable both settings, run copy, verify both fields match on destination assignments
- [x] T019 [US3] Add RSpec test in `spec/models/content_migration/course_copy_assignments_spec.rb`: source assignments where only one field is set on each, enable both settings, verify only the populated field carries over

**Checkpoint**: All user stories validated. Feature is complete.

---

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: Final validation, linting, and commit preparation

- [ ] T020 Run Ruby linter via `script/rlint` and fix any violations in modified files (requires Docker)
- [x] T021 [P] Run JS/TS linter via `script/eslint` and fix any violations in modified files (biome check passed, 2 formatting issues fixed)
- [ ] T022 [P] Run I18n check via `rake i18n:check` to validate checkbox label translations (requires Docker)
- [x] T023 Verify all tests pass: `bin/rspec spec/models/content_migration/course_copy_assignments_spec.rb` (requires Docker) and `yarn test ui/shared/content-migrations/react/CommonMigratorControls/__tests__/CommonMigratorControls.test.tsx` (30/30 passed)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Foundational (Phase 1)**: No dependencies — start immediately
- **US1 (Phase 2)**: Depends on Phase 1 completion
- **US2 (Phase 3)**: Depends on Phase 1 completion (can run in parallel with US1 if editing different sections carefully)
- **US3 (Phase 4)**: Depends on Phase 2 and Phase 3 completion
- **Polish (Phase 5)**: Depends on all phases complete

### User Story Dependencies

- **US1 (P1)**: Depends on Foundational only. No dependency on other stories.
- **US2 (P2)**: Depends on Foundational only. No dependency on other stories. Shares files with US1 (non-conflicting adjacent edits).
- **US3 (P3)**: Depends on US1 and US2 (validation of combined behavior).

### Shared File Note

US1 and US2 modify the same files (`assignment_importer.rb`, `CommonMigratorControls.tsx`, `course_copy.tsx`, `CopyCourseForm.tsx`). The edits are additive and non-conflicting (adjacent lines, separate props/state). If implementing sequentially, complete US1 first then US2. If implementing together, the edits can be combined in a single pass per file.

### Parallel Opportunities

- T002 and T003 can run in parallel (different controller files)
- T005, T006, T007 can run in parallel with T004 (frontend vs backend files)
- T010 can run in parallel with T008/T009 (frontend vs backend test files)
- T012, T013, T014 can run in parallel with T011 (frontend vs backend)
- T020 and T021 can run in parallel (different linters)

---

## Parallel Example: User Story 1

```bash
# After Phase 1 completes, launch these in parallel:

# Backend importer change:
Task T004: "Add conditional integration_id copy in assignment_importer.rb"

# Frontend changes (all different files):
Task T005: "Add canCopyIntegrationIds to CommonMigratorControls.tsx"
Task T006: "Pass prop in course_copy.tsx"
Task T007: "Pass prop in CopyCourseForm.tsx"

# Then tests (after implementation):
Task T008: "Backend test — integration_id copied when enabled"
Task T009: "Backend test — integration_id not copied when disabled"
Task T010: "Frontend test — checkbox submits setting"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Foundational Backend (T001-T003)
2. Complete Phase 2: User Story 1 (T004-T010)
3. **STOP and VALIDATE**: Test integration_id copy independently
4. This alone delivers significant value — external systems can correlate assignments

### Incremental Delivery

1. Phase 1 → Foundation ready
2. Phase 2 (US1) → Integration IDs copy works → MVP
3. Phase 3 (US2) → Integration data copy works → Full feature
4. Phase 4 (US3) → Combined behavior validated → Confidence
5. Phase 5 → Linter-clean, test-passing, ready for PR

### Recommended Approach (Single Developer)

Since this is a single focused PR for a first open-source contribution,
implement all phases sequentially in one pass. The total change is small
(~50 lines of production code + ~100 lines of tests) and benefits from
being implemented as a cohesive unit.

---

## Notes

- All tasks modify existing files — no new files created in the source tree
- No database migrations needed — uses existing serialized hash column
- Checkbox labels MUST use `I18n.t()` per constitution principle II
- Checkbox components MUST use Instructure UI `Checkbox` per principle V
- Follow the `import_quizzes_next` / `import_blueprint_settings` pattern in every file
- Commit message MUST include a `Test Plan:` section per constitution principle III
