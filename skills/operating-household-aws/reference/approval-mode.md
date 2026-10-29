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
| `hous aws plan <Stack>... --env <env> --task <ticket> --go "Approve <env> <scope words>" [--commit <sha>]` | Stacks in deploy order, all in one call. Builds change sets from the CI-built templates, **executes nothing**, checks them, prints the card |
| `hous aws ok <card-or-sitting-id> --words "<the chosen Approve label, verbatim>"` | Records his OK on a shown standalone card or complete sealed sitting summary only if it is exactly the `Approve <id> <12 hex>` line that card or summary printed; anything else (`Reject`, typed text) records nothing, exit 2 |
| `hous aws sitting card <card>... --task <ticket>` | Local only: seal fully built, shown cards in execution order; print every child's full content, ids and hashes |
| `hous aws sitting apply <sitting-id> --card <next-child> --task <ticket>` | Execute only the next listed card once through the existing apply/run gates; first failure stops the whole sitting |
| `hous aws sitting stop <sitting-id>` | Local only: close the sitting on a run-sheet check failure or when the session ends; unused permission cannot resume |
| `hous aws check <read-or-plan.json>` | Offline: check an AWS read/write-plan file against recorded CLI response structure and write its local check receipt; predicate results are information only; grants no AWS permission |
| `hous aws apply <card> --task <ticket>` | Runs exactly the change sets on an OK'd card, then the read-backs |
| `hous aws read <name or file.json> --env <env> [--stack <Stack>]` | Named reads: `stack-status`, `stack-outputs`, `termination-protection`, `change-sets`, `failed-events` (each needs `--stack`), or a read file. Reports every step (`PASS`/`FAIL`); exit 1 if any FAIL |
| `hous aws inbox-ownership` (task, row label, boundary, sitting OK, absolute scratch `infraDir`, full 40-hex `infraCommit`) | A read under the current explicitly scoped sitting OK (`--go "Approve staging reads for this sitting, including hous aws inbox-ownership and its fixed count reads"` or another `Approve staging <scope words>` naming these reads), solely through its pinned infra client: it assumes Household-Staging-InboxOwnershipReader in staging, signs a non-admin DbConnect token, and calls only household_inbox_checks.ownership_counts; it grants no setup, mapping, generic SQL, other AssumeRole or production route. Prints the row/boundary labels and four counts only; a failure is a fixed category with no fabricated zero counts |
| `hous aws run --allowlist` | Prints the write operations `run` supports (`tools/aws/src/run-allowlist.ts`). It is also how you learn whether the installed tooling exposes the refresh operations below |
| `hous aws run --card <plan.json> --task <ticket> --go "Approve <env> <scope words>"` | Makes a card for writes outside CloudFormation |
| `hous aws run --apply <card> --task <ticket>` | Runs exactly that card's commands (or its fixed `dsql-proof` operations, in order) |
| `hous aws hook status` | Is the write-blocking hook installed in Claude Code and Codex, are the Codex rules there, does Codex trust it |

Every verb halts at the first failure and never retries a write; `read` runs every step and reports each, but still stops at once with no session or access denied. The verbs that contact AWS (`whoami`, `plan`, `apply`, `read`, and `run` with `--card` or `--apply`) check the caller
and print `hous aws: evidence: <dir>` last. `sitting apply` preserves these checks for every child and records the ordered child evidence paths.
The local-only ones (`ok`, `check`, `sitting card`, `sitting stop`, the `hook` verbs, `run --allowlist`) make no AWS calls: `ok` only updates the card or summary (its words, time, relayer and approved hash),
and `hook check` logs a denial (or a note) to `~/.local/state/hous/aws/hook.log`.
Exit codes: 0 done, 1 stopped, 2 usage or refusal (including a `--go` or `--words` that is not the exact approval label), 4 no session, 10 an AWS failure inside `apply.sh`.

