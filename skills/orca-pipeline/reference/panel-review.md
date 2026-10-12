# PR panel review

Rules for the two-seat PR panel. Coordinators follow all of it. Reviewer seats are pointed at this file
and follow "Inputs" through "Finding format".

## Contents
- Inputs
- Reviewer may / may not
- Evidence: logs
- Scope
- Severity
- Finding format
- Outcomes
- Proportion (coordinator)
- Disagreement (coordinator)
- CRITICAL halt
- Round checklist (coordinator)

## Inputs
The coordinator gives each reviewer:
- PR number, head sha, approved `plan.md`, and the log folder for that head.
- Round 2+: every prior CRITICAL/HIGH/MEDIUM from BOTH reviewers, plus any coordinator PR replies that
  rejected a fix.
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
- Implementer and fix seats run a full `hous ci` (the repo's own workflows via act, so it matches GitHub
  CI; every suite the repo documents only when it has no CI targets) on their final commit (every round)
  and save the complete
  raw logs, untrimmed, to `~/household/agent-work/<TICKET>/logs/<40-hex sha>/<suite>.log`, a fixed path
  every seat can reach (seats' own scratchpads are private). Each log starts with the command, sha, exit
  code and start/end time. They post one line per suite on the PR: sha, test count, pass/fail, exit code.
- Reviewers treat the logs as proof. Log sha differs from the head, or the log lacks the runner's
  summary = CHANGES_NEEDED. Do not run tests to fill the gap.
- The PR summary lines are a record, never evidence.

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
- CRITICAL: escalate to the coordinator immediately (see CRITICAL halt), then finish and post the verdict.

Example blocking finding:
```
MEDIUM src/import/meals.ts:142
Trigger: an import file whose meal has an empty ingredients array (the mobile app sends this when the user deletes the last ingredient).
Wrong result: the meal is saved with ingredientCount=1; plan section 3 says empty meals are rejected with 422.
Proof: traced code path: parseMeal() defaults [] to [placeholder] at line 138 before validate() runs.
Fix: validate before defaulting (swap lines 138 and 142).
```

## Outcomes

| Finding | In scope | Out of scope |
|---|---|---|
| CRITICAL | CRITICAL halt | CRITICAL halt |
| HIGH / MEDIUM | Blocks merge | Never blocks. Coordinator dedupes and sends to Sean's queue with story and PR comment link; Sean decides whether to file it |
| LOW / NIT | Coordinator's follow-up list (new code Sean may revisit or discard); never blocks | Dropped (old code is noise at this level) |

- Rounds don't change outcomes: a HIGH/MEDIUM first raised in round 3 still blocks.
- Round 2+: verify every prior blocking finding and review the fix changes.
- Max 3 rounds. Anything blocking after round 3 goes to Sean; no round 4.
- A finding class that recurs in the same area: switch to a blunter fail-closed design instead of
  patching another case.

## Proportion (coordinator)
Good enough is the target, not perfect. Reviewers add real value and also drift toward perfection, the
Codex seat especially: rounds of test changes to prove the implausible can't happen are waste. A story
with hundreds of new tests is doing too much. Push back.
- Test requests that don't name a plan-required rule are LOW; downgrade without debate.
- If a fix would add new machinery (module, check layer, test framework) or more code than the change it
  fixes, you may reject it. Post a PR reply naming the smaller fix or accepting the residual risk.
  Next-round reviewers receive that reply and must answer it, not restate the finding.
- Test budget: past 1.5x the plan's estimated new-test count, stop, cut duplicate and non-plan tests, and
  exceed the budget only with Sean's OK.
- Trend: track per reviewer the blocking findings raised vs downgraded/rejected for proportion. Past half
  in a story, apply these rules more strictly to that reviewer and tell Sean in your next status. Never
  lower its model or effort.

## Disagreement (coordinator)
- No voting. Any valid blocking finding blocks, whatever the other reviewer said.
- You may downgrade a MEDIUM to LOW only with a one-line reason posted on the PR, and list every
  downgrade in your next status to Sean.
- You may downgrade a HIGH only for a missing format part, never on judgment. Disagreement about a
  well-formed HIGH goes to Sean.
- Only Sean downgrades a CRITICAL.
- The implementer may dispute a finding in its report but may not ignore it. You decide per the above.

## CRITICAL halt
1. The reviewer escalates to its coordinator immediately with the five-part finding, then finishes and
   posts its verdict.
2. All work stops, across every coordinator: no new seats, merges or pushes. Running seats stop at their
   next safe point (no half-written edits). Nothing is closed.
3. One exception: launch a fresh read-only Claude investigator. It reports the cause (file:line,
   mechanism, since when), impact (data, users, main/staging/prod), and 2-3 fix options with trade-offs,
   size, risk and a recommendation. It does not fix anything.
4. Only the top coordinator (EC, else SC) sends Sean the push notification, at once, then the
   investigation, and re-notifies until he responds. Other coordinators don't send their own alert.
5. Nothing resumes until Sean sets the direction. AWS work (secret rotation, deploys) stays Sean's.

## Round checklist (coordinator)
Copy per round:
```
Panel round N (head <sha>):
- [ ] Logs for this head exist, untrimmed, sha matches
- [ ] Both reviewers launched fresh together (opus high + gpt-6.1-sol high/xhigh), with inputs above
- [ ] Both verdicts posted, bound to this head; mutant results in
- [ ] Each finding sorted: CRITICAL halt / blocks / Sean's queue / follow-up / dropped
- [ ] Proportion applied; downgrades and fix rejections posted on the PR
- [ ] Log folder deleted; reviewer terminals closed and their worktrees removed
- [ ] Blocking findings open? -> fresh fix seat (rebase onto `origin/main`, full `hous ci` + new logs) -> round N+1 (max 3)
```
