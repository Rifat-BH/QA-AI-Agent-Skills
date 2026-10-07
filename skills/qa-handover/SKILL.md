---
name: qa-handover
description: "Prepares a QA handover when leaving a project, going on leave, or rotating off a team: open tickets and their exact state, pending retests, open bugs and who owns them, test data created and cleanup needed, environment/access notes, automation PRs in flight, and short messages to the people taking over, plus an optional farewell note. Use when the user says \"today is my last day\", \"handover\", \"I'm going on leave\", \"pass this to <name>\", or \"message to everyone\"."
---

# QA Handover

The next person should be able to continue without asking you anything.

## Collect

| Area | What to list |
|---|---|
| Tickets in QA | id, state (passed / in progress / blocked), what remains, where the final comment is |
| Bugs | id, priority, owner, status, retest needed? |
| PRs | automation or test PRs in flight, review comments resolved or pending, who takes over |
| Test data | records created (names/ids), what must be cleaned up and by whom |
| Environments | URLs, which VPN/access each needs, known quirks (tenant missing in one system, flaky host) |
| Test assets | where test cases, specs, scripts and reports live |
| Open questions | asked to whom, waiting since when |

## Output

1. **Handover note** (doc or message): the tables above, short.
2. **Per-person messages**: each names exactly what that person should do, with
   links. One ask per person where possible.
3. **Farewell message** (if wanted): short, warm, no project details.

Farewell template:
> Hi everyone, today is my last day on the team. It's been a real pleasure
> working with all of you, and I've learned a lot. Thank you for the support
> along the way. Wishing you all the very best, and I hope our paths cross again.

## Rules

- No credentials in the note; say where they are stored and who grants access.
- Mark anything you could not finish clearly as "not done".
- Keep client-confidential material where it belongs; share links, not copies.