A read file is `{"schemaVersion": 1, "env": "<env>", "steps": [{"argv": ["<service>", "<read-op>", ...], "region"?: "<r>", "expect"?: {"equals": v} | {"matches": "re"} | {"nonEmpty": true}, "project"?: "<dotted path>", "to"?: "<new file>"}]}`.
`hous` adds profile, region and output; it refuses an argv that is not a read.
A step with `project` (for example `Environment.Variables.PROOF_EMAIL_SHARED_SECRET`) keeps only that one value of the response: checked with `expect`, or written
to a new 0600 file with `to`, never printed; the response is never recorded in `run.json`. Use it for every read whose response holds a secret. The step that uses
the file deletes it, and the run sheet names that step.

A write plan is `{"schemaVersion": 1, "env": "<env>", "destructive"?: true, "steps": [{"region"?: "<r>", "before": {"argv": [...], "expect": {...}}, "write": {"argv": ["<service>", "<op>", ...]}, "after": {"argv": [...], "expect": {...}}}]}`.
`before` and `after` are reads with an `expect`. Values go inline (no `file://`). It lives with the run sheet.
A `before` or `after` read may take `project` (a dotted path, with its `expect`; `to` is refused, a write plan's reads write no file): the card shows that one value and `run --apply` compares only it with card time, so a field the step does not consume cannot refuse the card. Use it, expect only and never `to`, on the event-source-mapping reads of the writer stop and restart below (`project: State`) and nowhere else there: the function-concurrency reads carry no `project`, because they prove "no reservation" with the whole null response and a projected missing value is refused.
On staging only, `run` also takes the writer stop and restart operations, each in exactly one form: `lambda put-function-concurrency --function-name <name> --reserved-concurrent-executions 0` (only 0), `lambda delete-function-concurrency --function-name <name>`, and `lambda update-event-source-mapping --uuid <uuid> --enabled` or `--no-enabled`. The name is a plain function name (no ARN, no qualifier) and the uuid is lowercase. Use them only when `hous aws run --allowlist` lists the `lambda` operations. Their reads are `lambda get-function-concurrency`, which prints nothing when no reservation is set (so that expect is `{"equals": null}`, never `{}`, and `{"equals": {"ReservedConcurrentExecutions": 0}}` once stopped) and `lambda get-event-source-mapping` with `project: State`. The tool pins these reads when it parses the plan, before any AWS contact; the pins and their refusals are in [cards-and-stops.md](cards-and-stops.md#when-a-verb-stops).

A schema write plan (staging only) is `{"schemaVersion": 1, "env": "staging", "destructive": true, "steps": [{"write": {"operation": "dsql-proof.apply-step", "infraDir", "infraCommit", "apiDir", "apiCommit", "step": "<NNNN_name>", "mode": "rehearse"|"live", "rehearsalStep"?: <n>}}]}`:
each live step right after its own practice run (`rehearsalStep` = that entry), every entry on the same two scratch clones and full 40-hex pins. Nothing else can be named
(no argv, SQL, profile, region, endpoint, role or output path). Before Sean is asked, refresh both clones' `origin/main` through the App helper with explicit
`+refs/heads/main:refs/remotes/origin/main` refspecs and check each pin is on it, both clones clean, infra `npm ci` done: `run --card` refuses otherwise.
`run --card` (under the sitting OK) runs the tool's offline dry runs and the caller check only; `run --apply` (after the card OK) runs the four operations in
order, each with the tool's own preflight, statements, practice-run cleanup and ledger read-back, and stops at the first failure. A failed card is spent: after
Sean's recovery, a new card (fresh practice run and live pair, fresh result paths) and a new OK. If the failed card belonged to a sitting summary,
that summary is spent too: every remaining card stops, and replacement or later-built cards need a new complete summary and new OK.
Never delete a card's or summary's execution marker.
Give `run --apply` and `sitting apply` a tool timeout long enough for their printed bounds, or launch once in a resumable background session.
Never interrupt an in-flight apply to obtain a quicker answer; a lost tool reply is not permission to restart it.

## Database refresh cards
The `seed-catalogue` plan has the same closed shape as a schema plan (a practice entry, then its live entry with `rehearsalStep`); housetools' `tools/aws/README.md` has its example.
Three further fixed staging operations reset, recreate and check the database. Use them **only when `hous aws run --allowlist` lists them** (as `local dsql-proof.reset-init (staging only: ...)`, `local dsql-proof.apply-init (...)` and `local dsql-proof.verify-init (...)`); if the installed tooling does not, stop and wait for its update, never substitute a raw writer, a `--dir` input or generic SQL.
A refresh card is `{"schemaVersion": 1, "env": "staging", "destructive": true, "steps": [{"write": {"operation": "dsql-proof.reset-init" | "dsql-proof.apply-init" | "dsql-proof.verify-init", "infraDir", "infraCommit", "apiDir", "apiCommit"}}]}` with **exactly one** entry
(`dsql-proof.verify-init` also takes `artifactManifest`, an absolute path to a regular file you own). There is no `stack`, `mode`, `step`, `rehearsalStep`, command, SQL, role, endpoint, profile, region or output path, and no practice run; the target is derived from infra's pinned config.
Prepare as for a schema card: both scratch clones clean, each pin on the refreshed `origin/main`, `npm ci` done in both (the verifier needs the API clone's).
- `dsql-proof.reset-init` **deletes every household table and all its data** (the card says so first). Point it at the **previously applied** API chain: `0001` through its real pinned head, a real `main` ancestor, never a truncated clone and never the target chain.
  Before any drop it checks the live inventory, the completed full ledger and that no household job is pending; on a partial, divergent or newer ledger it refuses before any drop. Then stop and reconcile, and account for the stopped writers as the writer stop and restart rule below says.
  It drops no schema, role or IAM mapping, uses no CASCADE, repairs no ledger and backfills nothing; an already-empty schema is a checked no-op.
- `dsql-proof.apply-init` applies the **target** chain's `0001_init` only, onto an empty schema, with the four fixed `Household-Staging-{Dsql,Api,Realtime,Delivery}Runtime` mappings; grants come before the completion ledger, which it reads back as exactly one row. Numbered steps `0002` onward stay `apply-step` cards.
- `dsql-proof.verify-init` is a **writer, not a read**: one pinned API integrity runner checks the complete target chain against the shipping artifact manifest (matching head digest), committing and removing synthetic probe rows under a generated namespace, with separate admin and runtime identities.
  The card shows every statement in full. It does not repair or clean up an interrupted earlier run, and a matching digest does not prove the artifact is deployed: the run sheet binds the manifest to the published artifact.
- **Writer stop and restart.** A refresh stops the staging writers on one `hous aws run` stop card, and restarts them on a second card built later. The writers are the API function and the live-update function (each set to reserved concurrency 0) and the delivery worker's queue feed (its event source mapping, set to disabled). Disable the feed; do not give the worker concurrency 0, because throttled batches go back to the queue and their receive counts still climb until they dead-letter (trap 26). The run sheet reads the physical names and the feed's uuid inside the sitting (its first reads, under the sitting OK), not in offline prep.
  - **Stop card.** It is the first child of the refresh summary: API function, live-update function, then the feed. Between it and `dsql-proof.reset-init` the run sheet reads the feed back `Disabled` (bounded polling: `Disabling` is not yet stopped, and any other state or a read error stops the sitting), waits 35 seconds (longer than the 30 s worker and 29 s API timeouts), and records the dead-letter queue's depth and oldest-message age. Reset never starts before those pass.
  - **Restart card.** Build it only after the stop card has run, never before: its before reads must show the stopped state, so it can never be sealed with its stop. It re-enables the feed, then removes the live-update reservation, then the API reservation last (that is what lets users back in). On success it is the last child of the release summary, after the release change sets and the catalogue seed: never restart a writer before the deployed code matches the live schema.
  - **Refusal or partial stop before any drop** (the stop card failed partway, the feed read-back did not reach `Disabled` within its bound, or `reset-init` refused before any drop): the old code still matches the old schema. Read the live state, build a standalone recovery card from the stopped resources actually observed (never from what the stop card intended), in the restart order (feed, live-update function, API), with its own fresh OK. It is never a summary child, and a spent or summary-owned card is never reused.
  - **After any drop, until the release and seed checks pass,** the writers stay stopped (the old code cannot run on a partial schema) until Sean's recovery and a new complete summary. If the release summary stops at its restart card after those checks passed, restart what is still stopped on a standalone card built from the observed state, with its own fresh OK.
  - **A deploy does not undo the stop**: neither the reservation nor the feed's state is declared in any template, so a release leaves them as they are. Only the restart card and the end reads guard it. The end reads show the feed `Enabled`, no reservation on either function, the dead-letter depth and age against the baseline taken before the stop, and the alarms back to `OK`.
  - Tell Sean before he approves the summary that staging answers errors while the writers are stopped and that alarm emails to the alerts address (the tick-missing, oldest-message and throttle alarms, and 5xx) are expected.
- Each result passes only when the tool's own exact, operation-specific read-back passes: every intended drop (or the explicit empty no-op) and the preserved schema, role and mappings for reset; every statement and the ledger for init; a durable `COMPLETED` record **and** `passed: true` for verify. A zero exit alone is not a pass.
- The first failure, interruption, timeout or failed read-back **spends the card** and stops its sitting summary; a live child is never retried. A timeout can leave partial work or probes: do not run cleanup SQL. Recovery is Sean's; a replacement is a new card, a new summary and a new OK. The card prints its time bounds (the verifier's is fixed); give `run --apply` and `sitting apply` a tool timeout above them.

## Exact sitting summaries
Build all child cards before the write OK, under the existing sitting OK for reads and plan steps. Then run `hous aws sitting card <card>... --task <ticket>`.
Show its complete text, including every child's content, id and immutable-card hash, then ask with the interactive question picker
(SKILL.md "Asking for an OK"; exactly the two choices the summary prints: `Approve <sitting id> <12 hex>` and `Reject`), and, only if he selected the approval label, record that one label verbatim with `hous aws ok`. `Reject`, "Other" or typed text, cancellation or no answer stops: record and apply nothing.
Only `hous aws sitting apply --card <next-child>` may consume that approval, in the sealed order. Direct child apply/run and child OK are refused.
A database refresh runs in one summary, in exactly this order: the writer stop card, `dsql-proof.reset-init`, `dsql-proof.apply-init`, the numbered `apply-step` cards (each its own practice-then-live pair) in chain order, then `dsql-proof.verify-init`. The release change-set cards, the catalogue seed and the writer restart card, in that order, are a second summary built only after verification passes: the restart card is last and is built only after the stop card has run. Inbox setup is not a card and never a child of the summary: it is Sean's separate action in his own terminal, after verification, under its own separate exact OK. Check the step names against the target chain's directories before sealing. A reordered, dropped, added or later-built child is a different summary and needs a new OK.
Run the run sheet's required checks between children; no next card runs merely because the previous card passed.
If any external check fails, or the session ends, close remaining permission with `hous aws sitting stop <sitting-id>`; do not continue.
The ordered ids and whole immutable-card hashes are sealed by a summary SHA-256; `ok` refuses unless recomputation matches
the sealed hash already printed to Sean, then records that same hash, never a replacement computed later.
Apply rechecks it, every child and the existing live-state/pin gates. Child progress, read-back statuses and recorded OK metadata are not approval content.
No stack-name match, template similarity, regenerated card id or post-OK hash can substitute for exact membership.
The first failure (including a read-back), interruption or occupied one-time claim stops all remaining children; the summary cannot resume.
A card built later, a changed card or a replacement after recovery needs a new complete summary and a new OK.
Production is never combined with staging; its verified-staging gate and separate OK remain. Recovery cards keep their own fresh OK.
The existing sitting OK for reads and plan steps remains necessary before AWS planning; its picker choices are `Approve <env> <scope words>` and `Reject`, separate from the write OK's printed `Approve <id> <12 hex>` and `Reject`.
An approved summary is spent when its last listed card has run or at its first failure, and `sitting stop` ends it; no automatic expiry is built.

AWS read and write-plan files must pass `hous aws check` before use. Registration, an eligible recording of the same service/operation
(any request), and shape/consumed-path presence and type determine the verdict; predicate results are information only.
Receipts bind file bytes, checker/mask/shape versions and the recordings actually used. Adding a recording does not invalidate a receipt;
changing or removing a used recording does. The check is offline and does not authorize reads, plan steps or writes.
A first-use read needs a checked-in real-CLI recording from a fake endpoint when no eligible real response exists.
No hand-written response or warning-only bypass substitutes for it.
DSQL schema-step plans are exempt from this receipt gate: `run --card` already performs their offline checks before any AWS call,
including parsing, pairing, input/output checks, dry runs and continuity checks; existing Git provenance checks still apply.

## Sitting checklist
```
Sitting progress:
- [ ] 1. Prepare offline
- [ ] 2. Sean signs in and gives the sitting OK
- [ ] 3. Build every child card of the env before asking for the write OK
- [ ] 4. Show the complete sealed sitting summary, get one write OK, hous aws ok
- [ ] 5. hous aws sitting apply, then read the read-backs (standalone cards still use apply/run)
- [ ] 6. Production: own plan, separate summary/card, own OK
- [ ] 7. Lessons entry
```
1. **Prepare (no AWS).** PRs merged; `hous ci` green; `hous lambda check` for Lambda code; the `build-templates` run for the commit has finished
   (its artifact is `cdk-out-<sha>`); every changed stack has its `expected-changes/<env>/<Stack>.json` from the merged PR; the run sheet is reviewed.
   When the sitting's proof uses synthetic accounts, each of them signs in on staging now (one HTTPS sign-in per account, synthetic data, no OK needed) before Sean is asked to sign in,
   and again after any database refresh, restore or marker change; the accounts file existing proves nothing.
2. **Sign-in and sitting OK.** Sean types `! hous aws login household-<env>` (rule 2) and gives the sitting OK through the interactive question picker (exactly two choices: `Approve <env> <scope words>` and `Reject`) after a full plain-words account of the env, stacks and the read/plan scope; `Reject`, "Other" or typed text, cancellation or no answer stops before `--go`. Read the printed link and code from the output and relay them if he asks. Check with `hous aws login --status household-<env>`, then `hous aws whoami --env <env>`.
3. **Plan.** `hous aws plan <Stack>... --env <env> --task <ticket> --go "<the chosen Approve label, verbatim>"`. Stop on any refusal ([cards-and-stops.md](cards-and-stops.md#when-a-verb-stops)). Fix by PR, then plan again;
   a refused plan deletes the change sets it made. Note the evidence path and the card id.
4. **Summary or standalone card.** For multiple cards, build and paste the complete sitting summary exactly as printed.
   Then ask with the interactive question picker (one question, exactly the summary's two printed choices: `Approve <sitting id> <12 hex>` and `Reject`).
   Only if he selected the approval label: `hous aws ok <sitting-id> --words "<the chosen label, verbatim>"` (if `ok` refuses the label, stop; never reword or replay), and execute each next child with `hous aws sitting apply <sitting-id> --card <next-child> --task <ticket>`.
   One clear reply covers only the exact sealed list once; do not request another reply for each child.
   For one standalone card, paste its text exactly, ask with the picker (exactly the card's two printed choices: `Approve <card id> <12 hex>` and `Reject`), and only if he selected the approval label use `hous aws ok <card-id> --words "<the chosen label, verbatim>"`.
   `Reject`, "Other" or typed text (whatever it says), cancellation or no answer stop: do not invoke `hous aws ok`, pass `--go` or apply. A general go-ahead never counts.
5. **Apply.** For a standalone change-set card, `hous aws apply <card-id> --task <ticket>`; for an owned child, use `sitting apply` as above. It re-checks every change set, runs them in the card's order, watches each stack,
   enables termination protection only on `*_COMPLETE`, then runs the read-backs (`PASS`/`FAIL` lines). Report them to Sean in plain words with times in EDT, no ticket codes, no UTC stamps. Two failures differ:
   - A script or deploy failure stops all later stacks, prints the failed stack's status and failed events, and marks the card failed: go to [failed-stack.md](failed-stack.md).
   - A read-back `FAIL` after every change set ran leaves the card `applied` (spent). Every stack is already deployed, the other read-backs are still run and printed,
     and no status or events are printed: capture them yourself with reads ([failed-stack.md](failed-stack.md) step 1). Do not retry, and write nothing new without a new card and OK.
   A proof round that depends on the worker finishing an earlier round's task ends on that task's own final line (its task id with `delivered` or a terminal code), never on a timer alone; the next round starts only then.
   A log read that runs only on a failure (the raw twin of a verdict read) also includes the worker's error-level lines for the same window (`{ $.level >= 50 }` for household-api's pino logger), so a thrown handler's cause is seen without another OK.
6. **Production.** Only after the same templates are live and verified on staging at the same commit (`plan` checks this and stops otherwise).
   Own `plan`, separate summary or standalone card, own OK; never inherit staging's approval.
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
- `cards/<card-id>.md` and `.json`: standalone cards and sitting summaries; summary bindings hold the ordered child ids, immutable-card hashes and approved summary hash.
- A sitting's one-time claim and per-child ownership records prevent concurrent apply, standalone child execution and reuse after interruption; never remove them to retry.
- `infra/<sha>/` (scratch clone of household-infra at the commit), `artifacts/<sha>/` (the downloaded templates), `hook.log` (every hook deny).
Put the card text, the card id and the `run.json` paths in the sitting notes in the work directory.

## What is in place
- The hook (`hous aws hook check`, installed in Claude Code and Codex) denies AWS writes, `cdk deploy|destroy|bootstrap|rollback|import|refactor|watch|gc`,
  and every `dsql-proof` database writer outside the verb: the launcher (directly or through `sh`/`bash`) or `tools/dsql-proof/src/cli.ts` or `verify-init.ts` (through `node`, `tsx` or a package runner), for `bootstrap`, `proof`, `map-role`, `apply-sql`, `apply-init`, `reset-init`, `cleanup-proof-schema`, `purge-proof-rows`, `write-marker`, live or practice `apply-step` and `seed-catalogue`, and the infra `verify-init`;
  also household-api's verifier (`scripts/verify-dsql/cli.ts`, `npm run verify:dsql`). On a positively identified `dsql-proof` path an unknown verb or option is denied. This full list belongs to the fail-closed hook of the installed housetools that exposes the refresh operations; an older hook names fewer writers, and every writer here stays forbidden regardless.
  The hook also scans position-blind: any other command that names a writer entry point anywhere in its text (a word, a quoted string, an `opt=value` or assignment value) is denied whatever its flags, unless it is one of the read-only forms above through a runtime or wrapper the hook knows. The deny reason says a writer was "named anywhere in a command ... that is not its exact read-only form" (`hook.log` records it as `unknown-runner`); a runtime, wrapper or runner the hook does not know (`bun`, `stdbuf`, `arch`, `hous testers run --`) and anything behind `xargs` earns no read-only form.
  The one exemption is a plain path argument of `cat`, `head`, `tail`, `wc`, `grep`, `rg`, `ls`, `stat`, `file`, `diff`, `cmp`, `shasum`, `sha256sum`, `sed`, `git`, `echo` or `printf`, with no wrapper in front and in no form that runs a command (no `sed -i`, `-f` or `e`/`w`/`W` in a script, no `rg --pre`, `git` only with `-C`, `--no-pager`, `-P`, `--git-dir=`, `--work-tree=` and a known subcommand, never `-c`).
  Because the hook denies any command that names a DSQL writer path outside the read-only forms, put such text in a file (`--body-file`, `git commit -F`) or use the Read tool; `gh pr comment --body` and `orca orchestration send --body` with such a path in the text are denied.
- What the hook allows is read-only forms only, and its acceptance is never an AWS read OK (the sitting OK still applies):
  - no arguments, top-level `--help`/`-h`, and first-argument help for the verbs that implement it (not `bootstrap` or `proof`);
  - `verify-mapping`; `apply-step` or `apply-init` with `--dry-run` or `--verify-only`; `reset-init`, `apply-sql`, `cleanup-proof-schema` or `seed-catalogue` with `--dry-run`; `purge-proof-rows --count` without apply controls; the infra `verify-init`'s offline `--dry-run` or help form;
  - the API verifier only as `--selftest` alone, or both `--local-admin-url=<value>` and `--local-runtime-url=<value>` with nonempty values (through `npm run verify:dsql`, only after a literal `--`); the separate `verify:dsql:selftest` script stays allowed. `--help` or `--dry-run` does not exempt it, and neither does a flag-looking option value, a duplicate or unknown option, or a mixed form.
- Also passing: `hous aws inbox-ownership` (the one named read exception: fixed staging AssumeRole of Household-Staging-InboxOwnershipReader, non-admin DbConnect, fixed `household_inbox_checks.ownership_counts` call; it needs the current sitting OK to name the staging ownership reads, and the hook accepting it is not that OK; the IAM role card and database setup SQL stay separate write OKs; general AssumeRole stays a write), and `hous aws apply|run|sitting` pass (their own approval gates still apply).
- The inbox setup tool, `tools/inbox-ownership/src/setup-cli.ts`, run by `node`, `tsx` or `npx tsx`, is denied in every form except its offline `--show-plan` and `--help`. The live setup is Sean's alone, in his terminal; there is no card or exemption.
- The hook reads words, not the working directory. A visible `dsql-proof` writer inside `bash -c`, `eval` or a similar inline payload is denied, safe flags included; a script whose contents it cannot see (a shell fed on stdin included), or a computed command, is not inspected and is still forbidden. Check it with `hous aws hook status` (`hous doctor` prints it too).
- Built into every verb that contacts AWS: the card OK bound to change-set ARNs, the caller check, explicit profile and region, a clean environment, halt on failure, evidence.
  A schema or refresh card adds: the OK bound to one hash of the whole ordered batch, pinned clean scratch checkouts re-checked before every operation, fresh result paths, the dry runs repeated at apply, and a one-time claim before the first operation that can write. The local-only operations (`ok`, `check`, `sitting card`, `sitting stop`, `hook`, `run --allowlist`) have none of the AWS ones; they leave only the card, the hook settings and `hook.log` records.
- `hous ci` in household-infra runs the `checks` workflow: tests, lint, synth, `cfn-lint` on the templates (`scripts/ci/cfn-lint.sh`, a job-owned venv; never install it yourself), `assert:assembly`, and the static checks: cdk-nag (`test/cdk-nag.test.ts`), the stateful logical-ID lock (`test/stateful-ids.test.ts`, `config/stateful-ids.json`) and the expected-changes schema check (`schemas/expected-changes.v1.json`, `test/expected-changes.test.ts`). `build-templates` makes the artifact `plan` uses.
- The deploy buttons stay as a fallback. Their GitHub actor and OIDC restrictions are unchanged.
