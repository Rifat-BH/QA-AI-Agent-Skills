---
name: qa-spec-authoring
description: "Writes a QA Spec for a ticket BEFORE development starts: testable acceptance criteria, acceptance-level and end-to-end (incl. UI) scenarios in Gherkin, an oracle per scenario, a run gate, an Automate/Manual decision, and an automation brief (steps, locators, page objects, fixtures, assertion) a coding agent can turn straight into a test. Unit and integration coverage stays with dev. Use when the user asks to \"write a QA spec\", \"spec this ticket\", \"what acceptance and E2E cases do we need\", \"which tests can we automate for this ticket\", or before development begins on any ticket. If the ticket already has a PR, use qa-pr-review instead."
---

# QA Spec Authoring

Produce the QA half of a combined spec. The dev spec says what will be built;
the QA spec says what "built correctly" means at the two levels QA owns,
**acceptance** and **end-to-end (including UI)**, in a form a coding agent can
turn straight into test code.

## Ownership

| Level | Owner | Specified here? |
|---|---|---|
| Unit | Dev | No |
| Integration | Dev | No |
| **Acceptance** | **QA** | **Yes** |
| **End-to-end (incl. UI)** | **QA** | **Yes** |

Do not write unit or integration cases. When a criterion is only falsifiable
below acceptance level, **delegate it** to dev as a one-line coverage request
(section 5a of the output).

## Where this sits in the QA loop

1. Dev writes the dev spec on the ticket.
2. **This skill**: QA spec against the dev spec and ticket. Dev reconciles,
   answers open questions, accepts the delegation list. This is the gate.
3. Development and PR: unit + integration from the dev spec, acceptance + E2E
   from `qa-spec.json`.
4. `qa-pr-review`: does the PR contain the tests this spec asked for.
5. Merge and deploy.
6. `qa-manual-test-run`: execute the Manual scenarios, file bugs, post the
   coverage comment.

## When NOT to run

- **The ticket already has a PR** or a branch with commits: the shift-left
  window has closed. Hand off to `qa-pr-review`.
- **No dev spec exists yet.** Write what the ticket alone supports, mark it
  `DRAFT - pending dev spec`, and list what you need. Never invent the dev spec.
- **The ticket is a spike.** A spike has no acceptance criteria. List the
  decisions it must produce so the follow-up can be specced.

## Inputs

- **The ticket ID**, the only required input. Fetch title, description,
  technical details, acceptance criteria, labels, parent and sub-issues.
- **The dev spec**, read from the ticket, not requested.
- **Sibling context**: parent, sibling sub-issues, and any blocking spike.
- **Dev comments on the ticket.** A later dev or reviewer comment can change or
  correct the written ACs. Test against the latest agreed behaviour and say
  which comment changed what.
- **The target repo's own test conventions**: README files, testing guides,
  existing specs of the same layer. Never invent a framework, folder or naming
  scheme. If a needed layer does not exist, list it as a setup task.
- **Existing unit and integration tests**, so you do not duplicate them (Step 2).

Do not ask the user for ACs, test data or environments. Derive what you can;
put the rest in Open Questions.

## Step 1 - Normalize the requirement

Rewrite ticket + dev spec into individually testable criteria (`AC-1`, ...).
Sharpen vague language without inventing numbers: "loads quickly" becomes an
Open Question, never an AC with a number you chose. Split compound criteria;
they fail independently.

## Step 2 - Assign each criterion a level

**First, check what is already proven below.** Search unit and integration
tests for each AC's behaviour. Mapping tables, conversions, guard clauses, null
handling and parsing are usually unit-tested already. A higher level adds only:

- proof that the **deployed chain** carries the value between systems, and
- proof that the **far end stores or renders it correctly**.

Record existing coverage in section 3 and drop or narrow the scenario.

**Then assign the level:**

1. Provable against the API contract, a messaging boundary or a cross-system
   round trip? -> **acceptance**. Prefer this: faster and stabler.
2. Needs a real user journey through the UI? -> **E2E**.
3. Neither (pure internal logic)? -> delegation list for dev.

## Step 3 - Write the scenarios

```gherkin
Scenario: <name>
Given <starting state>
When <action>
Then <observable result>
And <further observable result>
```

