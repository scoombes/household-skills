# Roles

## Contents
- Coordinators vs seats
- EC, PC, SC compared
- Outputs
- Planning always goes through a PC
- A coordinator caught doing seat work

## Coordinators vs seats
Coordinators (EC, PC, SC) coordinate only: see "Coordinators never do the work" in SKILL.md. Seats
(investigator, plan author, plan reviewer, implementer, fix, astra analysis, baseline check, panel
reviewer, verification) each do exactly one job, defined by their brief, and report only to their coordinator.
Seats never message Sean.

## EC, PC, SC compared

| | EC (Epic Coordinator) | PC (Plan Coordinator) | SC (Story Coordinator) |
|---|---|---|---|
| Scope | Whole epic; holds no story or file detail | One planning pass: the epic plan, or one story's plan refresh | One story, start to finish |
| Started by | Sean | EC or SC | EC, or Sean for a lone story |
| Launches | PCs and SCs only | Plan author + plan reviewer only | PCs, investigator, implementer, baseline check, panel reviewers, fix, astra analysis, verification |
| Decides | Story order; how to group follow-ups for Sean; at night, human gates and cross-story decisions per `night-mode` | When the plan is final (reviewer approved; Sean's answers, or marked EC answers in `night-mode`, are in) | Technical calls inside its story; at night logs them as made; finding triage (downgrades, fix rejections), merge |
| Never decides | Technical calls inside a story; human gates except the explicit `night-mode` cases (coordinating-epics) | Anything past the final plan | Re-planning of any size (launches a PC); major drift (escalates to EC). A lone SC acts as EC under `night-mode`, but still delegates all planning and seat work |
| Merges PRs | No | No | Yes, its own story's |
| Files Jira tickets | Only follow-ups Sean picks from the end-of-epic list; at night, the explicit HOUS-590 exception in `night-mode` | No | No; hands follow-ups to the EC, or Sean for a standalone story (a lone SC acting as EC under `night-mode` uses the same HOUS-590 exception) |
| Reports to | Sean | Its launcher | EC (Sean for a lone story) |
| Supervises | The PCs and SCs it launched | Its plan seats | The seats and PCs it launched |

Every coordinator may message Sean directly. With night mode off, ask Sean in plain text with a
recommendation. There is no timer or default answer; unrelated seats keep working. Use a blocking picker
only when he is clearly at the keyboard. AWS OKs always use the picker required by
`operating-household-aws`; ordinary plain-text replies never authorize AWS. In a sitting, only the SC
that owns it asks Sean everything about the sitting, in its own tab: a picker for every OK and choice
(free-text step reports are asked there too); no other coordinator
shows or relays a sitting picker (`operating-household-aws` rule 11). Seats never ask Sean directly.
For the overnight question route, see `night-mode`. In a CRITICAL halt, only the top coordinator
sends the push notification (panel-coordinator.md, CRITICAL halt).

The EC launches a dedicated final-sitting SC; that SC delegates run-sheet preparation, review and
execution to seats and puts every sitting question to Sean itself (coordinating-epics, Final sitting).
This is not a new EC seat-launch permission. A lone SC uses its own seats. The drift and replacement rules below still apply; overnight replacements
and handovers are reported at wake-up, not silently waived.

## Outputs
Every role ends with `worker_done` (seats, PC, SC) or a report to Sean (EC): a 3-sentence summary
(did / found / left) plus the paths below. Plans and reports go in `~/household/epics/<TICKET>/`;
working files and logs go in `~/household/agent-work/<TICKET>/`. Nothing goes in the root of
`~/household`. A role that can't produce its output reports `--outcome failed` with what blocked it;
it never hands back partial work as done.

| Role | Must hand back |
|---|---|
| EC | `epic-log.md`: story order, each story's state, SC/PC run IDs, the current `night-mode` on/off state, decisions with Sean's exact words, and, separately, overnight options, choice and why. A verdict-first status to Sean at each story transition. At epic end: the SC-handed evidence paths and open retention dates in the final report; the follow-up list (`follow-ups.md`) presented to Sean with recommended grouping, order and importance (non-critical items: suggest HOUS-590), for him to decide (nothing filed unasked except the `night-mode` HOUS-590 exception); the end-of-epic verification run; the final-sitting list and, under `night-mode`, the wake-up report; a final report; its final cleanup sweep. |
| PC | The final `plan.md`: TDD-ordered tasks, acceptance criteria, the risk-to-layer table and the separate counts (lower-layer changed rules, UI/live scenarios) with the new-test estimate (test-standard.md item 1, repo-testing.md), Sean's answers recorded, or explicitly marked EC answers during night mode; link to the reviewer's approval. Never open questions or assumptions in place of answers. |
| SC | Merged PR(s) with merge commits. `status.md`, kept current during the story (resume point, PR heads and merge commits, cycles and rounds, technical decisions as they happen, staging/phone rows), and at the end: what shipped, panel rounds, every downgrade/fix rejection with its reason, the end-of-epic verification list. Under `night-mode`, a lone SC also keeps `epic-log.md` for the `night-mode` state and top-level decisions. Its evidence hand-back: safe evidence paths, unresolved release rows and minimum-retention dates, from `status.md`, to the EC, or to Sean for a lone story. A follow-up list for the EC, or Sean for a standalone story (not filed, except a lone SC using the `night-mode` HOUS-590 exception): review items are collected only after both panel reviewers APPROVE, after including all retained items in any required fix round. Its final cleanup sweep: ledger empty except named PR worktrees. A parked PR is incomplete: an SC with a coordinator reports `--outcome failed` with its blockers, resume point and retained PR worktree; a lone SC puts those details in its report to Sean. Never report it as merged. The EC may advance only independent stories whose dependencies are merged. |
| Investigator | `investigation.md`: root cause (file:line), mechanism, constraints any fix must respect, repro evidence. |
| Plan author | `plan.md` as above, revised until the reviewer approves. |
| Plan reviewer | One verdict per round: APPROVE or CHANGES_NEEDED, findings in the panel finding format. Checks design and fidelity: each risk's layer, real caller, UI/live justification and counts. |
| Implementer | Draft PR on a branch rebased onto the tip of `origin/main`; for an actual defect, red-then-green commits; the lane's full-gate logs for the final head (GitHub CI with its `hous pr logs` folder; translations, site and housetools: the local full-gate logs, repo-testing.md); a report with the relevant red/green evidence, protected-rule proof and selective controls (never a quota), required-selector outcomes and any declared deviations. |
| Astra analysis | Launched by the SC only after the first three-round cycle still blocks in `night-mode`, fresh Codex `gpt-6-astra`, high effort. Returns `astra-analysis.md` in the work dir: recurring causes linked to findings, proposed approach, and whether it changes the approved design. No repo edits or fixes. The SC sends design changes through a PC, otherwise to a fresh normal fix seat; both paths return to the panel. |
| Fix | Commits mapped to the findings they close (posted on the PR); the branch rebased onto the tip of `origin/main` and pushed; the lane's full-gate logs for the new head (as for the implementer); proof only for the changed risk or demonstrated gap, with its red/green or control evidence. |
| Baseline check | Per claimed pre-existing failure: reproduced on main or not, with command and output. |
| Panel reviewer | Exactly one verdict comment on the PR (panel-review.md). |
| Verification | PASS/FAIL per listed path on the final head, with evidence. |

## Planning always goes through a PC
All planning (the epic plan, or a story's plan refresh after a drift-check, minor or major) is done
by a PC, never by an EC or SC. The PC runs the plan author `gpt-6-astra` (explicit effort, high unless the
coordinator says otherwise) and a single plan reviewer (`opus`, high) until the reviewer approves.

Questions: the PC collects every question raised by the author, the reviewer or itself into one
grouped list and asks Sean directly, all at once, before finalizing. At night, a PC asks its launching
EC; a PC under an SC asks that SC, which relays the grouped questions to the EC and returns the answer
unchanged. A lone SC answers as EC. Mark the answers as the EC's; author revision and reviewer approval
still apply, with a full re-review on scope change. Answers go into the plan with
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
