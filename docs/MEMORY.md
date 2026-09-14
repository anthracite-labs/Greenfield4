# Project Memory

Append-only ledger. Newest entry at the bottom. The sandbox is destroyed
between sessions, so memory that is not committed does not exist.

Procedure: [`../.ecc/skills/project-memory.md`](../.ecc/skills/project-memory.md).

## How to use this file

- Append one entry per working session. Never rewrite or delete an old entry;
  correct it with a new one that says what changed and why.
- Record what was **verified**, with the command and its actual result — not
  what was intended.
- Record surprises and dead ends. A failed approach that is not written down
  gets retried by the next session.
- Durable trade-offs go in [decisions/](decisions/README.md) as ADRs; this file
  points at them rather than duplicating them.

Entry template:

```text
## YYYY-MM-DD — <short title>

**Context:** <issue / branch>
**Did:** <what changed>
**Verified:** <command → actual result>
**Learned:** <surprises, dead ends, constraints discovered>
**Next:** <what the following session should know or do>
```

---

## Template provenance (carried by App-Factory, not project history)

This repository's engineering foundation is App-Factory (see
[`../FOUNDATION_VERSION`](../FOUNDATION_VERSION)). App-Factory v0.1.0 was
derived from the reviewed Ditto Foundation source commit
`anthracite-labs/Ditto@5d9cc349d264f73e8da913da9d2cea664522237d`. That is
factory provenance only: none of the source project's product history,
debugging chronology, issue numbers, or branch names is carried here, and none
of it applies to this repository.

## Operating conventions inherited from the foundation

These are the conventions every session is expected to follow. They are
recorded here because they are the durable context a new session needs before
it has read anything else.

| Convention | Where it is enforced |
| :-- | :-- |
| Read `.ecc/BOOTSTRAP.md` first; load 1–2 skills on demand. | `bootstrap`, `skill_index` checks |
| `scripts/verify.sh` is the only accepted evidence of quality. | CI job `Foundation gate` |
| The gate is proven by negative tests, not by passing. | `scripts/selftest.sh` |
| No stack/product choice without an approved issue and an ADR. | `no_app_stack`, `lifecycle` checks |
| Lifecycle changes are config diffs, never edits to the gate. | `config/project.env` + `lifecycle` check |
| Never commit credentials; findings are reported redacted. | `secrets`, `env_files` checks |
| Work on the session branch; never push to `main`; never self-merge. | `.ecc/rules/git.md`, branch ruleset |
| ECC is adapted, not vendored, and never silently upgraded. | `provenance`, `attribution` checks |

---

## Session entries

<!-- Append below this line. Do not edit entries above it. -->

## 2026-09-06 — App-Factory v0.1.0 foundation created

**Context:** Issue #1, branch `arena/01a076c9-app-factory`
**Did:** Created the generic reusable foundation from the reviewed Ditto
source commit: genericized `.ecc/` adapter, added `FOUNDATION_VERSION`,
`config/project.env` lifecycle state, portable `config/main-ruleset.json`,
`scripts/init-project.sh`, lifecycle-aware no-stack guard, clean
product/domain/roadmap/memory docs, and `docs/FACTORY.md`.
**Verified:** `bash scripts/verify.sh` and `bash scripts/selftest.sh` — see the
PR body for the recorded output of both runs.
**Learned:** The source foundation's permanent `ALLOW_APP_STACK=0` constant
inside `verify.sh` could not survive in a reusable template: a generated
repository must be able to graduate to an application stack without editing the
gate. Moving the state into `config/project.env` and adding a `lifecycle` check
that requires phase + ADR consistency keeps the transition explicit and
reviewable. ECC stays pinned at v2.2.0; upgrading it is a separate version bump.
**Next:** This template is `PROJECT_PHASE=factory`. A generated repository
should run `scripts/init-project.sh` first, then complete the GitHub-admin
checklist in [FACTORY.md](FACTORY.md), which the template cannot do for it.

## 2026-09-10 — Greenfield preflight documentation audit

