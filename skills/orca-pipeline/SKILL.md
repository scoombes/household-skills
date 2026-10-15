---
name: orca-pipeline
description: Runs Sean's per-story Orca pipeline (investigate, plan, implement, PR panel review, merge) with his seat model/effort rules, merge gates, panel review rules, cleanup gates and test standard. Use whenever the orchestration skill is engaged or any `orca orchestration` command is about to run; it is the mandatory companion to `orchestration`, which owns the raw command surface.
---

# Orca Pipeline

Load `orchestration` first and run `orca skills get orchestration` for the version-matched commands.
This is the per-story pipeline. For an epic with 2+ stories, load `coordinating-epics` first: one Epic
Coordinator (EC), one Story Coordinator (SC) per story, each SC running this pipeline. Never run a
multi-story epic from one coordinator.

Terms: **coordinator** = EC, PC or SC; this pipeline is run by an SC. **Seat** = one agent launched for
one job (investigator, plan author, plan reviewer, implementer, fix, baseline check, panel reviewer,
verification). **Head** = the PR's current commit sha. **Work dir** = `~/household/agent-work/<TICKET>/`.

Reference files (read when the stage needs them; give seats the path):
- [reference/roles.md](reference/roles.md): EC/PC/SC differences, what every role hands back,
  planning via a PC, replacing a coordinator that does seat work.
- [reference/panel-review.md](reference/panel-review.md): reviewer rules: inputs, evidence, scope,
  severity, finding format. Give reviewers this path.
- [reference/panel-coordinator.md](reference/panel-coordinator.md): outcomes, proportion, disagreement,
  CRITICAL halt, round checklist.
- [reference/test-standard.md](reference/test-standard.md): what makes a test count.

## Coordinators never do the work

A coordinator reads, decides, launches and releases seats, and reports. Everything that produces work
goes to a seat.

A coordinator may:
- Read code, logs, PRs and plans, only enough to brief seats and decide on their results.
- Write its own files: briefs, ledger, `status.md`, handover, decision notes.
- Post decision comments on PRs; mark PRs ready; merge with `hous pr merge` (SC only).
- Message Sean directly.

A coordinator may not:
- Edit, create or delete repo files; commit, push or rebase; edit PR content. Not even one character.
- Run tests or `hous ci` as evidence.
- Write plans, investigations, reviews, findings, packages or docs.
- Take over a stalled or failed seat's work. Launch a fresh seat instead.

Why: work a coordinator does itself skips cross-vendor review, logs, the test budget and the panel,
and it burns the coordinator's context.

Self-check before any tool call that writes outside your own files or runs a test: this is a seat's
job; launch one. Seats never message Sean. Role differences and the drift procedure:
[reference/roles.md](reference/roles.md).

## Every seat brief includes

- Do NOT send heartbeats. Message the coordinator only for ask, escalation or worker_done. Pass this down.
- No `pkill`/`killall`/kill-by-pattern; stop only PIDs you started.
- Use housetools (`hous --help`; guide `~/household/housetools/AGENTS.md`): `hous test|ci|up|down` for
  checks, `hous pr status|wait|merge --task <id>` for merge gates (App token only), `hous worker
  start|status` for seats.
- PRs open as DRAFT. No CI on drafts.
- Any seat that touches AWS, household-infra code, a gate package or DNS: load the `operating-household-aws`
  skill first; it owns every AWS rule.
- Never `pgrep -f` a pattern your own wait loop contains. No plain `git fetch/pull/push/ls-remote/clone`: wrap
  every git network command and every `gh` call in `agent-token run`.
