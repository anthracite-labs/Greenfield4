# Plan — Reconcile discovery state with Issue #7 product-owner approval

Requirement: After PR #6 merged, the repository still describes Android TV / Google TV + Samsung Tizen as proposed candidates and states that product-owner approval does not exist. That is now stale: Issue #7 is closed and anthracite-labs recorded explicit product-owner approval of the V1 ecosystem direction, subject to documented PARTIAL security/legal validation and remaining discovery exit criteria. Reconcile PRODUCT.md, ADR-0005, the decisions index, current-state research notes, and MEMORY.md so they tell that same current story, without inventing broader approval, without moving out of discovery, and without weakening security requirements.

Acceptance: copied from Issue #8 (https://github.com/anthracite-labs/Greenfield4/issues/8). Verbatim criteria:

- Replace statements that product-owner approval does not exist; record that explicit approval now exists on Issue #7; do not fabricate broader approval than what was recorded.
- Update ADR-0005 Deciders to include the product owner based on the durable Issue #7 approval; reassess status; if the approval is sufficient to accept the ecosystem-selection decision, change ADR-0005 from proposed to accepted; preserve all unresolved technical/legal caveats.
- PRODUCT.md: Android TV / Google TV + Samsung Tizen is the approved V1 ecosystem direction; mark “choose the first two smart-TV ecosystems” complete; keep remaining discovery items open/PARTIAL; preserve separation between product-direction approval, shipping readiness, security validation, legal validation, and hardware validation.
- Do not convert IRDB or LIRC into approved shipping dependencies; keep IRDB conditional; keep LIRC unresolved.
- Keep PROJECT_PHASE=discovery, ALLOW_APP_STACK=0, STACK_DECISION_ADR empty; no application stack or architecture transition.
- Append a MEMORY.md entry; do not rewrite historical entries.
- PRODUCT.md, ADR-0005, decisions index, MEMORY.md, Issue #7 references, and PR body tell the same current story.
- Code/spec review; security review only if triggered; bash scripts/verify.sh; one issue, one PR; leave PR open; do not self-merge.

Approach: Docs-only governance/decision reconciliation. Accept ADR-0005 as the product-direction decision because Issue #7 approval is sufficient for that decision. Do not treat acceptance as security, legal, hardware, IR-source, naming, or architecture clearance.

Phases:
1. Create the GitHub issue with acceptance criteria matching this objective — verify: `gh issue view N`.
2. Update ADR-0005 status/deciders/decision framing; update decisions index — verify: status accepted, product owner in Deciders, caveats preserved.
3. Update PRODUCT.md current-state sections — verify: ecosystems approved, choose-first-two complete, remaining items PARTIAL/open.
4. Narrow current-state update in `docs/research/2026-09-14-ecosystem-evidence.md` so it does not contradict the reconciled story — verify: no “approval does not exist” in current-state sections; IR/security/legal caveats unchanged.
5. Append MEMORY.md — verify: new entry only, historical entries untouched.
6. Reviews + `bash scripts/verify.sh` — verify: exit 0, reviews recorded.
7. Commit, push session branch, open PR, leave open.

Risks/unknowns:
- Overstating Issue #7 as security/legal/IRDB/architecture approval. Mitigation: quote the recorded approval and keep PARTIAL items open.
- Editing historical MEMORY or research evidence. Mitigation: append MEMORY; change only current-state/governance notes in research.
- Accidental lifecycle/stack change. Mitigation: do not touch `config/project.env`.

Out of scope:
- New ecosystem research
- Architecture selection / implementation stack
- Application code
- Final naming
- Real-hardware execution
- Resolving vendor legal terms beyond preserving PARTIAL
- Self-merge
