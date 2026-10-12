# Approval mode

The normal path when `hous help aws apply` succeeds (SKILL.md "Pick the write mode first"). Source of truth for every verb,
flag and message: `housetools/tools/aws/README.md` and `tools/aws/src/`. The sitting itself is planned in a run sheet
(`household-infra/docs/runbooks/run-sheet-template.md`): goal, stacks, expected-changes files, reads, rollback. It holds no shell.

## Commands
`<env>` is `staging`, `production` or `management`. `<ticket>` is the story id (it names the change set `hous-<ticket>-<time>`).

| Command | What it does |
|---|---|
| `hous aws login household-<env>` | Sean's sign-in: he types `! hous aws login household-<env>` (or runs it in his terminal). It prints the link and code at once. Agents never run it for him; check with `hous aws login --status household-<env>` (SKILL.md rule 2) |
| `hous aws whoami --env <env> [--stack <Stack>]` | Prints the caller. Exit 4 no session, exit 1 wrong account |
| `hous aws plan <Stack>... --env <env> --task <ticket> --go "<sitting OK, verbatim>" [--commit <sha>]` | Stacks in deploy order, all in one call. Builds change sets from the CI-built templates, **executes nothing**, checks them, prints the card |
| `hous aws ok <card> --words "<Sean's reply, verbatim>"` | Records his OK on a shown card |
| `hous aws apply <card> --task <ticket>` | Runs exactly the change sets on an OK'd card, then the read-backs |
| `hous aws read <name or file.json> --env <env> [--stack <Stack>]` | Named reads: `stack-status`, `stack-outputs`, `termination-protection`, `change-sets`, `failed-events` (each needs `--stack`), or a read file. Reports every step (`PASS`/`FAIL`); exit 1 if any FAIL |
| `hous aws run --allowlist` | Prints the write operations `run` supports (`tools/aws/src/run-allowlist.ts`) |
| `hous aws run --card <plan.json> --task <ticket> --go "<sitting OK>"` | Makes a card for writes outside CloudFormation |
| `hous aws run --apply <card> --task <ticket>` | Runs exactly that card's commands |
| `hous aws hook status` | Is the write-blocking hook installed in Claude Code and Codex, are the Codex rules there, does Codex trust it |

Every verb halts at the first failure and never retries a write; `read` runs every step and reports each, but still stops at once with no session or access denied. The verbs that contact AWS (`whoami`, `plan`, `apply`, `read`, and `run` with `--card` or `--apply`) check the caller
and print `hous aws: evidence: <dir>` last. The local-only ones (`ok`, the `hook` verbs, `run --allowlist`) do neither: `ok` only updates the card (its words, time and relayer),
and `hook check` logs a denial (or a note) to `~/.local/state/hous/aws/hook.log`.
Exit codes: 0 done, 1 stopped, 2 usage or refusal, 4 no session, 10 an AWS failure inside `apply.sh`.

A read file is `{"schemaVersion": 1, "env": "<env>", "steps": [{"argv": ["<service>", "<read-op>", ...], "region"?: "<r>", "expect"?: {"equals": v} | {"matches": "re"} | {"nonEmpty": true}, "project"?: "<dotted path>", "to"?: "<new file>"}]}`.
`hous` adds profile, region and output; it refuses an argv that is not a read.
A step with `project` (for example `Environment.Variables.PROOF_EMAIL_SHARED_SECRET`) keeps only that one value of the response: checked with `expect`, or written
to a new 0600 file with `to`, never printed; the response is never recorded in `run.json`. Use it for every read whose response holds a secret. The step that uses
the file deletes it, and the run sheet names that step.

A write plan is `{"schemaVersion": 1, "env": "<env>", "destructive"?: true, "steps": [{"region"?: "<r>", "before": {"argv": [...], "expect": {...}}, "write": {"argv": ["<service>", "<op>", ...]}, "after": {"argv": [...], "expect": {...}}}]}`.
`before` and `after` are reads with an `expect`. Values go inline (no `file://`). It lives with the run sheet.

## Sitting checklist
```
Sitting progress:
- [ ] 1. Prepare offline
- [ ] 2. Sean signs in and gives the sitting OK
- [ ] 3. Plan every stack of the env: hous aws plan
- [ ] 4. Show the card exactly, get Sean's OK, hous aws ok
- [ ] 5. hous aws apply, then read the read-backs
- [ ] 6. Production: own plan, own card, own OK
- [ ] 7. Lessons entry
```
1. **Prepare (no AWS).** PRs merged; `hous ci` green; `hous lambda check` for Lambda code; the `build-templates` run for the commit has finished
   (its artifact is `cdk-out-<sha>`); every changed stack has its `expected-changes/<env>/<Stack>.json` from the merged PR; the run sheet is reviewed.
   When the sitting's proof uses synthetic accounts, each of them signs in on staging now (one HTTPS sign-in per account, synthetic data, no OK needed) before Sean is asked to sign in,
   and again after any database refresh, restore or marker change; the accounts file existing proves nothing.
2. **Sign-in and sitting OK.** Sean types `! hous aws login household-<env>` (rule 2) and gives the sitting OK. Read the printed link and code from the output and relay them if he asks. Check with `hous aws login --status household-<env>`, then `hous aws whoami --env <env>`.
3. **Plan.** `hous aws plan <Stack>... --env <env> --task <ticket> --go "<his OK>"`. Stop on any refusal below. Fix by PR, then plan again;
   a refused plan deletes the change sets it made. Note the evidence path and the card id.
4. **Card.** Paste the card text to Sean exactly as printed (also in `~/.local/state/hous/aws/cards/<card-id>.md`). Any clear OK counts.
   Then `hous aws ok <card-id> --words "<his reply, verbatim>"`. The OK covers only the change sets on that card and is spent once.
