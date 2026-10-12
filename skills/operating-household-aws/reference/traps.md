# Traps from real HouseHold runs

Number = trap number in the old guide's §(b) (`household-infra/docs/runbooks/agent-aws-execution.md`, history) and `aws-lessons-log.md`, where the evidence is.

1. Raw `Cfn*` resources created in parallel can be rejected (CloudTrail vs bucket policy). Add `DependsOn`, or use L2 constructs.
2. AWS CLI v2 errors start with `aws: [ERROR]:`. Parsers must match that.
3. `aws sso login` fails from a deleted working directory. `cd` to a real directory, or use `--use-device-code`.
4. `aws sso login` selects no profile. Pass `--profile` and `--region` on every command.
5. Retained, fixed-name resources from a failed create block the retry. Their removal needs its own OK.
6. CloudTrail's first delivery lags. Use a bounded poll.
7. Enable termination protection only on `CREATE_COMPLETE` or `UPDATE_COMPLETE`.
8. Login setup: one `sso-session` named `household`; profiles `household-management|staging|production` share it.
   Check with `command -v aws; aws --version` first. Two CLI installs exist, and PATH order picks one.
9. `cdk bootstrap` runs from an empty temp directory, with the pinned CLI, checked against a `--show-template` digest.
10. Live synth is scoped to the stacks being deployed (`HOUSEHOLD_LIVE_SCOPE`).
11. Actions logs: read via `agent-token` only.
12. Check a dispatched run's branch and head sha before counting it.
13. S3 can show the old state for about a second after a create or delete. Retry before concluding.
14. A retry loop also retries when the read itself errors.
15. A lost `wait` result does not mean failure. Re-read the stack status.
16. GoDaddy nameserver change: confirm the second "Continue & Verify" step. A reload discards the change.
17. Never write into Sean's checkout, even to remove a stray file.
18. Sean-runs mode (packages READY before the switch): a package edited after READY needs a fresh READY before it runs.
19. zsh does not word-split `$list`. Use arrays or `${=list}`.
20. A hand-written fake agrees with a wrong script. Check key names against real CLI shapes (housetools `tools/aws/shapes/`; the old `household-infra/tools/package-fixtures` is frozen).
21. Zips built on different machines differ byte for byte. Pin CI's artifact, never a local build.
22. A long command can hit the tool time cap. Use resumable polling (`hous pkg run --detach` + `hous pkg wait` for a READY package).
23. ACM validation: poll `Certificate.Status` until `ISSUED`. The CLI waiter can time out first.
24. A package's assumed base template can be stale (a staging deploy was skipped). Read the live template first (checks.md step 3).
25. An HTTP API answers 404 where a REST API answers 403. Check the API type before asserting a status code.
26. A schema refresh with the worker's queue feed on dead-letters its tick messages (HOUS-618, 2026-10-10). End the sheet with the DLQ depth read and a decision.
