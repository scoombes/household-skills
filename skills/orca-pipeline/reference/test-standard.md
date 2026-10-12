# Test standard

Applies only to things that run: application and infrastructure code, scripts, and executable command
blocks in runbooks and gate packages. Prose, guides and evidence docs get no tests.

The question for every test: could a wrong implementation pass it? Implementers apply this list before
opening the PR; reviewers enforce it. Stay inside the plan's test budget (see panel-review.md,
Proportion): this list makes each test count, it is not a reason to add more.

1. **Mutation proof per test**, including each `it.each` case: name the wrong implementation it catches.
   For rule-pinning tests (ordering, limit, identity, privacy, rollback), break the code that way, paste
   red output, restore, paste green. No grouped claims; no claims without pasted output.
2. **Both edges of every limit**: accepted at N, rejected at N+1.
3. **Precedence via compound inputs**: when check order is specified, each case has 2+ faults so wrong
   order yields a different result.
4. **Pin definitions, not existence**: indexes/constraints by full definition (columns, uniqueness,
   predicate). CHECK constraints in both directions, including a row a stricter rule would wrongly reject.
5. **Discriminating fixtures**: UUIDs with hex letters in mixed case; non-zero counts in every dimension;
   sentinels at every nesting level as values and unknown keys; explicit `null` cases for nullable fields.
6. **Our config vs framework default**: test at exactly our limit, not only limit+1 (which the default may
   also reject). Applies to body limits, timeouts, parser tables.
7. **Real path or declared deferred**: a test that passes only because a seam is stubbed is not evidence.
   Mark it deferred to the story that makes the path real; pin the pure function now.
8. **Scope tenant assertions** to the target household via parent relations; global counts hide leaks.
9. **Plan fixtures transcribed exactly**; declare every deviation. Fixture helpers get guard tests.
10. **Evidence hygiene**: every claimed check is pasted command + output. BSD grep treats `\|` as
    literal; use perl. Suite runs need a timeout of at least 1500 s (the API suite runs about 20 min) or
    nohup + poll; a run without the Vitest summary is not evidence. Never edit the real `.env`; use
    scratch copies.
11. **Commits**: tests red first, one commit per fix group, PR comment maps each commit to its finding.
