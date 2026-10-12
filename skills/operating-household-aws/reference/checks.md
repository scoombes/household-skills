# Checks before deploy

`<p>` = `household-<env>`, `<r>` = region. Every AWS command follows SKILL.md rules 1–4. In approval mode `hous aws plan`
does steps 3–5 itself and prints the result on the card; read this file to know what the card means and to check by hand when it stops.

```
Pre-deploy checks:
- [ ] 0. Changed resources exist in the target environment
- [ ] 1. Offline: synth, repo tests, linter
- [ ] 2. Stateful replacement scan
- [ ] 3. Live template matches the assumed base
- [ ] 4. Change-set validation results (needs a change set: a plan step)
- [ ] 5. IAM changes: Access Analyzer
- [ ] 6. Report
```

**0. Target environment.** Some resources exist only in one environment (for example the site publish role is
production-only). Check the stack code or `cdk.out` for each environment. If the changed resource is absent from
the target, say so: a staging pass then proves nothing about it.

**1. Offline (no AWS, no OK needed).** Work in a scratch clone under the work directory, after `npm ci`.
- `npx cdk synth <Stack>` in the scratch clone.
- `hous ci` must be green. It runs cdk-nag (`test/cdk-nag.test.ts`), the stateful logical-ID lock (`test/stateful-ids.test.ts`, `config/stateful-ids.json`), the expected-changes schema check (`schemas/expected-changes.v1.json`, `test/expected-changes.test.ts`) and `cfn-lint` (`scripts/ci/cfn-lint.sh`, a job-owned venv; never install it yourself; with rule I3510 on, a resource ARN of the wrong service or type for the action is a finding). Fix and repeat until clean.
- For each expected-changes `Modify`, also list every resource whose Properties reference its attributes through `Fn::GetAtt` or `${LogicalId.Attribute}` in `Fn::Sub`; a dependent already listed under any action is covered, and otherwise unchanged dependents appear as dynamic `Modify` (`test/expected-changes-dependents.test.ts`).

**2. Stateful replacement.** In `npx cdk diff <Stack>` (or the change set), find any `Replacement: True` or
`Conditional` on a DSQL cluster, S3 bucket, KMS key, Route 53 zone or certificate. Each one is a blocker until the
change explains it and Sean accepts it.

**3. Live vs assumed base** (read). Compare the live template with what the package or PR assumes:
```sh
aws cloudformation get-template --stack-name <S> --template-stage Original --profile <p> --region <r> --query TemplateBody --output json > live.json
```
Report a mismatch as a diff, naming the commit or stack that put it there. Never re-pin silently.

**4. Change-set validation.** Creating the change set is a plan step. In approval mode `hous aws plan` creates it under the sitting OK, reads
the validation results with `describe-events` (`describe-stack-events` does not return them), and stops on any FAIL; WARNs go on the card.
To look again at a built one (a read): a read file whose step is `["cloudformation", "describe-events", "--change-set-name", "<change-set-arn>"]`
(`hous aws read change-sets --env <env> --stack <S>` lists the sets). Fallback, Sean-runs mode only: Sean creates the change set; then
```sh
aws cloudformation describe-events --change-set-name <change-set-arn> --profile <p> --region <r> --output json \
  --query "OperationEvents[?EventType=='VALIDATION_ERROR']"
```
Any `ValidationFailureMode: FAIL` means the change set must not run. WARN results go into the report.

**5. IAM changes** (reads, billed per check, covered by the sitting OK). `old.json` is the policy document from the
live template (step 3), or from the base commit's `cdk.out` if the resource is new to that account. `new.json` is
the policy document from the PR's `cdk.out`. Use `RESOURCE_POLICY` for bucket, key and trust policies:
```sh
aws accessanalyzer validate-policy --policy-type IDENTITY_POLICY --policy-document file://new.json --profile <p> --region <r>
aws accessanalyzer check-no-new-access --policy-type IDENTITY_POLICY \
  --existing-policy-document file://old.json --new-policy-document file://new.json --profile <p> --region <r>
```
`check-no-new-access` FAIL means new access is granted: list exactly what. It can be a false FAIL, so read the flagged statement before reporting.

**IAM grants.** A statement whose resource ARN type is not in the action's resource list in the Service Authorization Reference is a review
finding (for example `sns:Publish` takes topic ARNs only; a platform-endpoint ARN never matches). cfn-lint I3510 reports a resource ARN of the
wrong service or type in any identity policy document with plain-string resources; it does not tell an SNS endpoint ARN from a topic ARN, and
skips `*` and intrinsic resources, so those are checked by hand against the Service Authorization Reference.

**6. Report.**
- One line per check: PASS, the finding, or "pending Sean" (step 4 in Sean-runs mode, or `hous aws plan` waiting for the sitting OK). Report partial results; don't wait silently.
- Replacements and deletions first.
- For Sean: plain words, no internal codes.
