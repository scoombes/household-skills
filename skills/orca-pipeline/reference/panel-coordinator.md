# PR panel: coordinator rules

The coordinator's half of the two-seat PR panel. Read [panel-review.md](panel-review.md) too: it holds the
reviewer inputs, evidence, scope, severity and finding format these rules act on.

## Contents
- Outcomes
- Proportion
- Disagreement
- CRITICAL halt
- Round checklist

## Outcomes

| Finding | In scope | Out of scope |
|---|---|---|
| CRITICAL | CRITICAL halt | CRITICAL halt |
| HIGH / MEDIUM | Blocks merge | Non-blocking; include in a required fix round, otherwise collect only after both reviewers APPROVE |
| LOW / NIT | Non-blocking; include in a required fix round, otherwise collect only after both reviewers APPROVE | Dropped (old code is noise at this level) |

- **Fix round:** when the panel requires changes and a fix round is needed, give the fix seat every
  unresolved, retained finding from both reviewers, including non-blocking items. Fix them together in
  that round; do not send the non-blocking items to the EC as follow-ups while fixes are underway.
- **Both APPROVE:** collect any remaining non-blocking items as follow-ups, deduped with the source
  story and PR comment link. Hand them to the EC at story completion, or to Sean for a standalone
  story. These items never trigger another fix or panel round. Sean decides whether to file them.
- Rounds don't change outcomes: an in-scope HIGH/MEDIUM first raised in round 3 still blocks.
- Round 2+: verify every prior retained finding and review the fix changes.
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
- [ ] Each finding sorted: CRITICAL halt / blocks / non-blocking / dropped
- [ ] Fix round needed? Include all unresolved retained findings from both reviewers, including non-blocking items
- [ ] Both APPROVE? Collect remaining non-blocking follow-ups; no new round for those items
- [ ] Proportion applied; downgrades and fix rejections posted on the PR
- [ ] Log folder deleted; reviewer terminals closed and their worktrees removed
- [ ] Blocking findings open? -> fresh fix seat (rebase onto `origin/main`, full `hous ci` + new logs) -> round N+1 (max 3)
```
