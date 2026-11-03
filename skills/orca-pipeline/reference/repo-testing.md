# Repo testing: lanes, counts, evidence

Per-repo proof and full-evidence lanes for the seven HouseHold repos, plus the evidence, count, time and
retention rules every seat shares. [test-standard.md](test-standard.md) says what makes a test count;
[panel-review.md](panel-review.md) and [panel-coordinator.md](panel-coordinator.md) say how it is reviewed.

## Contents
- Policy version
- Seven repos and three full-evidence lanes
- Full evidence at the exact head
- Historical targeted proof
- Mutation tools: selection and status
- Counts, time and authority
- Evidence capture and retention
- Adoption and transition

## Policy version

This guide and the matching rules in the other pipeline files are policy version **testing-guidance-v1**. The
version applies to a story only where the story's brief names it under the adoption or transition recorded in
[Adoption and transition](#adoption-and-transition). Work whose brief names an older pinned snapshot completes
under that snapshot. Approving or merging this text is distinct from installing it: installation (`hous skills
sync`) waits for the safe active-work boundary in that section, and nothing here contacts running work.

What testing-guidance-v1 sets:
- Fix-seat time: a 10-minute ordinary local proof allowance, a 5-minute mutation aim and reviewer probe, a
  45-minute all-elapsed checkpoint and one bounded SC extension, then Sean (the EC at night)
  ([Counts, time and authority](#counts-time-and-authority)).
- Counts: the 1.5× rule and its authority apply separately to UI/live scenarios and to lower-layer changed
  rules, and actual new tests and rows are still reported (same section).
- Evidence: SC-owned logs, safe minimum retention of 30 and 90 days, an explicit closeout handoff and
  Sean-directed cleanup ([Evidence capture and retention](#evidence-capture-and-retention)).
- Review: the test-request format with the changed-protected-rule proof-gap exception, and severity by real
  consequence for hostile input at an exposed boundary ([panel-review.md](panel-review.md), Severity and
  Finding format).
- Overlap: review preparation may run alongside the exact-head CI run; the final verdict waits for it
  ([Full evidence at the exact head](#full-evidence-at-the-exact-head)).
- Reuse: historical targeted proof, with exact invalidation ([Historical targeted proof](#historical-targeted-proof)).

Pilots (one iOS layer pilot, one Stryker API pilot, one Muter feasibility check) are separate, separately
authorized work. Nothing here installs or runs a tool, and no pilot is a prerequisite for the policy. The
mutation tools' current installation and integration status is in
[Mutation tools: selection and status](#mutation-tools-selection-and-status).

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
- Review preparation (reading the diff, plan and prior findings) may overlap the CI run.
  No final verdict, and no merge, before successful current-head full logs and the mandatory proof.
- Two head-bound approvals where applicable, and the existing `Rebase carry-over @ <new head> from
  <approved head>` protocol with a verified identical diff. A conflict or a changed diff needs renewed
  review.
- **Docs-only exception:** qualifying Markdown prose gets accuracy, consistency and resource/link review
  with no tests. Executable blocks, runbook command blocks and owned paths keep the full pipeline, and
  Code Owner rules still apply. Translation JSON and site HTML are shipped data and behavior, not prose.

## Historical targeted proof

Reusing an earlier targeted result is never a fresh-execution claim.
- Record: original source commit and the actual source and test trees (including staged or unstaged
  patches and fixture changes), test identity, selectors, config, toolchain, dependencies, environment,
  locale/device profile, and the compatibility-set SHAs. Keep patch identities and safe provenance, not
  private dumps.
- Explain the intervening changes and why none affect the claim. A relevant change or any uncertainty
  invalidates reuse. The same SHA alone is insufficient.
- Mark each row **reused** or **rerun**. Reusing a baseline also needs proven reset and cleanup. Within-run
  selector baselines exist today; cross-run cache reuse is future tooling, not a current command.

## Mutation tools: selection and status

**Rule.** When mutation proof is already warranted ([test-standard.md](test-standard.md) item 1: a protected
rule, or a plausible wrong implementation the targeted tests could pass) and a supported third-party tool is
compatible with the repo's stack and the changed code, prefer that tool over building another ad hoc general
mutation driver. Stryker covers API/Vitest; Muter covers iOS/Swift. This changes the choice of instrument,
not the amount of proof:
- No blanket mutation gate, score threshold, per-test or per-rule quota, routine UI/live mutation,
  automatic install or upgrade, or broad run. Genuine red/green still replaces equivalent mutants; proof stays
  at the lowest faithful layer, aimed at the changed code, inside the plan's existing counts and time
  ([Counts, time and authority](#counts-time-and-authority)).
- Hand-written, selector-targeted mutants through the supported `hous` commands (`hous test --mutants` for
  iOS, `hous sandbox --ephemeral` elsewhere) stay valid. `hous` remains the normal test and evidence path.
- A tool that is absent, unconfigured or incompatible on the host or the repo is a limitation to record, not
  a waiver of required proof: use the existing red/green, semantic control or `hous` mutant instead. Absence
  on any host never authorizes a network install or upgrade.
- This adds no permission gate. Use follows the story's ordinary plan, brief and authorization.

**Status (2026-11-03; re-check before relying).** Installed locally on Sean's Mac, not integrated anywhere.

| Tool | For | Local CLI | Repo integration and compatibility |
|---|---|---|---|
| Stryker core and the official Vitest runner, both 10.1.1 | household-api (Vitest) | `~/.local/bin/stryker` | None. No HouseHold Stryker config or dependency exists and no mutation run has been made. |
| Muter 16 | household-ios (Swift) | `~/.local/bin/muter` | None, and it is an **installed candidate pending feasibility** against modern HouseHold Swift. It was built from upstream source (tag 16, no prebuilt release) and only its version and help were checked. |

- Roots: `~/.local/share/household-mutation-tools/stryker-10.1.1` (an isolated prefix with its own Vitest
  4.1.11, which selects nothing for a HouseHold checkout) and
  `~/.local/share/household-mutation-tools/muter-16`. Checks were limited to version and help, plus, for
  Stryker, an import-only resolution of the Vitest plugin from the isolated prefix (no runner built, no
  Vitest started). A global CLI does not give a project checkout plugin or dependency resolution.
- This is a local Mac provision, not CI or any other host. Details and receipts:
  `~/household/agent-work/TESTING-GUIDANCE/tools-install.md`.
- No tested run command is recorded here. Before choosing a tool, read the repo's own `package.json` (or
  Xcode project), its test config and its `AGENTS.md`; do not invent a config. Take the invocation from the
  tool's own help and the repo's current setup, and keep the selection narrow: changed files, the tests that
  should catch them, and the plan's isolation (database, workers, simulator lock, external calls). If that
  cannot be done compatibly, fall back as above and say so in the evidence. Integration and the
  compatibility pilots remain separately authorized work.

## Counts, time and authority

**Counts.** A *logical upper-layer scenario* is a starting state, an action path and an observable
outcome; adding a journey leg consumes another scenario even inside an old method. Plans record separately:
lower-layer changed rules with their required boundary and parameter rows; UI/live scenarios; the actual
new tests and rows; launches; expected runtime. Equivalent splitting changes organization, not covered
behavior. Moving proof down frees upper-layer scope. HOUS-668's historic 16-case budget counted cases and
parameter rows against an estimate of 12; that record is not rewritten.
- The 1.5× rule and its authority (Sean; EC at night after cuts) apply separately to the plan's UI/live
  scenarios and to its lower-layer changed rules. Boundary and parameter rows are obligations of their
  rule, not separate budget units. Actual new tests and rows are still reported.
- New or extended iOS journeys keep the 120-second ordinary timeout default. A higher allowance names the
  fidelity need and expected cost in the plan or brief before execution. Do not retime untouched
  journeys or change runner code. Mutation mode has its own timeout mechanics.

**Time.** A **narrow fix** changes neither the approved design nor the plan; anything else takes the
existing PC/drift route without resetting panel counts. Fix-seat time:

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
- The SC owns `~/household/agent-work/<TICKET>/logs/<sha>/` through closeout, verifies
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

Coordinator-facing. testing-guidance-v1 takes effect for a story only through these steps. The rest of this
section does not itself authorize contacting running work, installing or syncing.
1. Version. Merging this text approves it and installs nothing. A story is under testing-guidance-v1 only
   when its brief names that version under the adoption or transition recorded here. A story whose brief
   names an older pinned snapshot completes under that snapshot, never under whichever installed file
   changed last. Adoption is of the whole version, not of single rules.
2. Before any sync, inventory every active story, PR and coordinator: its current policy version, plan
   and brief proof additions, ongoing checks, review cycle and repo-local instructions. Get Sean's
   authority before notifying running work. One merged story cannot show that all others are unaffected.
3. Declare the new-work boundary: briefs first issued after it name testing-guidance-v1. Each active story
   either completes under an explicitly pinned immutable old policy plus its brief (a snapshot outside the
   installed directories, all referenced files present) or records a reviewed transition at a safe stage
   boundary with the exact changed evidence obligations. New seats inherit that choice, never whichever
   installed file changed last. No stopped check is assumed finished, and panel counts are not reset.
4. Repo-local instructions that conflict (for example iOS `AGENTS.md` and `docs/testing-ci-script.md`,
   the translations and web `CLAUDE.md` data-preservation waivers, web `CI.md`) are resolved explicitly
   in the adoption record with Sean's direction. After adoption this policy is the canonical general
   testing contract; repo-local commands, safety rules and required fidelity still apply. Seats escalate
   an unresolved conflict through their coordinator instead of letting a stale brief or local file
   silently override policy, or ignoring an explicit user instruction. A temporary override names the
   conflicting path, replacement, affected work and its expiry at the correction PR. With no explicit
   resolution the affected work waits; unrelated adoption proceeds under the declared boundary.
5. Sync is held until steps 2 to 4 are done for the active work and Sean authorizes it. Then update the
   clean `~/household/household-skills` checkout to `origin/main`, run `hous skills sync` and inspect
   `hous skills status`. Never sync an unmerged worktree or edit installed copies to force convergence.
   Record acknowledgements and the policy version or pin in new briefs; unknown delivery is not adoption.
   If a reliable transition is impossible, hold the sync.

Relief starts for work under testing-guidance-v1: genuine red/green replaces equivalent mutants, selectors
batch, and repeated UI proof needs a real uncovered risk.
