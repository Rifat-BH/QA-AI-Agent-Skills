---
name: qa-api-testing
description: "Tests a REST API directly (via browser fetch, curl, Postman or an API client) for a ticket: discovers the contract from code or OpenAPI, runs happy path, validation, boundary, negative, conflict, auth and concurrency calls, checks status codes, response shape and error bodies, verifies side effects downstream, and records evidence. Use when the user says \"test the API\", \"call the endpoint\", \"check the 409/400 behaviour\", \"conduct testing by you\", or when UI testing is slower than hitting the API."
---

# QA API Testing

## Before calling anything

- Get the contract: request DTOs, validators, error messages, status mapping
  (from code or OpenAPI/Swagger). Note required headers (auth, CSRF, tenant).
- Confirm the base URL and environment, and that it is reachable (health
  endpoint). If unreachable, check VPN/network before anything else.
- Never put real personal data, credentials or tokens in requests you share.
  Use the session the user is already signed into; never sign in for them.

## Calls to make

| Type | Examples |
|---|---|
| Happy path | minimum fields; all fields |
| Validation | each required field missing; wrong format; whitespace; over-length; future/invalid dates |
| Business rules | duplicates, conflicts (409), state rules (pending, locked) |
| Not found / bad id | 404, malformed id 400 |
| Auth | no token 401, wrong role 403, other tenant |
| Idempotency | same request twice; two requests in parallel (`Promise.all`) |
| Contract | response shape, field names, nulls vs empty, no extra/missing fields |
| Side effects | downstream records, messages, links; poll for async results |

## Running from a browser session

- Open a page on the API's own origin (e.g. its health endpoint) so `fetch` is
  same-origin, then run small scripts.
- Use unique names with a run suffix. Keep scripts short; long waits may time
  out, so poll in steps.
- If output is filtered because a key looks sensitive, rename keys in your
  summary object.

## Evidence

Per call: method, path, key request fields, status, the relevant response
fields or error detail, and created ids. Summarise in a table:

| # | Call | Expected | Actual | Result |

## Rules

- Do not create data you cannot identify later; list created test records at
  the end for cleanup.
- Do not test destructive endpoints against shared or real data without asking.
- A 2xx is not a pass until the side effect is verified.
