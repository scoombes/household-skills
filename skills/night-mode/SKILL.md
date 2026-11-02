---
name: night-mode
description: Keeps HouseHold coordinators moving while Sean is away under his night-mode rules. Use when Sean turns night mode on, when your brief or coordinator says it is on, or when Sean says he is awake and asks for the report. Applies to an EC, its SCs and PCs, or the SC of a lone story.
---

# Night Mode

Night mode is an overlay on `coordinating-epics` and `orca-pipeline`. Their role boundaries, cross-vendor reviews and merge gates still apply. Coordinators decide and delegate; seats do the work. Sean: "you did stop working the entirety of the night which we need to avoid from now on".

## Turning it on and off

Only Sean turns night mode on; the time never does. If his intent is unclear, it is off: ask in plain text. Sean: "Until you hear from me you remain in night mode no matter the time." When he says he is awake, turn it off and send the report. Record the state in epic-log.md and every new SC or PC brief. Tell running coordinators when it changes, using the routing and confirmation rules in `coordinating-epics`. SCs pass the state to their PCs. Seats never choose the state; their briefs carry the rules their job needs. Overlay states in flight when Sean says he is awake (a PR in its second cycle, a waiting SC beside another SC) continue until Sean directs.

## Who decides

SCs make technical calls inside their story and keep the decisions current in status.md. The EC takes human gates and cross-story decisions that would have gone to Sean, except the waits below. With no EC, the SC acts as top coordinator, keeps epic-log.md as well as status.md, and sends the report. Log options, choice and why as each decision is made. Keep PR heads, merge commits, cycle and round counts current. If coding and other permitted work run out, stop that work and leave the report ready; keep the full-stop notification waiter armed when required below.

## Human gates

Major drift: the EC chooses the best long-term direction, logs it, and launches a PC to update the epic plan. The PC's reviewer must approve the update before affected work proceeds. List every overnight re-plan first in the report.

Plan questions: the EC answers using the long-term rule. The plan marks each answer "EC answer during night mode", with the choice and reason; the report lists it so Sean can reverse it. A PC launched by an SC sends its grouped questions to that SC, which relays them to the EC and returns the answer unchanged. A lone SC answers as EC. The author revises and the plan reviewer re-checks as usual.

A disputed well-formed HIGH stays blocking: treat it as valid and fix it, whether the reviewer, coordinator or implementer disputes it. Use the fix and astra path when needed. Rejecting a proposed fix does not clear the finding.

Cut duplicate and non-plan tests before exceeding the test budget. If still over, the EC may allow it with a logged reason and an entry in the report. The SC relays the request; the exception does not excuse unnecessary tests.

## After three review rounds

Sean: "activate a astra seat to review what is causing the reviews to keeps failing and looking for patterns and come up with a new approach to correct that and move and start a new 3 review cycle from that."

After the first cycle's third round still has blockers, the SC launches one fresh Codex `gpt-6-astra` seat at `high` effort to find the recurring cause and propose an approach. Night mode is Sean's opt-in for this analysis seat, not a frontier fix seat. If the approach stays within the approved plan, use a fresh normal fix seat. If it changes the design, use a scoped PC refresh and its reviewer's approval first; cross-story changes go through the EC. Start one new cycle of at most three rounds, with fresh Claude and Codex panel seats on each new head and the normal evidence and merge gates. Give each reviewer all prior retained findings and coordinator replies from both cycles, the astra report and the approved plan or refresh. The count restarts; the finding history does not.

If blockers remain after the second cycle, park the PR. Do not merge it or start a third cycle. Record the open findings, head, cycles, rounds, worktree and resume point; put it in the report. An SC with a coordinator reports `--outcome failed`, naming the parked PR and cleanup exceptions. A lone SC has no coordinator to send that outcome to; it puts the parked PR, blockers and cleanup exceptions in its report to Sean. Keep its PR worktree; close settled seats under the normal cleanup rules. The EC records the blocked story and dependencies, then moves to the next independent ready story. A parked PR is not a merged dependency or a completed story.

