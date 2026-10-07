---
name: qa-bug-analysis
description: "Investigates a suspected defect before it is reported: reproduces it, rules out test-data, environment and wrong-database causes, compares against a working control case, traces the behaviour to the exact code line and data row, and separates a confirmed root cause from a hypothesis. Also decides whether it is in scope of the ticket under test or a pre-existing/related issue, and whether it is a regression. Use when the user says \"why is this happening\", \"is this a bug\", \"find the root cause\", \"could it be a data issue\", \"check the code\", or when a test result looks wrong."
---

# QA Bug Analysis

Find out what is really wrong, with evidence, before anyone files or fixes
anything. The most expensive QA mistake is a confident wrong root cause.

## The rule

**State a root cause only when it is proven.** Proven means you can point to:

1. the **code line** (file and line) that produces the behaviour, and
2. the **data** (row, response, message, log line) showing that path was taken.

Otherwise write "cause not confirmed" and list what evidence would confirm it.
If you stated a cause earlier and new evidence contradicts it, say so plainly
and correct it. Do not defend a theory.

## Step 1 - Reproduce

- Exact steps, environment, build/version, account, tenant, record ids.
- Reproduce with **fresh data** (unique name, new record). If it only fails on
  an old record, suspect the data first.

## Step 2 - Rule out the usual false alarms

| Check | How |
|---|---|
| Wrong environment or database | Confirm DB name, host, tenant in the query results themselves |
| Not deployed | Fix merged? Deploy pipeline green? Version endpoint shows the build? |
| Stale or corrupted test data | Leftovers from earlier versions, half-deleted rows, rows written by old code. Clean or recreate, then retest |
| Environment outage | Host down, VPN off, expired session, CI agent cannot reach a host |
| Async delay | Eventually consistent flows: poll for a reasonable window before calling it lost |
| Not saved | The UI showed the edit but did Save actually complete? Check header/audit fields |
| Misread UI | The value shown is a placeholder, synthetic/demo data, or a cached view |
| Test artefact | A skipped or not-executed test, or an assertion that cannot fail |

## Step 3 - Find a control case

Compare a **working** case with the **failing** one, changing one thing at a
time: an old record vs a new one, one tenant vs another, one source system vs
another, created via path A vs path B. The single difference that flips the
result points at the cause. Show both side by side in a small table.

## Step 4 - Trace to code and data

- Find the code path for the behaviour (handler, mapper, validator, query).
- Look for: exact-match lookups against values stored in a different format,
  "first item" selection that ignores a qualifier (tenant, office, type),
  missing branches, swallowed errors, truncation, timezone conversion, null vs
  empty handling, ordering assumptions.
- Pull the supporting data with a precise read-only query (see
  `qa-db-verification`). Give the user the query if you cannot run it.

## Step 5 - Classify

| Question | Answer options |
|---|---|
| Is it a defect? | Defect / expected behaviour / environment / data / test issue |
| Scope | In scope of the ticket / pre-existing / caused by a related ticket |
| Regression? | Worked before (which build or case) / new behaviour |
| Impact | Who is affected, how many records (count them), data integrity risk |
| Priority | Urgent (wrong-record writes, data loss, security) / High / Medium / Low |

Scope rule: if the ticket did not change the faulty code, it is **not** a reason
to fail that ticket. Raise a new bug, link it as related, and say whether it
should block the parent epic.

## Output

```markdown
**<Confirmed bug | Not a bug | Not confirmed yet>**: <one line>

Evidence:
| Case | Value | Result |
(control vs failing)

Cause: <confirmed: file:line + data> | <not confirmed: what would confirm it>
Scope: <in scope / pre-existing / related to TICKET>
Impact: <who, how many>
Next step: <file bug (qa-bug-report) / retest on clean data / ask dev for logs>
```

## Rules

- Evidence over intuition. Quote the line, show the row.
- One variable at a time.
- Say what you could not check.
- Short answer first; details after.
