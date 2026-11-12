# hous commands for seats

`hous` commands to use instead of writing your own helper; "Every seat brief includes" in
[SKILL.md](../SKILL.md) points here. Seats read this when the job needs one; the brief gives the installed
path (`~/.agents/skills/orca-pipeline/reference/hous-commands.md`).

Who it is for:
- Every seat: test output, gate logs, mutant lists, `hous pr wait`, `hous test-api`, string catalogs, and the
  exit 74 and 75 meanings.
- household-ios seats only: `--retry-busy`, `--classes`, `--export-attachments`, and the patch-folder
  mutant form (`--write-patches`).
- ios-e2e seats, and any seat whose proof table requires an ios-e2e journey: `hous e2e list|run|cleanup`, and the
  ios-e2e form of `hous test` and `hous ci` below.
- Coordinators only: `hous worker wait --seats <path>`, and `hous worker check` as the waiter. A seat does
  not wait for a reply this way: after an ask it ends its turn and Orca wakes it with the answer, and after
  worker_done it stops and does not keep calling `check`.

`hous help <command>` has the detail; for `hous worker`, `hous pr` and `hous catalog` it is a summary, and
the flags are in `~/household/housetools/docs/commands.md`:
- Waiting (coordinators; a seat does not wait this way): `hous worker check` (your mail, nothing trimmed;
  exit 0 message, 3 none or timeout, 1 error, 74 reader stalled or closed early; without `--peek` it takes
  the mail, so read it directly, never through `head`), `hous worker wait --seats <path>` (seat state, never
  mail).
- Waiting for CI (seats and coordinators): `hous pr wait` (bound to the pushed head: `--head <sha>` or the
  checkout's HEAD).
- Test output: `hous test --summary [args]` and `hous test --failures <log>` (a saved log) show the failures
  and the runner's summary only. `--summary` also works on `hous ci`.
- Gate logs: `--task <ticket> [--suite <name>]` or `--log <dir> [--suite <name>]` on `hous test` and `hous ci`
  write the header and the complete output, and never overwrite.
- household-ios: `--retry-busy [--timeout <s>]` on `hous test` and `hous ci` (retries exit 75 only, bounded);
  on `hous test`, `--classes A,B,C` (named UI classes one at a time, one log each) and `--export-attachments
  <dir>` (named attachments under clean names).
- ios-e2e (on-demand journeys; guide `~/household/housetools/docs/commands.md`, the `hous e2e` section, and
  `tools/e2e/README.md`): `hous test` is its whole local gate (manifest check, host tests, build-for-testing of
  its UI target inside one lease of the shared simulator lock; zero journeys, no runner arguments; exit 75 =
  busy: UI build not started, host tests already ran), logged with `--task <ticket> [--suite <name>]` like any other. `hous ci`,
  `hous sandbox` and `hous test --repeat` are refused for it by name: it has no workflow and no sandbox, and
  its evidence is that gate at the exact head plus the journeys the proof table names. `hous e2e list [--repo
  <ios-e2e>]` (inspect) reads the suites of the committed `e2e.json`. `hous e2e run <suite> [--ios-ref r]
  [--api-ref r] [--e2e-ref r] [--task <ticket>] [--target ios-legacy] [--storekit-contract <abs>]` (local-write,
  with Git network through the App) runs one suite on demand: refs default to `main`, are resolved once to full
  SHAs and must be pushed; `--task` names the ticket for the App-token fetches and the report, it is not a gate
  log. Bounded leases (at most 25 minutes, at least 120 seconds free between), one owned simulator and a
  run-scoped TEST API instance (`ios-e2e-<run-id>`, the runner's alone: `hous test-api down` refuses it, even
  with `--takeover`). Exit 0 all passed and cleaned up; 1 failure; 2 refused before any resource; 75 busy at the
  first acquisition; 129/130/143 interrupted. The report is `~/household/agent-work/ios-e2e/<run-id>/report.json`
  and `report.md` with safe `logs/` and `attachments/`; no `hous` command deletes it. `hous e2e cleanup <run-id>`
  (destructive; one recorded run, only after its runner and every recorded process are gone, from any seat of
  the same user) removes that run's simulator, API instance and scratch and marks an unfinished report
  `incomplete`. Lane rules: [repo-testing.md](repo-testing.md), ios-e2e: journeys on demand.
- Mutant lists: `hous test --ephemeral --mutants <list.json>` runs for household-api, -infra, -web,
  -translations and housetools only. `--check` and `--propose-mutants` also work on household-ios, where
  `--write-patches` turns the list into its patch folder ([repo-testing.md](repo-testing.md), Mutation tools).
  household-site has neither lane.
- Live API: `hous test-api up|status|down` (the seeded TEST API on :3100 on its own disposable database;
  `status` proves who owns the port).
- String catalogs: `hous catalog compare|check|show`.
- Exit 74 from any of these: the output was not all read, so 74 is never a pass; the stderr message names the
  command's own exit code. Exit 75 (household-ios): lock held or load over the limit, and nothing started.
