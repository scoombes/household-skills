---
name: coordinating-epics
description: Runs a multi-story epic to completion with Orca orchestration through a lean Epic Coordinator (EC) that plans via a Plan Coordinator (PC) and runs one disposable Story Coordinator (SC) per story, so no coordinator exhausts its context. Use for an epic with 2+ linked stories to plan, order and merge, or when a coordinator is running low on context mid-epic. Not for a single story (use orca-pipeline directly).
---

# Coordinating Epics

An epic is run by a lean **EC** that never holds story detail. Planning goes to a **PC**; each story
goes to a disposable **SC** that runs `orca-pipeline`. Splitting the levels is what stops a
coordinator running out of context mid-epic.

Load `orchestration` (command surface) and `orca-pipeline` (per-story rules) first. Role
differences, what each role hands back, and the coordinator-drift procedure are in orca-pipeline's
[reference/roles.md](../orca-pipeline/reference/roles.md) (`~/.agents/skills/orca-pipeline/reference/roles.md`). All coordinators coordinate only; see
"Coordinators never do the work" in orca-pipeline.

When Sean or the brief says night mode is on, load `night-mode`. Record on/off in epic-log.md and every
new SC or PC brief. Every new SC or PC brief also names the testing policy version (testing-guidance-v1 or a
pinned older snapshot) and any active transition ([repo-testing.md](../orca-pipeline/reference/repo-testing.md), Adoption and transition); testing
budgets and time allowances live there and in orca-pipeline, not here.

## The EC loop

Copy and track:
```
Epic <id>:
- [ ] Read every story's shape (titles, deps, acceptance), not their guts
- [ ] Epic plan: launch a PC (unless an approved epic plan exists); reviewer approval makes it final
- [ ] Order stories from the dependency graph
- [ ] Per story: launch SC -> SC done -> record follow-ups, staging/phone rows -> tear SC down (a night-mode SC waiting only on Sean stays open) -> next story
- [ ] End-of-epic live verification (orca-pipeline stage 6 list)
- [ ] Final sitting with Sean at the keyboard (Final sitting); AWS/phone rows settled
- [ ] Present the follow-up list (day: Sean decides; night: only the HOUS-590 exception); final report; final sweep
```

1. Read every story for its shape, not its implementation detail.
2. No approved epic plan? Launch a PC (roles.md, Planning). Its reviewer-approved plan is final; Sean
   has no plan-approval step.
3. Set the order from the dependency graph.
4. The approved epic plan is the go-ahead for the whole epic. Start each story as soon as
   its dependencies are merged; don't ask per story. Goal: the epic runs story to story to completion
   without Sean, stopping only at a human gate, with the explicit `night-mode` outcomes when it is on (below).
5. Run **one SC at a time** unless Sean says otherwise. Each SC gets its own workspace (SC
   placement). In `night-mode`, an SC waiting only on Sean (Code Owner approval, machine recovery, a
   contained CRITICAL) stays open and does not count toward one SC at a time (the scheduling exception); the EC may launch the next
   independent ready story whose dependencies are merged. Keep the waiting SC and paused seats on the
   ledger; do not force settlement or final cleanup. Full stops, the stopped scope of a contained CRITICAL,
   and host-wide recovery pauses still stop launches in their scope. A new SC's brief names any stopped
   scope.
6. On an SC's done: add its follow-ups to the follow-up list in `epic-log.md` and its staging/phone rows to
   the final-sitting list, tear the SC down (Cleanup), and launch the next ready story in the same turn.
   Never end a turn on "advancing next" without doing it. In `night-mode`, a waiting SC is not done: the
   same exception applies to the next-story rule. On a parked PR's failed settlement, record the blockers,
   dependencies and retained worktree, release settled seats, and advance only an independent ready story;
   it is not a done story.
7. All stories merged: run the end-of-epic verification and the final sitting (Final sitting), then report
   to Sean with the follow-up list for him to decide on (Follow-ups). Unfinished stories and open
   sitting/verification rows keep the epic incomplete. Missing critical proof (orca-pipeline stage 6) is a
   release blocker: the epic stays incomplete and the blocker goes to Sean; noncritical local gaps stay on
   the verification list. The final report includes the evidence paths and open retention dates the SCs
   handed back.

Give Sean a verdict-first status at each story transition, naming each seat's model and effort. It
informs; don't wait for a reply.

