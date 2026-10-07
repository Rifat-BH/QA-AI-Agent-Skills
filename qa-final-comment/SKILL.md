---
name: qa-final-comment
description: "Writes the short final QA comment for a ticket or epic: pass/fail/blocked verdict, what was verified with evidence ids, what is N/A or known limitation, open and related bugs, regression result, and what blocks closure. Also writes one-line status updates for a lead. Use when the user asks for \"the final comment\", \"QA comment for this ticket\", \"epic comment\", \"pass comment\", \"reopen comment\", \"give me the comment again\", or \"just a line to update my lead\"."
---

# QA Final Comment

The comment goes on the ticket and is read by dev, PO and lead. It must be
scannable in 20 seconds and still be useful evidence months later.

## Verdicts

| Verdict | Use when |
|---|---|
| **QA passed** | All in-scope criteria verified; open bugs are out of scope or non-blocking |
| **QA passed with known issues** | Passed, but linked bugs or limitations the PO should accept |
| **Reopened** | An in-scope criterion fails |
| **Blocked** | Cannot verify: environment, access, dependency |
| **Epic: ready to close / stays open** | Children passed; list what blocks closure |

## Ticket template

```markdown
QA passed on <env>.

- <criterion in plain words>: <result> (<evidence id: record id, external id, response code>)
- <criterion>: <result> (<evidence>)
- <edge case>: <result>

N/A: <flow not reachable / not on this environment, and why>
Covered by automated tests: <what QA cannot exercise directly>
Regression: <suite/run id>: <n/n passed; any expected failure and why>
Known limitation: <later ticket / phase>
Open: <BUG-ID> (<one line>, priority)
Related (not caused by this ticket): <BUG-ID> (<one line>)
Test data: <names/ids created, for cleanup>
```

## Epic template

```markdown
QA status: <all children pass | n of m pass>; epic <ready to close | stays open because ...>.

- <child>: <passed / bug raised>
- <child>: <passed>
- <child>: no longer needed, please cancel or fold into <child>

End to end: <one sentence proving the whole flow with example ids>.
Blocking: <BUG-ID> (<one line>).
Ready to close once <condition>.
```

## One-line updates

When the user wants "just a line" for a lead: `<tickets> done; <n> bugs raised
(<ids>); <blocker if any>.`

## Rules

- **Verdict first.**
- Bullets of one line each. No paragraphs.
- Every pass claim has an evidence id.
- Distinguish in-scope bugs from related or pre-existing ones.
- Say what was not tested and why (N/A, automated, needs logs).
- Plain words a PO understands; technical detail only as ids.
- If a later finding corrects a posted comment, give the corrected line and
  say which line to replace.
- If the user has no tracker access, format it to paste wherever they keep
  comments (doc page, chat) and suggest who should copy it onto the ticket.
