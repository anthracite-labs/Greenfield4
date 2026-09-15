# Plan — Issue #14: Harvest / Adopt / Reject evidence reconciliation

Requirement: Reconcile the current Greenfield4 discovery/security record with proven open-source implementations, preserving accepted findings and explicitly correcting any contradiction. Use Harvest / Adopt / Reject as the working strategy: reuse proven implementation patterns, adopt only those that satisfy Greenfield4 invariants, and reject shortcuts that weaken security or obscure legal/hardware/product-owner gaps. Run a second targeted pass over authoritative upstream/vendor sources only for load-bearing unresolved claims; do not repeat broad research where credible OSS has already solved the engineering problem.

Acceptance (verbatim from Issue #14):
- [ ] Preserve the current accepted Greenfield4 findings and explicitly reconcile any contradiction rather than silently rewriting history.
- [ ] Produce a durable Harvest / Adopt / Reject matrix covering the remaining discovery gaps for Samsung Tizen, Android/Google TV, IR data, legal/terms, hardware validation, and any directly affected product constraints.
- [ ] For implementation gaps already solved by credible OSS, identify the reusable pattern and pin primary source evidence; avoid redundant broad research.
- [ ] For security-critical claims, distinguish compatibility bypasses from actual mitigations and reject global certificate-verification disablement / insecure fallback as Greenfield patterns.
- [ ] Run a second, targeted pass over authoritative upstream/vendor sources for the unresolved load-bearing claims.
- [ ] Keep hardware-only questions labelled hardware-required; do not convert OSS evidence into Greenfield hardware validation.
- [ ] Keep human/legal/product-owner decisions explicit; do not treat OSS prevalence as legal clearance or owner approval.
- [ ] Keep PROJECT_PHASE=discovery, ALLOW_APP_STACK=0, and STACK_DECISION_ADR empty.
- [ ] Update project memory with evidence, surprises, and the next action.
- [ ] Run the repository verification gate before PR and leave the PR open for independent review.

Approach: Keep the existing trust-model and ecosystem research as the baseline. Create one reconciliation matrix that classifies each remaining area as HARVEST, ADOPT, REJECT, or STILL-OPEN, with pinned OSS evidence and a separate upstream-authority column. Only deepen research where the current record is load-bearing and unresolved: Samsung first-use identity, Samsung reconnect validation feasibility, Android Remote v2 pairing/reconnect identity, retry/session semantics where upstream can help, vendor/public API status, and IR dataset licensing/provenance. Do not claim hardware behavior without Greenfield hardware evidence.

Phases:
1. Reconcile repository state and accepted findings against current source docs — verify: direct read of PRODUCT.md, trust model, ecosystem/IR research, ADR-0005, and issue #14.
2. Harvest credible OSS patterns at pinned revisions and classify bypass vs mitigation — verify: GitHub primary-source files/commits.
3. Target upstream authoritative sources for only unresolved load-bearing claims — verify: source/vendor pages with exact URLs/revisions where available.
4. Write the durable matrix and minimal PRODUCT.md cross-reference/status clarification; append MEMORY.md — verify: diff review against acceptance criteria.
5. Review and verify — run repository gate where execution is available; otherwise report the exact limitation and use CI on the PR head as the executable witness.

Risks/unknowns:
- Vendor documentation may be incomplete, archived, or inaccessible; absence of public documentation is not proof of prohibition.
- OSS prevalence is not legal clearance and cannot satisfy vendor-terms review by itself.
- Hardware behavior (rate limits, retry/session lifecycle, certificate persistence on real firmware) cannot be promoted from OSS/source evidence to Tested.
- Samsung first-use identity may remain a product-owner residual-risk decision even after the deeper pass.

Out of scope:
- No application implementation, framework, database, UI, auth scheme, or stack decision.
- No lifecycle transition out of discovery.
- No claim that OSS usage satisfies vendor legal terms.
- No substitution of community testing for Greenfield hardware validation.
- No broad re-ranking of TV ecosystems already approved in ADR-0005 unless new primary evidence directly invalidates that decision.
