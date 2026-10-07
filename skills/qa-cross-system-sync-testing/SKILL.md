---
name: qa-cross-system-sync-testing
description: "Tests data synchronisation between systems (for example a central service and one or more source/legacy systems connected by CDC, message queues, outbox/worker patterns or APIs): create-on-demand flows, links between ids, both directions of updates, echo-loop suppression, idempotency, conflict handling, field limits and mapping, and eventual consistency. Use when the user is testing \"sync\", \"write-back\", \"inbound/outbound\", \"CDC\", \"create in the other system\", \"linked ids\", or asks why a change in one system did not reach another."
---

# QA Cross-System Sync Testing

Sync bugs hide in the seams: id formats, directions, timing and loops. Test the
seams deliberately.

## Map the flow first

For each direction, write one line: `source change -> capture (CDC/event) ->
transport (queue/topic) -> worker/handler -> target write -> confirmation/link`.
Note for each hop: where you can observe it (UI, API, DB row, message, log) and
who can access it.

## Core scenarios

| Area | Check |
|---|---|
| Create in target | Record appears once, with correct field mapping, in the right tenant/office |
| Link established | Both sides store the link; note the **exact stored format** of the external id |
| Pending state | What is stored before the target confirms; what is blocked while pending |
| Pending forever | Target rejects the write (length, invalid date, missing required field): is anyone told? |
| Update A -> B | Edit in source reaches target |
| Update B -> A | Edit in target reaches source |
| Search/derived fields | Record findable by its new external id after linking |
| Echo loop | Target's own change event does not overwrite source or re-publish endlessly |
| Idempotency | Redelivered command/event creates nothing extra |
| Conflict | Linking an id another record already owns is rejected and reported |
| Multi-tenant | An id from tenant X is never used in tenant Y |
| Field limits | Over-length values truncated or rejected as designed in every target |
| Unsupported target | A system without the feature yet ignores the command safely |

## The control-case technique

When sync fails for some records, compare a record that syncs with one that does
not (created by a different path, older vs newer, different tenant). Compare
their link rows directly. A different stored id format with an exact-match
lookup on the other side is a classic cause.

## Timing

- Poll with a sensible budget (seconds to minutes) before calling it lost.
- Record timestamps on both sides; they show where it stalled.

## Evidence

Per scenario: ids on both sides, link rows, timestamps, and the UI/API view.
If a hop needs logs or queue access you do not have, mark it "covered by
automated tests / needs dev logs" and say what to look for (the log message
that would prove skip vs success).

## Reporting

Report each direction separately. Count affected records with a grouped query.
State confirmed cause only with code line + data; otherwise "not confirmed".
