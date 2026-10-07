---
name: qa-bug-report
description: "Drafts a clear, ready-to-post bug report: title, priority with reason, links, environment, exact steps, expected vs actual, confirmed or unconfirmed cause, impact, evidence and cleanup. Decides whether to raise a new bug or reopen a ticket, and which tickets to link. Use when the user says \"write a bug report\", \"prepare the bug\", \"give me a bug with priority and ticket link\", \"post this bug\", \"combine these into bugs\", or after qa-bug-analysis confirms a defect."
---

# QA Bug Report

A bug report is read by someone busy. They should understand what is broken,
how to see it, and why it matters, in under a minute.

## New bug or reopen?

| Situation | Action |
|---|---|
| Ticket still in QA and its own AC fails | Reopen / send back with comment |
| Ticket is Done and a regression appears | **New bug**, linked to the ticket |
| Defect in code the ticket did not change | **New bug**, linked as related; do not fail the ticket |
| Defect caused by an interaction of two tickets | New bug linked to both; say which caused it and which it breaks |
| Same root cause as an existing bug | Add evidence to the existing bug instead |

Mark a bug as **blocking** a parent epic when the epic's goal cannot be met
without the fix.

## Priority

| Priority | When |
|---|---|
| Urgent | Data written to the wrong record or tenant, data loss, security or privacy exposure, production down |
| High | Core flow broken, silent failure with no error, data stops syncing, many records affected |
| Medium | Feature works with a workaround, wrong message, partial data |
| Low | Cosmetic, rare edge, minor wording |

Always give a one-line reason.

## Template

```markdown
**Title:** <what is broken, in user terms, specific>

**Priority:** <level> (<one-line reason>)
**Label:** Bug
**Links:** Blocks <epic> | Related to <ticket> | Caused by <ticket>
**Environment:** <env>, build <version/sha>, <tenant/account>

**Steps**
1. ...
2. ...

**Expected:** <observable correct behaviour>

**Actual:** <observable wrong behaviour, with ids/values>

**Control (if any):** <similar case that works, and the one difference>

**Cause:**
- <Confirmed: file:line, what it does wrong, data that proves it>
- <or: Not confirmed. Needs: logs / query / dev check>

**Impact:** <who and how many; count affected records where possible>

**Evidence:** <screenshots, request/response, query + result, record ids>

**Cleanup:** <test or orphan data that must be removed, with ids>
```

## Rules

- **Title says the symptom, not the guess.** "Edits made in system A never reach
  records created from system B" beats "Sync broken".
- Steps reproduce from a clean start with concrete values.
- Expected and Actual are both observable; no "should work".
- Separate confirmed cause from hypothesis, explicitly.
- Count the impact when you can (a query returning how many records are
  affected turns "some" into a number).
- No credentials, personal data, or internal hostnames that should not leave
  the team. Use test records.
- One defect per report. Several failed tests with one cause = one report.
- If the user wants several findings "combined", give a short table first
  (title, priority, ticket) and then each report.
- Offer one line on where to file it and what to link; do not file it yourself
  unless asked.
