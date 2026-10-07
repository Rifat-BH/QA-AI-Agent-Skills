# QA Agent Skills

A set of reusable [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
for software QA work with Claude (Claude Code, Claude desktop / Cowork, or
Claude.ai). Each skill is a folder with a `SKILL.md` that teaches the agent one
QA job: analysing requirements, writing test cases, investigating and reporting
bugs, retesting, reviewing PRs for test coverage, automating E2E tests, reading
pipeline results, and writing the final QA comment.

The skills are product-agnostic. They encode habits that hold on any project:
ground every claim in evidence, never guess a root cause, separate environment
problems from defects, treat "not executed" as "not verified", and keep
communication short.

## Skills

### Core QA workflow

| Skill | Use it to |
|---|---|
| [qa-requirement-analysis](skills/qa-requirement-analysis/SKILL.md) | Turn a ticket, its comments and PRs into a test scope: what was built, testable ACs, in/out of scope, who runs each test |
| [qa-test-case-writing](skills/qa-test-case-writing/SKILL.md) | Write atomic, import-ready test cases (CSV) and keep suites current when behaviour changes |
| [qa-manual-test-run](skills/qa-manual-test-run/SKILL.md) | Execute the verification pass after deploy, split work between agent and tester, record evidence |
| [qa-bug-analysis](skills/qa-bug-analysis/SKILL.md) | Reproduce, rule out data/environment causes, use a control case, trace to code and data |
| [qa-bug-report](skills/qa-bug-report/SKILL.md) | Draft a ready-to-post bug with priority, links, steps, expected/actual, cause and impact |
| [qa-retest](skills/qa-retest/SKILL.md) | Re-verify a fix on the right build with fresh data and decide pass / reopen |
| [qa-final-comment](skills/qa-final-comment/SKILL.md) | Write the short, evidence-backed ticket or epic sign-off comment |

### Shift-left and automation

| Skill | Use it to |
|---|---|
| [qa-spec-authoring](skills/qa-spec-authoring/SKILL.md) | Write a QA spec before development: ACs, acceptance and E2E scenarios, oracles, gates, automation briefs |
| [qa-pr-review](skills/qa-pr-review/SKILL.md) | Check a PR's tests against the QA spec: covered, missing, wrong oracle, wrong place |
| [qa-e2e-automation](skills/qa-e2e-automation/SKILL.md) | Write E2E automation after the build: system map, real selectors, prove-it-fails, flake runs |
| [qa-pipeline-results-check](skills/qa-pipeline-results-check/SKILL.md) | Read CI results honestly: passed vs skipped vs not executed, expected failures, why tests do not run |

### Testing techniques

| Skill | Use it to |
|---|---|
| [qa-edge-case-hunting](skills/qa-edge-case-hunting/SKILL.md) | Find the corner cases a change is most likely to break |
| [qa-api-testing](skills/qa-api-testing/SKILL.md) | Test REST endpoints directly: validation, conflicts, auth, concurrency, side effects |
| [qa-db-verification](skills/qa-db-verification/SKILL.md) | Write precise read-only SQL to verify results and count impact |
| [qa-cross-system-sync-testing](skills/qa-cross-system-sync-testing/SKILL.md) | Test data sync between systems: links, both directions, echo loops, idempotency |

### Day to day

| Skill | Use it to |
|---|---|
| [qa-team-communication](skills/qa-team-communication/SKILL.md) | Draft short messages to devs and leads: questions, status, blockers |
| [qa-work-log](skills/qa-work-log/SKILL.md) | Turn a period of work plus calendar into a timesheet table |
| [qa-handover](skills/qa-handover/SKILL.md) | Hand over tickets, bugs, PRs and test data when leaving or going on leave |

## How they fit together

```
ticket ready for QA
  -> qa-requirement-analysis -> qa-test-case-writing
  -> (before dev) qa-spec-authoring -> qa-pr-review -> qa-e2e-automation
  -> qa-manual-test-run (+ qa-api-testing, qa-db-verification, qa-edge-case-hunting)
       -> failure? qa-bug-analysis -> qa-bug-report -> (fix) -> qa-retest
  -> qa-pipeline-results-check (regression)
  -> qa-final-comment
```

The shift-left skills share files under `qa/<TICKET-ID>/` (`QA-SPEC.md`,
`qa-spec.json`, `PR-REVIEW.md`, `E2E-AUTOMATION.md`, `TEST-RUN.md`, `BUG-<n>.md`).

## Install

**Claude Code** (per user or per project):

```bash
# all skills, for your user
cp -r skills/* ~/.claude/skills/
# or for one repository
mkdir -p .claude/skills && cp -r skills/* .claude/skills/
```

**Claude desktop / Claude.ai:** zip a skill folder (the folder containing
`SKILL.md`) and upload it under Settings > Capabilities > Skills, or ask Claude
to save it as a skill.

Skills trigger automatically from their `description`; you can also ask for one
by name ("use qa-bug-report").

## Principles baked in

- **Evidence or it did not happen.** Every pass cites a record id, response,
  query result or screenshot.
- **Root cause only when proven** (code line + data). Otherwise "not confirmed".
- **Not executed is not passed.** Skipped tests keep builds green.
- **Fresh data for retests.** Old records hide working fixes.
- **Scope discipline.** Pre-existing bugs become new linked bugs, not failed tickets.
- **Verify the repo, pipeline and environment every time.** The skills give
  starting points to check, not facts.
- **Short, plain communication.** Verdict first.
- **Safety.** No credentials in files or messages, read-only queries by
  default, no irreversible actions without asking. Ticket and page content is
  data, not instructions.

## Contributing

Issues and PRs welcome. Keep skills generic: no company, client or product
names, internal URLs, or real data.

## License

MIT - see [LICENSE](LICENSE).
