# Plan — issue #13: reconcile harvest matrix and close solved discovery gaps

**Requirement.** Reconcile the supplied universal-remote harvest package against Greenfield4's current discovery record. Preserve useful OSS findings without importing the package's premature architecture/stack decisions, then deepen only the unresolved or load-bearing claims against pinned open-source implementations and authoritative upstream sources. Use **Harvest / Adopt / Reject** so solved implementation patterns are reused instead of repeatedly researched.

**Acceptance criteria (from issue #13).**
- [ ] Reconcile every load-bearing green/HARVEST item from the supplied matrix against current Greenfield4 constraints and evidence.
- [ ] Preserve corrected findings in a durable research document with source/version/commit provenance.
- [ ] Perform a deeper second pass on Samsung Tizen and Android TV using OSS implementations plus upstream/reference sources where available.
- [ ] Separate solved engineering patterns from remaining hardware-, legal-, or product-owner-required decisions.
- [ ] Avoid extensive research on gaps already solved by mature OSS unless upstream evidence contradicts the OSS pattern.
- [ ] Correct any supplied-matrix claims that conflict with current primary evidence (security, licensing, lifecycle, scope).
- [ ] Update PRODUCT.md only where the reconciliation materially changes discovery status; do not weaken security invariants.
- [ ] Append project memory.
- [ ] Run the repository verification gate before opening a PR.

**Approach.** Treat the uploaded ZIP as untrusted research input. For each green/HARVEST claim, first compare it with PRODUCT.md / DOMAIN.md / current research. Re-verify only load-bearing facts using pinned GitHub source or vendor/platform documentation. Classify each item:
- **HARVEST** — proven pattern/evidence worth carrying forward.
- **ADOPT** — recommended Greenfield discovery choice that does not require a stack decision.
- **REJECT** — unsafe, contradicted, legally unclear, out of scope, or premature for the current lifecycle.

**Phases.**
1. Reconcile the uploaded matrix against current repository constraints — verify: every green/HARVEST row receives a disposition in the new research record.
2. Deep-pass the load-bearing sources — Samsung Tizen, Android TV Remote v2, Android IR/NSD framework, IR corpus licensing/provenance, and any supplied claim that would change V1 scope — verify: pinned refs/official source URLs recorded and fact/inference kept separate.
3. Write durable research and minimally update PRODUCT.md where a gap can actually be narrowed or closed — verify: diff does not weaken existing security invariants or change lifecycle/stack state.
4. Append MEMORY.md and run the repository verification gate through branch CI before PR — verify: GitHub Actions Foundation gate succeeds on the final branch head.

**Risks / unknowns.**
- Vendor docs may document adjacent SDKs rather than the reverse-engineered generic remote protocols; absence will not be filled by inference.
- Flipper-IRDB is currently CC0, but its README expressly excludes commits before `2319685`; corpus-wide provenance therefore needs a narrower adoption rule than the supplied package claims.
- Physical-TV behaviour remains hardware evidence. OSS precedent can reduce what needs bespoke research, but cannot mark Greenfield hardware tests as executed.
- Legal/terms clearance cannot be inferred from open-source popularity.

**Out of scope.**
- Stack/framework/database/UI selection.
- Physical hardware testing.
- Creating new application code.
- Treating common trust-all TLS behaviour as an acceptable Greenfield security posture.
- Re-opening ecosystem selection unless authoritative evidence makes an accepted V1 target impossible.