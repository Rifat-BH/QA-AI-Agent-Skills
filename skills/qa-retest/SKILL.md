---
name: qa-retest
description: "Re-verifies a fixed bug or a ticket returned to QA: confirms the fix is deployed, re-runs the original failing steps on fresh data, checks the fix did not break nearby behaviour, re-verifies previously passed criteria that the fix touched, and gives a clear pass / reopen decision with evidence. Use when the user says \"retest\", \"reverify\", \"the PR is merged, check again\", \"dev says it's fixed\", \"back to QA ready\", \"what remains to retest\", or \"can we pass now\"."
---

# QA Retest

A retest answers one question with evidence: is it fixed, and did fixing it
break anything near it?

## Step 1 - Confirm what changed and that it is deployed

- Read the fix PR: what changed, which files, and any new behaviour or new
  error messages. A fix can change expected results of other cases.
- Confirm deployment: PR merged, deploy pipeline green for that commit, the
  environment shows the new build. Do not retest an old build.
- Read the dev's comment on the fix; note any behaviour they changed on purpose.

## Step 2 - List what to retest

| # | What | Why |
|---|---|---|
| 1 | Original failing steps | The bug itself |
| 2 | Variants of the bug (other inputs, other path, other tenant) | Fix may be narrow |
| 3 | Previously passed cases touching the changed code | Regression |
| 4 | New behaviour introduced by the fix | It needs its own check |

Mark what is already verified and does not need repeating. Mark what cannot be
retested by QA (needs logs, fault injection) and how it is covered instead.

## Step 3 - Retest on fresh data

- **Create new test data.** Records created before the fix may hold the broken
  state and will make a working fix look broken. If an old record still shows
  the problem but a fresh one does not, the fix works and the old record needs
  a data cleanup (say so, with ids).
- If the original record must be reused, clean it first and say how.
- Use the same environment, tenant and steps as the original report.

## Step 4 - Decide

| Result | Decision |
|---|---|
| Original and variants pass, no regression | **Pass** |
| Original passes, a variant fails | Reopen with the variant as new steps |
| Original passes, unrelated older behaviour fails | Pass the fix; raise a **new linked bug** |
| Original still fails on fresh data | **Reopen** with new evidence and build number |
| Cannot verify (access, environment down) | **Blocked**, say exactly what is needed |

If an earlier theory about the cause was wrong, say so in the update.

## Output

```markdown
**Retest <TICKET/BUG>: <Pass | Reopen | Blocked>** (build <version>)

| # | Check | Result | Evidence |
...

Data cleanup needed: <ids or none>
Next: <close / reopen with steps / new bug for X>
```

Then produce the ticket comment with `qa-final-comment`.

## Rules

- Never retest without confirming the build.
- Fresh data first; old data second, as a separate check.
- Evidence per row (record id, response, screenshot, query result).
- Keep it short; the reader wants the decision first.
