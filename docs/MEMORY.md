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

## 2026-09-14 — Discovery-state reconciliation: Issue #7 product-owner approval is durable (issue #8, branch arena/01a0a0b0-greenfield4)

**Context:** Current `main` after PR #6 merge (`769b9f2`). Issue #7 is closed. Explicit product-owner approval is recorded on Issue #7 by anthracite-labs (comment 2026-09-14T16:08:57Z): proceed with the Android TV / Google TV + Samsung Tizen V1 ecosystem direction in ADR-0005, subject to documented PARTIAL security/legal validation and remaining discovery exit criteria. Session branch `arena/01a0a0b0-greenfield4`. This is governance/decision reconciliation, not new research and not architecture work.

**Did:**
- Created Issue #8 for this reconciliation with acceptance criteria matching the objective.
- Replaced stale current-state statements that product-owner approval does not exist. Recorded only the Issue #7 approval actually present; did not invent security, legal, IRDB/LIRC, naming, hardware, or architecture approval.
- Updated ADR-0005 Deciders to include product owner (anthracite-labs via Issue #7). Reassessed status: approval is sufficient for the ecosystem-selection **product-direction** decision, so ADR-0005 changed from `proposed` to `accepted`. Preserved all unresolved technical/legal caveats. Did not weaken PRODUCT.md or DOMAIN.md security requirements.
- PRODUCT.md now records Android TV / Google TV + Samsung Tizen as the approved V1 ecosystem direction. Marked “choose the first two smart-TV ecosystems” complete at product-direction level. Remaining discovery items stay open/PARTIAL: vendor legal/terms; Samsung and Android TV security; IR source/legal; real-hardware matrix execution; naming; casting/voice.
- IRDB remains conditional on documented license obligations and separate product-owner/legal acceptance (not part of Issue #7). LIRC remains unresolved and not an approved shipping source.
- Kept `PROJECT_PHASE=discovery`, `ALLOW_APP_STACK=0`, `STACK_DECISION_ADR` empty. No application stack, framework, source code, database, auth, hosting, or UI. Did not move to architecture.
- Updated current-state notes in ecosystem-evidence and IR-dataset research docs so they do not contradict the reconciled story. Did not rewrite historical MEMORY entries. Wrote `docs/plans/discovery-approval-reconciliation.md`. Updated decisions index.

**Verified:**
- GitHub starting state via `gh api`: Issue #7 `state=closed`, `state_reason=completed`, `closed_at=2026-09-14T16:09:20Z`; approval comment id 5666976043 by anthracite-labs; PR #6 `merged=true`.
- `bash scripts/verify.sh` → PASS — 15 passed, 0 failed, 3 skipped (`shell_lint` shellcheck not installed; `agentshield` scanned 0 files advisory; `workflows_yaml` no YAML parser). Lifecycle check: `phase=discovery, allow_app_stack=0`. `no_app_stack` passed. Links 54 resolve. Secrets 58 files scanned, none found.
- Code review: docs-only governance reconciliation; no CRITICAL/HIGH; no secrets; no new deps; no app stack. Security review skipped: no `.ecc/rules/security.md` triggers and `config/project.env` unchanged.
- Spec review against Issue #8: see PR body conformance table.

**Learned:** Issue #7 approval is product-direction only and is explicitly subject to PARTIAL security/legal validation. Accepting ADR-0005 is correct for that decision and must not be read as shipping readiness.

**Next:** Independent review of this PR. Remaining discovery work: Samsung and Android TV security/TOFU hardware validation; vendor legal/terms with human portal access; IR source/legal validation (IRDB obligations, LIRC unresolved); real-hardware matrix execution; naming; remaining casting/voice validation. Do not move to architecture.

**ADR:** docs/decisions/0005-v1-ecosystem-selection.md (accepted, product-direction)
**Issue:** #8
**Refs:** Issue #7 (closed, product-owner approval recorded)