For every AC cover: happy path, each actor, negative and edge input, and any
cross-system or feature-flag path it implies. A scenario with no AC means an AC
is missing: add the AC.

**Multiply scenarios only where the code multiplies.** A mapping table earns a
case per value; a cross-system path earns a case per direction. A pass-through
string earns one case for the awkward input, riding on an existing scenario.
Use `qa-edge-case-hunting` for the edge-case checklist.

## Step 4 - Pin the oracle, and check where it works

For each scenario state **how pass/fail is observed**, at the real boundary.
Prefer, in order:

1. **A signal your own system exposes**: HTTP response, a status, an external
   id or audit record served by your own API. Portable to every environment.
2. **The target system's own API.**
3. **The target system's UI.**
4. **Direct SQL against another system's database.** Usually **development
   only**; QA rarely has DB access in staging or production. Choosing it confines
   the scenario to dev.

Rules:

- Different external systems expose different mechanisms (one has an API,
  another only a DB or a UI). Confirm per system; never assume symmetry.
- **An E2E scenario asserts at the UI.** If the honest oracle is an API
  response, it is an acceptance case with the wrong label.
- **State per scenario which environments its oracle works in.**
- **No portable oracle?** Ask in 5b for the product to expose one. A write that
  fails silently is invisible in production too.
- State pre-requisite data and whether it is seeded or found. Credentials are
  never test data: use `"$ENV_VAR_NAME"` placeholders.

## Step 5 - Flags, versions, environments

Per scenario: which feature-flag states it must pass under, which contract
version it targets, and which environments it runs in. An AC with no flag
answer is an Open Question.

## Step 6 - Assign a run gate (by dependencies, not by level)

| Gate | What belongs there | Trigger |
|---|---|---|
| `pr` | needs only your own stack; externals stubbed at the boundary | every push, blocks merge |
| `post-deploy` | needs an external system, a real message bus, or a deployed UI | on deploy to dev |
| `nightly` | long or broad regression | schedule |

- A scenario whose oracle is an external system is **never** `pr`.
- A UI scenario on your own stack **may** be `pr`.
- Prefer `post-deploy` over `nightly`: failure lands next to its cause.
- Say what a non-`pr` gate costs: **not proven at merge time**.
- **Read the pipeline; do not assume the gate exists.** Check: does the stage
  exist and is it enabled (no `condition: false`)? Is the pipeline registered
  in CI at all? What triggers it (change-detection scripts may need extending)?
  Can its agent pool reach every system the scenario needs?
- A suite that only runs on a developer's machine is not a gate. Mark those
  scenarios `Manual` until a pipeline exists.

## Step 7 - Non-functional expectations

Only where ticket or dev spec implies one, and only as an agreed number.
"Real-time" and "fast" are Open Questions.

## Step 8 - Automate or Manual, by testability

`Automate` if code can execute the scenario and check its oracle in a layer and
stage that exists. Otherwise `Manual`. **There is no target ratio.**

`Manual` is right when: the oracle is human judgement (visual fidelity,
readability); it is exploratory; the layer or stage does not exist yet; or the
oracle needs access automation lacks. Say which. "Manual because a layer is
missing" is a setup task to name, not a permanent state.

## Step 9 - Automation brief for every `Automate` scenario

1. **Concrete step sequence**: the real path, one line per step, naming the
   element or call. Not a restatement of the Gherkin.
2. **Locators**: accessible role + name, then label/placeholder, then a stable
   test id. Never CSS/XPath tied to layout, never positional (third card,
   first row). **Missing hook? Do not invent a selector; ask dev in 5b.**
3. **Page objects / API client**: reuse first, name any new ones.
4. **Fixtures, setup, teardown**: independent, repeatable, parallel-safe.
   Unique data per run (suffix a run id). State what teardown removes.
5. **The assertion**: exact field and value, schema, row/column, message
   property. "Assert success" is not an assertion. Every assertion carries a
   failure message in product terms.

Stability rules: **wait on state, never on time**; name every async boundary
and its poll timeout (eventually consistent flows need a poll, not one read).

**Script timing** per scenario:

