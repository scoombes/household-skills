---
name: operating-household-aws
description: Applies HouseHold's AWS operating rules (who may run reads and writes, Sean's OK on an approval card, caller and profile checks, halt-on-failure) and gives the procedures for checking CDK/CloudFormation changes, reading live state, running Sean's AWS sittings and diagnosing failed stacks. Uses AWS's external CloudFormation, CDK, IAM and security guides when needed, under HouseHold overrides. Use before running any `aws` or `cdk` command against a HouseHold account, when writing or reviewing household-infra code or a run sheet, when a stack fails, for DNS or Route 53 changes, and for IAM, Identity Center or GitHub OIDC changes.
---

# Operating HouseHold AWS

## Terms
- **Sean**: the human owner. Only he signs in to AWS.
- **Read**: an AWS call that only describes, gets, lists or looks up (`describe-*`, `get-*`, `list-*`, `lookup-*`,
  `sts get-caller-identity`, `accessanalyzer validate-policy|check-*`), plus `cdk diff`, `cdk synth` and
  `cdk --unstable=diagnose diagnose`.
- **Write**: every other AWS call, including `route53 change-resource-record-sets`, `cdk deploy|bootstrap|destroy|rollback|import`
  and executing a change set.
  - Building a change set and uploading CDK assets are **plan steps**. They run only inside `hous aws plan`, under the sitting OK.
- **Sitting OK**: Sean's explicit OK for a named scope (which stacks, which account), recorded verbatim in the work
  directory's run notes and passed as `--go` to `hous aws plan` / `hous aws run --card`. It covers reads and plan steps, never a write.
  - A general go-ahead ("I trust you, just do it") is not an OK for a write.
- **Card**: the plain-words approval card `hous aws plan` (or `run --card`) prints. Sean's **card OK**, in his own words, is recorded
  verbatim with `hous aws ok` and covers exactly the change sets (or commands) on that card.
- **Sitting**: one session in which Sean signs in, reads a card and says OK.
- **Run sheet**: the 40-80 line plan for a sitting (`household-infra/docs/runbooks/run-sheet-template.md`). It holds nothing executable.
- **Work directory**: `~/household/agent-work/<TICKET>/`. Evidence, notes and scratch clones go here.

## Pick the write mode first
Run `hous help aws apply`. Probes like this, file reads, `cdk synth` and `hous ci` need no OK.
- **It succeeds: approval mode. This is the normal path.**
  - Writes run only through `hous aws apply` / `hous aws run`, after Sean's card OK (the one exception is the packages READY before the switch, below). A hook in Claude Code and Codex denies any other AWS write, `cdk deploy`, `cdk destroy` and `cdk bootstrap`.
  - Follow [reference/approval-mode.md](reference/approval-mode.md) exactly.
  - New work gets a run sheet. A gate package, harness, fakes, mutants, receipts or `packages-index.md` is never written (rule 13).
