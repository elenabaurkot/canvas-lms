<!--
Sync Impact Report
==================
Version change: 0.0.0 (template) → 1.0.0
Modified principles: N/A (initial population)
Added sections:
  - Core Principles (7 principles)
  - Contribution & Pull Request Process
  - Development Workflow & Code Review
  - Governance
Removed sections: None
Templates requiring updates:
  - .specify/templates/plan-template.md — ⚠ not found (no action needed)
  - .specify/templates/spec-template.md — ⚠ not found (no action needed)
  - .specify/templates/tasks-template.md — ⚠ not found (no action needed)
  - .specify/templates/commands/*.md — ⚠ not found (no action needed)
Follow-up TODOs: None
-->

# Canvas LMS Constitution

## Core Principles

### I. Code Style Compliance

All contributions MUST follow the established Canvas LMS style guides.
Non-compliance will block acceptance.

- **Ruby**: Follow the [bbatsov Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide)
  with Canvas-specific exceptions: `!!` coercion is acceptable, trailing
  commas in Array/Hash literals are acceptable, `raise` over `fail`,
  stabby lambdas (`->`) preferred, 120-character line limit, normal
  indent over alignment to first paren.
- **JavaScript**: Follow [AirBnB's JS Style Guide](https://github.com/airbnb/javascript).
  Exception: `underscored_method_name` casing is acceptable when
  dealing with `.to_json`-serialized Ruby data. Never use `$.fn.html`
  to set content (use Handlebars templates). Never manually construct
  HTML snippets. Use `js_env` to pass data from Rails to JS.
- **ActiveRecord/SQL**: Hash condition syntax is preferred, then array,
  then literal. Pass AR objects instead of IDs. Use `exists?` over
  `count > 0`. Prefer `preload` over `includes`. No options on
  associations. No `default_scope`. Quote table names in raw SQL FROM
  clauses. Prefer `<>` over `!=` in SQL.
- **CSS**: Use soft-tabs with two-space indent. Prefer SCSS for new
  files. Document components for the styleguide.
- **Selenium**: No xpath. Use CSS selectors via `f`/`ff`/`fj`/`ffj`
  helpers. Do not depend on element types or exact descendant paths.
  Do not rely on English link text. Prefer `name` attributes, then
  `id`, then `class` for selectors.
- **Linters**: Run `script/eslint` (JS), `script/rlint` (Ruby), and
  `script/stylelint` (CSS/SCSS) before submitting. Contributed code
  MUST pass linters, but sweeping lint-only changes to existing code
  MUST be avoided.
- **Motto**: "Always leave code better than you found it."

### II. Internationalization (I18n)

All user-facing strings, dates, times, and numbers MUST be
internationalized. This is non-negotiable and verified on every commit.

- Use `t("English text")` — do not manage `.yml` files manually.
- Always use literals as the default translation argument, never
  variables or expressions.
- Call `I18n.t` at runtime; never cache translations in constants or
  singletons.
- Interpolate, do not concatenate — preserve word-order flexibility
  for translators.
- Use `count:` for pluralization instead of `pluralize`.
- Use Canvas date/time formatters (`date_string`, `localize`) instead
  of raw `strftime`.
- Use Rails number helpers (`number_to_currency`,
  `number_with_precision`, etc.) for locale-aware formatting.
- Validate with `rake i18n:check` before submitting.

### III. Testing Discipline

Every commit MUST include tests and a test plan. The test plan MUST
be included in the commit message.

- Tests MUST cover all necessary cases for the change.
- The commit message MUST contain a `Test Plan:` section enumerating
  manual verification steps.
- Run targeted tests locally before submitting a pull request.
- JS tests: `yarn test`, `yarn test:vitest`, or `yarn test:watch`.
- Ruby tests: `bin/rspec path/to/spec`.
- Selenium tests: follow the Selenium Styleguide — do not test
  animations, avoid redundant page loads, seed data instead of
  UI-creating it repeatedly.

### IV. Multi-Tenant & Sharding Awareness

Canvas runs as a multi-tenant environment with database sharding
(Switchman). All code MUST respect this architecture.

- Enhancements affecting all institutions MUST use the Plugin
  architecture so they can be scoped to specific root accounts.
- Use hash syntax in ActiveRecord queries to enable automatic shard
  inference and ID translation.
- Prefer `preload` over `includes` to avoid broken cross-shard
  queries.
- Quote table names (`Model.quoted_table_name`) in raw SQL FROM,
  JOIN, UPDATE, and DELETE targets to support multi-shard databases.

### V. Accessibility

All UI changes MUST be accessible to screen readers and other
assistive technology devices. This is a non-negotiable review
criterion.

- New UI MUST be built in React using the documented Canvas API.
- Follow WCAG guidelines and use Instructure UI components where
  available.
- Reviewers will verify accessibility compliance during code review.

### VI. Performance

Code MUST remain performant under heavy load.

- Use `exists?` instead of `count > 0` or `first.present?`.
- Use `pluck` instead of `select(:col).map(&:col)` to avoid
  instantiating AR objects unnecessarily.
- Use `bulk_insert` for large batch inserts.
- Use `preload` over `eager_load` unless a JOIN is specifically
  needed for filtering.
- For expensive queries that do not require real-time data, run on
  the replica: `Shackles.activate(:slave) { ... }`.
- Use `limit` with `update_all`/`delete_all` in data fixups to
  process in batches.

### VII. Security

Code MUST not introduce XSS, SQL injection, or other
vulnerabilities.

- Never use `$.fn.html` with user data — use `$.fn.text` or
  Handlebars templates.
- Never manually construct HTML snippets with string concatenation.
- Prefer hash syntax in ActiveRecord conditions (immune to SQL
  injection).
- Never use string interpolation for SQL values
  (`where("col=#{val}")`).
- Use array syntax with placeholders when hash syntax is not
  possible (`where("col=?", val)`).

## Contribution & Pull Request Process

All contributions MUST follow the Instructure contribution process:

- Sign the Instructure Contributor Agreement (ICA) before submitting.
- Develop against the `master` branch, not `stable`.
- Each pull request SHOULD consist of a single, focused commit (or a
  small set of focused commits — no "train of thought" history).
- Commit messages MUST follow this format:
  ```
  Summary of the commit (Subject)

  Further explanation focusing on why the approach was chosen.

  closes gh-<issue_number>

  Test Plan:
    - Step-by-step verification instructions
    - Include role context (student, teacher, admin)
  ```
- Keep each line in commit messages under 60 characters.
- Review checklist applied to every commit:
  1. Changeset is checked out and tested locally by reviewer.
  2. Commit message includes a test plan.
  3. Tests and test plan cover all necessary cases.
  4. Code follows language coding conventions.
  5. Code is well designed and architected.
  6. All user-facing strings/dates/times/numbers are internationalized.

## Development Workflow & Code Review

- All code is reviewed by at least one Instructure engineer.
- Every line is read; the reviewer checks out and tests the change.
- After code review passes, QA engineers execute the test plan.
- UI/behavior changes also require product manager review.
- Feature enhancements SHOULD use Canvas Plugins or Feature Flags for
  gradual rollout in the multi-tenant environment.
- Coordinate through the mailing list for anything beyond a bug fix.

## Governance

This constitution is the authoritative reference for contribution
standards to Canvas LMS. All pull requests and code reviews MUST
verify compliance with these principles.

- Amendments require documentation in this file, version increment,
  and team review.
- Version follows semantic versioning:
  - MAJOR: Principle removals or backward-incompatible redefinitions.
  - MINOR: New principles added or materially expanded guidance.
  - PATCH: Clarifications, wording fixes, non-semantic refinements.
- Compliance is verified during code review. Non-compliant commits
  will be rejected or returned for revision.
- Use AGENTS.md for runtime development guidance (commands, Docker
  tips, project structure).

**Version**: 1.0.0 | **Ratified**: 2026-02-22 | **Last Amended**: 2026-02-22