**Context:** Issue #3, branch `chore/greenfield-preflight-cleanup`.
**Did:** Aligned the README lifecycle summary with the committed lifecycle by
including the `factory` phase, and clarified that GitHub's **Template repository**
setting is administrative state rather than something committed repository files
can prove or enable.
**Verified:** Read-only GitHub repository metadata reported `is_template=false`;
the repository rulesets endpoint returned no live rulesets at the time of this
audit. The repository content itself still carries `config/main-ruleset.json`
as the portable policy definition. No GitHub administrative setting was changed
by this documentation task. CI on the exact PR head is the acceptance evidence
for the repository edits.
**Learned:** Calling App-Factory a template source and GitHub marking it as a
template repository are separate states. The greenfield workflow must verify
both repository contents and live GitHub configuration instead of inferring one
from the other.
**Next:** A maintainer should enable GitHub's **Template repository** setting
before relying on **Use this template**, and separately decide whether to apply
the portable Main ruleset to App-Factory itself. Generated repositories must
still receive their own live governance because GitHub administrative settings
are not inherited.

## 2026-09-13 — Greenfield4 deliberately entered product discovery

**Context:** Issue #4, branch `discovery/repurpose-greenfield4`.
**Did:** The product owner explicitly chose to repurpose Greenfield4 itself as
the product repository rather than creating a separate application repository.
Recorded that exception in ADR-0001, moved `PROJECT_PHASE` to `discovery` while
leaving `ALLOW_APP_STACK=0` and `STACK_DECISION_ADR` empty, migrated the
completed product-owner discovery decisions into `docs/PRODUCT.md`, and added
established domain vocabulary to `docs/DOMAIN.md`. The final product name and
slug remain intentionally unset.
**Verified:** Repository/file state was read through the GitHub connector before
the change. Local `bash scripts/verify.sh` and `bash scripts/selftest.sh` were
**not run** because the execution environment could not resolve `github.com`
when cloning the branch (`Could not resolve host: github.com`). GitHub Actions
on the final PR head is therefore the executable verification witness for this
change and must be checked before merge.
**Learned:** The earlier direct factory-to-discovery attempt was mechanically
valid but process-incomplete because it silently contradicted the default
fresh-repository path in `docs/FACTORY.md`. The correct way to use Greenfield4
itself is to make the exception explicit, approved, and durable rather than to
pretend the default path does not exist. Uploaded remote-control research packs
remain evidence only. The Rust-core/switchable-UI idea remains an architecture
preference, not an accepted stack decision.
**Next:** Review and merge Issue #4's PR only after both required CI jobs pass.
Then continue product discovery from `docs/PRODUCT.md`: finish naming, select
the first two smart-TV ecosystems using the recorded evidence bar, and do not
move to architecture until the product definition is reviewed.

## 2026-09-14 — Discovery continuation: V1 ecosystem selection and IR/naming research

**Context:** No open issue, branch `arena/01a0a053-greenfield4`, PROJECT_PHASE=discovery, ALLOW_APP_STACK=0. Continuing from recorded state per BOOTSTRAP.md Step 0 and MEMORY.md Next from 2026-09-13.
**Did:** Established evidence for smart-TV ecosystems via web_search (secondary) + GitHub primary sources (samsungtvws v3.0.5, LGWebOSRemote, kud/androidtv-remote) + PyPI. Created docs/research/2026-09-14-ecosystem-evidence.md (market share Q4 2024 TechInsights: Samsung 16.9%, LG 11.1%, Android/Google TV >24%; security assessment: Samsung token auth, Android TV TLS cert+PIN, LG pairing, Roku ECP no auth fails security bar), docs/research/2026-09-14-ir-dataset.md (IRDB CC0 500k+ codes 10k brands, LIRC DB, coverage 80-85%, matching strategy with user verification), docs/research/2026-09-14-naming.md (criteria: premium, short, no vendor trademark, candidates Tier1 Lumen/Sora/Hearth/Beacon). Recorded ADR-0005 selecting Android TV/Google TV + Samsung Tizen as V1, deferring LG webOS, rejecting Roku ECP for V1 due to security invariants. Updated docs/decisions/README.md index and docs/PRODUCT.md fixed constraints, V1 scope, compatibility promise, and research still required checklist (ecosystem done, IR done, hardware matrix criteria defined, naming criteria done). Created docs/plans/discovery-ecosystem-selection.md per planning skill.
**Verified:** `bash scripts/verify.sh` → PASS (15 passed, 0 failed, 3 skipped: shell_lint, agentshield, workflows_yaml) — executed via bash tool, output recorded. Links check PASS 54 links resolve. No secrets, no app stack artifacts. Code review: no CRITICAL/HIGH, docs-only, no new deps, no injection. Security review: no secrets, no auth changes, no CI changes, no prompt injection acted upon, reduced legal coverage reported.
**Learned:** Market reach max with Android TV + Samsung (40.9% shipment) vs Samsung+LG 28%. Roku ECP simplicity is offset by security failure: no auth means any LAN device can control, violating PRODUCT.md "Security wins over popularity" and DOMAIN.md Paired device invariants. Samsung token rotation observed in community requires robust re-pair handling. Egress allowlist blocks vendor developer portals, so legal terms evaluation must report reduced coverage. IRDB CC0 licensing suitable, LIRC GPL data vs code distinction. Naming must avoid vendor trademarks per LG forum guidance. Duplicate ADR 0001 files exist from prior session (0001-ecc-on-arena-adapter and 0001-repurpose) — index now lists both to avoid hidden file.
**Next:** Product owner to review ADR-0005 and research docs, finalize product name/slug (set PROJECT_NAME/SLUG in config/project.env via reviewed PR), execute real-hardware matrix for Tested claims (2 models/firmware per ecosystem), re-validate legal terms with human access to official portals, then review PRODUCT.md before moving to architecture. Do not introduce stack artifacts until architecture ADR and implementation transition.
**ADR:** docs/decisions/0005-v1-ecosystem-selection.md