- Implementer and fix seats: follow test-standard.md, and include this log requirement with the real
  path filled in:
  > Before worker_done, run a full `hous ci` on your final commit (every fix round too; a new commit needs
  > new logs). It runs the repo's own GitHub workflows via act, so it matches GitHub CI; never substitute
  > hand-picked suites. Only if `hous ci --list` shows no targets, run every suite the repo documents.
  > Save each run's complete output, untrimmed, to
  > `~/household/agent-work/<TICKET>/logs/<40-hex sha>/<suite>.log`. Start each log with these header lines:
  > `command:`, `commit:`, `exit code:`, `start:`/`end:` (UTC). Post one line per suite on the PR: sha,
  > test count, pass/fail, exit code. Report the log folder path in worker_done.

  Include these two paragraphs verbatim:
  > Keep working until everything the task asks for is done, and only stop to ask when you can't go on
  > without the coordinator or before a risky step. Never end a turn with a summary that announces the
  > next step; do the step. End only with worker_done, an ask, or an escalation.
  >
  > When the work the task asks for is done and checked, stop and report. Don't add features, tests,
  > files, docs or refactors that weren't asked for. If you think one would help, mention it in your
  > report instead of doing it.
- Reviewer seats: follow panel-review.md (give the full path).
- The Orca preamble says not to post to GitHub. If the task needs a push or draft PR, say so explicitly.

## Stages

Copy and track:
```
Story <id>:
- [ ] 1 Investigate -> investigation.md
- [ ] 2 Plan -> plan.md (approved; includes new-test estimate)
- [ ] 3 Implement -> draft PR + logs for head
- [ ] 4 Baseline check (parallel with 3)
- [ ] 5 PR panel -> two APPROVE verdicts on head
- [ ] 6 Untested paths resolved or on end-of-epic list
- [ ] 7 Merge + cleanup sweep
```

One Run, all Tasks created upfront with deps, one shared worktree unless files conflict.

1. **Investigate** (Claude). Live repro with real evidence before discussing fixes. Open every screenshot;
   measure frames; characterize triggers. Output `investigation.md`: root cause (file:line), mechanism,
   constraints any fix must respect.
2. **Plan.** Launch a PC (roles.md, Planning). It runs author `fable` (high) and single reviewer
   `gpt-6.1-sol` (high) until approved, gets Sean's answers to all open questions first, and hands back
   the final `plan.md`: TDD-ordered tasks, acceptance criteria, estimated count of new tests. The author
   verifies the investigation's claims against the code. `fable` here is the one sanctioned frontier use.
3. **Implement** (Claude). Red regression test committed first, then the fix. Plan deviations only when
   empirically forced, documented with red-test evidence. Before the full `hous ci`: `git fetch` and rebase
   onto the tip of `origin/main` (resolve any conflicts and re-run the tests they touch). Then run a full
   `hous ci` on that commit, save logs, push, open a draft PR. Do not merge.
4. **Baseline check** (separate seat). Independently verify every "pre-existing" or "unrelated" failure
   claim against main. A real failure on main is fixed or escalated, never waved through.
5. **PR panel.** Follow [reference/panel-coordinator.md](reference/panel-coordinator.md). Fresh Claude (`opus`,
   high) + fresh Codex (`gpt-6.1-sol`, high/xhigh) reviewer, every round; max 3 rounds.
6. **Untested paths.** A reviewer-flagged untested path blocks merge only while a check for it is queued
   or running; never merge past a running check, and answer "what's the holdup" with that check.
   Otherwise add it to the end-of-epic live-verification list. Live verification runs once per epic: a
   seat exercises each listed path on the final head and reports PASS/FAIL per path.
7. **Merge** when: both APPROVE verdicts are bound to the head, the head contains current main, a full `hous ci`
   passed on it, and no more pushes are planned. Main moved after the approvals: a fresh seat rebases onto
   `origin/main`. With conflicts, it is a fix round (full `hous ci`, new panel round). Without conflicts the
   seat pushes and posts one PR comment whose first line is exactly
   `Rebase carry-over @ <40-hex new head> from <40-hex approved head>`. After several rebases, still name
   the commit both seats approved, not the previous rebase's head: verdicts live only on the approved
   commit, so naming another head carries nothing and the PR waits for a new panel. No new panel and no `hous ci`:
   GitHub CI on the new head is the check. `hous pr merge` carries the approvals only if the change (files,
   added and removed lines) is identical, and otherwise asks for a new panel round. Then mark the PR ready (CI runs now). If it needs
   Sean's Code Owner approval, request it with `hous pr status <n> --task <id> --request-review` (App
   token; never `gh` under Sean's login outside housetools). Merge with
   `hous pr merge <n> --task <id> --delete-branch` once zero checks are pending.
   An infra PR's merge deploys nothing: the AWS change happens in a sitting from a run sheet, with Sean's OK on
   the card.

