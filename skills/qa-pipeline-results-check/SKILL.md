---
name: qa-pipeline-results-check
description: "Reads CI/CD test results honestly: finds the right pipeline and run (branch, trigger, commit), separates passed, failed, flaky, skipped and not-executed tests, checks whether stages were disabled, filtered out or unable to reach their dependencies, explains why a test is not running, and decides whether a regression run is acceptable for sign-off. Use when the user asks \"how do I run the regression check\", \"is the regression green\", \"why are these tests skipped\", \"why did this pass here but fail there\", or shares a pipeline run or log."
---

# QA Pipeline Results Check

A green pipeline is not the same as verified behaviour. Read what actually ran.

## Step 1 - Find the right run

- Which pipeline holds the tests (read the YAML: trigger, schedule, branch,
  working directory, environment variables).
- Scheduled suites run on a cron against a branch (often `main`). A ticket is
  covered only by runs **after** its merge on that branch.
- Note the run's **branch, trigger (manual/scheduled/PR), commit and date**. A
  manual run on a feature branch is not the main-branch regression.

## Step 2 - Classify every relevant test

| State | Counts as verified? |
|---|---|
| Passed | Yes |
| Failed | No; analyse |
| Flaky (passed on retry) | Only if the failure is unrelated timing; say so |
| Skipped / Not executed | **No** |
| Missing (test does not exist) | **No** |

Check the test-results API or the Tests tab, not only the job colour.

## Step 3 - Explain non-executed tests

Common causes, in order of likelihood:

1. Runtime `skip` because a config/env variable is empty in that environment's
   config file (the build stays green).
2. Stage `condition: false` or change-detection did not select the stage.
3. Pipeline never registered (a YAML file nobody connected).
4. Agent pool cannot reach a dependency (hosted pools reach only public
   endpoints; private hosts need a pool inside the network). Probe DNS/HTTPS
   from the agent to get the real error.
5. Test filter (`grep`, tags, project) excludes it.

Name the cause with evidence (the YAML line, the config line, the probe output).

## Step 4 - Explain failures

- Compare against the change: does the failing test assert the **old**
  behaviour that the ticket intentionally changed? Then it is an expected
  failure; the test needs updating or removal. Check whether a branch/PR already
  does that, and whether removing it leaves a coverage hole.
- Timing/UI flake (element not ready, overlapping elements) that passes on
  rerun: unrelated, note it.
- Real failure: hand to `qa-bug-analysis`.
- Environment failure (host down, expired credentials): not a product bug.

## Step 5 - Compare two runs

When a test passes in one run and fails in another, diff: branch, commit, test
count, environment, agent pool, config. The difference in test count often
explains it (a test was removed or added).

## Output

```markdown
**Regression: <OK for TICKET | Not OK>**

| Run | Branch | Tests | Result |
| <id> | main | 67 (64 pass, 2 flaky, 1 fail) | Failed |

Failure: <test>: <expected because ... / real defect / environment>
Not executed: <tests + cause>
Line for the ticket comment: <one line>
```

If the user asks how to run it: give the pipeline link pattern, which run to
open, which tab, which tests matter, and how to trigger a manual run (branch,
expected duration).