**Human gates.** With night mode off, stop the affected work and ask Sean; everything else proceeds. With it
on, load `night-mode` for each gate's outcome. AWS reads wait too; Code Owner approval and machine recovery
still require Sean.
- Major drift (below).
- A CRITICAL finding (panel-coordinator.md, CRITICAL halt).
- Any AWS deploy or AWS write, staging included: the gate is Sean's OK on the approval card of a sitting (agents then run
  `hous aws apply|run`). Packages READY before the switch (`hous help aws apply` succeeding) finish with `hous pkg run`,
  including the writes their Waivers section names; nothing new uses that path. Every other step only Sean can do
  (`operating-household-aws` skill).
- A blocking finding still open after 3 panel rounds, a disputed well-formed HIGH, a test budget
  overrun, or a second overrun of the fix-seat time checkpoint (panel-coordinator.md; the checkpoint is
  in repo-testing.md).
- A plan question only Sean can answer (the PC asks him before finalizing): Sean's answer by day; a marked
  EC answer under `night-mode`.
AWS and phone work is queued for the Final sitting; it does not block independent code/infra PRs whose
acceptance and merge gates are met.
Watch host load (`uptime`). If it thrashes, pause new launches and tell Sean.

## Planning: the full epic up front
The PC plans every story to acceptance criteria in one epic plan, with all of Sean's questions
answered before it is final (Sean's answers by day; marked EC answers under `night-mode`). The reviewer's
approval makes it final; Sean answers questions, he does not approve the plan.
- Never defer a story's planning to its start, and never replace the epic plan with skeletons to
  plan later; that hides the epic's real shape from the plan review.
- Staleness is handled by each SC's drift-check, against the existing plan.
- The epic plan names the shared version and compatibility set (for example the translations, API and
  client SHAs that stories must agree on) and the owner of each pending verification row. Each story's
  slice carries its risk-to-layer table and counts (orca-pipeline stage 2).

## Story Coordinator
1. Read the epic and its own story.
2. **Drift-check** its slice of the epic plan against merged reality:

   | Drift found | Action |
   |---|---|
   | Minor: affects this story only | SC launches a PC to refresh its slice, then proceeds |
   | Major: changes other stories' assumptions or the order | Escalate to the EC; don't resolve it |

3. Run the story through `orca-pipeline`.
4. Report done to the EC with the SC outputs in roles.md: merged PRs, `status.md`, the follow-up
   list (never filed in Jira by the SC, except a lone SC using the `night-mode` HOUS-590 exception), its
   staging/phone rows, and its final cleanup sweep.

## Major drift
With night mode off, re-planning mid-epic is Sean's call, not the EC's:
SC escalates major drift → EC alerts Sean with directional options and waits → Sean picks a direction
→ EC launches a PC to update the epic plan → the next SC proceeds. SCs still escalate cross-story drift
and never write the update themselves.

Red flag, with night mode off: "I'll just update the contract and mark the downstream stories stale." That
is a silent re-plan. Stop and alert Sean.

At night the EC selects and logs the direction, then launches a PC for the update and its reviewer's
approval. Put that re-plan first in the wake-up report.

## SC placement
Every SC gets its own Orca workspace, never the EC's (`--worktree current` from the EC is wrong).
- Workspace name includes the story number, e.g. `hous-446` (branch `scoombes/hous-446-<slug>`), on
  the story's primary repo. A slice adds a suffix: `hous-448-g6`.
- Create the workspace first, then launch into it:
  `orca worktree create --name hous-xxx --repo name:<repo> --base-branch main --no-parent --setup skip --json`,
  then `hous worker start ... --worktree path:<worktree path> --agent claude --model <m> --effort <e>`.
- Claude Code overwrites the tab title, so `--display-name` and `terminal rename` don't stick. After
  launch, send `/rename SC:HOUS-xxx` (`orca terminal send --terminal <handle> --text "/rename SC:HOUS-xxx" --enter`)
  and confirm with `orca terminal show`. A slice is `SC:HOUS-448-G6`.
- The SC's seats run in that workspace (or further story-numbered worktrees it creates); the SC
  cleans those up, and the EC removes the SC's workspace at close-out, except a retained parked PR's.

## EC → SC and PC messaging
Once an SC binds its own sub-worker Run, mail to `dispatch:<SC dispatch>` on the EC's Run never
reaches it (`consumer_fenced`); only replies to its own asks get through.
- At launch, have the SC report its sub-run ID in its first ask; record it in `epic-log.md`. Every SC and
  PC reports its sub-run ID in its first ask to its launcher; record it.
- Send follow-ups to `run:<SC sub-run>`, or answer through its pending ask.
- For anything urgent, also `orca terminal send` a one-line pointer into the SC's terminal.
- Never assume a mailed decision was applied; ask the SC to confirm it.
- On each `night-mode` on/off change, send to `run:<sub-run>` plus a one-line terminal pointer and require
  confirmation. The EC does this for its SCs and directly launched PCs; SCs do the same for their PCs and
  confirm propagation. If the sub-run is unknown, use a pending ask to obtain it and convey state. With no
  sub-run and no pending ask, the terminal pointer carries the state and requests the ID. Require
  confirmation either way; never assume delivery. A new coordinator's brief carries state before work
  starts. Record any actual delivery refusal.

## Final sitting
SCs hand staging and phone rows to the EC with the action, dependencies, required proof and what needs
Sean. Keep one final-sitting list alongside the separate end-of-epic verification list; cross-reference
shared rows so they run once. Each row also carries its owner, reason, exact version set and
PASS/FAIL/PENDING status. After the code stories merge, the EC coordinates one sitting with Sean at the
keyboard through a dedicated SC. That SC launches seats to consolidate the run sheet, obtain its normal
cross-vendor review and execute the approved work. The EC never writes the run sheet or runs commands
itself. A lone-story SC coordinates its own final sitting through seats.

A coordinator who can reach Sean presents the complete AWS cards or summary and gets the AWS skill's exact
picker replies, relaying them unchanged to the execution seat. A dispatched seat never uses a local picker
to ask Sean. If the top coordinator cannot reach him through a picker, AWS waits until he is present in a
session that can. Existing sign-in, approval, caller, stop-on-failure and Sean-only operations still apply;
one sitting is not one blanket permission. Overnight, prepare offline and queue these rows for wake-up. Run
permitted non-AWS verification without him; phone rows that need him wait. Keep pending rows explicit and
do not declare the epic verified or complete until they are settled.

## Follow-ups
Nobody in the pipeline files follow-up tickets on Sean's behalf: not the SC, not the EC, except the EC's
explicit `night-mode` HOUS-590 exception.

A follow-up story produces no follow-ups. Every finding, including panel LOW/NIT findings, is fixed
within that story's scope or dropped with a one-line reason in its `status.md` (for a UX residual the reason
cannot be "cheap to leave"); do not add it to the end-of-epic follow-up list. The remaining rules below apply to stories that are not follow-up stories.
- **Track:** SCs return follow-ups to the EC. The EC keeps one running `## Follow-ups` list in
  `epic-log.md`: each item, the story/PR it came from, and why it matters. Challenge each list first:
  an unmet story acceptance criterion gets fixed in that story, not listed.
  For review findings, a required fix round addresses all retained items from both reviewers,
  including non-blocking ones. SCs collect remaining non-blocking review follow-ups only after both
  panel reviewers APPROVE; those items do not trigger another round. See
  [panel coordinator rules](../orca-pipeline/reference/panel-coordinator.md) for finding triage.
