# hous commands for seats

`hous` commands to use instead of writing your own helper; "Every seat brief includes" in
[SKILL.md](../SKILL.md) points here. Seats read this when the job needs one; the brief gives the installed
path (`~/.agents/skills/orca-pipeline/reference/hous-commands.md`).

Who it is for:
- Every seat: test output, gate logs, mutant lists, `hous pr wait`, `hous test-api`, string catalogs, and the
  exit 74 and 75 meanings.
- household-ios seats only: `--retry-busy`, `--classes`, `--export-attachments`, and the patch-folder
  mutant form (`--write-patches`).
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
- Mutant lists: `hous test --ephemeral --mutants <list.json>` runs for household-api, -infra, -web,
  -translations and housetools only. `--check` and `--propose-mutants` also work on household-ios, where
  `--write-patches` turns the list into its patch folder ([repo-testing.md](repo-testing.md), Mutation tools).
  household-site has neither lane.
- Live API: `hous test-api up|status|down` (the seeded TEST API on :3100 on its own disposable database;
  `status` proves who owns the port).
- String catalogs: `hous catalog compare|check|show`.
- Exit 74 from any of these: the output was not all read, so 74 is never a pass; the stderr message names the
  command's own exit code. Exit 75 (household-ios): lock held or load over the limit, and nothing started.
