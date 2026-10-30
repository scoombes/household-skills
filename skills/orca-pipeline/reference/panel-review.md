# PR panel review

Rules for each PR panel reviewer. The coordinator's side (outcomes, proportion, disagreement, CRITICAL
halt, round checklist) is in [panel-coordinator.md](panel-coordinator.md); reviewers don't need it.

## Contents
- Inputs
- Reviewer may / may not
- Evidence: logs
- Scope
- Severity
- Finding format

## Inputs
The coordinator gives each reviewer:
- PR number, head sha, approved `plan.md`, and the log folder for that head.
- Round 2+: every prior retained finding from BOTH reviewers, including non-blocking items assigned
  to the fix round, plus any coordinator PR replies that rejected a fix.
- This opening line: "Small household app; one developer maintains every test and safeguard forever.
  Find real problems in this change, rate them honestly, propose the smallest fix. The coordinator
  decides what is worth doing."

Reviewers do not dig through PR history for any of this.

## Reviewer may
- Read code, PR, plan and logs.
- Post exactly one verdict comment per round. First line exactly:
  `Panel review (<Claude|Codex> seat) @ <40-hex head sha>: APPROVE|CHANGES_NEEDED`
  (`hous pr merge` counts only verdicts bound to the current head).
- Run up to 5 mutants per round, only via `hous sandbox --ephemeral -- <cmd>`, each against the single
  test file that should catch it. Each mutant must differ from every mutation proof the implementer
  listed; name the closest one. If `hous help sandbox` does not list `--ephemeral`, run no mutants.

## Reviewer may not
- Write anything else: no commits, pushes, file edits, PR edits or labels.
- Run any test the logs cover. Never run `hous ci` or a full suite.
- Read the other reviewer's verdict for the current head before posting your own.
- Judge the plan. It went through its own review rounds. Approach, design, and asks beyond the plan's
  acceptance criteria are neither findings nor escalations.

## Evidence: logs
1. Repos with a PR workflow: the implementer and fix seats save the GitHub CI run's logs for the head with
   `hous pr logs` into `~/household/agent-work/<TICKET>/logs/<40-hex sha>/`, one file per job, each starting
   with `command:`, `commit:`, `exit code:` (the job's GitHub conclusion), `start:`/`end:` and the run and job
   URLs. They post one line per job on the PR: sha, job, conclusion, test count.
2. Repos with no PR workflow (housetools, household-translations): the implementer and fix seats run a full
   `hous ci` (every suite the repo documents only when it has no CI targets) on their final commit (every
   round) and save the complete raw logs, untrimmed, to
   `~/household/agent-work/<TICKET>/logs/<40-hex sha>/<suite>.log`, a fixed path every seat can reach (seats'
   own scratchpads are private). Each log starts with the command, sha, exit code (the command's exit code)
   and start/end time. They post one line per suite on the PR: sha, test count, pass/fail, exit code.
3. Reviewers treat the logs as proof. Common to both kinds: a log whose `commit:` differs from the head, or a
   log without the runner's summary where the job or suite runs a test runner, = CHANGES_NEEDED. The success
   rule differs by kind, because the header line means a different thing in each:
   - GitHub job log (`command: GitHub Actions job ...`): `exit code:` must be the conclusion `success`.
   - Local `hous ci` log (`command: hous ci ...` or the suite command): `exit code:` must be `0`.

   3a. A job of the deciding run with no log file at all = CHANGES_NEEDED.
   3b. A header-only file with `exit code: skipped` and `source: none (job skipped)` is acceptable only for a
   job the workflow skips by design (a path filter or matrix condition), and only when the deciding run passes
   the CI gate, that is at least one job of the run concluded `success` (`hous pr status` shows the run as
   passing; `checks.ts:179-182`). A run whose every job is skipped is a draft-era result and never evidence.
   No workflow changed by HOUS-685 has such a mixed set today (api `CI`, infra `checks`, ios `CI` have one job
   each; both `costs` matrix jobs run), so 3b describes future workflows, not a current case.

   Do not run tests to fill the gap. The PR summary lines are a record, never evidence.

## Scope
- In scope: the changed lines and anything they break (callers, weakened tests). A bad pattern copied
  into new code is in scope; the original is not.
- Out of scope: code that existed before the PR. It was already reviewed and accepted; it never blocks.

## Severity
Every finding except CRITICAL needs a realistic accidental trigger. A trigger that needs deliberate
construction (adversarial input, contrived layout, crafted race) makes it LOW at most.
- **CRITICAL**: (a) out-of-scope code on main that breaks the app: security hole, data loss/corruption,
  or a broken core flow; deliberate-exploit paths count here. Or (b) any secret (key, token, password)
  anywhere in the PR's commits.
- **HIGH**: data loss/corruption, cross-household or secret exposure, broken core flow, or a
  deploy/migration that fails or can't roll back.
- **MEDIUM**: wrong result in a less common case; wrong error handling (swallowed, wrong status, no
  rollback); a plan-required rule with no test, or a test a mutant survives. A request for more tests is
  MEDIUM only if it names such a plan-required rule.
- **LOW**: deliberate-construction only, no current trigger, or a weak test for a rule the plan doesn't
  require.
- **NIT**: naming, style, comments, structure.

## Finding format
- CRITICAL/HIGH/MEDIUM need all five parts: file:line; trigger; wrong result vs what the plan says; proof
  (log line, mutant result, or traced code path); smallest fix in 1-2 lines, not a redesign. A missing
  part makes it LOW.
- LOW/NIT: file:line plus one line. Max 10 per review, ranked. CRITICAL/HIGH/MEDIUM are never capped.
- Out-of-scope findings go in a separate `Out of scope` section of the verdict.
- CRITICAL: escalate to the coordinator immediately with the five-part finding, then finish and post the
  verdict.

Example blocking finding:
```
MEDIUM src/import/meals.ts:142
Trigger: an import file whose meal has an empty ingredients array (the mobile app sends this when the user deletes the last ingredient).
Wrong result: the meal is saved with ingredientCount=1; plan section 3 says empty meals are rejected with 422.
Proof: traced code path: parseMeal() defaults [] to [placeholder] at line 138 before validate() runs.
Fix: validate before defaulting (swap lines 138 and 142).
```
