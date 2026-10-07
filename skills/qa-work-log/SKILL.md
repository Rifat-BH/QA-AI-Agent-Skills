---
name: qa-work-log
description: "Builds a timesheet or work log from a period of QA work: groups activities by day, assigns hours and activity categories (testing, test case writing, bug reporting, requirement analysis, R&D, peer review, documentation, meetings, communication), adds meetings from a calendar and fixed daily items, hits a target total, and outputs a table ready to enter into a time-tracking tool. Use when the user asks to \"prepare my work log\", \"fill my timesheet\", \"make it 40 hours\", \"tabular form\", or shares a calendar and a list of tasks."
---

# QA Work Log

## Inputs

- The work done (tickets, bugs, PRs, investigations), from the conversation,
  notes, or another log the user pastes.
- Calendar for meetings (titles, durations). Skip cancelled ones.
- Fixed items the user specifies (e.g. 30 minutes of misc per day).
- The target total (e.g. 40-45 h, or 50+ h) and the date range.
- The activity categories available in the user's tool. If some are not
  visible, use the closest and tell the user which to swap.

## Method

1. List every work item with its date (best evidence of when it happened).
2. Add meetings from the calendar with their real lengths.
3. Add fixed daily items.
4. Distribute remaining hours over work items so each day is plausible and the
   total hits the target. Keep the user's own entries' hours unless asked.
5. One line per entry: concrete outcome, ticket id, no fluff.

## Output

| Date | Time | Activity | Work done |
|---|---|---|---|

Subtotal row per day, grand total at the end. Times in `XhYY` format.

## Rules

- Descriptions say what was achieved ("verified X; raised BUG-n"), not "worked on X".
- Do not invent work that did not happen; only redistribute time.
- No confidential detail beyond ticket ids if the log leaves the company.
- Flag assumptions (estimated meeting lengths, missing days) in two or three
  bullets after the table.
