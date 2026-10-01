---
name: feedlab-design-tests
description: Design manual test cases from the current codebase and register them in FeedLab via its MCP tools. Use when the user asks to design, write, or generate test cases for a feature or app, to add or sync tests to FeedLab, or to update FeedLab tests after code changes.
---

# Design tests for FeedLab

You turn code into **manual test cases** a human tester runs in a FeedLab test session, then register them through the `feedlab` MCP tools. The reader of every case is a tester with the app open and no access to the code.

If the `feedlab` tools (`list_projects`, `list_tests`, `create_test_cases`, …) are not available, stop and tell the user to create a token in FeedLab under Settings → API Tokens and connect with:

```bash
claude mcp add --transport http feedlab https://feedlab.cloud/api/mcp --header "Authorization: Bearer <token>"
```

## Steps

### 1. Scope

Pin down which features are in scope:
- **Named feature** ("the timetable"): that feature and the flows that touch it.
- **Recent changes** ("update tests for what I changed"): features touched by `git diff` against the base branch, or the commits the user names.
- **Whole app**: list the feature areas you find, confirm the list with the user, then work one area at a time.

Done when you can name every feature in scope, one line each.

### 2. Connect

Call `list_projects` and pick the project matching this repo (name, website URL, or ask if it's ambiguous). Call `list_tests` for that project. The existing suites, groups and cases are the **coverage** you build on: reuse their names and match their style.

### 3. Survey

Read the code for each feature in scope and build a **coverage map**: every behaviour a tester can observe. Hunt through:
- **Entry points**: routes, pages, screens, menu items, deep links.
- **Actions**: forms, buttons, uploads, API calls the UI triggers.
- **Rules**: validation, required fields, limits, date/time logic, calculations.
- **Roles**: who can see and do what; what a lower role is refused.
- **States**: empty, loading, error, and each status an entity moves through.
- **Integrations**: emails, notifications, payments, third-party calls.

Done when every entry point and action in scope appears on the map with its roles and rules. Cite the file for each so the plan is checkable.

### 4. Draft

Write cases from the coverage map using the case rules below. Cover the **happy path**, the **refusals** (validation and permission), and the **edges** (empty, limits, boundaries) for each behaviour. File every case in the tree:
- **Suite**: one area or module of the app, as users would name it ("Timetable", "Assignments").
- **Group**: one flow or sub-feature inside it ("Creating a timetable", "Permissions").

Reuse an existing suite or group whenever one fits.

### 5. Diff

Compare the draft with the coverage from step 2 and sort every case into one bucket:
- **New**: behaviour with no existing case.
- **Update**: an existing case whose steps or expected result the code no longer matches (give its `case_id`).
- **Archive**: an existing case for behaviour that no longer exists (give its `case_id` and the evidence).
- **Covered**: already right; leave it.

### 6. Plan

Show the user the plan before writing anything: per suite › group, a table of new cases (title, priority), then updates and archives with a one-line reason each, and totals. Wait for approval, and apply their edits to the plan.

### 7. Register

- `create_test_cases` once per group with `suite_name` and `group_name` (both are created if missing), at most 50 cases per call.
- `update_test_case` and `archive_test_case` for the approved updates and archives.

Done when every approved case is created, updated, or archived. Report the counts per suite › group and anything returned in `skipped_duplicates`, and tell the user the cases are ready under the project's Tests tab.

## Case rules

- **One behaviour per case.** The title names it as an outcome: "Teacher can publish a timetable", "Student cannot edit a published timetable".
- **Preconditions** state the role and the data that must exist ("Logged in as a teacher with one class and no timetable"). A tester sets these up before step 1.
- **Steps** are an array of short imperative actions in the tester's words: what to click, type, or open, using the labels the UI shows. Include concrete test values ("Enter `08:00` as start time"). 3–8 steps; split a longer case.
- **Expected result** is observable on screen or in an email/notification: text shown, item listed, button disabled, redirect to a page. One concrete outcome per case.
- **Priority**:
  - `high`: core flow of the feature, money, data loss, security or permissions.
  - `medium`: secondary flows, validation, common edge cases.
  - `low`: cosmetic, rare edges, copy.
- Write for the UI, in product language. Code names, file paths, and function names belong in the plan's citations, never in the case.