Docs-only work (prose, guides, evidence) skips this pipeline: one medium-effort Codex reviewer checks
accuracy against raw output, no tests, batched per milestone, agents merge after that approval (Code
Owner paths still need Sean). Code, executable command blocks in runbooks/gate packages, and owned files
get the full pipeline.

## Review rules

- The two-seat panel is used ONLY for the PR merge gate. Every other review (plan, spot check, artifact)
  gets exactly one reviewer.
- Always cross-vendor. Implementers are Claude, so single reviewers are Codex.
- Every round, review or fix, uses brand-new seats. Never reuse a terminal across rounds or roles.

## Seat models

Every launch names model AND effort explicitly; defaults drift. Orca rejects any effort for `haiku`
(1.4.224: "does not support effort"), so Claude seats use `sonnet` at `medium` instead. Confirm what applied
(`hous worker status`, or `launch.effective` in the `worker-start` receipt) before relying on the seat.
State each seat's model+effort in the kickoff status.

| Tier | Claude | Codex |
|---|---|---|
| Frontier (only plan author, or when Sean opts in) | `fable` (claude-fable-5-1) | `gpt-6-astra` |
| Heavy (automated ceiling) | `opus` (claude-opus-5-5) | `gpt-6.1-sol` high/xhigh |
| Balanced | `sonnet` (claude-sonnet-5-5) | `gpt-6.1-sol` low/medium (fallback `gpt-5.6-terra`) |
| Fast | `haiku` (claude-haiku-5-5; not launchable as a seat, see above) | `gpt-6-luna` |