## Follow-ups

After the normal finding-triage rules, the EC may add non-critical follow-ups only under HOUS-590. Read it first, avoid duplicates, record each addition and list it in the report. Everything else waits for Sean. SCs hand items to the EC; a lone SC acts as EC. This does not turn unmet acceptance criteria or blocking findings into follow-ups, or allow follow-up stories to spawn follow-ups.

## Work that waits

Sean: "All aws work waits for me to be at the keyboard so if you run out of coding work you do stop then".

No live AWS commands, including reads, caller checks, plans or writes, run while night mode is on. Continue offline infra code, infra PR merges (a merge deploys nothing), and run-sheet preparation through seats. Local help and `hous aws check` against recorded responses are offline. `cdk synth` is allowed only when it makes no AWS calls; if live context lookup is needed or cannot be ruled out, queue it for Sean. Staging proof API/app HTTPS checks with synthetic data remain allowed under the AWS skill's rule 10; they grant no AWS access. Queue AWS and phone rows for the standing final sitting in [coordinating-epics](../coordinating-epics/SKILL.md#final-sitting). All AWS rules and picker OKs still apply.

Code Owner approval and machine recovery still wait for Sean. Only the affected PRs or seats wait; record them for the report and keep independent work moving. An SC with a coordinator sends it one escalation naming the wait; the EC logs it and applies the scheduling exception. An SC waiting only on Sean stays open under the scheduling exception in `coordinating-epics`; it does not report completion or run a final cleanup while a seat is paused. Never skip a running check or a required approval.

## CRITICAL findings

The top coordinator classifies from the finding's evidence; an SC under an EC sends it up at once and holds as CRITICAL halt says. A broken core flow on main with no data or security harm is contained: only work depending on it or merging into that repo stops. A secret, a security hole, possible data loss or corruption, or anything uncertain is a full stop. Containment is never a downgrade; only Sean may downgrade CRITICAL. Follow [CRITICAL halt](../orca-pipeline/reference/panel-coordinator.md#critical-halt): stop at safe points and launch one fresh read-only Claude investigator in either branch. It reports cause, impact and fix options and does not fix.

Only the top coordinator sends the immediate push through `PushNotification`. If the push is unavailable, failed or not sent, tell Sean directly that push delivery is unavailable, log the failure, and keep the halt; never claim a push was sent. Re-notify at each `check --wait` timeout; during a full stop keep a timed waiter armed, even with no seat running, until Sean responds. Use the direct channel at those checkpoints if push remains unavailable. A contained finding gets one notification and an entry in the report; the affected work waits for Sean's direction in the morning, while unrelated work may continue under the scheduling exception in `coordinating-epics`. Send the investigation to Sean in either case.

## Wake-up report

Sean: "When I wake up I will tell you I am awake and I would like a report of all prs done, how many rounds it took. The decisions you made and anything that stands out as something I would like to know."

Keep it ready from epic-log.md and each story's current status.md; send it when Sean says he is awake. Use plain words; keep the HOUS-590 name when reporting its additions. There is no automatic wake time, timer or default answer.

1. Overnight re-plans, first: direction, reason and approved updated plan.
2. Every merged PR, merge commit and rounds by cycle; in-flight PRs with head, cycle and round; parked PRs and their blockers.
3. Decisions: options, choice and why, including marked EC plan answers, test-budget exceptions and HOUS-590 additions.
4. Incidents, contained CRITICALs, rules broken, coordinator replacements, verification gaps and machine problems; PRs waiting for Code Owner approval, paused seats, and the final-sitting AWS/phone rows.

## Changing this skill

Edit this skill only in the household-skills repo (`skills/night-mode/`), merge it, update `~/household/household-skills` to `origin/main`, then run `hous skills sync` (it reads that checkout, so it installs nothing new until the update). Never edit the installed copy under `~/.agents/skills` or `~/.claude/skills`: sync reports that as a conflict and refuses to run until the change is made in the repo.
