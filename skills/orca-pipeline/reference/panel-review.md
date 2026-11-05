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
- PR number, head sha, approved `plan.md` (with its risk table and counts), the log folder for that head,
  and any historical-proof table (rows marked reused or rerun).
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
- Run bounded novel probes, at most 5 mutants per round. The five is a ceiling, not a quota: a probe is
  for a plausible fault the logs and the implementer's proof do not already cover, never for repeating
  covered proof. Aim at most 5 minutes per probe; a larger or expanding run goes to the
  coordinator for a bounded decision. Prefer the lowest faithful layer. A UI/live mutant needs a named
  uncovered fault and the cheapest selector. For iOS, use `hous test --ephemeral --mutants <folder>` (a
  disposable host copy, not an isolated sandbox). For household-api, -infra, -web, -translations and housetools,
  use `hous test --ephemeral --mutants <list.json> --out <new folder>` (one disposable copy, worktree untouched)
  after `hous test --mutants <list.json> --check`; keep the list and the folder outside the checkout. A one-off
  command a list can't express, such as per-mutant environment variables, uses `hous sandbox --ephemeral -- <cmd>`.
  Target only the test file that should catch each mutant (for iOS, the patch's `Tests: -only-testing:<…>` line;
  for a list, the entry's `tests`). Each mutant must differ from every mutation proof or control the implementer
  listed; name the closest one. If `hous help test` does not list this repo's form (`--mutants <folder>` for iOS;
  otherwise `--mutants <list.json>`, naming this repo), run no mutants.

## Reviewer may not
- Write anything else: no commits, pushes, file edits, PR edits or labels.
- Run any test the logs cover. Never run `hous ci` or a full suite.
- Read the other reviewer's verdict for the current head before posting your own.
- Judge the plan's design. It went through its own review rounds. Approach, design, and asks beyond the
  plan's acceptance criteria are neither findings nor escalations. Do enforce those criteria: the
  risk table's actual caller, selector and UI/live fidelity requirements and the justification for each new
  or extended UI/live scenario. That is not permission to redesign an approved plan.

## Evidence: logs
1. Repos with a PR workflow (household-api, -infra, -ios, -web): the implementer and fix seats save the GitHub CI run's logs for the head with
   `hous pr logs` into `~/household/agent-work/<TICKET>/logs/<40-hex sha>/`, one file per job, each starting
   with `command:`, `commit:`, `exit code:` (the job's GitHub conclusion), `start:`/`end:`, the run and job
   URLs and `source:`. They post one line per job on the PR: sha, job, conclusion, test count.
2. Repos with no PR workflow: the implementer and fix seats run the repo's full local gate (translations
   `hous ci --web <checkout> --ios <checkout>`; site `hous ci`; housetools the host `hous test` full suite plus
   the documented full specialized suites that apply, as housetools has no `hous ci` capability) on their
   final commit (every round) and save the complete raw logs, untrimmed, to
   `~/household/agent-work/<TICKET>/logs/<40-hex sha>/<suite>.log`, a fixed path every seat can reach (seats'
   own scratchpads are private). Each log starts with the command, sha, exit code (the command's exit code)
   and start/end time. They post one line per suite on the PR: sha, test count, pass/fail, exit code.
3. Reviewers treat the logs as proof. Common to both kinds: a log whose `commit:` differs from the head, or a
   log without the runner's summary where the job or suite runs a test runner, = CHANGES_NEEDED. The success
   rule differs by kind, because the header line means a different thing in each:
   - GitHub job log (`command: GitHub Actions job ...`): `exit code:` must be the conclusion `success`.
   - Local gate log (`command: hous ci ...`, `hous test ...` or the suite command): `exit code:` must be
     `0`.

   3a. A job of the deciding run with no log file at all = CHANGES_NEEDED.

   3b. A header-only file with `exit code: skipped` and `source: none (job skipped)` is acceptable only for a
   job the workflow skips by design (a path filter or matrix condition), and only when the deciding run passes
   the CI gate, that is at least one job of the run concluded `success` (`hous pr status` shows the run as
   passing; the successful-run branch of `checks.ts`). A run whose every job is skipped is a draft-era result
   and never evidence.
   No workflow changed by HOUS-685 has such a mixed set today (api `CI`, infra `checks`, ios `CI` have one job
   each; both `costs` matrix jobs run), so 3b describes future workflows, not a current case.

   Do not run tests to fill the gap. The PR summary lines are a record, never evidence.

4. Full-head evidence and historical targeted evidence are different. A full gate never carries from an old
   head. A reused targeted result counts only with the identity, intervening-change and
   invalidation record in [repo-testing.md](repo-testing.md) (Historical targeted proof), and is never
   treated as freshly executed; a relevant change or uncertainty means ask for a rerun. A required
   selector skipped, absent or failed in setup is incomplete evidence, not the intended red. Evidence
   that holds secrets or private state is incomplete, never a reason to retain or quote them.

## Scope
- In scope: the changed lines and anything they break (callers, weakened tests). A bad pattern copied
  into new code is in scope; the original is not.
- Out of scope: code that existed before the PR. It was already reviewed and accepted; it never blocks.

## Severity
Every finding except CRITICAL needs a realistic accidental trigger. A trigger that needs deliberate
construction (adversarial input, contrived layout, crafted race) makes it LOW at most. Legitimate
hostile input at an exposed auth, parser or identity boundary is graded by its real consequence, not LOW
merely because it is deliberate; ordinary speculative or nonconsequential cases stay bounded.
- **CRITICAL**: (a) out-of-scope code on main that breaks the app: security hole, data loss/corruption,
  or a broken core flow; deliberate-exploit paths count here. Or (b) any secret (key, token, password)
  anywhere in the PR's commits.
- **HIGH**: data loss/corruption, cross-household or secret exposure, broken core flow, or a
  deploy/migration that fails or can't roll back.
- **MEDIUM**: wrong result in a less common case; wrong error handling (swallowed, wrong status, no
  rollback); a plan-required rule with no test, or a test a relevant mutant survives. A survivor is
  diagnostic until it was activated, reached and relevant, and demonstrably exposes a gap in a
  plan-required or protected rule; equivalent, setup-failed or unrelated survivors create no blocking
  test request. A request for more tests is MEDIUM only if it names such a plan-required rule (or, under the
  test request format below, a proof gap in a changed protected rule). A traced wrong result is a defect
  finding graded by its consequence under the rules above.
- **LOW**: deliberate-construction only, no current trigger, or a weak test for a rule the plan doesn't
  require.
- **NIT**: naming, style, comments, structure.

## Finding format
- CRITICAL/HIGH/MEDIUM need all five parts: file:line; trigger; wrong result vs what the plan says; proof
  (log line, mutant result, or traced code path); smallest fix in 1-2 lines, not a redesign. A missing
  part makes it LOW.
- **Test request format:** name the consequential wrong behavior or the plan requirement,
  the actual production path, the gap in existing proof and the cheapest faithful test. Meeting the format
  does not make a request blocking. A request with no plan-required rule stays LOW unless it identifies a
  proof gap for a changed protected rule (auth, tenant, privacy, identity, wrong-database,
  destructive-operation, rollback, data, draft, pending-write).
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
