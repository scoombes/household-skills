# DNS and Route 53 changes

- The `householdtech.ca` zone and all its records, including staging and production names, live in the
  **management** account, `us-east-1`. Stacks: `Household-Management-DnsZone` and `Household-Management-DnsRecords`.
  "A staging DNS change" is still a management-account change.
- Records are changed only through `Household-Management-DnsRecords` (CDK, operator synthesizer). In approval mode that is
  `hous aws plan Household-Management-DnsRecords --env management --task <ticket> --go "<sitting OK>"`, then the card, `hous aws ok`, `hous aws apply`.
  The card says `Staging: no staging copy` and `CloudFormation runs as: your session`; his OK on it applies the change. (Sean-runs mode: Sean pastes the write block.)
- Never type `route53 change-resource-record-sets` in the shell (the hook denies it, and the record would drift from the stack). `hous aws run` supports it,
  but only for a change Sean asks for outside the stack, as a write plan on its own card.
- The zone has `Retain`. Replacing it would change the nameservers, so any zone replacement is a hard stop.

```
DNS change:
- [ ] 1. checks.md steps 0–5 on Household-Management-DnsRecords (profile household-management, us-east-1)
- [ ] 2. List each record added, changed or removed (name, type, old -> new value, TTL)
- [ ] 3. Write runs: plan, card, Sean's OK, apply (approval-mode.md)
- [ ] 4. Verify at the authoritative nameservers, then at public resolvers (`hous aws read stack-status --env management --stack Household-Management-DnsRecords` first)
- [ ] 5. Report and lessons entry
```

**4. Verify.** Ask the authoritative servers first; public resolvers can serve the old answer until the old TTL expires:
```sh
for ns in $(dig +short householdtech.ca NS); do echo "$ns: $(dig +norecurse +short @"$ns" <name> <type>)"; done
dig @1.1.1.1 <name> <type>; dig @8.8.8.8 <name> <type>
```
Pass = every authoritative nameserver returns the new value. Public resolvers must agree once the old record's TTL
has passed; until then, report them as "pending TTL", not as a failure. Full verification patterns: `household-infra/docs/dns/post-cutover.md`.

**Registrar (GoDaddy) nameserver changes** are Sean's in the browser. They need the second "Continue & Verify"
step, and reloading at that point discards the change (traps.md 16). Procedure: `household-infra/docs/dns/delegation-change.md`.