- **It fails (unknown command): Sean-runs mode, the fallback before the switch or if the tool is not updated.**
  - Sean runs every write himself (the deploy button, or the package's write block), except the packages below. His own pastes stay his.
  - Write no new package. Prepare the run sheet and wait for the switch.

**Packages READY before the switch (either mode).** The switch is `hous help aws apply` succeeding. HOUS-450, HOUS-548 sitting 2 and
HOUS-514's short sitting finish with `hous pkg run <package> <block> --go "<Sean's OK>"`, including the writes their Waivers section names.
Sean's own pastes stay his. Nothing new uses this path: no new package, harness, fakes, mutants, receipts or `packages-index.md`.

Apart from that, never run a write any other way in either mode. If the hook denies a command, use the verb. Never work around it.

## Rules (both modes)
1. **Reads need Sean's OK** for the sitting.
   - The OK does not carry to a write or to a later session.
   - S3 object contents, secret values and KMS decrypts must be named beforehand and covered by the OK.
   - If Sean is unavailable, do the offline work, queue one batched ask, and wait. Never infer an OK.
   - Billed reads (Access Analyzer) are ordinary reads under the same OK.
   - `npm ci` of a repo's own lockfile is not an install that needs an OK.
2. **Session missing or expired:** ask Sean to type `! hous aws login household-<env>` here (or run it in his own terminal).
   Through `!` it prints the link and code at once and returns, and it asks the Mac browser to open the link; on his phone he
   opens the printed link. Read the link and code from the command output and relay them if he asks. Check completion with
   `hous aws login --status household-<env>` (no AWS contact), then `hous aws whoami --env <env>`. Never start the sign-in
   for him. Never ask for a one-time code or MFA value.
3. **Account and region come from `household-infra/config/targets.json`** (by stack name).
   - Profiles: `household-management`, `household-staging`, `household-production`.
   - Management DNS zone and records and Billing are `us-east-1`; most other stacks are `ca-central-1`.
   - Every `hous aws` verb that contacts AWS drops the `AWS_*` variables, sets profile and region and checks the caller itself.
     The local-only operations (`ok`, the `hook` verbs, `run --allowlist`) never touch AWS, so they do none of that.
   - A hand-typed read (a read file is better) starts with this header, then passes `--profile "$P" --region "$R"` on every command:
   ```sh
   ( set -euo pipefail
     for v in $(env | sed -n "s/^\(AWS_[A-Za-z0-9_]*\)=.*/\1/p"); do unset "$v"; done
     export AWS_PAGER=""
     P=household-<env>; R=<region>
     arn=$(aws sts get-caller-identity --profile "$P" --region "$R" --query Arn --output text)
     case "$arn" in arn:aws:sts::<account-id>:assumed-role/AWSReservedSSO_*) ;; *) echo "STOP: caller $arn"; exit 1;; esac
     # commands
   )
   ```
   `<account-id>` is the id that `household-infra/config/accounts.json` gives for that environment. If the caller
   check stops, ask Sean to sign in to the right profile (rule 2). This header wins over older runbook blocks that lack it.
4. **Halt at the first failure** (the verbs and the header do this). Never continue past a failed `wait`.
5. **Before any write, know the exact change.** The card lists it; anything replaced or deleted comes first, with its reason.
6. **GitHub:** App token only (`agent-token`; `household-infra/docs/runbooks/agent-token.md`). Never Sean's `gh` login or git keychain.
7. **Never write into Sean's checkouts** (`~/household/household-*`). Use a scratch clone or worktree in the work directory.
8. **Stateful resources** (DSQL clusters, S3 buckets, KMS keys, Route 53 zones) keep `DeletionPolicy: Retain`.
   - Deleting a `ROLLBACK_COMPLETE` stack, or any retained resource, needs its own card and OK (`hous aws run`; it refuses buckets, zones, keys and clusters outright).
   - Never delete an `UPDATE_ROLLBACK_COMPLETE` stack.
9. **Lambda code:** before any deploy that ships it, `hous lambda check` must pass in household-api on the commit being deployed.
10. **Staging proof API and app:** HTTPS calls with synthetic data need no OK. This grants no AWS access.
11. **Messages to Sean:** plain words, no internal codes (HOUS-/E-/D- ids stay in files and agent messages), times
    in EDT. Batch every request into one sitting. A card goes to him exactly as the tool printed it.
12. **Lessons:** `hous aws apply` and `hous aws run --apply` end with a reminder. Every sitting appends at least one
    entry to the work directory's `lessons-pending.md` (what happened, which verb or text should have caught it, the closing change).
    A clean sitting records one entry saying it had no lessons; that entry needs no closing change.
    - Entries move into `household-infra/docs/runbooks/aws-lessons-log.md` by PR from a scratch clone, each with a `Closed by:` line.
    - A lesson closes only as a tool test, a rule in this skill, or an automatic check, or by Sean's explicit "no change". Nothing else.
13. **Frozen harness:** no new gate package, harness, fakes, mutants, receipts or `packages-index.md`. Run sheets replace packages.
    - Packages READY before the switch finish as they are, through `hous pkg run`, including the writes their Waivers section names; bug fixes only. A new `harness/` folder or index is a plan violation to raise with Sean.

## Jobs
| Job | Read |
|---|---|
| Check a template or infra PR before deploy | [reference/checks.md](reference/checks.md) |
| Diagnose a failed or stuck stack | [reference/failed-stack.md](reference/failed-stack.md) |
| DNS or Route 53 change | [reference/dns.md](reference/dns.md) |
| Avoid repeating past incidents | [reference/traps.md](reference/traps.md) |
| Run a sitting (plan, card, OK, apply, read-backs) | [reference/approval-mode.md](reference/approval-mode.md) |
| Plan a sitting | the run sheet template, `household-infra/docs/runbooks/run-sheet-template.md` |
| Write or change CDK code | the external `aws-cdk` guide (see External AWS guides below); code changes go through the normal PR pipeline |
| Add an S3 bucket | `protectedBucketProps()` in `household-infra/lib/audit-stack.ts` (versioned, encrypted, private, TLS-only, Retain), plus a test |
| Prevent stateful replacement in CDK | external `aws-cdk/references/refactor-and-prevent-replacement.md` |
| Change templates safely (exports, conditions, Retain, size) | external `aws-cloudformation/references/template-safety-guidance.md` |
| Change IAM policies, role trust or GitHub OIDC | the external `aws-iam` guide and `aws-iam/references/common-pitfalls.md` |
| Security posture or findings (read-only) | the external `aws-security` guide |

## External AWS guides

AWS's reference library is maintained in [Agent Toolkit for AWS](https://github.com/aws/agent-toolkit-for-aws). It is downloaded separately and stays outside this repository. Consult only the guide or reference needed for the current task.

| Collection | Upstream download source |
|---|---|
| `aws-cdk` | [CDK guide and references](https://github.com/aws/agent-toolkit-for-aws/tree/188af2f810ce4df1b699cb55dd02f28bfa8eb2c8/skills/core-skills/aws-cdk) |
| `aws-cloudformation` | [CloudFormation guide and references](https://github.com/aws/agent-toolkit-for-aws/tree/188af2f810ce4df1b699cb55dd02f28bfa8eb2c8/skills/core-skills/aws-cloudformation) |
| `aws-iam` | [IAM guide and references](https://github.com/aws/agent-toolkit-for-aws/tree/188af2f810ce4df1b699cb55dd02f28bfa8eb2c8/skills/core-skills/aws-iam) |
| `aws-security` | [Security guide and references](https://github.com/aws/agent-toolkit-for-aws/tree/188af2f810ce4df1b699cb55dd02f28bfa8eb2c8/skills/core-skills/aws-security) |

1. Reuse an existing copy first. The current HouseHold installation stores the four collections under `~/.agents/skills/operating-household-aws/vendor/`; their entrypoints are named `GUIDE.md`. Leave these files in place.
2. If a needed document is missing, download that document and the supporting references needed for the task from the corresponding source above. Save downloads under `~/household/agent-work/<TICKET>/aws-guides/`, preserving collection names and relative paths. Upstream entrypoints are named `SKILL.md`; read them as reference material. Do not install them as independent skills or fetch the whole library by default.
3. The links use commit `188af2f810ce4df1b699cb55dd02f28bfa8eb2c8`, the revision recorded by the existing installation. Preserve upstream license/notice files with saved copies. If using a newer revision, record it and review changed instructions before relying on it.

## Overrides of upstream AWS guidance

These HouseHold rules apply to both existing and newly downloaded AWS guides. This file wins every conflict:
- Where a vendor step runs a write "after user approval" (`execute-change-set`, `create-stack`, `update-stack`,
  `delete-stack`, `cdk deploy`, `cdk rollback`, `cdk import`, `cdk refactor`, `cdk destroy`, IAM role creation),
  run it through `hous aws apply|run` after Sean's OK on a card instead (a package READY before the switch uses its own `hous pkg run` block). Ignore "do not ask for confirmation" in `aws-iam`.
- Never add `Metadata.AWSToolsMetrics` or `com.aws.cloudformation.Context` metadata.
- Ignore "prefer the AWS MCP Server / `call_aws` / `retrieve_skill`". Use the AWS CLI under the rules above.
- Install nothing (`cfn-lint`, `cfn-guard`, `uv`, …) without Sean's OK. Repo dev dependencies are added by PR.
  Run `npx cdk` only from a clone where `npm ci` installed the repo's pinned CDK.
- Never use `--disable-validation`, Express mode or `--hotswap` on HouseHold stacks.
- Ignore vendor advice to set `removalPolicy: DESTROY` or `autoDeleteObjects: true`. Stateful resources stay Retain (rule 8).
