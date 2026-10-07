---
name: qa-team-communication
description: "Drafts short, clear QA messages for chat or email: questions to a developer, status updates to a lead, replies to a dev who says something is fixed, blocker escalations, requests for access or logs, and nudges to move a ticket. Keeps them brief, specific, polite and evidence-based. Use when the user says \"what should I ask the dev\", \"make a short message\", \"reply to him\", \"update my lead\", \"just a line\", or \"tell them about this bug\"."
---

# QA Team Communication

Busy people read the first line. Put the point there.

## Shapes

**Question to dev**
> Hi <name>, on <TICKET>: <observed behaviour, with one id>. Is that expected,
> or should it <expected behaviour>? <optional: I can raise a bug if not.>

**Status to lead (one line)**
> <tickets> done; <n> bugs raised (<ids>); <ticket> blocked on <thing>.

**Reply to "it's fixed"**
> Thanks, retesting on build <x> now. Will update <ticket> by <time>.
> or: Retested on <build>: <fixed / still failing with id X>. <next step>.

**Blocker / access request**
> I'm blocked on <TICKET>: need <access/log/data> to verify <criterion>.
> Could you <specific action>? Without it I'll mark <scenario> as not verified.

**Asking someone to post or move a ticket**
> <TICKET> QA is done and passes. Final comment is here: <link>. Could you post
> it and move the ticket on? It also mentions <bug> (pre-existing, not from this
> ticket).

**Bug heads-up**
> Found a <priority> issue while testing <TICKET>: <one-line symptom>. Not caused
> by this ticket; raised as <BUG>. Details and evidence in the bug.

## Rules

- Lead with the point; one ask per message.
- Concrete ids beat descriptions.
- No blame, no hedging words; state what you saw.
- Say what you will do next and when.
- Never paste credentials, tokens or personal data.
- Match the user's tone and language; keep it short unless asked otherwise.
