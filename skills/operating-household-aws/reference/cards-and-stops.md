# Reading a card, and when a verb stops

Lookup material for a sitting run under [approval-mode.md](approval-mode.md). Read "A real card" before showing
Sean his first card in a session; read "When a verb stops" when a verb prints a refusal or error.

## A real card
This is the card from housetools' test fixture (`tools/aws/test/fixtures/card.md`). It shows an update with replacements, a new stack
(CREATE), a "no staging copy" stack, a stack with nothing to do, and the closing lines.
```
AWS changes for production (984381715450)

Household-Production-Foundation: 1 added, 3 changed, 1 removed
  REPLACED or DELETED: ArtifactsBucket (AWS::S3::Bucket), replaced: the bucket moves to the new naming rule
  REPLACED or DELETED: Dist (AWS::CloudFront::Distribution), may be replaced: the origin id changes
  REPLACED or DELETED: OldTopic (AWS::SNS::Topic), deleted
  add NewAlarm (AWS::CloudWatch::Alarm)
  change DeployRole (AWS::IAM::Role)
  Checks: validation WARN: AWS::Early::Validation: bucket policy too broad (Resources/ArtifactsBucket); IAM new access: DeployRole.Policies.Deploy; matches PR #321 yes
  IAM DeployRole.Policies.Deploy: new access: New access in the statement with index: 1.
  Staging: verified at this commit
  CloudFormation runs as: bootstrap exec role (arn:aws:iam::984381715450:role/cdk-hnb659fds-cfn-exec-role-984381715450-ca-central-1)
  Change set: arn:aws:cloudformation:ca-central-1:984381715450:changeSet/hous-HOUS-614-20261009T140000Z/0aB1c2D3-e4F5-a6B7-c8D9-e0F1a2B3c4D5

Household-Production-SiteEdge: new stack, 2 added
  add Distribution (AWS::CloudFront::Distribution)
  add EdgeFunction (AWS::CloudFront::Function)
  Checks: validation PASS; IAM grants nothing new; matches PR #322 yes
  Staging: no staging copy
  CloudFormation runs as: your session
  Change set: arn:aws:cloudformation:us-east-1:984381715450:changeSet/hous-HOUS-614-20261009T140000Z/1bC2d3E4-f5A6-b7C8-d9E0-f1A2b3C4d5E6

Household-Production-Guardrails: no changes, nothing to apply
  Checks: matches PR #323 yes

No staging copy: Household-Production-SiteEdge. Your OK on this card applies them without a staging run.
Templates: household-infra commit 8c57685bc4683769ba8d34ae53ff1f23ff670d85, build-templates run 7001.
To apply exactly these changes, choose one:
Approve production-20261009T140000Z-0a1b2c fb3415602630
Reject
```
Reading it: `REPLACED or DELETED` lines come first for a stack (`new stack, N added` marks a CREATE). `Staging: no staging copy` is the standing
rule for the management stacks and production SiteOrigin and SiteEdge: Sean decided that his OK on a card carrying that line is enough. Identity, Guardrails and Foundation
have a staging copy and get `Staging: verified at this commit`. A stack with a kept template parameter (production SiteEdge's `CertificateArn`, planned as an update) also
prints `  parameter CertificateArn kept at its previous value`: the value is not changed and never shown. A first create of such a stack is refused by `plan.sh` (a first create needs the operator route).

A `run --card` card reads: `AWS writes outside CloudFormation for <env> (<account>)`; then per step `Step n: aws <service> <op> ... (<region>)`, its `Before:` read with
what was seen now, its `After:` read; `DELETE: step n ...` lines first when a step deletes; `Run as: <caller>`; then `To run exactly these commands, choose one:`, `Approve <card id> <12 hex>` and `Reject` (the picker's two options).

A schema card (`run --card` with `dsql-proof.apply-step` entries) reads: `AWS writes outside CloudFormation for staging (<account>)`; then every statement that drops
something, first, as `DROPS: entry n <file>: <reason>` (a practice run's own scratch table or schema, or a constraint the same entry added, validated and renamed into
the dropped one's place; anything else says `reason not recognized by hous; read the SQL below before OK`: read it before showing the card); the IRREVERSIBLE notice;
the cluster ARN and endpoint and both pins; per entry the step id, `practice run` or `live; requires practice run n PASS`, the full `fromDigest → toDigest`, the
statement counts and the result path, and under each live entry every statement in order (cut at 300 characters with `...`; the card JSON keeps them whole);
`Run as: <caller>; database schema role: ...; runtime role: ...`; then `To run exactly these operations, choose one:`, `Approve <card id> <12 hex>` and `Reject`. Show it unedited; it goes to Sean exactly as printed.

A single-operation card (`dsql-proof.verify-init`, or `dsql-proof.backfill-owners`; only where the installed tooling lists them, see [approval-mode.md](approval-mode.md#schema-and-data-change-sittings)) is read the same way, with these differences: the one entry's effect warning comes first (verify: `WRITES SYNTHETIC PROBE ROWS`), there are no separate `DROPS:` lines,
and every write statement is shown in full, never cut, with the identities, both pins, the chain and its digest, the manifest identity (verify), the generated namespace (verify), the result path, the time bound and the irreversible effects. A verify card is a write: its probe statements are on it. A backfill card's tool-printed card instead shows the effect first (writes owner flags), the cluster, both pins, the chain boundary (0005 and 0006 with their digests, the ledger head that must hold), the module source hash and its file hashes, the infra policy hash, the runner, the only data statement (a fixed `UPDATE` template, one primary-key row per statement), the selection and write rules, the limits, the baseline (path, sha256 and private copy), the live argv, the result path with its time bound, and the result it accepts; it does not list per-row statements (see `hous help aws run` and housetools' `tools/aws/README.md`). Show it unedited. `hous aws ok` on a verify card prints `OK recorded (1 operation)`. There is no reset or init card: staging `dsql-proof.reset-init` and `dsql-proof.apply-init` are retired, so a card that names either is refused, never shown.

## Sitting summaries
The combined summary prints every child in execution order, without truncating its content, plus each id and whole immutable-card hash.
Sean's one reply covers exactly that list, not other cards with matching stack names. The summary must exist in full before the reply.
For a staging schema or data change the order is fixed: the writer stop runs first as a standalone card with its own OK, and after the drain reads and the authoritative precheck (the baseline) comes the one change summary: the numbered apply-step cards and any fixed data card in chain order (for the ownership change: the 0005 pair, `dsql-proof.backfill-owners`, then the 0006, 0007 and 0008 pairs), then verify-init when the run sheet calls for it; then, in a second summary built after verification, the release cards, the seed card and the writer restart card last (built only after the stop card has run). Sean sees every one of them, in that order, before replying. Inbox setup is not a card and never a child of the summary: it is Sean's separate action in his own terminal, after verification, under its own separate exact OK.
Use `hous aws sitting apply` to execute it; do not OK or apply its children separately. Any failed child/read-back or interrupted apply spends
the summary and stops the remaining list. A replacement or later-built card needs a new summary and a new OK. Production stays separate.

## When a verb stops
| Message (start of it) | Meaning and what to do |
|---|---|
| `no AWS session for household-<env>: ask Sean to run hous aws login ...` (exit 4) | Rule 2. Wait for Sean. |
| `STOP: caller <arn> is not an Identity Center session in the <env> account <id>` | Wrong profile or account. Stop; Sean signs in to the right one. |
| `unknown option`, `--task is required`, `--go must hold Sean's sitting OK`, `name at least one stack`, `<Stack> is named twice` (exit 2) | Fix the call; the usage line follows. |
| `--words must be the approval label printed on <id>, exactly as chosen in the picker: nothing was recorded (Reject, typed text or any other words are not an OK)` (`ok`, exit 2; `tools/aws/src/approval.ts:15`, `tools/aws/README.md:358`) | Nothing was recorded. Stop: Sean chose `Reject`, or the reply is not the exact `Approve <id> <12 hex>` label the card or summary printed (no trimming, case folding or partial hash). Never reword or replay it; ask again only with the card or summary shown in full. |
| `--go must be the sitting OK's approval label, exactly as chosen in the picker: Approve <env> <scope words> (a refusal, a deferral or any other text is not an OK)` (`plan`, `run --card`, `inbox-ownership`, exit 2; `tools/aws/src/approval.ts:27`, `tools/aws/README.md:359`) | Nothing ran. The env in the label must be exactly the run's env. Only the picker's `Approve <env> <scope words>` is a sitting OK; stop and ask Sean, never reword his reply. |
| `--go names no scope: Approve <env> <scope words>` (exit 2; `tools/aws/src/approval.ts:29`) | Nothing ran. The label has the env but no scope words; ask again with the scope named. |
| `<Stack> is not in household-infra config/targets.json`, `belongs to <x>, not <env>`, `is not selected` | Wrong stack name or env, or the stack is not deployable. Check `config/targets.json`. |
| `--commit <sha> is not on household-infra main` | Plan a commit that is on `main`. |
| `no build-templates artifact cdk-out-<sha> (absent or expired)` | Wait for the `build-templates` run of that commit, or re-run it. Never build templates locally. |
| `the artifact ... says commit ... not ...`, `does not match household-infra's own derivation` | The artifact is not the commit's. Stop and report; re-run `build-templates`. |
| `staging is not live at this commit: ...` | Production needs the same templates applied and read back on staging first. Do staging, then plan production. |
| `its staging copy ... is not selected`, `cannot classify <Stack> for production after staging` | The stack cannot be classified. Stop and report. |
| `<Stack> is <STATUS>: plan works only on an absent, REVIEW_IN_PROGRESS or settled stack` | A failed or busy stack. Triage with [failed-stack.md](failed-stack.md). |
| `pending CREATE change set <arn> from an earlier plan` | A leftover from an earlier plan. Apply it from its card, or delete it with a `hous aws run` write plan (`cloudformation delete-change-set`) on its own card; then plan again. |
| `no expected-changes/<env>/<Stack>.json` / `does not follow schemas/expected-changes.v1.json` | The PR that changes the stack must commit the file. Fix by PR. |
| `the change set does not match expected-changes/... (PR #n)` with `expected, not in the change set` / `in the change set, not expected` lines | The template and the PR disagree. Stop. Fix the file or the template by PR; never edit around it. |
| `replacement not listed in expected-changes/...` | A replacement the PR did not explain. Blocker until it is explained and Sean accepts it. |
| `change set validation FAIL: ...` | CloudFormation's own check failed. Fix by PR. A WARN is on the card, not a stop. |
| `change set <arn> is <status>/<exec>, not CREATE_COMPLETE/AVAILABLE` | CloudFormation could not build it; the reason follows. Fix by PR. |
| `scripts/deploy/plan.sh stopped with exit N` / `did not report <Stack> as planned` | The script's refusal text is above it. Exit 2 refusal, 3 digest or asset mismatch, 8 guard, 10 AWS failure. Stop and report. |
| `plan: refused: <Stack> is new and its parameter(s) ... have no Default: a first create needs the operator route` | A first create of a stack with a required parameter is not a `plan` job. Stop; tell Sean. |
| `scripts/deploy/plan.sh kept ... but <Stack> is planned as CREATE` | Same cause. Stop and report. |
| `deleted change set <arn>` / `could not delete change set ...; delete it before planning again` | A refused plan cleans up after itself; on the second message delete the named set (a `run` plan on its own card) before planning again. |
| `card <id> has no recorded OK` | Show the standalone card or complete sitting summary, then record Sean's reply with `hous aws ok`. A general sitting go-ahead is insufficient. |
| `card <id> is <state>, not shown; an OK is recorded once` (`ok`) | The OK is already recorded or spent. Plan again for a new card. |
| `card <id> was already applied`, `failed earlier`, `is a write plan: run it with hous aws run --apply` | An OK is spent once. Plan again, or use the other verb. |
| `change set <arn> is missing`, `is not AVAILABLE`, `was re-created since the card`, `changed since the card`, `the live template changed since the card` | Something moved after Sean's OK. Nothing ran. Plan again for a new card and a new OK. |
| `the stack is <x>, not REVIEW_IN_PROGRESS, so this CREATE change set cannot be the first execution` / `another change set is pending` | Same. Plan again (delete the extra set first). |
| `scripts/deploy/apply.sh stopped with exit N: <Stack> is the first stack not reported deployed` | A stack failed. Later stacks were not applied (`not applied: ...`, their change sets stay AVAILABLE). Read the printed status and failed events, then [failed-stack.md](failed-stack.md). |
| `the change sets were executed, but a read-back failed` | The change ran and the card stays `applied`. Read the `FAIL` lines (every read-back was still run); no status or events were printed, so capture them with reads ([failed-stack.md](failed-stack.md)). A production plan needs every staging read-back to be PASS. |
| `step n: ... use hous aws plan and hous aws apply` | `run` never touches `create-stack`, `update-stack`, `create-change-set`, `execute-change-set`, `deploy` or `cdk`. Use `plan`/`apply`. No flag changes this. |
| `step n: aws <service> <op> is not a supported run operation` | Not on the allow-list. Add it by a reviewed housetools PR, never by a flag. |
| `step n: aws ... is refused with no override`, `--force is refused` | `s3 rb`, `s3api delete-bucket`, `route53 delete-hosted-zone`, `kms schedule-key-deletion`, `dsql delete-cluster`, `backup delete-backup-vault`, `iam delete-*`, anything forced. Stop; this is Sean's decision outside the tool. |
| `step n is destructive ...: the plan must say "destructive": true` | Add it. The card then prints `DELETE` first. |
| `delete-stack runs only on a ROLLBACK_COMPLETE stack` / `is UPDATE_ROLLBACK_COMPLETE: ... never deleted` / `termination protection is on` | Rule 8. Do not delete. Turning protection off is Sean's decision. |
| `reads a file (file://): put the value inline` | The card must show exactly what runs. |
| `the before read does not hold now` / `differs from card time: plan again` | Live state moved. Make a new card. |
| `the card was edited after it was shown` | Do not edit cards. Make a new one. |
| `step n ran, but its after read failed` | The write happened and later steps did not run. Read the live state before anything else. |
| `aws <service> <op> is not a read; hous aws read runs reads only`, `unregistered AWS operation` | Use a supported read. A new read operation needs a housetools change. |
| Hook deny: `AWS write <service> <operation>: writes run only through hous aws apply|run after Sean's OK` | A direct write. Do not retry or rephrase. Use the verb. `bash -c`, scripts and `python -c` are not inspected by the hook; they are still forbidden. |
| `step n: aws lambda put-function-concurrency takes exactly --function-name <function name> --reserved-concurrent-executions 0, on staging only` / `step n: aws lambda delete-function-concurrency takes exactly --function-name <function name>, on staging only` / `step n: aws lambda update-event-source-mapping takes exactly --uuid <mapping uuid> and one of --enabled or --no-enabled, on staging only` | The writer stop or restart step is not in its one closed form (production, an ARN or qualifier, any reservation but 0, an uppercase uuid, extra flags). Nothing ran. Fix the plan; there is no override. If `step n: aws lambda ... is not a supported run operation` appears instead, the installed tooling lacks the operations (`hous aws run --allowlist`): wait for the update, and do not stop the writers another way. |
| A writer stop or restart read pin (exit 2, at `run --card` plan parsing, before any AWS contact; `tools/aws/src/run-pins.ts`, called from `run.ts`): `step n before: a put-function-concurrency stop must pin "no reservation" ({"equals": null}) on its own function's get-function-concurrency (--function-name <name>): ...` / `step n after: a delete-function-concurrency restart must prove "no reservation" ({"equals": null}) on its own function's get-function-concurrency (--function-name <name>)` / `step n before: a --no-enabled stop must pin State Enabled on its own mapping's get-event-source-mapping (--uuid <uuid>): ...` / `step n after: an --enabled restart must prove the feed is Enabled (or Enabling) on its own mapping's get-event-source-mapping (--uuid <uuid>)` | The plan's own read does not pin the state the step restores to; nothing ran and no live state was read, so this is not a before-state mismatch. Fix the plan file; there is no override. The required reads: a function stop's `before` and a function restart's `after` are `lambda get-function-concurrency --function-name <the same name>` with `{"equals": null}` and **no `project`** (it must accept the null response and reject `{}` and any reservation); a feed stop's `before` is `lambda get-event-source-mapping --uuid <the same uuid>` with `project: State` and `{"equals": "Enabled"}`; a feed restart's `after` is the same read with `project: State` accepting `Enabled` (`^(Enabling\|Enabled)$` is the usual form) and none of `Disabled`, `Disabling`, `Creating`, `Updating` or `Deleting`. The stop pins state "none" and "Enabled" so that the restart can put back exactly what was there. A refusal that begins `step n: the before read does not hold now` is the other kind: the live state moved (next row). |
| `step n before: to is for hous aws read files only: a write plan's reads write no file` | A write plan's read took `to`. Use `project` with `expect` alone. |
| A writer stop card's `run --card` refuses: `step n: the before read does not hold now: ...` | A writer is not in the state the stop expects (already reserved, or the feed not `Enabled`). Nothing ran. Read the live state (the run sheet's state read) before building anything; do not edit the expect to make it pass. |
| The standalone stop card or the change summary stopped before any schema or data change: the stop card failed partway (`step n ran, but its after read failed` or a failed step), the feed read-back did not reach `Disabled` within its bound, the dead-letter read or the after-drain precheck failed, or the first schema or data card refused before writing | The old code still matches the old schema. Read the live state; build a **standalone** restart card from the stopped resources actually observed (restart order: feed, live-update function, API), with its own fresh OK; never a summary child, never a spent or summary-owned card. A restart card already owned by a summary is not used. If a summary exists, closing it comes first (`hous aws sitting stop`). |
| The change summary stopped after a schema or data change (a step, the backfill or later failed, timed out or was interrupted) | The writers stay stopped. No restart card, standalone or otherwise, until reviewed forward completion (Sean's recovery route for a state the tool does not support) and a new complete summary. Never a reset. |
| The release summary stopped at a release or seed child (a change set failed or was refused, or the catalogue seed failed or its read-back failed) | The writers stay stopped and no restart card is built, standalone or otherwise: the deployed code may not match the changed schema, and old code must not be run against it. Read the evidence (`failed-stack.md` for a failed stack; for a failed seed, recovery is Sean's), follow the run sheet's recovery, and restart only from a fresh complete summary whose release and seed checks pass. |
| The release summary stopped at the restart card, after the release and seed checks passed (the restart card's before read did not hold, or its write or after read failed or was refused) | The deployed code matches the live schema, so restarting is safe. Whatever is still stopped stays so until a standalone restart card built from the observed state has its own fresh OK. Read the stopped set first; a deploy that cleared a reservation changes which steps are needed. |
| `step n: live entry must immediately follow its matching rehearsal` | Repair the schema plan's order or `rehearsalStep`, then `run --card` again; nothing ran. |
| `step n: fromDigest does not continue the preceding live toDigest` | The selected steps are out of order. Repair the plan and make a new card; a dry run grants no permission to skip a step. |
| `DSQL apply-step supports staging only` / `unsupported DSQL operation, mode or field` | Correct the fixed shape. There is no override and no argv route. A single-operation card likewise takes exactly one entry and no `stack`, `mode`, `step`, `rehearsalStep` or override; an operation missing from `hous aws run --allowlist` is not installed, so wait for the tooling update. `dsql-proof.reset-init is retired: ...` and `dsql-proof.apply-init is retired: ...` (also `card <id>: ...`, `sitting <id> lists card <id>: ...; nothing in it runs`) mean nothing ran: no route wipes or re-initializes staging, so change the schema forward with apply-step cards and never work around it. |
| `step n: checkout is dirty` / `checkout HEAD differs from pinned commit` / `... is not on the checkout's origin/main` | Restore the scratch clone at its pin (refresh `origin/main` through the App helper), then `run --card` again. Never work around it. |
| `step n: output path already exists` | Keep the old evidence. Make a new card (it gets fresh result paths). |
| `step n: dry-run produced no valid JSON` / `step n: dry-run failed` | The tool could not plan the step offline; its streams are in the run's `dsql/dryrun/`. Fix the input or invocation before any card. |
| `card <id>: execution marker already exists; this OK is spent` | An apply already claimed this card. Do not delete the marker or retry: read the evidence, triage, then a new card and a new OK. |
| `step n: rehearsal did not PASS including cleanup` / `step n: apply-step failed or evidence is incomplete` | That operation failed; nothing later ran and the card is spent. Read the ledger (`--verify-only` under the sitting OK) and follow the run sheet's recovery: recovery SQL is Sean's, in his viewer; then a new card and a new OK. |
| A single-operation card's child failed, timed out, was interrupted, failed a read-back or left a `STARTED` or `FAILED` record (backfill-owners, verify-init) | The card is spent and its summary stops; nothing is retried. A timeout may leave partial work or verifier probes: do not run cleanup SQL. Read the evidence; recovery is data-preserving and forward (Sean's route when the tool does not support the state, never a reset); then a new card, a new summary and a new OK. |
| `step n: the offline plan differs from card time` / `step n: <field> differs from card time` | The checkout or target moved after the OK. Nothing ran. Make a new card and get a new OK. |
| Hook deny: `dsql-proof apply-step writes run only through hous aws run --apply after Sean's card OK (live and rehearse)` | A direct schema write. Do not retry or rephrase; use the schema card. |
| Hook deny naming another `dsql-proof` writer (`reset-init`, `apply-init`, `verify-init` and the rest), an unknown `dsql-proof` verb or option, household-api's `verify:dsql` verifier, or a writer "named anywhere in a command ... that is not its exact read-only form" (`unknown-runner`) | A direct database write, or a form the hook cannot prove read-only. Do not retry or rephrase: use the card route if the installed tooling lists the operation, otherwise stop. Only the read-only forms listed in [approval-mode.md](approval-mode.md#what-is-in-place) pass. |
| Hook deny naming `tools/inbox-ownership/src/setup-cli.ts` | The inbox setup tool is Sean's alone, in his terminal; agents run only its `--show-plan` and `--help`. |
| `card <id> is owned by sitting <sitting>`, `card <id>: child hash differs from the approved summary`, `card <id> is not listed in sitting <sitting>`, `card <id> is card n of sitting <sitting>; the next card is ...` | No standalone child execution or membership substitution. Stop; a changed or later card needs a new complete summary and OK. |
| `sitting <id>: sitting execution marker already exists`, `card <id>: its sitting claim already exists`, `sitting <id> failed at card ...`, `sitting <id> failed earlier`, `sitting <id> was stopped`, `sitting <id> is applied` | No retry or resume. Read evidence; recovery and replacement need a fresh summary/card and OK. |
| `sitting <id> changed since it was shown` (`ok`), `sitting <id> changed after the OK` (`sitting apply`) | No OK was recorded, or nothing ran. Seal a new complete summary and show it. |
| `<file> has no hous aws check receipt for its current content`, `the hous aws check receipt for <file> is stale` | Run the offline check; no AWS call is allowed to begin until its receipt matches. |