- **Present:** when the last story is completed, the EC merges like items and writes
  `~/household/epics/<EPIC>/follow-ups.md`, then gives Sean the list in plain words in its final
  report: each item, where it came from, a recommended importance, and a recommendation (one grouped
  ticket, fold into an existing ticket, add under HOUS-590, or drop). Default proposal: as few
  tickets as possible.
- **HOUS-590** is Sean's standing epic for non-critical follow-ups (flaky tests, small fixes,
  reviewer notes). Before presenting, read it so the list doesn't repeat items already there
  (mark those "already in HOUS-590"). For a non-critical item, suggest "add under HOUS-590" by
  name; Sean shouldn't have to remember the epic. Never add without his OK except non-critical additions
  under `night-mode`, after reading it to avoid duplicates; log and report each addition.
- **Decide:** Sean decides. The EC files, folds or drops only what he says (except the `night-mode`
  HOUS-590 exception), records his exact words in `epic-log.md`, and files nothing he hasn't picked (same
  exception).
- With night mode off, a genuinely blocking find goes to Sean right away (still as a question, not a
  ticket). At night, route it through `night-mode`; it never becomes a follow-up ticket. Everything
  else waits for the end-of-epic list.

## Cleanup
- **SC:** its own ledger (orca-pipeline, Cleanup).
- **EC:** release each finished SC's and PC's seat (they can't release themselves after reporting
  done), remove the SC's workspace (`git status --ignored` first, as in orca-pipeline's Cleanup; then
  `worktree rm`; release alone leaves it) except a retained parked PR's, and run a final sweep. An SC waiting only on Sean under `night-mode` is not finished: do not release it.

## Changing this skill

Edit this skill only in the household-skills repo (`skills/coordinating-epics/`), merge it, update `~/household/household-skills` to `origin/main`, then run `hous skills sync` (it reads that checkout, so it installs nothing new until the update). Never edit the installed copy under `~/.agents/skills` or `~/.claude/skills`: sync reports that as a conflict and refuses to run until the change is made in the repo.
