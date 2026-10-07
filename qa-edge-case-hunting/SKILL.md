---
name: qa-edge-case-hunting
description: "Finds corner and edge cases for a feature or change using a structured checklist: input boundaries, dates and time zones, formats and encodings, state and lifecycle, concurrency and idempotency, multi-tenant and cross-system identity, async and eventual consistency, permissions, limits of downstream systems, and failure handling. Prioritises the cases most likely to break given the actual code change. Use when the user asks \"any corner cases?\", \"what edge cases remain?\", \"what else should I test?\", \"is there any gap?\", or before signing off a ticket."
---

# QA Edge-Case Hunting

Edge cases live where the code makes an assumption. Read the change, find the
assumptions, then attack them. Do not dump the whole checklist on the user:
pick the items the change actually touches and rank them.

## Checklist

**Input values**
- Empty, whitespace-only, leading/trailing spaces, very long (max, max+1)
- Special characters: apostrophe, hyphen, accents, emoji, `<script>`, SQL quotes
- Numbers: 0, negative, decimals, very large, leading zeros, formatted (`(555) 123-4567`)
- Duplicates within one request (same phone twice, same id twice)

**Dates and time**
- Today, future, far past (systems often reject dates before 1753 or 1900)
- Invalid dates (31 Feb), partial input, locale formats
- Time zones: storage vs display, DST change, slots near midnight
- Business hours, closed days, lunch blocks, "before hours"

**Downstream limits**
- Field lengths and types in every system the data reaches (a value valid in
  one system may be truncated or rejected in another)
- What happens on truncation: is the result still valid (a cut ZIP+4 keeps the
  hyphen and loses a digit)?
- Required fields in the downstream system that the upstream form does not send

**State and lifecycle**
- Create, edit, delete, restore, re-create with the same key
- Pending/in-progress states: what can and cannot be done while pending
- Records stuck forever in an intermediate state, and whether anyone is told
- Old records created by earlier code versions

**Identity across systems and tenants**
- The same id in two tenants/organisations/offices: is the qualifier checked?
- Lookups that take "the first match" and ignore the tenant or type
- Different stored formats for the same id (`123` vs `OFFICE:123`)
- Linking to an id that does not exist in the other system

**Concurrency and idempotency**
- Double-click submit; two requests in parallel; retry after timeout
- Redelivered messages; replayed events; out-of-order arrival
- Optimistic concurrency (stale version), last-write-wins surprises

**Async and sync**
- Eventual consistency windows; poll, don't sleep
- Echo loops (a write comes back as an inbound change)
- Both directions of a sync, separately
- Search indexes and derived fields updated (can the record be found by every
  identifier after the change?)

**Permissions and privacy**
- Other tenants' data never visible; role without permission; expired session
- Sensitive fields masked in UI, logs and responses

**Failure handling**
- Downstream down, slow, or returning an error: is the user told? Is input kept?
- Partial failure: nothing half-written, or a clear compensating action
- Error messages: specific, correct, not the generic fallback

**UI**
- Required-field markers vs real validation; disabled buttons with no reason shown
- Unsaved-changes prompts; back/close; refresh mid-flow
- Displays for missing values (`undefined`, `null`, `NaN` leaking to screen)
- Flows removed or replaced by later changes

## How to use

1. Read the diff or the feature. List the assumptions it makes.
2. Pick the checklist items that attack those assumptions.
3. Rank: data integrity and wrong-record writes first, silent failures second,
   validation third, cosmetic last.
4. For each, give: case, steps (short), expected, who can run it.
5. Run what you can; hand the rest over with exact steps.

## Output

| # | Edge case | Why it might break | Expected | Who |
|---|---|---|---|---|

End with the top one or two you would run first.