Claude effort (per Anthropic's Opus 5.5 / Sonnet 5.5 guidance):
- Never above `high` for Claude seats. At `xhigh`/`max` Sonnet starts its own review rounds and extra
  fixes, which is the over-engineering this pipeline pushes back on; Opus 5.5 thinks much longer per
  turn there for gains only worth paying for when measured.
- Never `low` for implementer or fix seats: Sonnet at `low` can skip verifying its change.
- `opus`: `medium` by default (Opus 5.5 at medium matches Opus 5 at high); `high` for panel reviewers,
  investigators and hard or long implementation.
- `sonnet`: `medium` for well-specified tasks; `high` for harder or longer ones.

Assignment:
- PR panel reviewers, every round: `opus` (high) + `gpt-6.1-sol` (high/xhigh), regardless of diff size.
- Implementer: `sonnet` (`medium`) when the plan contains the complete code or the fix is single-file
  mechanical; `sonnet` (`high`) for multi-file/integration/debugging or prose specs; `opus` for design judgment.
- Single (non-panel) reviewers: at least balanced tier; scale to diff risk.
- Stuck fix loop: next fix seat goes one tier up (and from `medium` to `high`). At the Claude ceiling
  (`opus` high) use a fresh seat, never higher effort or frontier. Codex seats may raise effort.
- Prefer fewer turns over cheaper tokens: cheap models often take 2-3x the turns on multi-step work.
- Codex: `gpt-6.1-sol` defaults to LOW effort, so always pass it. Efforts: low, medium, high, xhigh, max,
  ultra (luna tops at max). Never use hidden slugs (`gpt-reserve`, `codex-auto-review`). If the lineup
  looks stale, re-check `~/.codex/models_cache.json`.

## Launch mechanics

- Default: `hous worker start --run <run> --task <task>|--spec <text> --worktree <selector> --agent
  <claude|codex> --model <id> --effort <level> --role <role> --name <name> --seats <abs path outside any
  repo>`. It wraps `orca orchestration worker-start`, records the seat, verifies what was applied, and
  handles the Codex update prompt. `hous worker status --seats <path>` reconciles seats with Orca. Run
  `hous help worker start` for flags.
- It needs an existing worktree. To make one: `orca worktree create --name <n> --repo name:<repo>
  --base-branch main --no-parent`, then pass `--worktree path:<path>` (record it in the ledger).
- Custom argv only when `hous worker start` can't express it:
  - Codex: `codex --yolo -c mcp_servers={} -m <model> -c model_reasoning_effort="<effort>"`. Stuck on
    "Starting MCP servers" for 2-3 min = dead: `terminal close`, relaunch.
  - Claude: `claude --dangerously-skip-permissions --model <alias> --effort <level>`. Fresh worktrees may
    show a folder-trust prompt; send `--enter`.
  - Sequence: `terminal create` -> `terminal wait --for tui-idle` -> read the screen and confirm the home
    prompt (input during startup is silently dropped) -> `dispatch --inject` -> read back to confirm the
    task landed. On these seats `worker-release`/`worker-read` return `dispatch_not_found`; use
    `terminal close`/`terminal read`.

## Coordinator loop

- Wait with `check --wait --types worker_done,escalation,question` via Bash `run_in_background` only. A
  `&`-detached waiter never notifies; two waiters conflict. A timeout is a checkpoint: run the cleanup
  sweep, re-arm, and peek `worker-read` or git state if unsure.
- `send --to dispatch:` mail is read only at the seat's next check. Put decision-critical input in a new
  dispatch, or `terminal send` to a mid-turn Claude.
- Status to Sean at each stage transition: brief, verdict first, plain words (no internal codes), times
  in EDT.

## Cleanup (gates, not reminders)

Clean up only what you created. Never touch a worktree or terminal you didn't create: Sean leaves some
open on purpose.

- **Ledger.** Record every worktree and terminal you create in `<work dir>/ledger.md` when you create
  it. The ledger is the only list you may clean.
- **Settlement includes cleanup.** For each settled seat: `worker-release`; if it returns
  retained/user_takeover, read the screen, confirm idle, `terminal close`; confirm the terminal is gone;
  then `check --ack <id>`. Not confirmed closed = not processed.
- **Sweep before every launch and at every wait timeout:** `terminal list --worktree <wt>` for each
  ledger worktree; close every idle ledger terminal with no running dispatch.
- **Remove ledger worktrees when their job ends:** the PR worktree when its PR merges. Panel reviewers
  are read-only and run in the PR worktree; they need no worktree of their own. Never remove one with uncommitted or unpushed changes; report
  it instead.
- **Never close a working seat.** Read the screen first.
- **Don't finish with open items.** An SC's `worker_done` includes its final sweep showing every ledger
  item closed or removed, except named worktrees kept for an open PR. The EC checks this before launching
  the next SC, then removes the SC's own workspace.
- **Not on your ledger** (Sean's, or left by a dead coordinator): leave it, and mention it once in your
  next status to Sean.

## Artifacts

- Work dir (`~/household/agent-work/<TICKET>/`): briefs, `ledger.md`, `status.md`, `investigation.md`,
  `panel-*.md`, `verification.md`, `logs/<sha>/`, opened-screenshot folders. Every seat can reach it.
- `~/household/epics/<TICKET>/`: `plan.md` and final reports.
- Never in the root of `~/household`. Never commit throwaway probe UITests; keep copies next to the
  reports. Durable follow-ups go into PR bodies and the follow-up list before the story ends.

## iOS simulator host limit

Around 15+ consecutive xcodebuild/simulator cycles leak CoreSimulatorService processes (load avg 400+).
Watch `uptime` on sim-heavy runs. Recovery needs `killall CoreSimulatorService`, which agents may not
run: pause the seat between runs and escalate to Sean; never interrupt a running xcodebuild. Treat odd
simulator/launch errors right after recovery as environmental, not code defects.
