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
  story. These items never trigger another fix or panel round. Sean decides whether to file them. At night, only the HOUS-590 exception in `night-mode` applies.
- Rounds don't change outcomes: an in-scope HIGH/MEDIUM first raised in round 3 still blocks.
- Round 2+: verify every prior retained finding and review the fix changes. A second cycle keeps all
  prior retained findings and coordinator replies; its first round receives them too.
- With night mode off, max 3 rounds: anything blocking after round 3 goes to Sean; no round 4.
  `night-mode` permits one astra analysis and one second cycle of up to 3 rounds, then parks the PR.
- A finding class that recurs in the same area: switch to a blunter fail-closed design instead of
  patching another case.

## Proportion (coordinator)
Proportion governs review churn: test requests, implausible edge cases and extra machinery. For product,
architecture and UX choices, follow the long-term rule in SKILL.md; do not drop a UX residual just because
leaving it is cheap. Reviewers add real value and also drift toward perfection, the
Codex seat especially: rounds of test changes to prove the implausible can't happen are waste. A story
with hundreds of new tests is doing too much. Push back.
- Test requests that don't name a plan-required rule are LOW; downgrade without debate.
- If a fix would add new machinery (module, check layer, test framework) or more code than the change it
  fixes, you may reject it. Post a PR reply naming the smaller fix or accepting the residual risk.
  Next-round reviewers receive that reply and must answer it, not restate the finding. This does not
  override that UX rule or clear a well-formed HIGH that stays blocking under `night-mode`; choose
  another fix.
- Test budget: past 1.5x the plan's estimated new-test count, stop, cut duplicate and non-plan tests, and
  exceed the budget only with Sean's OK. In `night-mode`, the EC may allow an overrun after those cuts, with a
  logged reason and report entry.
- Trend: track per reviewer the blocking findings raised vs downgraded/rejected for proportion. Past half
  in a story, apply these rules more strictly to that reviewer and tell Sean in your next status. Never
  lower its model or effort.

## Disagreement (coordinator)
- No voting. Any valid blocking finding blocks, whatever the other reviewer said.
- You may downgrade a MEDIUM to LOW only with a one-line reason posted on the PR, and list every
  downgrade in your next status to Sean.
- You may downgrade a HIGH only for a missing format part, never on judgment. Disagreement about a
  well-formed HIGH goes to Sean. In `night-mode`, treat it as valid and fix it; the reviewer, coordinator
  or implementer's disagreement does not clear it. Use the fix/astra path if needed.
- Only Sean downgrades a CRITICAL.
- The implementer may dispute a finding in its report but may not ignore it. You decide per the above.

## CRITICAL halt
With night mode off, these steps halt all work. In `night-mode`, the top coordinator classifies from the
finding's evidence using its full-stop/contained rule; containment is not a downgrade. An SC under an EC
sends the finding to the EC at once, stops its own merges, pushes and new seats until the EC classifies it,
and launches the investigator; the EC relays the result to any other running coordinator. A lone SC is the
top coordinator and classifies the finding itself.

1. The reviewer escalates to its coordinator immediately with the five-part finding, then finishes and
   posts its verdict.
2. Stop all work for a full stop, or only work depending on the broken flow or merging into its repo for a
   contained `night-mode` halt: no new seats, merges or pushes in that scope, across every coordinator;
   running seats stop at their next safe point (no half-written edits). Nothing is closed. An SC waiting
   only on Sean stays open under the scheduling exception in `coordinating-epics`.
3. One exception, in both branches: launch a fresh read-only Claude investigator. It reports the cause (file:line,
   mechanism, since when), impact (data, users, main/staging/prod), and 2-3 fix options with trade-offs,
   size, risk and a recommendation. It does not fix anything.
4. Only the top coordinator (EC, else SC) sends Sean the push notification, at once, then the
   investigation. Other coordinators don't send their own alert. Use `PushNotification`. If unavailable,
   failed or not sent, tell Sean directly that push delivery is unavailable, log it, and keep the halt; do
   not claim a push was sent. Re-notify at each `check --wait` timeout; during a full stop keep a timed
   waiter armed, even with no seat running, until Sean responds. Use the direct channel at those
   checkpoints if push remains unavailable. For a contained `night-mode` finding, notify once and include it
   in the report.
5. Stopped work resumes only when Sean sets direction. Unrelated work may continue only for a contained
   `night-mode` finding. AWS work (secret rotation, deploys) stays Sean's. AWS rules still apply.

## Round checklist (coordinator)
Copy per round:
```
Panel round N (head <sha>):
- [ ] Logs for this head exist (`hous pr logs`, one per job; local `hous ci` only for housetools and
  household-translations), untrimmed, commit matches
- [ ] Both reviewers launched fresh together (opus high + gpt-6.1-sol high/xhigh), with inputs above
- [ ] Both verdicts posted, bound to this head; mutant results in
- [ ] Each finding sorted: CRITICAL halt / blocks / non-blocking / dropped
- [ ] Fix round needed? Include all unresolved retained findings from both reviewers, including non-blocking items
- [ ] Both APPROVE? Collect remaining non-blocking follow-ups; no new round for those items
- [ ] Proportion applied; downgrades and fix rejections posted on the PR
- [ ] Log folder deleted; reviewer terminals closed and their worktrees removed
- [ ] Blocking findings open? -> fresh fix seat (rebase onto `origin/main`, push, green GitHub CI at
  the new head, `hous pr logs`; housetools/translations: full `hous ci` + new logs) -> next round within this cycle (max 3); at the limit follow Outcomes
```