- `in-loop`: API/messaging acceptance tests. Written in the same PR.
- `after-build`: UI tests. Locators can only be confirmed against a built UI.
  Name who writes it and when.

## Naming and placing a spec file

Always computed from the repo, never remembered:

1. Find the target project from the scenario's level.
2. List every existing spec file across all its folders.
3. Match the pattern you find. If there is a flat numbered sequence
   (`TC<NNN>_...`), the next number is `max(all) + 1` across all folders.
4. State the computed name and mark it **provisional** (parallel tickets collide).

## Output - `qa/<TICKET-ID>/QA-SPEC.md`

```markdown
# QA Spec - <TICKET-ID>: <Title>
Scope: acceptance + end-to-end (incl. UI). Unit and integration: dev spec.
Status: <READY | DRAFT - pending dev spec | BLOCKED - see section 12>

## 1. What QA understands this to be
## 2. Acceptance Criteria (testable)
- AC-1: ...
## 3. Out of scope for QA on this ticket
- Already proven below: <behaviour> - <test file>
## 4. Coverage map
| AC | Level | Where it belongs | Gate | Why |
## 5. Asks of dev
### 5a. Delegated unit/integration coverage
| AC | Why not provable higher | Ask |
### 5b. Testability asks (test ids, seed/reset endpoints, completion signals)
| Need | Where | Why | Scenario(s) |
## 6. Scenarios
### S-1: <name> - AC-n - <acceptance|E2E> - <Automate|Manual> - gate: <...>
- Pre-requisite / Test data / Oracle / Oracle works in / External deps
(gherkin)
**Automation brief** (Automate only): spec file (provisional), written
(in-loop|after-build, by whom), steps, locators, page objects, fixtures,
assertion + failure message, async poll + timeout, skip-if precondition absent.
## 7. Run matrix
| Scenario | Gate | Flags | Version | Environments |
## 8. Not proven at merge time
## 9. Non-functional expectations
## 10. Automated vs manual (with reason per Manual)
## 11. Planned touch surface (intent-based; PR review re-derives from diff)
## 12. Open Questions (blocking)
## 13. Repo and pipeline state checked (what, when, from which file)
## 14. Definition of ready
- [ ] Every AC in section 4 or 5a
- [ ] Existing lower-level coverage checked
- [ ] Every scenario has oracle, data, deps, environments
- [ ] No `pr` scenario depends on an external system
- [ ] Every Automate scenario has a complete brief
- [ ] Every locator is role/label/test id; gaps are 5b asks
- [ ] No open blocking question
- [ ] Dev accepted 5a, 5b and section 11
```

Also emit `qa/<TICKET-ID>/qa-spec.json` (one object per scenario: `id`, `ac`,
`layer` (acceptance|e2e), `path`, `automate`, `gate`, `ui`, `external_deps`,
`prerequisite`, `test_data`, `oracle`, `oracle_environments`, `covered_below`,
`gherkin`, `flags`, `contract_version`, `environments`, and `automation_brief`
when `automate` is true). `qa-pr-review`, `qa-e2e-automation` and
`qa-manual-test-run` all read it.

## Hard rules

- Acceptance and E2E only; lower levels are a request to dev.
- Never duplicate coverage that exists below.
- **Never invent behaviour.** Unknowns go to Open Questions.
- Prefer acceptance over E2E; justify each E2E in one line.
- Every scenario has an oracle, its environments, and a gate set by its
  dependencies.
- Automate/Manual is a testability judgement, never a ratio.
- Locators are role, label or test id; never CSS/XPath or positional.
- No fixed sleeps. Every async boundary has a poll and a timeout.
- Respect any banned-dependency list in the repo or org instructions, and
  state it in the spec so the dev agent inherits it.
- Credentials are never test data, and env/session files are never committed.
- **No fact about a repo is carried in without checking it.** Record what you
  verified in section 13. If the repo contradicts this skill, follow the repo.
- An open blocking spike is a stop: status `BLOCKED`.
- Keep it reviewable in a few minutes.

## Handoff

Present `QA-SPEC.md` and stop. QA posts it on the ticket; dev reconciles and
answers Open Questions. Tell the user the dev should point the coding agent at
`qa-spec.json` explicitly, not just the ticket ID.
