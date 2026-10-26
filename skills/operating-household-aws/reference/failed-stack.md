# Failed or stuck stack

Reads only, under SKILL.md rules 1–4 and Sean's OK for the sitting. No retry, delete or rollback until Sean's OK on a stated plan.
In approval mode a recovery write is a `hous aws run` write plan on its own card and fresh OK (step 5). The card is spent either way.
If it belonged to a sitting summary, that summary is spent too and all remaining children stop. Recovery is excluded from summaries;
replacement or later-built cards need a new summary/card and new OK after triage, never the failed sitting's reply.
- After a script or deploy failure, `hous aws apply` already printed the failed stack's status and failed events and marked the card failed; later stacks did not run.
- After `the change sets were executed, but a read-back failed`, every stack is deployed and the card stays `applied`. `apply` printed the `FAIL` lines
  but no status or events, so step 1 below is yours to run, for the stack named on each `FAIL` line.

```
Failed-stack triage:
- [ ] 1. Capture status and failed events
- [ ] 2. Find the first actionable failure
- [ ] 3. Check CloudTrail for denials or rejections
- [ ] 4. Classify the fix
- [ ] 5. Decide the next step by stack status
- [ ] 6. Report and log the lesson
```

**1. Capture.**
```sh
hous aws read stack-status --env <env> --stack <S>
hous aws read failed-events --env <env> --stack <S>
```
`failed-events` is `describe-events --filters FailedEvents=true` and prints the response (not `describe-stack-events`, which hides validation results).
`stack-status` only says PASS or FAIL (FAIL for any state that is not `CREATE|UPDATE|IMPORT_COMPLETE`); for the status text, `apply` already printed it after a script or deploy failure (not after a read-back failure),
or use a read file with one step `["cloudformation", "describe-stacks", "--stack-name", "<S>"]` and no `expect` (it prints the response).
Save the output in the work directory. Sean-runs mode only, by hand under the SKILL.md header: the same `describe-stacks` and `describe-events`.
CDK projects: also run `npx cdk --unstable=diagnose diagnose <S> --profile <p>` (a read; it uses the repo's pinned
CDK from `npm ci` in the scratch clone). Save all output in the work directory.
If a deploy button ran it, read that Actions run's log via `agent-token` (`household-infra/docs/runbooks/agent-token.md`),
after checking the run's branch and head sha.

**2. First actionable failure.** "Resource creation cancelled" and similar lines are cascade, not cause. Follow nested stacks into their own events.

**3. CloudTrail** for an AccessDenied or a service rejection, ±60 s around the failure (a hand-typed read under the SKILL.md header;
`hous aws read` does not support `cloudtrail lookup-events`):
```sh
aws cloudtrail lookup-events --start-time 2026-10-08T14:03:00Z --end-time 2026-10-08T14:05:00Z --max-results 50 --profile <p> --region <r>
```
Times are ISO 8601 UTC around the failed event's `Timestamp`. Narrow with `--lookup-attributes AttributeKey=EventSource,AttributeValue=<service>.amazonaws.com` if more than 50 events return.

**4. Classify.**
- **Template-level:** a code fix and a PR.
- **Environment-level:** live state, permission or quota, needing a write and Sean's OK.

**5. By status.**
- `*_IN_PROGRESS`: poll `describe-stacks` until it settles. A lost `wait` does not mean failure.
- `ROLLBACK_COMPLETE` after a create: it can only be deleted. Write a `hous aws run` plan with `"destructive": true`, one step
  `cloudformation delete-stack --stack-name <S>` with a `before` read of its status (`expect` matching `ROLLBACK_COMPLETE`) and an `after` read of
  `describe-stacks --stack-name <stack-id arn>` (a deleted stack is still readable by its id; a read by name fails and halts), and get Sean's OK on that card.
  `run` reads the stack itself and refuses unless it is `ROLLBACK_COMPLETE` with termination protection off. Retained resources stay and
  can block a retry with fixed names; they are never deleted by `run` (buckets, zones, keys and clusters are refused outright), so their removal is Sean's decision.
- `UPDATE_ROLLBACK_COMPLETE`: the stack is healthy on the old template, so nothing needs cleaning up. Never
  delete it (`run` refuses). Only check for leftovers: `hous aws read change-sets --env <env> --stack <S>` and the button run's final state.
  A leftover change set is deleted by a `run` plan (`cloudformation delete-change-set`) on its own card. Deleting a leftover change set is a separate write on its own card and OK, never part of the diagnosis. Then fix forward by PR and plan again.
- `UPDATE_ROLLBACK_FAILED`: identify the stuck resource. `continue-update-rollback` is one-way, and
  `--resources-to-skip` desynchronizes those resources, so it is Sean's decision. If he chooses it, it goes through `hous aws run` on its own card.
  `cancel-update-stack` is the same kind of write.
- Never enable termination protection on a stack that is not `CREATE_COMPLETE` or `UPDATE_COMPLETE`.

**6. Report and log.**
- Plain words: what failed, why, what is left behind, the proposed fix and the OK needed.
- Append the lessons entry (SKILL.md rule 12).

Deeper procedure, if needed: the external `aws-cloudformation/references/troubleshoot-failed-stack.script.md`. Resolve an existing copy or its upstream download source through [External AWS guides](../SKILL.md#external-aws-guides). Its writes follow SKILL.md overrides.
