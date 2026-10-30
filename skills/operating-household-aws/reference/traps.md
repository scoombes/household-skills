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
26. A schema refresh with the worker's queue feed on dead-letters its tick messages (HOUS-618, 2026-10-10). End the sheet with the DLQ depth read and a decision. Disable the feed (the event source mapping), not the worker's concurrency: with concurrency 0 throttled batches return to the queue and their receive counts still climb, but a disabled feed receives nothing, so nothing dead-letters. While it is off, the tick-missing alarm (about 15 minutes), the oldest-message alarm and the throttle alarms email the alerts address; that is expected, and the run sheet says so before Sean approves.
27. Schema-chain cards: after a live DDL step, the next entry's practice preflight can refuse on a short-lived "pending household DDL job" and spend the card (HOUS-573, 2026-10-20). Put each live step on its own card, or run a `--verify-only` read showing `pending-household-jobs: none` before the next card.
28. Dead-letter reads in a run sheet: record the known depth before the sitting and pair the depth read with the oldest-message age (`ApproximateAgeOfOldestMessage`), so known old messages pass and only a new one stops (HOUS-573, 2026-10-20).
29. Web or app rows that need a sign-in: the run sheet names, for each such row, who types it, at which step and from where (Sean in the browser or app at step N using his tester list, or a protected in-process route; never argv), and Sean's directions name the Orca workspace and tab (HOUS-656, 2026-10-23).
30. Run-sheet prep: run `hous aws check` on every read the run sheet names, including each lookup that feeds a template placeholder and every read-back it asks for (an IAM policy, a log group name); a read with no recording is fixed in prep, by picking a recorded read that projects the same value or queuing the recording as code before the sitting, not mid-sitting (HOUS-573, 2026-10-27).
31. SQL for Sean is one idempotent run-all file extracted from the repo, never retyped, handed over with `pbcopy < <file>` or opened in his database viewer; never copied from chat and never block by block, because the chat display clips long lines (HOUS-573, 2026-10-27).
32. A schema card with an unrecognized `DROPS: ... reason not recognized by hous` line for a planned `ALTER COLUMN ... DROP NOT NULL`: read the SQL. If it removes no row, column or value (a constraint relaxation), proceed and name it to Sean in plain words. Any other unrecognized reason still stops the sitting. The run sheet lists each expected `DROPS:` line by card (HOUS-573, 2026-10-30).
33. Between children of a sitting summary, a pending DSQL `ANALYZE` job on a household table can show in `--verify-only` after a live step although it is not the step's own job. Re-read `--verify-only` every 60 s for up to 10 min until `pending-household-jobs: none` before the next card; still pending at 10 min stops the sitting (extends trap 27; HOUS-573, 2026-10-30).
34. Every agent-owned staging inbox address must match an inbox receipt rule (`lib/staging-inbox-stack.ts`): only the sw2, sw3 and sw4 mailboxes and their plus labels. A made-up local part is accepted by the API but its mail is never stored. Check each address against the rules in prep (HOUS-573, 2026-10-30).
35. SQL handed to Sean never contains blocks meant to be skipped or "fallback" statements (extends trap 31): a select-all run applies them. Before a Sean-run setup tool, run its read-only probes in his viewer first (HOUS-573, 2026-10-30).
36. Prep rows: before telling Sean "ready", check that each prep row's output exists (folder, clone, file). The report's word "ready" is not the check (HOUS-573, 2026-10-30).
37. Tester and fixture sign-ins on one address are at least 30 s apart (60 s for kept testers). Four sign-ins within about 10 s get `429 auth.rate_limited`; wait it out (90 s worked) and keep the spacing in drivers and run-sheet rows (HOUS-573, 2026-10-30).
38. Agent-made tester rows replace the old row, never add a second password line: Sean tried the old one and got "incorrect email or password". Hand Sean a password through the clipboard (`pbcopy`), not the chat (HOUS-573, 2026-10-30).
39. Event pushes reach attendees only, and the event API's `memberIds` takes USER ids, not household member ids (a wrong id is dropped silently). A push cue names the phone's account as an attendee and reads the attendees back, stopping if it is missing. Triage a push miss from the outbox task's recipients and device outcomes before telling Sean anything (HOUS-573, 2026-10-30).
40. Before planning a simulator leg, check a Simulator app exists on this Mac (`Developer/Applications/Simulator.app`) and a booted window is drivable; a built simulator app is not enough. Otherwise name the Mac HTTPS client as the second client in the run sheet up front (HOUS-573, 2026-10-30).
