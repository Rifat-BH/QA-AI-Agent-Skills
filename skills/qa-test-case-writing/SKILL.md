---
name: qa-test-case-writing
description: "Writes clear, atomic manual test cases for a feature, ticket or epic in a test-management-ready format (CSV importable into tools such as Testmo, TestRail, Zephyr, Xray or Azure Test Plans): title, objective, preconditions, test data, numbered steps, expected result, priority and status. Covers happy path, validation, boundaries, negative, permissions, cross-system and regression cases, maps every case to an acceptance criterion, and keeps existing suites current when behaviour changes. Use when the user asks to \"write test cases\", \"create test cases for this ticket/epic\", \"update the test cases\", \"export test cases to CSV\", or \"which test cases does this ticket cover\"."
---

# QA Test Case Writing

Produce test cases a tester who has never seen the feature can execute, and a
test-management tool can import without editing.

## Inputs

- The ticket(s) or epic, acceptance criteria, design notes, and any existing
  test-case file for the area.
- The analysed scope from `qa-requirement-analysis` if available.

## Format

Default columns (match the user's existing file if they have one):

| Column | Rule |
|---|---|
| Test Case Description | `Verify user can ...` / `Verify user receives 400 when ...` - one behaviour |
| Objective | Why it matters, with the AC or ticket reference |
| Pre-condition | State that must exist first (environment, role, data, flag) |
| Test Data | Concrete values, or N/A. Never real personal data or credentials |
| Steps | Numbered `1. ... 2. ...`, one action per step, UI labels in quotes |
| Expected Result | Observable and checkable; names the field, message or value |
| Priority | High / Medium / Low (High = core flow, data integrity, security) |
| Status | Not Executed / Passed / Failed / Blocked / N/A |

A template is in `templates/test-cases-template.csv`.

## Coverage checklist (per AC)

1. **Happy path** with minimum required fields, then with all optional fields.
2. **Validation**: each required field missing; invalid format; whitespace-only.
3. **Boundaries**: max length, max length + 1, min, zero, empty, today, future,
   very old dates.
4. **Negative and error handling**: server error, timeout, conflict (409),
   not found (404), unauthorised (401/403). The user sees a clear message and
   input is preserved.
5. **Idempotency and concurrency**: double-click submit, two users at once,
   repeated request.
6. **Cross-system**: the record reaches each downstream system with correct
   values; edits flow both directions; no duplicates; echo loops do not
   overwrite.
7. **Permissions and tenant isolation**: other organisations' data never shows.
8. **Regression**: neighbouring flows still work.
9. **UI states**: loading, empty, error, disabled buttons, unsaved-changes
   prompts.

Use `qa-edge-case-hunting` for a deeper list.

## Rules

- **One behaviour per case.** If the expected result has "and" joining two
  unrelated checks, split it.
- **Every case maps to an AC or a stated risk.** A case with no AC means an AC
  is missing; flag it.
- **Expected results are observable**: "Error 'First name is required' appears
  under First Name and Save stays disabled", not "validation works".
- **Write for the current behaviour.** When a later ticket changes behaviour,
  update the expected result of existing cases, mark removed flows `N/A` with
  the reason, and list the changed case numbers in your reply.
- **Mark Blocked with the reason** (feature not deployed, dependency missing).
- No client names, real customer or personal data, or credentials in shared files.
- Keep wording consistent so cases sort and filter well.

## Mapping cases to a ticket

When asked which cases a ticket covers, output:

| # | Case | Result |
|---|---|---|

Then list: cases to update (behaviour changed), cases now N/A, and new cases to
add (tested but not in the suite).

## Output

Write the CSV (UTF-8, header row, quoted multi-line cells) to the user's
folder and summarise: count by priority, ACs covered, and gaps.