## 2026-09-14 — Discovery correction: IR licensing, security PARTIAL, casting/voice per independent review #6

**Context:** Branch `arena/01a0a053-greenfield4`, PR #6 CHANGES_REQUESTED, issue #7 open (discovery issue with acceptance criteria). Independent review found overstated claims: IRDB described as CC0/public domain, security claimed fully met, casting/voice verified, Roku primary docs not cited, samsungtvws version outdated.
**Did:** 
- Verified primary sources: `gh api repos/probonopd/irdb/readme` decoded — primary README says "one of largest" no counts, license points to LICENSE.md; `fetch_page https://github.com/probonopd/irdb` confirmed; web_search for pinning found TVgrip PR #2 (per-TV serverCertSha256 encrypted, buildRemoteSslContext pins fingerprint, DANE-style, no trust-all) and hafa-remote PR #15 (pin exact cert for reconnects) as SECONDARY evidence of viable pinning, and omarchy-plugin issue #5386 (self-signed cert keyed to UUID, CERT_NONE MITM) and wdesimini Swift TVCertificatePinner note.
- Corrected `docs/research/2026-09-14-ir-dataset.md`: removed CC0/public domain, documented custom permission from primary LICENSE.md (notify via GitHub issue, include attribution notice `Contains/accesses irdb by Simon Peter and contributors...`, provide up to 3 copies free on request), marked LIRC DB license null/unresolved (cannot infer from GPL), moved coverage 500k+/10k/80-85% to SECONDARY ESTIMATE from infishark blog, noted runtime CDN access recommendation per primary README.
- Corrected `docs/research/2026-09-14-ecosystem-evidence.md`: security PARTIAL with primary code evidence `samsungtvws connection.py ssl.CERT_NONE` and `kud/androidtv-remote pairing-manager.ts`/`remote-manager.ts` `rejectUnauthorized:false` VERIFIED, added pinning design TOFU capture server cert SHA256/fingerprint store encrypted verify on reconnect fail-safe re-pair per DOMAIN.md identity change, casting PARTIAL (Samsung WS API no casting, separate Smart View SDK/DIAL/Google Cast 2026 requires validation), voice NOT VERIFIED (Samsung VoiceControl is on-TV Web API not WS remote, Android remote-manager comments voice not implemented), provenance corrected to reverse-engineered (kud README credits louis49, very new June 2026), Roku now cites primary `developer.roku.com/dev/docs/external-control-api` OS14.1 Control by mobile apps Enabled + search sunset OS12 + in-app ECP sunset + "may not be sent from 3rd-party platforms", samsungtvws refreshed to v3.0.6 released 2026-09-11.
- Corrected `docs/PRODUCT.md`: removed CC0 from fixed constraints and V1 scope, added IRDB obligations, reverted legal/terms and IR validation and security validation from [x] done to [ ] PARTIAL, corrected casting to PARTIAL and voice to NOT VERIFIED, noted security PARTIAL pending pinning.
- Corrected `docs/decisions/0005-v1-ecosystem-selection.md`: status accepted→proposed, included pinning assessment, PARTIAL validation, primary Roku docs, corrected IR licensing, updated consequences and follow-ups, date corrected 2026-09-14 session 2.
- Updated `docs/decisions/README.md` index status proposed.
- Linked PR #6 to issue #7 and updated PR body to remove overstated MET claims.
**Verified:** `bash scripts/verify.sh` → PASS (to be re-run in this session), spec/security reviews to be re-run via skills.
**Learned:** IRDB is NOT CC0 — primary LICENSE.md is custom permission, secondary blog incorrectly described as public domain. LIRC database license null means unresolved, cannot treat as cleared. Security: both reference libs disable server cert verification because TV serves self-signed cert keyed to UUID — MITM risk unless pinning implemented; pinning viable via TOFU fingerprint stored encrypted per TVgrip PR #2 and hafa-remote PR #15 but requires hardware validation for cert stability across reboots/firmware. Casting: Samsung remote WS API does not include casting, requires separate SDK/DIAL/Google Cast 2026. Voice: Samsung VoiceControl is on-TV Web API, Android remote-manager.ts explicitly not implemented. Roku primary docs now reachable and explicitly restrict 3rd-party mobile app ECP commands, plus Control by mobile apps must be Enabled OS14.1. Android TV protocol is reverse-engineered not official public API, kud lib very new June 2026 increases maintenance risk vs earlier assessment.
**Next:** Run `scripts/verify.sh` + spec/security reviews, push corrected branch, update PR #6 description linking Closes #7, reply mapping finding→change/evidence, await re-review. Do not move to architecture until security pinning design validated on hardware and legal terms validated with human portal access. Product owner approval still required.
**ADR:** docs/decisions/0005-v1-ecosystem-selection.md (proposed)
**PR:** #6
**Issue:** #7

