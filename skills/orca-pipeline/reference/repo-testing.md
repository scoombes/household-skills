# Repo testing: lanes, counts, evidence

Per-repo proof and full-evidence lanes for the seven HouseHold repos, plus the evidence, count, time and
retention rules every seat shares. [test-standard.md](test-standard.md) says what makes a test count;
[panel-review.md](panel-review.md) and [panel-coordinator.md](panel-coordinator.md) say how it is reviewed.

## Contents
- Status and proposed items
- Seven repos and three full-evidence lanes
- Full evidence at the exact head
- Historical targeted proof
- Counts, time and authority
- Evidence capture and retention
- Adoption and transition

## Status and proposed items

**DRAFT, HELD.** This guide is part of a draft policy PR that is not merged or synced. It stays held until
Sean adopts its terms. Items marked **Proposed [P#]** here and in the other pipeline files are
proposals, not decisions. Each takes effect only for work whose brief names an adoption version that
includes it; until then the "Until adopted" rule applies. Sean's actual terms replace the markers
(and this banner) in the same PR, re-reviewed before merge, so no partly adopted wording is installed.

| ID | Proposed item | Until adopted |
|---|---|---|
| P1 | Fix-seat time allowance: 10-minute ordinary local proof, 5-minute mutation aim and reviewer probe, 45-minute all-elapsed checkpoint, one bounded SC extension ([Counts, time and authority](#counts-time-and-authority)) | No fixed allowance. The brief names checks and an estimate; the 1.5× count rule is unchanged. |
| P2 | Count replacement: the 1.5× rule applies separately to UI/live scenarios and to lower-layer changed rules | Actual new tests and rows count against the plan's estimated new-test count, as today. |
| P3 | Evidence retention: SC-owned logs, 30/90-day minimums, closeout handoff of paths and dates ([Evidence capture and retention](#evidence-capture-and-retention)) | The old unconditional log deletion is not reinstated. Keep the safe folder until Sean directs cleanup. |
| P4 | Test-request format, and the changed-protected-rule proof-gap exception | A test request that names no plan-required rule is LOW. |
| P5 | Hostile input at an exposed auth, parser or identity boundary is graded by real consequence | A trigger that needs deliberate construction is LOW at most (except CRITICAL (a)). |
| P6 | Review preparation may overlap the exact-head CI run; the final verdict waits for it | Reviewers start after the implementer or fix seat hands off with green full-gate logs. |
| P7 | Historical targeted proof may be reused when its identity and invalidation checks hold ([Historical targeted proof](#historical-targeted-proof)) | Every claimed check is executed at the head. |

Pilots (one iOS layer pilot, one Stryker API pilot, one Muter feasibility check) are separate, separately
authorized work. Nothing here installs or runs a tool, and no pilot is a prerequisite for the policy.

## Seven repos and three full-evidence lanes

Full gate = the repo's complete applicable gate at the final commit, plus the required targeted proof.
Capability names describe the inspected snapshots (2026-11-03), not queried GitHub protection settings.
Snapshot SHAs are `origin/main` of each repo at that date; recheck before relying on a path.

| Repo (snapshot) | Lowest faithful targeted proof / retained fidelity | Full-evidence lane |
|---|---|---|
| household-api (`71ccca0`) | Model/service and real route callers; auth, tenant and refusal effects; real isolated database for transactions, migration preservation, wrong-target refusal and recovery; the relevant artifact/load/fault boundary | **GitHub PR CI at the current head**: the test job and applicable contract/type/migration/artifact checks. `hous ci` is debugging parity, not extra merge evidence. |
| household-infra (`1d92ebe`) | Offline synthesized/trust/retention/artifact contracts and the existing isolated tool/SQL proof; deployment stays a separate protected operation | **GitHub PR CI at the current head**: checks and the applicable costs jobs. Merge deploys nothing. AWS work follows `operating-household-aws`. |
| household-ios (`d7e1d01`) | Rules and races below the UI; real owner/shell component proof; affected cache, sync, reload, draft and pending-write continuity; selected navigation, accessibility, locale and profile fidelity; never the whole UI bundle | **GitHub PR CI at the current head**: script tests and English HouseHoldTests. CI runs no XCUI, so the selected UI results are additional. |
| household-web (`ffae490`) | Real handlers and components; built-export navigation and credential no-JS/hydration browser probes where affected; real rendering for layout | **GitHub PR CI at the current head**: verify, type, lint, tests, export. The separate `qa` probes remain additional; the nightly shuffle is not the PR gate. |
| household-translations (`e88d2e3`) | Keys, placeholders, staleness and serialization plus semantic language review; generator rollback and unrelated-data preservation; exact consumer parity | **Native `hous ci --web <checkout> --ios <checkout>`** at the translation head; record both committed client SHAs. Generated client PRs have their own gates. |
| household-site (`342aa5c`) | Static, runtime and edge checks plus focused browser proof for shipped layout, links, motion and JS; no application stack added | **Native `hous ci`** (`site-tests`) at the head. The browser test is separate. HTML copy is shipped behavior, not a prose exemption. |
| housetools (`06f772b`) | Real CLI entry, exit and effects with controlled process and filesystem boundaries; refusal-before-write and foreign-resource preservation; the affected documented integration only where needed | **Host `hous test`**, the full offline suite, at the head, plus the applicable documented full specialized suites. housetools has no `hous ci` capability (so no `hous ci --list` fallback), and Linux subsets are not macOS parity. |

Sources, repo-relative: API `AGENTS.md`, `docs/testing.md`, `.github/workflows/ci.yml`; infra `AGENTS.md`,
`.github/workflows/checks.yml`, `costs.yml`; iOS `AGENTS.md`, `docs/testing-ci-script.md`,
`.github/workflows/ci.yml`, `scripts/ci.sh`; web `AGENTS.md`, `CI.md`, `qa/README.md`,
`.github/workflows/ci.yml`; translations `.hous.json`, `scripts/ci.sh`, `README.md`; site `.hous.json`,
`README.md`, `tests/run.sh`; housetools `.hous.json`, `lib/adapters.ts`, `AGENTS.md`.

## Full evidence at the exact head

- A full gate never carries from an old head, including across a conflict-free rebase. Targeted proof
  carries only as [historical proof](#historical-targeted-proof).
- PR-workflow repos (api, infra, ios, web): `hous pr wait --ci-only`, then `hous pr logs`, as in SKILL.md.
  Keep the head headers, the GitHub conclusion versus local exit-code distinction, the runner summaries,
  required job and selector coverage, and the failure on a missing log. No merge past queued or running
  checks or an unexplained failure. Independently verify any claimed pre-existing failure against main.
- No-workflow repos (translations, site, housetools): `hous pr status` may pass the remote gate with "no
  remote CI configured". That cannot enforce the local logs. The implementer and fix seats produce them,
  reviewers inspect them, and the SC checks them before merge. No tool enforcement is claimed.
- Each required selector's executed outcome is recorded even when its class passes. A skipped, absent
  or setup-failed required selector is incomplete evidence, not a pass.
- **Proposed [P6]:** review preparation (reading the diff, plan and prior findings) may overlap the CI run.
  No final verdict, and no merge, before successful current-head full logs and the mandatory proof.
- Two head-bound approvals where applicable, and the existing `Rebase carry-over @ <new head> from
  <approved head>` protocol with a verified identical diff. A conflict or a changed diff needs renewed
  review.
- **Docs-only exception:** qualifying Markdown prose gets accuracy, consistency and resource/link review
  with no tests. Executable blocks, runbook command blocks and owned paths keep the full pipeline, and
  Code Owner rules still apply. Translation JSON and site HTML are shipped data and behavior, not prose.

## Historical targeted proof

**Proposed [P7].** Reusing an earlier targeted result is never a fresh-execution claim.
- Record: original source commit and the actual source and test trees (including staged or unstaged
  patches and fixture changes), test identity, selectors, config, toolchain, dependencies, environment,
  locale/device profile, and the compatibility-set SHAs. Keep patch identities and safe provenance, not
  private dumps.
- Explain the intervening changes and why none affect the claim. A relevant change or any uncertainty
  invalidates reuse. The same SHA alone is insufficient.
- Mark each row **reused** or **rerun**. Reusing a baseline also needs proven reset and cleanup. Within-run
  selector baselines exist today; cross-run cache reuse is future tooling, not a current command.

## Counts, time and authority

**Counts.** A *logical upper-layer scenario* is a starting state, an action path and an observable
outcome; adding a journey leg consumes another scenario even inside an old method. Plans record separately:
lower-layer changed rules with their required boundary and parameter rows; UI/live scenarios; the actual
new tests and rows; launches; expected runtime. Equivalent splitting changes organization, not covered
behavior. Moving proof down frees upper-layer scope. HOUS-668's historic 16-case budget counted cases and
parameter rows against an estimate of 12; that record is not rewritten.
- **Proposed [P2]:** the 1.5× rule and its authority (Sean; EC at night after cuts) apply separately to the
  plan's UI/live scenarios and to its lower-layer changed rules. Boundary and parameter rows are
  obligations of their rule, not separate budget units. Actual new tests and rows are still reported.
- Until adopted, actual new tests and rows count against the 1.5× ceiling, with no silent exemption.
- New or extended iOS journeys keep the 120-second ordinary timeout default. A higher allowance names the
  fidelity need and expected cost in the plan or brief before execution. Do not retime untouched
  journeys or change runner code. Mutation mode has its own timeout mechanics.

**Time.** A **narrow fix** changes neither the approved design nor the plan; anything else takes the
existing PC/drift route without resetting panel counts. **Proposed [P1]:**

| Event | Action and authority |
|---|---|
| Ordinary fix-seat local proof | Plan 10 minutes including setup, baseline, tests, controls and cleanup; aim at most 5 of them on mutation. The SC may approve one named increase before execution, with reason, remaining risk and end time. |
| Known mandatory proof exceeds 10 minutes | The brief names the checks, the reason and an estimate before execution. Nothing mandatory is omitted; its cost stays visible in the total. |
| Reviewer wants a small independent probe | Up to 5 minutes per probe, at the reviewer's discretion, never an obligation or a pool shared with another seat. The five-mutant ceiling is not a quota. Larger or expanding execution goes to the SC for a bounded decision, with no duplicate covered proof. |
| 45 minutes elapsed since the fix seat started | Counts authoring, setup, queue and host waits, local evidence, cleanup and CI. Report and stop expansion. If only the exact-head full gate is pending, report and continue that gate (GitHub CI keeps its wait and timeout route). Otherwise the SC may allow one extension with the named remaining risk and an end time. A further overrun goes to Sean; the EC may decide at night with a logged reason and report entry. Local allowance changes do not reset the clock. |
| More than 1.5× the applicable plan estimate | Stop and cut duplicate and non-plan proof. Only Sean permits the remaining excess (EC at night). SC time authority does not extend to counts. |
| Missing or failed critical proof, or a new design | Stays blocking or takes the PC route. No timer or extension waives correctness, clears a valid finding or authorizes a redesign. |

The checkpoint needs a concrete remainder, not automatic repeated extensions. Keep one simulator, the
lock and load protections and the host recovery authority; no parallel UI mutants.

## Evidence capture and retention

- Capture and redact safely. Never print or retain private state, cookies, credentials, typed secrets or
  live-journey xcresults. "Complete" does not authorize secret retention. Keep safe headers, summary and
  failure lines, assertion identity and omission markers. If sanitation prevents verification, the
  evidence is incomplete. Secret-bearing artifacts stay out of PRs and reports and follow the existing
  strict deletion rules; no checksum or masked dump stands in for safe provenance.
- **Proposed [P3]:** the SC owns `~/household/agent-work/<TICKET>/logs/<sha>/` through closeout, verifies
  safe evidence before closing a seat, and records each minimum-retention expiry in `status.md`.
  - Keep complete safe full-gate logs and the red/green/control provenance through review and at least
    **30 days after merge**. Unresolved release rows and pending or parked work keep their sanitized
    evidence at least **90 days after capture** and while unresolved.
  - These are minimum keep periods, not timers or automatic deletion dates, and promise no future owner.
    Safe logs go only in a later cleanup Sean directs. Pending evidence is never silently lost.
  - At closeout the SC hands evidence paths, unresolved rows and dates to the EC, or to Sean for a lone
    story. The top coordinator puts open dates in Sean's final report. Durable outcome, source and
    run-link summaries go in `~/household/epics/<TICKET>/`.
  - Close owned terminals and worktrees without deleting required safe evidence.

## Adoption and transition

Coordinator-facing; the policy takes effect only through these steps. The rest of this section does not
authorize contacting running work, installing or syncing.
1. Sean adopts or amends the proposed items above and the cutover. Until then keep the old counting and
   authority and the "Until adopted" column. Re-review the affected text before merge.
2. Before any sync, inventory every active story, PR and coordinator: its current policy version, plan
   and brief proof additions, ongoing checks, review cycle and repo-local instructions. Get Sean's
   authority before notifying running work. One merged story cannot show that all others are unaffected.
3. Declare an adoption version and a new-work boundary: briefs first issued after it use the new policy.
   Each active story either completes under an explicitly pinned immutable old policy plus its brief (a
   snapshot outside the installed directories, all referenced files present) or records a reviewed
   transition at a safe stage boundary with the exact changed evidence obligations. New seats inherit
   that choice, never whichever installed file changed last. No stopped check is assumed finished, and
   panel counts are not reset.
4. Repo-local instructions that conflict (for example iOS `AGENTS.md` and `docs/testing-ci-script.md`,
   the translations and web `CLAUDE.md` data-preservation waivers, web `CI.md`) are resolved explicitly
   in the adoption record with Sean's direction. After adoption this policy is the canonical general
   testing contract; repo-local commands, safety rules and required fidelity still apply. Seats escalate
   an unresolved conflict through their coordinator instead of letting a stale brief or local file
   silently override policy, or ignoring an explicit user instruction. A temporary override names the
   conflicting path, replacement, affected work and its expiry at the correction PR. With no explicit
   resolution the affected work waits; unrelated adoption proceeds under the declared boundary.
5. Merge only on Sean's direction. Then update the clean `~/household/household-skills` checkout to
   `origin/main`, run `hous skills sync` and inspect `hous skills status`. Never sync an unmerged
   worktree or edit installed copies to force convergence. Record acknowledgements and the policy or pin
   in new briefs; unknown delivery is not adoption. If a reliable transition is impossible, hold the sync.

Relief starts at adoption: genuine red/green replaces equivalent mutants, selectors batch, and repeated
UI proof needs a real uncovered risk.