5. **Apply.** `hous aws apply <card-id> --task <ticket>`. It re-checks every change set, runs them in the card's order, watches each stack,
   enables termination protection only on `*_COMPLETE`, then runs the read-backs (`PASS`/`FAIL` lines). Report them to Sean in plain words with times in EDT, no ticket codes, no UTC stamps. Two failures differ:
   - A script or deploy failure stops all later stacks, prints the failed stack's status and failed events, and marks the card failed: go to [failed-stack.md](failed-stack.md).
   - A read-back `FAIL` after every change set ran leaves the card `applied` (spent). Every stack is already deployed, the other read-backs are still run and printed,
     and no status or events are printed: capture them yourself with reads ([failed-stack.md](failed-stack.md) step 1). Do not retry, and write nothing new without a new card and OK.
   A proof round that depends on the worker finishing an earlier round's task ends on that task's own final line (its task id with `delivered` or a terminal code), never on a timer alone; the next round starts only then.
   A log read that runs only on a failure (the raw twin of a verdict read) also includes the worker's error-level lines for the same window (`{ $.level >= 50 }` for household-api's pino logger), so a thrown handler's cause is seen without another OK.
6. **Production.** Only after the same templates are live and verified on staging at the same commit (`plan` checks this and stops otherwise).
   Own `plan`, own card, own OK.
7. **Lessons.** `apply` and `run --apply` end with a reminder: every sitting appends at least one entry to `lessons-pending.md`; a clean sitting records one saying it had no lessons (SKILL.md rule 12).

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
Reply OK to apply exactly these changes.
```
Reading it: `REPLACED or DELETED` lines come first for a stack (`new stack, N added` marks a CREATE). `Staging: no staging copy` is the standing
rule for the management stacks and production SiteOrigin and SiteEdge: Sean decided that his OK on a card carrying that line is enough. Identity, Guardrails and Foundation
have a staging copy and get `Staging: verified at this commit`. A stack with a kept template parameter (production SiteEdge's `CertificateArn`, planned as an update) also
prints `  parameter CertificateArn kept at its previous value`: the value is not changed and never shown. A first create of such a stack is refused by `plan.sh` (a first create needs the operator route).

A `run --card` card reads: `AWS writes outside CloudFormation for <env> (<account>)`; then per step `Step n: aws <service> <op> ... (<region>)`, its `Before:` read with
what was seen now, its `After:` read; `DELETE: step n ...` lines first when a step deletes; `Run as: <caller>`; `Reply OK to run exactly these commands.`

## Foundation
Every Foundation deploy at a new commit shows exactly one `Modify` of the SSM contract parameter, `ParamFoundationContractV1CC26A8C8` (`AWS::SSM::Parameter`,
no replacement), because the parameter embeds the infra commit. Its `expected-changes/<env>/Household-<Env>-Foundation.json` lists that Modify. A missing one
stops `plan` ("in the change set, not expected"). Any other change next to it needs its own line from the PR.

## Where the evidence lands
All under `${XDG_STATE_HOME:-~/.local/state}/hous/aws/` (mode 0700; never in a repo):
- `runs/<stamp>-<verb>-<env>-<rand>/run.json`: verb, arguments, `--go`, caller, commit, every AWS call with its output, the scripts' output, exit.
- `cards/<card-id>.md` (the text for Sean) and `.json` (the binding: change-set ARNs, state `shown` -> `ok` -> `applied|failed`, his words, read-back results).
- `infra/<sha>/` (scratch clone of household-infra at the commit), `artifacts/<sha>/` (the downloaded templates), `hook.log` (every hook deny).
Put the card text, the card id and the `run.json` paths in the sitting notes in the work directory.

## When a verb stops
| Message (start of it) | Meaning and what to do |
|---|---|
| `no AWS session for household-<env>: ask Sean to run hous aws login ...` (exit 4) | Rule 2. Wait for Sean. |
| `STOP: caller <arn> is not an Identity Center session in the <env> account <id>` | Wrong profile or account. Stop; Sean signs in to the right one. |
| `unknown option`, `--task is required`, `--go must hold Sean's sitting OK`, `name at least one stack`, `<Stack> is named twice` (exit 2) | Fix the call; the usage line follows. |
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
| `card <id> has no recorded OK` | Show the card, then `hous aws ok`. |
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

## What is in place
- The hook (`hous aws hook check`, installed in Claude Code and Codex) denies AWS writes, `cdk deploy|destroy|bootstrap|rollback|import|refactor|watch|gc`.
  Reads, `hous aws apply|run` and everything else pass. Check it with `hous aws hook status` (`hous doctor` prints it too).
- Built into every verb that contacts AWS: the card OK bound to change-set ARNs, the caller check, explicit profile and region, a clean environment, halt on failure, evidence. The local-only operations (`ok`, `hook`, `run --allowlist`) have none of the AWS ones; they leave only the card, the hook settings and `hook.log` records.
- `hous ci` in household-infra runs the `checks` workflow: tests, lint, synth, `cfn-lint` on the templates (`scripts/ci/cfn-lint.sh`, a job-owned venv; never install it yourself), `assert:assembly`, and the static checks: cdk-nag (`test/cdk-nag.test.ts`), the stateful logical-ID lock (`test/stateful-ids.test.ts`, `config/stateful-ids.json`) and the expected-changes schema check (`schemas/expected-changes.v1.json`, `test/expected-changes.test.ts`). `build-templates` makes the artifact `plan` uses.
- The deploy buttons stay as a fallback. Their GitHub actor and OIDC restrictions are unchanged.
