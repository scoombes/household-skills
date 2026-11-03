# Test standard

Applies only to things that run: application and infrastructure code, scripts, and executable command
blocks in runbooks and gate packages. Prose, guides and evidence docs get no tests.

The question for every test: could a wrong implementation pass it? Implementers apply this list before
opening the PR; reviewers enforce it. Stay inside the plan's test budget (see panel-coordinator.md,
Proportion): this list makes each test count, it is not a reason to add more. Per-repo lanes, counts, time
and evidence retention: [repo-testing.md](repo-testing.md).

1. **Place each rule at the lowest faithful layer, with discriminating proof.**
   - The plan holds one small table covering the changed rules or risks, a row each: consequence; the actual production
     caller; the plausible wrong result; the cheapest faithful layer, selector and assertion; any UI/live
     fidelity added and why; expected cost; source, evidence and invalidation; pre-merge or release
     owner. The plan reviewer enforces the design. PR reviewers enforce its acceptance criteria,
     including a justification for every new or extended UI/live scenario, never a redesign. A fix round
     adds proof only for a changed risk or a demonstrated gap, not because another round happened.
   - Use deterministic boundaries, decision tables, controlled clocks and responses, independently
     expected outputs and forbidden effects. Control transport, time, scheduler and storage boundaries
     while keeping the production policy and the causal caller real. Database semantics need isolated
     real storage when a mock cannot establish the claim.
   - **Real defect:** run the same regression assertion against the actual unfixed behavior, then the
     fix, and keep the source and test identities with the genuine red and green. If old-code
     reproduction is impossible, say why and run the closest faithful semantic control, stating its
     limit. New behavior gets an old-code red where meaningful.
   - **Protected rules** need a meaningful executed red plus green, from an actual defect or a specific
     semantic control: auth, tenant, privacy, identity, wrong-database, destructive-operation,
     rollback, data, draft and pending-write rules. Related rules or rows may share evidence that
     discriminates each of them.
   - **Ordinary low-risk rules** (ordering, limits and similar) use selective negative controls where a
     plausible wrong implementation could pass. There is no per-rule, per-row or per-test executed
     mutation quota and no mutation score gate. Items 2 and 3 still apply.
   - UI/live mutants are not routine and have no ordinary quota. An exceptional one names the uncovered
     fault, the cheapest selector, the intended assertion, its cost and a stop condition. Batch
     compatible selectors and patches after the fix settles, using the existing `hous test` mutation
     support. Do not rerun old rounds automatically.
   - Not the intended red: a compile failure, a setup or splash failure, a wrong assertion, absent tests
     or a skipped required selector. Record every required selector's executed outcome, even when the
     class passes. Classify production kills and survivors, equivalent, noncovered, invalid, timed-out
     and inconclusive cases, and harness failure injection separately. Cleanup injection proves cleanup
     only if the cleanup effect was observed. Fixture pollution, uncontrolled clocks and unexplained
     failures need diagnosis, not retries until green.
2. **Both edges of every limit**: accepted at N, rejected at N+1.
3. **Precedence via compound inputs**: when check order is specified, each case has 2+ faults so wrong
   order yields a different result.
4. **Pin definitions, not existence**: indexes/constraints by full definition (columns, uniqueness,
   predicate). CHECK constraints in both directions, including a row a stricter rule would wrongly reject.
5. **Discriminating fixtures**: UUIDs with hex letters in mixed case; non-zero counts in every dimension;
   sentinels at every nesting level as values and unknown keys; explicit `null` cases for nullable fields.
6. **Our config vs framework default**: test at exactly our limit, not only limit+1 (which the default may
   also reject). Applies to body limits, timeouts, parser tables.
7. **Real caller or declared deferred**: a test that passes only because a seam is stubbed is not evidence.
   A helper, a static function on a shell type or a copied call sequence cannot prove that the shell
   dispatches to it. Caller evidence must break if the actual call site is deleted or bypassed; run a
   deletion mutant only when that is uncertain. A seam that cannot prove view lifetime does not replace
   rendered continuity. If no production caller exists yet, name the story that will add it, pin the pure
   function now and add the real-caller proof in that story; no premature shell is required. This never
   defers proof for a caller already shipped or changed.
8. **Scope tenant assertions** to the target household via parent relations; global counts hide leaks.
9. **Plan fixtures transcribed exactly**; declare every deviation. Fixture helpers get guard tests.
10. **Evidence hygiene**: every claimed check is pasted command + output, or a retained safe log path with
    its summary lines. Labelled historical proof (repo-testing.md) is never presented as freshly run. BSD
    grep treats `\|` as literal; use perl. Suite runs need a timeout of at least 1500 s (the API suite
    runs about 20 min) or nohup + poll; a run without the Vitest summary is not evidence. Never edit the
    real `.env`; use scratch copies. Never keep private state, cookies, credentials or typed secrets in
    evidence.
11. **Commits**: for an actual defect, the regression test goes in red first (against the unfixed code),
    then the fix; one commit per fix group; the PR comment maps each commit to its finding. Semantic
    controls, historical proof and docs-only work need no staged red commit: put the control and its
    result in the evidence, and never fabricate a red commit for prose.
12. **Isolation**: the smoke runner's network denial stays. New spawned children use that guard, or an
    explicit allowlisted environment and endpoints suited to their lane; do not describe child
    containment as universal today. Fake credentials and a healthy loopback endpoint are not isolation or
    database-identity proof. Keep ownership and identity checks and immutable applied migration history,
    and test populated-data preservation.
