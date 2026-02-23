# Feature Specification: Clone Assignment Integration IDs and Data

**Feature Branch**: `001-clone-integration-data`
**Created**: 2026-02-22
**Status**: Draft
**Input**: User description: "I would like to build a feature as part of the course clone process. The feature should provide options during the course clone process — one option to copy integration IDs and a second option to copy integration_data so that when these are selected the integration_id and integration_data on any existing assignments gets copied over to the assignments of the newly cloned course."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Copy Integration IDs During Course Clone (Priority: P1)

A course administrator or teacher initiates a course copy and wants the
integration IDs on assignments to carry over to the cloned course. This is
essential when the institution uses external systems (SIS, LTI tools, or
third-party integrations) that rely on `integration_id` to correlate
Canvas assignments with records in those systems. Without this option, the
administrator must manually re-enter integration IDs on every assignment
in the new course.

The user sees a new option during the course copy setup — a checkbox
labeled to indicate that assignment integration IDs will be carried over.
When selected, every assignment copied to the destination course retains
the `integration_id` value from the corresponding source assignment.

**Why this priority**: Integration IDs are the primary key used by
external systems to link to Canvas assignments. Losing them during a
clone breaks those integrations entirely, creating the most immediate
pain point.

**Independent Test**: Can be fully tested by cloning a course with the
option enabled and verifying that each assignment in the new course has
the same `integration_id` as the source assignment.

**Acceptance Scenarios**:

1. **Given** a source course with assignments that have `integration_id`
   values set, **When** the user initiates a course copy with the "copy
   integration IDs" option enabled, **Then** each assignment in the
   destination course has the same `integration_id` as the corresponding
   source assignment.

2. **Given** a source course with some assignments that have
   `integration_id` set and some that do not, **When** the user copies
   the course with the option enabled, **Then** only assignments that had
   an `integration_id` in the source course have one in the destination;
   assignments without one remain without one.

3. **Given** a source course with assignments, **When** the user copies
   the course without the "copy integration IDs" option enabled, **Then**
   no `integration_id` values are carried over (existing default behavior
   is preserved).

---

### User Story 2 - Copy Integration Data During Course Clone (Priority: P2)

A course administrator or teacher initiates a course copy and wants the
integration data on assignments to carry over. `integration_data` is a
free-form data store used by external integrations to persist
configuration, metadata, or state associated with an assignment. Losing
this data during a clone means the external integration must be
reconfigured manually for each assignment.

The user sees a second, independent option during course copy setup — a
checkbox indicating that assignment integration data will be carried
over. When selected, every assignment copied to the destination course
retains the `integration_data` value from the corresponding source
assignment.

**Why this priority**: Integration data supports richer integration
scenarios but is secondary to the identifier itself. An integration can
potentially re-derive its data from the `integration_id`, but not vice
versa.

**Independent Test**: Can be fully tested by cloning a course with the
option enabled and verifying that each assignment in the new course has
the same `integration_data` as the source assignment.

**Acceptance Scenarios**:

1. **Given** a source course with assignments that have
   `integration_data` populated, **When** the user copies the course with
   the "copy integration data" option enabled, **Then** each assignment
   in the destination course has the same `integration_data` as the
   corresponding source assignment.

2. **Given** a source course with assignments, **When** the user copies
   the course without the "copy integration data" option enabled,
   **Then** no `integration_data` values are carried over (default
   behavior preserved).

---

### User Story 3 - Both Options Used Together (Priority: P3)

A user enables both the "copy integration IDs" and "copy integration
data" options simultaneously during a course copy. Both fields are
carried over independently without conflict.

**Why this priority**: This is a natural combination of the two
independent options but does not introduce new functionality — it
validates that they compose correctly.

**Independent Test**: Clone a course with both options enabled and verify
both `integration_id` and `integration_data` are present on each
assignment in the destination course.

**Acceptance Scenarios**:

1. **Given** a source course with assignments that have both
   `integration_id` and `integration_data` set, **When** the user copies
   the course with both options enabled, **Then** each destination
   assignment has both `integration_id` and `integration_data` matching
   the source.

