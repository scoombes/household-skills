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
3. **Plan.** `hous aws plan <Stack>... --env <env> --task <ticket> --go "<his OK>"`. Stop on any refusal ([cards-and-stops.md](cards-and-stops.md#when-a-verb-stops)). Fix by PR, then plan again;
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

## Reading cards and handling stops
How to read a card (an example, the `Staging:` lines, kept parameters, `run --card` cards) and what each
refusal or error message means are in [cards-and-stops.md](cards-and-stops.md). Open it before showing
Sean a card, and whenever a verb stops.

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

## What is in place
- The hook (`hous aws hook check`, installed in Claude Code and Codex) denies AWS writes, `cdk deploy|destroy|bootstrap|rollback|import|refactor|watch|gc`.
  Reads, `hous aws apply|run` and everything else pass. Check it with `hous aws hook status` (`hous doctor` prints it too).
- Built into every verb that contacts AWS: the card OK bound to change-set ARNs, the caller check, explicit profile and region, a clean environment, halt on failure, evidence. The local-only operations (`ok`, `hook`, `run --allowlist`) have none of the AWS ones; they leave only the card, the hook settings and `hook.log` records.
- `hous ci` in household-infra runs the `checks` workflow: tests, lint, synth, `cfn-lint` on the templates (`scripts/ci/cfn-lint.sh`, a job-owned venv; never install it yourself), `assert:assembly`, and the static checks: cdk-nag (`test/cdk-nag.test.ts`), the stateful logical-ID lock (`test/stateful-ids.test.ts`, `config/stateful-ids.json`) and the expected-changes schema check (`schemas/expected-changes.v1.json`, `test/expected-changes.test.ts`). `build-templates` makes the artifact `plan` uses.
- The deploy buttons stay as a fallback. Their GitHub actor and OIDC restrictions are unchanged.
