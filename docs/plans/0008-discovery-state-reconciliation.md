# Plan — Issue #8: reconcile discovery state with durable Issue #7 product-owner approval

**Context:** Issue #8, branch `arena/01a0a0b5-greenfield4`, base `main` @ `769b9f2`
(PR #6 merge commit). `PROJECT_PHASE=discovery`, `ALLOW_APP_STACK=0`,
`STACK_DECISION_ADR=` empty.

## Requirement

PR #6 merged and the product owner recorded explicit approval on Issue #7, which
is now closed as completed. The repository still says the opposite — ADR-0005 is
`proposed` and its Deciders field asserts "product-owner approval does NOT exist",
PRODUCT.md describes the two ecosystems as "proposed/leading candidates … not
fixed/accepted pending explicit product-owner approval", and the discovery exit
criteria still list "obtain explicit product-owner approval on Issue #7".

This is a narrow state-reconciliation change: record the approval that actually
exists, accept the ecosystem-direction decision, and keep every unresolved
technical/legal caveat open. It is not new product research and not architecture
work.

## Durable approval actually on record (verbatim scope)

`gh api repos/anthracite-labs/Greenfield4/issues/7/comments` returns exactly one
comment, id `5666976043`, author `anthracite-labs` (`author_association=OWNER`),
`2026-09-14T16:08:57Z`:

> Product-owner approval: Approved. Proceed with the proposed Android TV / Google
> TV + Samsung Tizen V1 ecosystem direction recorded in ADR-0005, subject to the
> documented PARTIAL security/legal validation and remaining discovery exit
> criteria. Approved to merge PR #6 after final independent review confirms head
> and CI are unchanged.

Issue #7: `state=closed`, `state_reason=completed`, `closed_at=2026-09-14T16:09:20Z`,
`closed_by=anthracite-labs`; `reactions` length `0`.

**What this approval covers:** the Android TV / Google TV + Samsung Tizen V1
ecosystem direction recorded in ADR-0005, explicitly *subject to* the documented
PARTIAL security/legal validation and the remaining discovery exit criteria; and
merging PR #6 once final independent review confirmed head/CI unchanged.

**What it does not cover:** security validation, vendor legal/terms validation,
IRDB license-obligation acceptance, LIRC database licensing, hardware matrix
execution, casting/voice validation, or naming. Nothing in this task may treat
those as resolved.

## Acceptance (verbatim from Issue #8)

- [ ] Replace statements that product-owner approval does not exist. Record that explicit approval now exists on Issue #7. Do not fabricate any broader approval than what was actually recorded.
- [ ] Update ADR-0005 **Deciders** so the product owner is correctly included based on the durable Issue #7 approval.
- [ ] Reassess ADR-0005 status against repository ADR rules. If the approval is sufficient to accept the ecosystem-selection decision, change ADR-0005 from `proposed` to `accepted`.
- [ ] Preserve all unresolved technical/legal caveats. Acceptance of the product-direction decision does not mean security, legal, hardware, casting, voice, or IR-source validation is complete.
- [ ] Do not weaken any PRODUCT.md or DOMAIN.md security requirement merely to make the ADR accepted.
- [ ] PRODUCT.md: make Android TV / Google TV + Samsung Tizen the approved V1 ecosystem direction if consistent with the approved ADR.
- [ ] PRODUCT.md: mark the discovery item "choose the first two smart-TV ecosystems" complete if the decision is now accepted.
- [ ] Keep remaining discovery items open/PARTIAL where evidence is still incomplete: vendor legal/terms validation; Samsung and Android TV security validation; IR source/legal validation; real-hardware matrix execution; naming; any remaining casting/voice validation.
- [ ] Preserve clear separation between product-direction approval, shipping readiness, security validation, legal validation, and hardware validation.
- [ ] Do not convert IRDB or LIRC into approved shipping dependencies unless a separate durable approval/legal resolution exists. Keep IRDB conditional on its documented license obligations and product-owner/legal acceptance if that acceptance was not part of Issue #7. Keep LIRC unresolved if its database/config licensing remains unresolved.
- [ ] Keep `PROJECT_PHASE=discovery`, `ALLOW_APP_STACK=0`, and `STACK_DECISION_ADR=` empty. Do not introduce any application stack, framework, source code, database, auth scheme, hosting choice, or UI implementation. Do not move to `architecture`.
- [ ] Append a new `docs/MEMORY.md` entry recording: Issue #7 product-owner approval is now durable; ADR-0005/product-state reconciliation performed; what remains PARTIAL/open; actual verification results; the next discovery work. Do not rewrite historical MEMORY entries.
- [ ] Ensure PRODUCT.md, ADR-0005, decisions index, MEMORY.md, Issue #7 references, and PR body all tell the same current story. Remove stale "approval pending/no approval exists" language from current-state sections.
- [ ] Perform required code/spec review; perform security review only if the diff triggers it under repository rules; run `bash scripts/verify.sh`; run any other verification required by BOOTSTRAP for the changed surfaces.
- [ ] One PR per issue; push only the session branch; open the PR with verification results and known limitations; leave the PR open for ChatGPT independent review; do not self-merge.

## Approach

Edit the four current-state surfaces plus the research doc's governance
paragraph, and append one MEMORY entry. No research is re-run, no new evidence is
gathered, no lifecycle key is touched.

Is accepting ADR-0005 legitimate under the repository's ADR rules?

- `.ecc/skills/decisions.md` requires "**Status is honest.** `proposed` until it
  is actually in force." The decision *is* in force now: the product owner
  approved this exact direction and closed Issue #7 as completed.
- `docs/decisions/README.md` forbids editing **accepted** ADRs to change the
  decision. ADR-0005 is currently `proposed`, so a status change is permitted;
  and the *decision itself does not change* — the same two ecosystems are
  selected, the same alternatives stay rejected. Only status, deciders, and
  now-stale governance prose change. This is recorded explicitly in the ADR and
  in MEMORY so a reviewer can see the rule was checked, not dodged.
- ADR-0005 carried two reasons for `proposed`: (1) no product-owner approval,
  (2) security trust model PARTIAL. Reason (1) is now resolved. Reason (2) is
  *still open*, but the approval is explicitly "subject to the documented PARTIAL
  security/legal validation" — i.e. the owner accepted the direction while that
  work remains open. So (2) is carried as an open consequence, not as a blocker
  on the direction decision, and stays prominently PARTIAL in the ADR.

Accepting the ADR cannot unlock any implementation work: `scripts/verify.sh`
`validate_stack_transition` requires `PROJECT_PHASE=implementation`, a non-empty
`STACK_DECISION_ADR`, and that file to carry `**Decision Type:** application-stack`
plus `**Status:** accepted`. `STACK_DECISION_ADR` is empty, and
`grep -rn "Decision Type" docs/decisions/` matches only the template comment and
ADR-0004 prose — ADR-0005 has no such marker. The no-stack guard stays up.

## Phases

1. **ADR-0005** — status `proposed`→`accepted`; Deciders add the product owner
   with the comment reference; retitle `(proposed candidates)`→`(approved direction)`;
   rewrite the two stale "approval does not exist" governance passages; add an
   explicit "what acceptance does not mean" boundary. Keep every PARTIAL security,
   legal, hardware, casting, voice, and IR caveat verbatim in substance.
   Verify: `grep -n "Status:\|Deciders:"` shows accepted + product owner;
   no residual "approval does NOT exist".
2. **Decisions index** — ADR-0005 row status `proposed`→`accepted`, title matches
   the ADR, date notes the reconciliation. Verify: row matches the ADR header.
3. **PRODUCT.md** — approved V1 ecosystem direction in Fixed V1 constraints,
   V1 scope, and compatibility promise; tick "choose the first two smart-TV
   ecosystems" `[x]`; leave the other five checklist items open/PARTIAL; rewrite
   the discovery exit criteria to drop the satisfied approval item and keep the
   rest. Verify: `grep -n "^- \[ \]\|^- \[x\]" docs/PRODUCT.md` shows 1 done / 5 open.
4. **Research doc governance paragraph** — narrow correction of the stale
   current-state "approval pending" statement only. No evidence, statistics,
   security, licensing, or next-steps content is altered.
5. **MEMORY.md** — append one new entry. No historical entry touched.
   Verify: `git diff --stat docs/MEMORY.md` shows additions only (no deletions).
6. **Verification + review** — `bash scripts/verify.sh` (must exit 0),
   `bash scripts/selftest.sh` for gate integrity, code review, spec review against
   Issue #8 criteria, security-review trigger assessment.
7. **Commit + PR** — conventional commit, push `arena/01a0a0b5-greenfield4` only,
   `gh pr create` with `Closes #8`, conformance table, verification output, known
   limitations. Leave open; do not merge.

## Risks / unknowns

- **Over-claiming.** The approval is scoped. Every edit must stay inside it — the
  main failure mode here is drift from "direction approved" to "ecosystem
  validated". Mitigated by the explicit boundary note and by preserving caveats.
- **Accidentally weakening security.** PRODUCT.md "Security and privacy product
  requirements" and DOMAIN.md invariants must be byte-identical after the change.
  Verify: `git diff` shows no hunk in those sections.
- **Rewriting history.** MEMORY.md is append-only; ADR historical reasoning must
  survive as rationale even where its governance conclusion is superseded.
- **Links.** `verify.sh` `links` check resolves every relative Markdown link
  (54 at baseline); any new link must resolve or the gate fails.
- **No CI on the exact head until push.** Local PASS is not the independent
  witness; GitHub Actions on the PR head is. Report both, do not conflate them.

## Out of scope

- New ecosystem research, market data, or re-evaluating the selection.
- Architecture selection, application stack, source code, database, auth, hosting, UI.
- `config/project.env` changes of any kind — phase stays `discovery`.
- Final naming decision; `PROJECT_NAME`/`PROJECT_SLUG` stay blank.
- Real-hardware execution or new hardware criteria.
- Resolving vendor legal terms; IRDB obligations or LIRC licensing.
- Editing any historical MEMORY entry or an already-accepted ADR.
- Merging the PR.

## References

- Issue #8 acceptance criteria; Issue #7 approval comment
  `…/issues/7#issuecomment-5666976043`; PR #6 (merged, `769b9f2`).
- `.ecc/BOOTSTRAP.md` hard rules 1, 2, 4, 9; `.ecc/skills/decisions.md`;
  `.ecc/skills/planning.md`; `.ecc/skills/project-memory.md`;
  `.ecc/skills/verification.md`; `.ecc/skills/code-review.md`;
  `.ecc/skills/spec-review.md`; `.ecc/rules/security.md`.
- `docs/decisions/README.md` ADR rules; `docs/DOMAIN.md` business rules;
  `scripts/verify.sh` `check_lifecycle` / `check_no_app_stack`.
