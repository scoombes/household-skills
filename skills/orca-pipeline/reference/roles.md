# Roles

## Contents
- Coordinators vs seats
- EC, PC, SC compared
- Outputs
- Planning always goes through a PC
- A coordinator caught doing seat work

## Coordinators vs seats
Coordinators (EC, PC, SC) coordinate only: see "Coordinators never do the work" in SKILL.md. Seats
(investigator, plan author, plan reviewer, implementer, fix, baseline check, panel reviewer,
verification) each do exactly one job, defined by their brief, and report only to their coordinator.
Seats never message Sean.

## EC, PC, SC compared

| | EC (Epic Coordinator) | PC (Plan Coordinator) | SC (Story Coordinator) |
|---|---|---|---|
| Scope | Whole epic; holds no story or file detail | One planning pass: the epic plan, or one story's plan refresh | One story, start to finish |
| Started by | Sean | EC or SC | EC, or Sean for a lone story |
| Launches | PCs and SCs only | Plan author + plan reviewer only | PCs, investigator, implementer, baseline check, panel reviewers, fix, verification |
| Decides | Story order; how to group follow-ups for Sean | When the plan is final (reviewer approved, Sean's answers in) | Finding triage (downgrades, fix rejections), merge |
| Never decides | Technical calls inside a story; anything at a human gate (coordinating-epics) | Anything past the final plan | Re-planning of any size (launches a PC); major drift (escalates to EC) |
| Merges PRs | No | No | Yes, its own story's |
| Files Jira tickets | Only follow-ups Sean picks from the end-of-epic list | No | No; hands follow-ups to the EC |
| Reports to | Sean | Its launcher | EC (Sean for a lone story) |
| Supervises | The PCs and SCs it launched | Its plan seats | The seats and PCs it launched |

Every coordinator may message Sean directly. In a CRITICAL halt, only the top coordinator sends the
push notification (panel-coordinator.md, CRITICAL halt).

## Outputs
Every role ends with `worker_done` (seats, PC, SC) or a report to Sean (EC): a 3-sentence summary
(did / found / left) plus the paths below. Plans and reports go in `~/household/epics/<TICKET>/`;
working files and logs go in `~/household/agent-work/<TICKET>/`. Nothing goes in the root of
`~/household`. A role that can't produce its output reports `--outcome failed` with what blocked it;
it never hands back partial work as done.

| Role | Must hand back |
|---|---|
| EC | `epic-log.md`: story order, each story's state, SC/PC run IDs, decisions with Sean's exact words. A verdict-first status to Sean at each story transition. At epic end: the follow-up list (`follow-ups.md`) presented to Sean with recommended grouping, order and importance (non-critical items: suggest HOUS-590), for him to decide (nothing filed unasked); the end-of-epic verification run; a final report; its final cleanup sweep. |
| PC | The final `plan.md`: TDD-ordered tasks, acceptance criteria, new-test estimate, Sean's answers recorded; link to the reviewer's approval. Never open questions or assumptions in place of answers. |
| SC | Merged PR(s) with merge commits. `status.md`, kept current during the story (resume point), and at the end: what shipped, panel rounds, every downgrade/fix rejection with its reason, the end-of-epic verification list. A follow-up list for the EC (not filed). Its final cleanup sweep: ledger empty except named PR worktrees. |
| Investigator | `investigation.md`: root cause (file:line), mechanism, constraints any fix must respect, repro evidence. |
| Plan author | `plan.md` as above, revised until the reviewer approves. |
| Plan reviewer | One verdict per round: APPROVE or CHANGES_NEEDED, findings in the panel finding format. |
| Implementer | Draft PR on a branch rebased onto the tip of `origin/main` before its `hous ci`; red-then-green commits; full `hous ci` logs for the final head; a report with mutation proofs and any declared deviations. |
| Fix | Commits mapped to the findings they close (posted on the PR); the branch rebased onto the tip of `origin/main` before its `hous ci`; full `hous ci` logs for the new head. |
| Baseline check | Per claimed pre-existing failure: reproduced on main or not, with command and output. |
| Panel reviewer | Exactly one verdict comment on the PR (panel-review.md). |
| Verification | PASS/FAIL per listed path on the final head, with evidence. |

## Planning always goes through a PC
All planning (the epic plan, or a story's plan refresh after a drift-check, minor or major) is done
by a PC, never by an EC or SC. The PC runs the plan author (`fable`, high) and a single plan reviewer
(`gpt-6.1-sol`, high) until the reviewer approves.

Questions: the PC collects every question raised by the author, the reviewer or itself into one
grouped list and asks Sean directly, all at once, before finalizing. Answers go into the plan with
Sean's words recorded; the author revises and the reviewer re-checks the affected parts. If an answer
changes scope, the whole plan goes back through the reviewer. Only then does the PC hand back the
final plan. No plan, the epic plan included, waits for Sean's approval.

## A coordinator caught doing seat work
The supervisor acts: the EC for its SCs and PCs, the SC for its PCs, Sean for the EC.

1. **First slip:** tell it at once, naming what it did. It stops and hands that work to a seat. Any
   repo change it made is unreviewed: a seat redoes it or it goes through the normal panel. Never
   merge it as is.
2. **Second slip:** replace it.
   1. Tell it to stop and write `handover.md` in its work dir: story state; what it did itself
      (commits, files, PR changes); open seats with dispatch IDs; its ledger; decisions made and
      pending; next step.
   2. Check the handover against reality (git log, the PR, `hous worker status`, the ledger). Fill
      gaps by reading, not by asking it to keep working.
   3. Release it and close its terminal. Its seats keep running; the new coordinator adopts them.
   4. Launch a fresh coordinator with the handover, the original brief, and this line: "Your
      predecessor was replaced for doing seat work; treat everything it did itself as unreviewed."
   5. Tell Sean who was replaced, why, and where the handover is.
