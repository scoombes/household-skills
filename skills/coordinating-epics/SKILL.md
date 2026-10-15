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
[reference/roles.md](../orca-pipeline/reference/roles.md). All coordinators coordinate only; see
"Coordinators never do the work" in orca-pipeline.

## The EC loop

Copy and track:
```
Epic <id>:
- [ ] Read every story's shape (titles, deps, acceptance), not their guts
- [ ] Epic plan: launch a PC (unless an approved epic plan exists); reviewer approval makes it final
- [ ] Order stories from the dependency graph
- [ ] Per story: launch SC -> SC done -> record follow-ups -> tear SC down -> next story
- [ ] End-of-epic live verification (orca-pipeline stage 6 list)
- [ ] Present the follow-up list to Sean (he decides; file nothing unasked); final report; final sweep
```

1. Read every story for its shape, not its implementation detail.
2. No approved epic plan? Launch a PC (roles.md, Planning). Its reviewer-approved plan is final; Sean
   has no plan-approval step.
3. Set the order from the dependency graph.
4. The approved epic plan is the go-ahead for the whole epic. Start each story as soon as
   its dependencies are merged; don't ask per story. Goal: the epic runs story to story to completion
   without Sean, stopping only at a human gate (below).
5. Run **one SC at a time** unless Sean says otherwise. Each SC gets its own workspace (SC
   placement).
6. On an SC's done: add its follow-ups to the follow-up list in `epic-log.md`, tear the SC down
   (Cleanup), and launch the next ready story in the same turn. Never end a turn on "advancing next" without doing it.
7. All stories merged: run the end-of-epic verification, then report to Sean with the follow-up list
   for him to decide on (Follow-ups).

Give Sean a verdict-first status at each story transition, naming each seat's model and effort. It
informs; don't wait for a reply.

**Human gates** (stop the affected work and ask Sean; everything else proceeds):
- Major drift (below).
- A CRITICAL finding (panel-coordinator.md, CRITICAL halt).
- Any AWS deploy or AWS write, staging included: the gate is Sean's OK on the approval card of a sitting (agents then run
  `hous aws apply|run`). Packages READY before the switch (`hous help aws apply` succeeding) finish with `hous pkg run`,
  including the writes their Waivers section names; nothing new uses that path. Every other step only Sean can do
  (`operating-household-aws` skill).
- A blocking finding still open after 3 panel rounds, a disputed well-formed HIGH, or a test budget
  overrun (panel-coordinator.md).
- A plan question only Sean can answer (the PC asks him before finalizing).
Watch host load (`uptime`). If it thrashes, pause new launches and tell Sean.

## Planning: the full epic up front
The PC plans every story to acceptance criteria in one epic plan, with all of Sean's questions
answered before it is final. The reviewer's approval makes it final; Sean answers questions, he does
not approve the plan.
- Never defer a story's planning to its start, and never replace the epic plan with skeletons to
  plan later; that hides the epic's real shape from the plan review.
- Staleness is handled by each SC's drift-check, against the existing plan.

## Story Coordinator
1. Read the epic and its own story.
2. **Drift-check** its slice of the epic plan against merged reality:

   | Drift found | Action |
   |---|---|
   | Minor: affects this story only | SC launches a PC to refresh its slice, then proceeds |
   | Major: changes other stories' assumptions or the order | Escalate to the EC; don't resolve it |

3. Run the story through `orca-pipeline`.
4. Report done to the EC with the SC outputs in roles.md: merged PRs, `status.md`, the follow-up
   list (never filed in Jira by the SC), and its final cleanup sweep.

## Major drift: the one hard human gate
Re-planning mid-epic is Sean's call, not the EC's:
SC escalates major drift → EC alerts Sean with directional options and waits → Sean picks a direction
→ EC launches a PC to update the epic plan → the next SC proceeds.

Red flag: "I'll just update the contract and mark the downstream stories stale." That is a silent
re-plan. Stop and alert Sean.

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
  cleans those up, and the EC removes the SC's workspace at close-out.

## EC → SC messaging
Once an SC binds its own sub-worker Run, mail to `dispatch:<SC dispatch>` on the EC's Run never
reaches it (`consumer_fenced`); only replies to its own asks get through.
- At launch, have the SC report its sub-run ID in its first ask; record it in `epic-log.md`.
- Send follow-ups to `run:<SC sub-run>`, or answer through its pending ask.
- For anything urgent, also `orca terminal send` a one-line pointer into the SC's terminal.
- Never assume a mailed decision was applied; ask the SC to confirm it.

## Follow-ups
Nobody in the pipeline files follow-up tickets on Sean's behalf: not the SC, not the EC.
- **Track:** SCs return follow-ups to the EC. The EC keeps one running `## Follow-ups` list in
  `epic-log.md`: each item, the story/PR it came from, and why it matters. Challenge each list first:
  a gap in the story's own scope gets fixed in that story, not listed.
- **Present:** when the last story is completed, the EC merges like items and writes
  `~/household/epics/<EPIC>/follow-ups.md`, then gives Sean the list in plain words in its final
  report: each item, where it came from, a recommended importance, and a recommendation (one grouped
  ticket, fold into an existing ticket, add under HOUS-590, or drop). Default proposal: as few
  tickets as possible.
- **HOUS-590** is Sean's standing epic for non-critical follow-ups (flaky tests, small fixes,
  reviewer notes). Before presenting, read it so the list doesn't repeat items already there
  (mark those "already in HOUS-590"). For a non-critical item, suggest "add under HOUS-590" by
  name; Sean shouldn't have to remember the epic. Never add to it without his OK.
- **Decide:** Sean decides. The EC files, folds or drops only what he says, records his exact words in
  `epic-log.md`, and files nothing he hasn't picked.
- A genuinely blocking find goes to Sean right away (still as a question, not a ticket); everything
  else waits for the end-of-epic list.

## Cleanup
- **SC:** its own ledger (orca-pipeline, Cleanup).
- **EC:** release each finished SC's and PC's seat (they can't release themselves after reporting
  done), remove the SC's workspace (`worktree rm`; release alone leaves it), and run a final sweep.