## 2026-09-14 — Discovery governance fix: proposed candidates, IR candidate sources, TOFU candidate mitigation, no invented approval (session 3)

**Context:** Branch `arena/01a0a053-greenfield4`, PR #6 second independent review CHANGES_REQUESTED at head 9f26651, Issue #7 open, no product-owner approval in durable GitHub record. Objective per user: correct remaining governance, product-state, security, licensing inconsistencies, do not merge, leave PR open for re-review. Follow .ecc/BOOTSTRAP.md.

**Did:**
- Inspected Issue #7 durable record: created by arena-ai-coding-agent[bot], open, zero comments/reactions, no product-owner approval. Determined explicit product-owner approval does NOT exist — do not invent, keep ADR-0005 proposed, describe Android TV/Google TV + Samsung Tizen only as current proposed/leading candidates, not fixed/accepted/complete.
- **PRODUCT.md governance:**
  - Removed Android TV/Samsung Tizen from Fixed V1 constraints as fixed requirement; reworded to "V1 will support two evidence-selected ecosystems, current proposed/leading candidates per ADR-0005 proposed are Android TV + Samsung Tizen — not fixed/accepted pending explicit product-owner approval and security validation".
  - Built-in phone IR kept as fixed capability; IRDB and LIRC treated as candidate/evaluated data sources not fixed shipping dependencies while legal status unresolved/conditional; IRDB requires product-owner/legal acceptance still pending.
  - V1 scope reworded to proposed candidates, not fixed selection, with PARTIAL security pending trustworthy identity analysis.
  - Compatibility promise reworded to proposed candidates.
  - Research checklist: "Choose first two ecosystems" changed from [x] done to [ ] PARTIAL/proposed, explicitly noting product-owner approval does not exist in durable record (Issue #7 open, no approval), preserving distinction between product-direction approval (pending) and unresolved security/legal validation (PARTIAL). IR validation updated to note LIRC not approved as shipping source, IRDB requires product-owner/legal acceptance pending, runtime CDN does not remove obligations because license covers network access.
- **ADR-0005 governance:**
  - Title updated to "(proposed candidates)", Date corrected session 3.
  - Deciders field corrected to reflect actual durable approval record: discovery research session + independent review feedback; explicitly states product-owner approval does NOT exist (Issue #7 open, zero comments/reactions, no approval), product owner not listed as decider for proposed state.
  - Context updated to note governance per BOOTSTRAP.md hard rule 4 (approved issue + ADR required for durable product requirements), and that Issue #7 lacks approval so ADR remains proposed and PRODUCT.md must describe as proposed candidates.
  - Decision status remains proposed because (1) no product-owner approval, (2) security trust model PARTIAL.
  - Consequences Negative rewritten to separated trust analysis, candidate mitigation not demonstrated, first-use MITM resistance:
    - Samsung: token flow not cryptographically bound to TLS cert, first-use MITM resistance unresolved, TOFU pinning candidate mitigation only protects subsequent connections.
    - Android TV: pairing-manager.ts SHA-256 over client/server cert moduli/exponents + PIN VERIFIED primary, providing stronger binding, but first-use trust still TOFU and persistence needs verification; pinning candidate mitigation, not demonstrated, must not be generalized from Samsung or third-party projects.
  - Follow-ups updated to require explicit product-owner approval before treating as fixed, and separated Samsung vs Android TV first-use trust analysis with hardware validation.
- **IR data-source state:**
  - IRDB: retained exact primary-license obligations verbatim, removed unsupported interpretation that Play Store access or APK satisfies up to three copies obligation; stated only what license requires, operational consequence requires product-owner/legal confirmation.
  - Clarified runtime CDN access recommended by README for updateability but does not remove obligations because license explicitly covers network access.
  - LIRC: remains unresolved, not approved as shipping source.
  - Matching strategy and licensing compliance sections updated to candidate/evaluated sources.
- **Ecosystem evidence doc:**
  - Samsung security section rewritten to separated analysis, candidate mitigation not demonstrated, first-use MITM unresolved, no generalization from other ecosystems.
  - Android TV security section rewritten to separated analysis, verified primary code showing hash binding cert material + PIN, candidate mitigation not demonstrated, first-use trust analysis, hardware validation required.
  - Recommendation section re-evaluated separating VERIFIED facts, secondary evidence, inference, proposed design, unresolved questions, product-owner decisions; governance note added.
  - Next steps updated to reflect session 3 requirements.
- **Decisions index:** Title updated to proposed candidates, date session 3.
- **Plans:** Existing plan docs remain, new corrections covered by this entry and docs/plans/discovery-correction-session2.md plus this session's work.

**Verified:**
- `bash scripts/verify.sh` → to be run in this session (pending), expected PASS 15 passed 0 failed 3 skipped per previous runs; no app stack artifacts, secrets, links resolve.
- Spec review: Issue #7 fetched fresh via gh issue view, no product-owner approval found in durable record (Issue #7 open, zero comments/reactions, no approval), so governance corrections applied.
- Security review: separated trust analysis, candidate mitigation wording, first-use MITM unresolved for Samsung, Android TV binding verified but persistence unresolved, no new deps, no secrets.
- No product-owner approval invented; recorded only approval actually present (none).
- PROJECT_PHASE=discovery, ALLOW_APP_STACK=0 unchanged, no app stack introduced.

**Learned:**
- Issue #7 has no product-owner approval in durable GitHub record — created by bot, open, no comments. BOOTSTRAP hard rule 4 requires approved issue before introducing fixed product requirements, so PRODUCT.md must not treat proposed candidates as fixed constraints.
- IRDB license wording "up to three fully licensed copies/units" must be quoted verbatim without interpretation; CDN access does not remove obligations because LICENSE.md covers both include and network access.
- Samsung token flow does not cryptographically bind TLS cert — token bearer secret sent over WSS with CERT_NONE, first-use MITM resistance unresolved. Android TV PIN pairing hash does bind cert material (client modulus/exponent + server modulus/exponent + PIN) VERIFIED primary, providing stronger first-use binding but still TOFU and requires persistence validation.
- TOFU pinning must be described as candidate mitigation, not demonstrated solution, and must not be generalized across ecosystems; TVgrip PR #2 and hafa-remote PR #15 are secondary viability patterns for Android TV protocol, not proof for Samsung.
- State consistency requires PRODUCT.md, ADR-0005, research docs, decisions index, MEMORY.md, Issue #7 conformance, PR body all tell same story distinguishing VERIFIED facts, secondary, inference, proposed design, unresolved, product-owner decisions.

**Next:**
- Run `bash scripts/verify.sh` + spec/code/security review passes via .ecc/skills/INDEX.md, push corrected branch, update PR #6 body with governance fix and Closes #7, respond to latest independent review with concise finding→change/evidence mapping, leave PR open for ChatGPT independent re-review. If Issue #7 still lacks product-owner approval after corrections, leave ecosystem choice proposed and report approval as remaining human action.
**ADR:** docs/decisions/0005-v1-ecosystem-selection.md (proposed)
**PR:** #6
**Issue:** #7 (open, no approval)

## 2026-09-14 — Discovery-state reconciliation: Issue #7 approval is durable, ADR-0005 accepted (issue #8, branch arena/01a0a0b5-greenfield4)

**Context:** Issue #8, branch `arena/01a0a0b5-greenfield4`, base `main` @ `769b9f2`
(the PR #6 merge commit). Narrow discovery-state reconciliation — no new research,
no architecture work, no lifecycle change.

**Done:**
- **Verified the durable GitHub state before editing anything.** Issue #7:
  `state=closed`, `state_reason=completed`, `closed_at=2026-09-14T16:09:20Z`,
  `closed_by=anthracite-labs`. Exactly one comment, id `5666976043`,
  `author=anthracite-labs`, `author_association=OWNER`,
  `created_at=2026-09-14T16:08:57Z`; `reactions` length `0` (the approval is in
  comment text, not a reaction). PR #6: `MERGED`, merge commit `769b9f2`.
  Approval text quoted verbatim: "Product-owner approval: Approved. Proceed with
  the proposed Android TV / Google TV + Samsung Tizen V1 ecosystem direction
  recorded in ADR-0005, subject to the documented PARTIAL security/legal
  validation and remaining discovery exit criteria. Approved to merge PR #6 after
  final independent review confirms head and CI are unchanged."
- **ADR-0005** → `Status: accepted`. Deciders now lead with the product owner plus
  the comment reference and timestamp. Added an explicit "Scope of the recorded
  approval" boundary and a "Status history" line. Recorded the disposition of both
  original `proposed` reasons rather than deleting them: (1) missing approval —
  **RESOLVED**; (2) security trust model PARTIAL — **STILL OPEN, NOT RESOLVED**.
  Title `(proposed candidates)` → `(approved direction)`. All PARTIAL security,
  legal, hardware, casting, voice, and IR caveats preserved verbatim in substance.
- **Decisions index** row → `accepted`, title aligned, plus a scope note stating
  that acceptance is product direction only and that ADR-0005 is *not* the
  application-stack ADR.
- **PRODUCT.md** — approved V1 ecosystem direction in Fixed V1 constraints, V1
  scope, and compatibility promise; "choose the first two smart-TV ecosystems"
  ticked `[x]`; the other five discovery items left `[ ]` PARTIAL/open; discovery
  exit criteria rewritten to drop the now-satisfied approval item and keep
  security, legal, IR licensing, naming, and hardware matrix; added an explicit
  "approval states are separate" paragraph.
- **Research doc** (`2026-09-14-ecosystem-evidence.md`) — corrected only the
  current-state governance statements; added a "partly superseded" banner over the
  session-3 next-steps list instead of rewriting it. No evidence, statistic,
  security, or licensing claim was altered.
- **Plan** written to `docs/plans/0008-discovery-state-reconciliation.md`.
- **Not touched:** `config/project.env` (verified zero diff — `PROJECT_PHASE=discovery`,
  `ALLOW_APP_STACK=0`, `STACK_DECISION_ADR=` empty), `docs/DOMAIN.md` (zero diff),
  and the PRODUCT.md security/privacy requirements (verified zero changed lines).
  No application stack, framework, source, database, auth, hosting, or UI added.

**Verified:**
- `gh api repos/anthracite-labs/Greenfield4/issues/7/comments` → 1 comment, the
  approval above; `gh issue view 7 --json state` → `CLOSED`;
  `gh pr view 6 --json state` → `MERGED`. Issue #8 already existed with matching
  acceptance criteria, so no new issue was created.
- `bash scripts/verify.sh` → **PASS — 16 passed, 0 failed, 2 skipped**
  (`links` 55 relative links resolve, up from 54 because of the new Issue #7
  anchor links; skips are `shell_lint` — shellcheck absent — and `agentshield` —
  scanned 0 files, no Claude config surface, advisory only).
- `bash scripts/selftest.sh` → **PASS — 128 cases behaved as asserted**.
- **Dead end worth recording:** the first `selftest.sh` run **FAILED — 127 passed,
  1 failed**, case `workflows_yaml/corrupted` ("gate exited 0; the fault was not
  caught"). Cause: no YAML parser, so `verify.sh` `workflows_yaml` reported
  `SKIP`, and a skipped check cannot catch an injected fault. Confirmed
  **pre-existing and unrelated to this diff** by re-running with the changes
  stashed — identical 127/1 failure. Fixed by installing the parser
  (`python3 -m pip install --break-system-packages pyyaml` → PyYAML 6.0.3; plain
  `pip install` is refused by PEP 668 externally-managed-environment). After that,
  `workflows_yaml` reports **PASS (1 workflow file parses)** and the full selftest
  is green. Environment-only change; nothing installed is committed.
- Code review (`.ecc/skills/code-review.md`) over `git diff main...HEAD`: no
  CRITICAL/HIGH. Two MEDIUM accuracy findings self-caught and fixed before commit:
  the research doc's "Unresolved" bullet still lumped ecosystem approval in with
  genuinely open items, and the supersession banner said "two" bullets were stale
  when the session-3 list had more. No new dependencies, no secrets, no shell or
  CI changes.
- Security review **not triggered**: the diff touches only `docs/**` Markdown — no
  auth, secrets, input handling, paths, shell, network calls, dependencies,
  payment/personal data, CI workflow, `.ecc/**`, `AGENTS.md`, or
  `config/project.env`. Assessed against the `.ecc/rules/security.md` trigger
  list rather than assumed.

**Learned:**
- The Issue #7 approval is **scoped by its own wording**: it approves the ecosystem
  *direction* "subject to the documented PARTIAL security/legal validation and
  remaining discovery exit criteria". It does not approve IRDB obligations, LIRC
  licensing, vendor legal terms, security validation, hardware validation, or
  naming. The easiest failure in this task was letting "direction approved" drift
  into "ecosystem validated", so every edit carries the boundary explicitly.
- Accepting ADR-0005 is safe with respect to the no-stack guard. `verify.sh`
  `validate_stack_transition` requires `PROJECT_PHASE=implementation`, a non-empty
  `STACK_DECISION_ADR`, and that file to carry `**Decision Type:** application-stack`
  **and** `**Status:** accepted`. `STACK_DECISION_ADR` is empty and
  `grep -rn "Decision Type" docs/decisions/` matched only the template comment and
  ADR-0004 prose — ADR-0005 has no such marker. A product ADR being `accepted`
  therefore cannot unlock implementation.
- `docs/decisions/README.md` forbids editing **accepted** ADRs to change the
  decision. ADR-0005 was `proposed` when edited and its decision content is
  unchanged, so supersession was not required — but the reasoning is now written
  into the ADR so a reviewer can check it instead of trusting it.
- A `SKIP` in `verify.sh` is not a pass, and it silently disables the matching
  `selftest.sh` negative case. If the selftest reports a fault "not caught", check
  whether the corresponding check skipped before assuming a regression.
- `docs/plans/` and `docs/research/` are historical records as much as current
  state. Banning stale language there wholesale would mean rewriting history;
  the working pattern is a dated "partly superseded" banner plus a pointer to the
  current-state file.

**Still PARTIAL / open after this change:**
- Security validation, both ecosystems: reference implementations disable server
  cert verification (`ssl.CERT_NONE`, `rejectUnauthorized:false`); first-use MITM
  resistance unresolved for Samsung; Android TV cert-binding verified but
  persistence unproven; TOFU pinning is candidate mitigation, not demonstrated.
- Vendor legal/terms validation: portals blocked by the egress allowlist, reduced
  coverage, needs a human with portal access.
- IR sources: IRDB remains a candidate conditional on its custom-permission
  obligations and product-owner/legal acceptance; LIRC database licensing remains
  unresolved and is not an approved shipping source.
- Hardware matrix: criteria defined, execution not started. Naming: criteria and
  candidates researched, no decision. Casting PARTIAL, voice NOT VERIFIED.

**Next:**
- Independent review of this PR by ChatGPT; do not self-merge.
- Highest-value discovery work now is the security trust analysis, since it is the
  one gate the product owner explicitly conditioned the approval on and it is
  blocked on real hardware for the pinning-stability evidence.
- Legal/terms validation needs a human with vendor-portal access; the sandbox
  cannot close it.
- Do not move to `architecture` until security validation is resolved or the
  product owner explicitly accepts it as PARTIAL, and legal/terms validation is
  complete. Approval of the direction is no longer an outstanding condition;
  those two still are.

**ADR:** docs/decisions/0005-v1-ecosystem-selection.md (accepted — product direction only)
**Issue:** #8 (this task), #7 (approval, closed as completed)
**Base:** PR #6 merged at `769b9f2`