2. **Given** a source course with assignments where only one of the two
   fields is set on each assignment, **When** the user copies the course
   with both options enabled, **Then** each destination assignment
   carries over whichever field was set on the source, leaving the other
   blank.

---

### Edge Cases

- What happens when the user selects only specific assignments for
  copying (selective content mode) with the integration options enabled?
  Only selected assignments should carry over the integration fields.
- What happens when `integration_id` or `integration_data` is null or
  empty on a source assignment? The destination assignment should also
  have null/empty for that field — no fabrication of values.
- What happens if the course copy is performed via the API rather than
  the UI? The same options MUST be available as API parameters and
  behave identically.
- What happens if a user without course copy permissions attempts to
  use the integration copy API parameters directly? The system should
  reject the request as part of existing course copy authorization.

## Clarifications

### Session 2026-02-22

- Q: Should the integration copy options apply only to the direct course copy flow, or also to other content migration types? → A: Course copy only for the initial PR. Blueprint sync and course templates are planned follow-ups to keep the first contribution focused and reviewable.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST present an option during course copy setup
  to copy assignment integration IDs.
- **FR-002**: The system MUST present a separate, independent option
  during course copy setup to copy assignment integration data.
- **FR-003**: When the "copy integration IDs" option is enabled, the
  system MUST copy the `integration_id` value from each source
  assignment to the corresponding destination assignment.
- **FR-004**: When the "copy integration data" option is enabled, the
  system MUST copy the `integration_data` value from each source
  assignment to the corresponding destination assignment.
- **FR-005**: Both options MUST default to disabled (unchecked), so
  that existing course copy behavior is unchanged for users who do not
  opt in.
- **FR-006**: Both options MUST be available through the course copy
  API as well as the UI.
- **FR-007**: When selective content copying is used, the integration
  fields MUST only be carried over for assignments the user has
  selected to copy.
- **FR-008**: The options MUST be visible to any user who has permission
  to initiate a course copy. No additional permission check is required
  beyond the existing course copy authorization.
- **FR-009**: The feature scope is limited to the direct course copy
  flow. Blueprint sync, course templates, Canvas cartridge
  export/import, and Common Cartridge import are excluded from this
  initial contribution.

### Key Entities

- **Assignment**: The primary entity affected. Assignments have
  `integration_id` (a string identifier used by external systems) and
  `integration_data` (a free-form hash of metadata used by external
  integrations). During a standard course copy, both fields are
  currently cleared.
- **ContentMigration**: Represents a course copy operation. Stores
  migration settings and copy options that control what gets copied.
  The new integration copy options will be stored as additional
  settings on this entity.
- **Course**: The source and destination entities in a course copy
  operation.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: When the integration ID copy option is enabled, 100% of
  copied assignments retain their source `integration_id` values.
- **SC-002**: When the integration data copy option is enabled, 100% of
  copied assignments retain their source `integration_data` values.
- **SC-003**: When neither option is enabled, course copy behavior is
  identical to the current behavior — no integration fields are
  carried over.
- **SC-004**: The options are available through both the UI and the API,
  allowing automated workflows to use them.
- **SC-005**: The integration copy options appear for every user who can
  initiate a course copy, with no additional permission barriers.

## Assumptions

- The feature applies only to assignments. Other content types that may
  have integration fields (e.g., quizzes, discussion topics) are out of
  scope for this feature unless explicitly requested later.
- The two options are independent — enabling one does not require or
  imply enabling the other.
- The feature does not validate whether a copied `integration_id` is
  unique in the destination context. External systems are responsible
  for handling any collisions.
- The feature works for both "Copy all content" and "Select specific
  content" modes of course copy.

## Future Work

- **Blueprint sync**: Carry over integration fields when a Blueprint
  course syncs to associated courses.
- **Course templates**: Carry over integration fields when a course is
  created from a template.
- **Other content types**: Extend to quizzes, discussion topics, or
  other entities that have integration fields.
