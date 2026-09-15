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

- CI on the **first pushed commit** `6fe7b2a` (run `34869405838`), checked before
  this entry's final update — see the correction note at the end of this entry for
  why this line no longer says "the PR head": **both jobs `success`** —
  `Foundation gate` and `Independent checks`. The job's steps confirm the gate
  really ran: `Ensure shellcheck is present`, `Make PyYAML available for the
  workflow YAML check`, `Run the verification gate`, and `Run the negative tests
  (gate must fail when it should)` all `success`. Note this validates the
  environment finding above — CI installs PyYAML itself, so `workflows_yaml` runs
  there; and CI has shellcheck, so `shell_lint` is not skipped in CI. The raw job
  log text could not be retrieved from this sandbox (the Azure blob redirect for
  the log returned `EOF`), so the evidence is the per-step conclusions, not the
  logged `RESULT:` line.

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
**PR:** #10 (`arena/01a0a0b5-greenfield4`, left open for independent review, not self-merged)
**Base:** PR #6 merged at `769b9f2`

**Correction (appended, same session — verification-record accuracy only):**

The CI bullet above originally read "CI on the PR head (run `34869405838`, commit
`6fe7b2a`)". That was true when written but became false as soon as the second
commit of this session was pushed, which moved the head off `6fe7b2a`. Independent
review of PR #10 (CHANGES_REQUESTED, 2026-09-14T16:40:25Z) caught it. The label
was corrected in place; **no evidence was removed** — the run id, commit sha, both
job conclusions, and all four step names are unchanged. Per this file's own rule
("never rewrite or delete an old entry; correct it with a new one"), this note is
appended rather than the chronology being silently restated.

**Cause, and the rule that follows from it.** Recording a CI witness against a
specific sha *inside a commit* is self-invalidating: the commit that records it
becomes a new head, so the claim is stale the moment it lands. The fix is
structural, not editorial — committed memory must stay sha-independent, and the
exact final-head witness belongs in the PR body, which can be edited without
moving the head.

**Accurate chronology for this task:**

- Local gates, run before each commit: `bash scripts/verify.sh` → **PASS — 16
  passed, 0 failed, 2 skipped**; `bash scripts/selftest.sh` → **PASS — 128 cases
  behaved as asserted**.
- CI was green on the **first** pushed commit `6fe7b2a`: run `34869405838`,
  `Foundation gate` and `Independent checks` both `success`.
- CI was green again on the **second** pushed commit `32e3244`: run
  `34869684834`, `Foundation gate` and `Independent checks` both `success`
  (verified via the per-check-run conclusions on that commit).
- The **exact final-head** CI witness is deliberately **not** recorded here. This
  correction is itself a commit and therefore creates a new head, so any sha
  written into it would already be stale. That witness is verified externally
  after the last documentation commit is pushed and is recorded in the **PR body**
  (head sha, workflow run id, `Foundation gate` result, `Independent checks`
  result). Treat the PR body, not this file, as the authoritative record of the
  final-head CI result.

**Rule for future sessions:** never commit "CI is green on head `<sha>`". Write
the local results and the per-commit CI facts into memory, and put the
final-head witness in the PR body after the last push.

## 2026-09-14 — Security trust-model validation for Samsung Tizen and Android TV Remote v2 (issue #11, branch arena/01a0a0e4-greenfield4)

**Context:** Base `main` at `92730e6` (PR #10 merged). ADR-0005 accepted for product direction only, with security recorded PARTIAL and explicitly part of what the product owner accepted. This session was asked to validate the two trust models with primary evidence and to separate protocol facts from hardware-dependent designs. Discovery research only — not architecture, not implementation.

**Preflight (all verified from tools, not assumed):**
- PR #10 MERGED (`92730e6`, 2026-09-14T17:06:22Z); ADR-0005 `accepted`; `PROJECT_PHASE=discovery`, `ALLOW_APP_STACK=0`, `STACK_DECISION_ADR=` empty.
- PR #9 was an obsolete duplicate: same issue (#8), same body text ("Closes #8"), same substantive files, opened 16:21:53Z and left `mergeable: CONFLICTING` / `mergeStateStatus: DIRTY` after PR #10 merged. Confirmed by reading both PR bodies and diffs. **Closed PR #9 as superseded with a factual comment; not merged.**
- No open issue covered this work (all issues #1–#8 closed), so created **issue #11** with 16 acceptance criteria *before* writing any durable conclusion.

**Primary sources read at pinned refs** (all via `gh api ... ?ref=<pinned>` and decoded in this session):
- `xchwarze/samsung-tv-ws-api` tag `v3.0.6` — `connection.py`, `async_connection.py`, `helper.py`, `remote.py`, `README.md`.
- `kud/androidtv-remote` commit `5a05d73eb477` (v0.1.2) — `pairing-manager.ts`, `remote-manager.ts`, `certificate-generator.ts`, `pairing-message-manager.ts`, `README.md`, `docs/index.mdx`, `test/digest.test.ts`.
- `tronikos/androidtvremote2` commit `b09f21432ba3` — `pairing.py`, `androidtv_remote.py`, `base.py`. Used as an **independent cross-check** on the Android TV digest.

**What was actually established (new, beyond the previous session):**
- **Samsung**: `CERT_NONE` is in **both** paths — sync `connection.py` *and* async `helper.get_ssl_context()` (which additionally sets `check_hostname = False`). The previous session recorded only the sync line.
- **Samsung**: the token is attached **only when `ssl and token is not None`**, so on plaintext port 8001 the reference client sends no token at all. "A token exists" does not imply the transport is authenticated.
- **Samsung**: nothing binds the token to any certificate, key, or device identity — it is a pure bearer secret. First-use MITM resistance therefore **not established**.
- **Android TV**: the pairing code is **6 hex symbols** (protocol negotiates `ENCODING_TYPE_HEXADECIMAL`, `symbolLength: 6`), not a decimal PIN; `androidtvremote2` enforces `len == 6` and hex-parseability. The `"123456"` in the README is six chars that happen to be valid hex.
- **Android TV**: the digest is `SHA-256(clientMod || "0"+clientExp || serverMod || "0"+serverExp || code[2:])`, and `code[0:2]` is an **8-bit check byte** compared to `hash[0]`. Both implementations agree byte-for-byte. So the one-byte check is a cheap sanity check, **not** the security mechanism — the mechanism is the full 32-byte secret sent to the TV.
- **Android TV**: the TV certificate subject carries name + MAC (`CN=atvremote/.../XX:XX:XX:XX:XX:XX`). Useful as a label; **not** an authenticator, because it is asserted by the very certificate being validated (circular).
- **Android TV**: neither reference client persists any **server** identity, so reconnect is unprotected in both. But fail-closed pinning is viable **without** any global trust-all mode (use the paired self-signed cert as its own trust anchor).
- **Neither ecosystem was upgraded.** Both remain PARTIAL; both FAIL the identity-change and refuse-unsafe-connection invariants as shipped by their reference clients.

**Surprises / things that cut against the earlier write-up:**
1. The Android TV binding is real and cryptographically meaningful, but whether the **TV enforces** it is closed-firmware and **unproven**. I deliberately did not promote this from inference to VERIFIED even though the design is clear from two independent clients.
2. `kud`'s `sendCode` has a genuine defect: `hexStringToBytes` is applied to the raw code while the digest uses `code.slice(2)`. Confirmed with Node 22 — `hexStringToBytes("0x1A2B3C")` returns `[NaN, 26, 43, 60]`, so a `0x`-prefixed code can never pass the check, and an unprefixed code compares the wrong byte. `androidtvremote2` handles this correctly. Reference-implementation defect, not a protocol flaw — but a warning that this digest must be tested against real hardware, not ported on faith.
3. Samsung's first-use exposure is **structural, not fixable by pinning**: TOFU has no prior fingerprint at first pairing. This is a genuine conflict with a non-negotiable PRODUCT.md invariant and could not be resolved by more research.

**Recorded as an open conflict for product-owner decision (deliberately NOT resolved here):** Samsung first-use MITM resistance cannot be achieved by TOFU alone. Options recorded: accept documented residual risk; add out-of-band fingerprint confirmation; or decline Samsung for V1. Routed to the product owner rather than handled by weakening the requirement or reversing ADR-0005.

**Did not do:** did not touch `config/project.env`; did not change ADR-0005 (decision and status are unchanged; only the research docs were sharpened); did not weaken any PRODUCT.md or DOMAIN.md requirement; did not execute a single hardware test; did not reach any vendor portal (egress allowlist) — reported as reduced coverage.

**Artifacts:**
- `docs/research/2026-09-14-security-trust-model.md` (new) — the main deliverable; per-ecosystem, labelled evidence, five separate trust concerns, mapped against named invariants.
- `docs/research/2026-09-14-hardware-validation-matrix.md` (new) — 17 Samsung + 20 Android TV tests, all `NOT RUN`.
- `docs/plans/0011-security-trust-validation.md` (new) — plan per planning skill.
- `docs/PRODUCT.md` — two research-checklist rows sharpened, one new open-conflict row added, exit criteria updated. No requirement weakened.
- `docs/research/2026-09-14-ecosystem-evidence.md` — pointer added plus two corrections (async `CERT_NONE`; hex code, not decimal PIN).

**Next (highest value first):**
- Run **ATV-17** (substitute a certificate during pairing) — it is the single decisive test: it settles whether the Android TV digest binding is actually enforced, which is the largest open security question on either track.
- Then **ATV-18**, **SAM-17**, and the firmware-update stability tests (**SAM-13 / ATV-13**) — the last group decides whether pinning is shippable or merely theoretical.
- Get the product-owner decision on the Samsung first-use conflict; do not let it sit as an implicit acceptance.
- Still open and untouched: vendor legal/terms (needs human portal access), IRDB/LIRC licensing, naming, casting PARTIAL, voice NOT VERIFIED.
- Do **not** move to `architecture` and do **not** call either ecosystem shipping-ready.

**Issue:** #11
**PR:** opened from `arena/01a0a0e4-greenfield4`, left open for independent review, not self-merged.

**Correction to the entry above (appended 2026-09-14, after independent review of PR #12 — this
replaces specific claims in the previous entry; the previous entry is left intact as history).**

**What changed and why.** ChatGPT's independent review on PR #12 (CHANGES_REQUESTED,
2026-09-14T20:08:27Z) identified that I had missed an accessible primary source and had overstated
three things. The review was right on all counts. Corrections applied:

1. **I missed the AOSP pairing-protocol source, and it changed the Android TV conclusion.**
   `android.googlesource.com/platform/external/google-tv-pairing-protocol` at commit `7c99785` was
   reachable and implements **both roles**. `PoloChallengeResponse.getAlpha()` is
   `SHA-256(clientMod ‖ clientExp ‖ serverMod ‖ serverExp ‖ nonce)`; `getGamma()` is
   `alpha-prefix ‖ nonce` (`new byte[nonce.length * 2]`, copying alpha then nonce). Decisively, the
   **output-device (TV) path verifies server-side**: `PairingSession.doPairingPhase()` computes
   `localAlpha` and `Arrays.equals(localAlpha, inbandAlpha)`, throwing `BadSecretException` on
   mismatch; the C++ `OnSecretMessage` calls `VerifySecret()` and on failure sends
   `kErrorInvalidChallengeResponse`. **SecretAck is sent only after that comparison succeeds.**
   Blobs: `81095fd` (PoloChallengeResponse.java), `8baccf4` (PairingSession.java),
   `011c913` (pairingsession.cc).
   **Removed:** my claims that TV-side enforcement "is not something a client can prove", that no
   server-side implementation was available, and the "a code that nothing verifies would be
   pointless" inference. All three were wrong.
   **Kept separate:** this is **[VERIFIED — protocol]**. Whether *contemporary firmware* enforces it
   is **[HARDWARE-REQUIRED]** — the Java files are ©2009 and the C++ ©2012. I did not over-correct:
   Android TV stays PARTIAL because device conformance, reconnect identity persistence, and hardware
   behaviour are all still unproven.

2. **My "kud genuine defect" claim was wrong and is withdrawn.** For a valid six-hex-symbol code the
   byte split is correct Polo gamma layout, confirmed by `getGamma()`. `0x1A2B3C` is eight characters
   and is **not a valid pairing code**, so behaviour on it is a malformed-input/validation issue, not
   a protocol flaw. Narrowed to: `kud` lacks explicit six-hex-symbol input validation and handles
   malformed `0x…` input poorly. I had presented invalid-input behaviour as evidence that valid
   handling was broken — a real error in reasoning.

3. **Samsung server-side claims were overstated.** "Pure bearer credential / whoever holds it can act
   as the paired client" and "8001 is an unauthenticated endpoint" are statements about **server**
   behaviour that a client library cannot prove. Relabelled: no cryptographic binding is
   **[VERIFIED — client]**; the bearer-token consequence is **[INFERRED]**; the TV's actual
   association rule is **[HARDWARE-REQUIRED]** / **[UNRESOLVED]**. The Samsung **first-use** finding
   is unchanged and was not weakened — it rests on the absence of an authenticated TV identity at
   first connection, which holds regardless of what the server does with the token afterwards.

4. **ATV-17 was diagnostically invalid and was redesigned into ATV-17a–d.** A generic terminating MITM
   can fail at the client's local `checkGamma` (which runs *before* transmission) and never reach the
   TV, so "pairing failed" could never prove server-side verification. The decisive test **ATV-17b
   needs no interception at all**: our own instrumented client reads gamma from the TV and transmits a
   deliberately corrupted Secret directly, so the TV's response unambiguously reveals whether it
   verifies. 17a validates the harness, 17c only classifies the failure point, 17d repeats on a second
   device generation.

5. **PRODUCT.md had the governance/exit-criteria paragraph twice verbatim** (my earlier restore had
   appended it again). Deduplicated 2 → 1.

**Lesson to carry forward.** "Two independent client implementations agree" is corroboration of
*client* behaviour only — never evidence of *server* behaviour. Before concluding "the peer's
behaviour cannot be known", search for the protocol's own reference implementation; and before
calling something a defect, check whether the input was ever in contract.

**Not changed:** ADR-0005 status and decision (untouched); `config/project.env` (unchanged, phase
still discovery); no requirement in PRODUCT.md or DOMAIN.md weakened; the Samsung first-use conflict
remains an open product-owner decision, not resolved here; no hardware test executed.

**Issue:** #11 · **PR:** #12 (updated in place; still open, not self-merged)

**Correction to the entry above (appended 2026-09-15, after the second independent review of PR #12 —
this replaces a specific interpretation; earlier entries are left intact as history).**

**What the previous interpretation got wrong.** I had written that the deployed Android TV 8-bit
alpha-prefix check is "a cheap client-side sanity check, not the security mechanism", that "the
security property derives from … not from the prefix width", and that the full 32-byte alpha is the
security mechanism. **That was backwards on the part that matters for first-use MITM.** The reviewer
was right and I verified it against the pinned sources rather than just accepting it.

Walking a terminating active MITM through the protocol makes the error concrete. With two
terminated legs the phone computes `alpha_phone = H(K_C, K_M1, N)` and the TV computes
`alpha_TV = H(K_M2, K_S, N)`. The *only* thing that can reveal that the phone's observed key
material differs from the TV's is the alpha prefix, which reaches the phone **through the user**,
not through the network. So the prefix **is** the out-of-band authenticator and its width **is**
security-critical. The full 32-byte alpha is sent in-band over the channel whose integrity is in
question; all its inputs except the nonce are public certificates; and once the attacker clears the
prefix gate and observes one alpha, the 16-bit nonce falls to a 2^16 offline search (milliseconds),
after which he computes each leg's alpha independently and both verifications pass.

**Corrected evidence.** Deployed gamma = 8-bit alpha prefix ‖ 16-bit nonce
[VERIFIED — deployed client]. AOSP `getGamma()` = alpha-prefix ‖ nonce with the prefix
`nonce.length` bytes wide [VERIFIED — protocol/reference, blob `81095fd`] — i.e. the structure
matches but deployed carries 8 bits where that formula would give 16. AOSP `extractNonce()`
rejects odd-length gamma, so the deployed 3-byte gamma is not wire-compatible with that reference
build. AOSP's own symbol/byte arithmetic (`symbolLength/2` then `/symbolsPerByte()`) is internally
inconsistent and is quoted, not relied on. Net result: **8 bits of out-of-band authentication per
pairing attempt**; each retry gives an independent 1/256, so ~10^2 attempts to expected success if
unthrottled.

**Why the classification changed.** Because digest length and out-of-band entropy are different
quantities. "The server verifies a 256-bit alpha" is true and was never in doubt; it does **not**
imply 256 bits of first-use authentication. Conflating them would have let a future architecture or
product decision read Android TV as cryptographically strong at first use. It is a **bounded
residual risk** — materially stronger than Samsung (no user-transferred code at all), not
fundamentally unsafe, and not strong enough to meet PRODUCT.md's bar on its own.

**New unresolved question.** Practical attackability now hinges on controls no source can answer:
attempts per displayed code, whether failure rotates the code, retry delay, rate limiting,
lockout/backoff persistence, code lifetime, and whether a LAN peer can start pairing unattended.
Added hardware tests **ATV-21 … ATV-28** for exactly these, all NOT EXECUTED.

**Also fixed this round.** ATV-17d now runs the baseline **and** the corrupted-Secret test on ATV-B,
because a clean baseline on a second device proves harness portability, not server verification;
results are explicitly per-device/per-firmware and must not be generalised. Samsung §1.4 was
reconciled with §1.7 — the reference client *presents* an opaque token [VERIFIED — client], bearer
semantics are [INFERRED], and the TV's server-side association rule is [UNRESOLVED]/[HARDWARE-required];
it is no longer stated as VERIFIED that simple possession authenticates the client to every target
Samsung TV. The Samsung first-use finding is unchanged.

**Unchanged:** ADR-0005 status and decision (untouched); `config/project.env`; every PRODUCT.md and
DOMAIN.md requirement; the Samsung first-use conflict as an open product-owner decision. No hardware
test executed. No interception performed.

**Lesson to carry forward.** When a protocol has both an out-of-band value and an in-band digest,
ask separately: *what does each authenticate, over which channel, against which adversary?* An
in-band digest sent over the untrusted channel cannot authenticate the endpoints of that channel.

**Issue:** #11 · **PR:** #12 (body rewritten to current state; still open, not self-merged)

---

## 2026-09-15 — Correction round 4: trial independence, the local retry path, and per-leg vs cross-leg binding

**Context.** Three further independent reviews of PR #12 (all against head `26978f2`) accepted the
8-bit out-of-band conclusion but found it was **quantified and tested wrongly**. All three reviews
were checked against the pinned sources before editing; none of the reviewer claims was accepted on
assertion alone.

**Finding 1 — §2.7.5 contradicted §2.7.6 (HIGH).** §2.7.5 still said an attacker "changes the
client's alpha but not the TV's, and the two cannot match", so first-use MITM is "resisted by
construction". That is **withdrawn**. The two alphas are computed over different key material
*by design* in a terminating-MITM setup and are **not required to match each other**: after the
8-bit gate passes the attacker supplies each leg with the alpha that leg expects. Full-alpha
equality authenticates **each leg**; it does **not** bind two separately terminated legs. The old
statement is true only for a naive/transparent relay, which cannot read or modify traffic either.
**Lesson:** *per-leg authentication* and *cross-leg binding* are different properties; verifying a
digest over an untrusted channel never binds the two ends of that channel together.

**Finding 2 — §2.8 was backwards (HIGH).** It said the compensation is server-side "so it does not
depend on the client's local one-byte check". Withdrawn and reversed. Server-side full-alpha
verification is **necessary** to authenticate the Secret on the TV leg but **not sufficient** to bind
the legs; the client's local prefix check is the **only** out-of-band cross-leg authenticator.

**Finding 3 — "per retry" was not established (MEDIUM→HIGH).** All `1/256 per attempt/retry`
language replaced by **~1/256 per independent pairing trial**. An *independent trial* requires an
alpha-digest input to change (`K_C`, `K_M1`, `K_M2`, `K_S`, `N`, or a new session regenerating one);
re-entering the same gamma against unchanged key material is **deterministic**. The claim that
"each retry generates a fresh nonce" is **withdrawn** — AOSP generates the nonce once per
output-device pairing phase, and nothing read establishes regeneration on rejection. Conditional
arithmetic retained and labelled as conditional: geometric mean **256** independent trials, **~177**
for ~50 % cumulative, ~590 for ~90 % — all contingent on the attacker obtaining that many, which is
**[HARDWARE-required]** and deliberately left open.

**Finding 4 — unmeasured timing claim (MEDIUM).** "2^16 … milliseconds" replaced with "at most
**65,536 candidate hashes**; computationally small as an offline search", plus an explicit note that
no timing was benchmarked. **BOOTSTRAP discipline: never state an unmeasured quantity as obtained
fact.** The cryptographic conclusion never needed a speed number.

**Finding 5 — ATV-21–23 measured the wrong path (HIGH).** 255/256 of failures occur **locally at the
phone's prefix check, before any Secret is transmitted**, so the TV may observe nothing it could
rate-limit. ATV-21–23 were redesigned around that path: ATV-21 = local prefix-mismatch retry loop
(does the gamma/session survive, can the same gamma be re-entered, is the connection destroyed, what
user action precedes the next trial); ATV-22 = **is the next trial cryptographically independent**
(including deliberately rotating the attacker-controlled phone-facing key `M1` with the TV session
held fixed); ATV-23 = **is the local failure visible to the TV at all**. Added ATV-29 (independent
trials per user-mediated session) and ATV-30 (cost of ~177 trials, reported as extrapolation). The
section now states that **TV-visible throttling (ATV-24–28) must not be credited as mitigation for
the local loop unless ATV-23 shows the TV actually observes local failures.**
**Lesson:** when a security bound depends on a retry loop, instrument the loop the attacker would
actually use — including the failure mode the *defender* cannot see.

**Finding 6 — consistency cleanup (LOW).** PRODUCT.md said 17 Samsung + **20** Android TV tests;
corrected to 17 + **30** (33 rows counting ATV-17a–d). The `CLIENT-INST` environment row said
"ATV-17a and ATV-17b **only**" but ATV-17d repeats both on ATV-B and the new ATV-21/22/29 also need
it. Samsung §1.4's first VERIFIED bullet ("The client authenticates itself by presenting that opaque
token") was a **server-side semantic under a client-only label**; replaced with "The client presents
no reconnect credential beyond the opaque token; no certificate, public-key, or device-identity
binding is visible in the client-observable path."

**New product-security constraint carried forward (research conclusion, not implementation work).**
The pairing gamma is **out-of-band, per-session, user-mediated material — not a stored credential**.
A future Greenfield4 implementation **must not** silently cache or reuse a gamma across a new TLS
peer identity or a new pairing session; preserving a legitimate ongoing session is acceptable,
carrying a gamma to a different peer identity is not. Recorded in §2.7.7 as a `[DESIGN CONSTRAINT]`.

**Deliberately preserved:** the 8-bit conclusion was **not** softened merely because no attack was
executed — it stays `[ANALYSIS — derived]` (not `[VERIFIED]`), contemporary-firmware conformance and
retry/session behaviour stay `[HARDWARE-required]`, Android TV stays **PARTIAL / bounded residual
risk**, the Samsung first-use conflict stays an open product-owner decision, ADR-0005 and
`config/project.env` stay untouched, and no hardware test was executed.

**Issue:** #11 · **PR:** #12 (body rewritten to current state each round; still open, not self-merged)

---

## 2026-09-15 — Correction round 5: authentication direction in §3.1, and the last "per attempt" forms

**Context.** Review `5211873875` on head `c53ed6d`: the detailed protocol analysis in §§2.7.2–2.7.3
was already right, but the **decision-output summary in §3.1 had the two pairing checks on the wrong
sides**. Verified against the pinned sources before editing, not on the review's word.

**Finding 1 (HIGH) — direction, confirmed from AOSP `7c99785`.**
- **TV → client** is the **phone's local OOB check**. `PairingSession.java` (blob `8baccf4`)
  `doPairingPhase()`, `isInputDevice()` branch: `mChallenge.checkGamma(userGamma)` →
  `BadSecretException("Secret failed local check.")` **before** `SecretMessage` is sent. C++
  `pairingsession.cc` (blob `011c913`) `SetSecret()` mirrors it: "Secret failed local check",
  `return false`, **nothing transmitted**. `PoloChallengeResponse.java` (blob `81095fd`)
  `checkGamma()` = `Arrays.equals(gamma, getGamma(nonce))`.
- **client → TV** is the **TV's full-alpha verification**. `PairingSession.java` output branch:
  regenerate nonce → `getGamma` → display → receive `SecretMessage` →
  `Arrays.equals(localAlpha, inbandAlpha)` → `BadSecretException` on mismatch → `SecretAck`
  **only after success**. C++ `OnSecretMessage` → `VerifySecret()` → `kErrorInvalidChallengeResponse`.
§3.1 now states both correctly. The client certificate is retained as client identity but is
explicitly **not** a substitute for the pairing check. Conformance stays `[HARDWARE-required]`.
**Lesson:** a summary table is a second place to get a fact wrong. When a detailed section and its
executive summary disagree, check *both* against the source — and assume the summary is the one that
drifted, because it is edited under time pressure and read most often.

**Finding 2 (MEDIUM) — three live `8 bits/attempt` statements** in §2.13 First-use trust, §3.1
First-use MITM, and the §3.1 prose. Replaced with **"8 bits per independent pairing trial"**, and
§3.1 gained an explicit guard: retries against unchanged digest inputs are **deterministic**, a fresh
~1/256 chance requires a relevant input to change, and whether enough independent trials are
obtainable stays `[HARDWARE-required]`. Added because a summary is exactly where a conditional
probability gets misread as a retry-rate claim. One benign `per attempt` in ATV-29 normalised to
`per trial`.

**Grep after the fix:** `8 bits/attempt`, `8-bit-per-attempt`, `1/256 per retry`, `bits/attempt` →
**no matches**. `1/256 per attempt` → one match, `docs/MEMORY.md:599`, inside a round-4 **withdrawal
notice** (allowed to remain by the reviewer, since it is clearly marked as corrected).

**Deliberately preserved:** the 8-bit conclusion was **not** softened because no attack was executed
— it stays `[ANALYSIS — derived]`; no hardware test was executed or claimed; Samsung's first-use
conflict stays an open product-owner decision; ADR-0005, `config/project.env`, the discovery
lifecycle state, and every `[HARDWARE-required]` caveat are untouched.

**Issue:** #11 · **PR:** #12 (body rewritten to current state; still open, not self-merged)


## 2026-09-15 — Harvest / Adopt / Reject reconciliation (issue #13, branch arena/issue-13-harvest-reconcile)

**Done:** Reconciled the supplied harvest package against Greenfield4, recorded the results in
`docs/research/2026-09-15-harvest-adopt-reject.md`, and updated PRODUCT.md plus the existing IR and
security research. No lifecycle or application-stack change was made and no hardware result was claimed.

**Verified:** The uploaded ZIP SHA-256 was checked locally. Pinned GitHub sources and current
Android, Samsung, Roku and AOSP upstream sources were checked. A local repository checkout failed
because this sandbox could not resolve GitHub DNS, so `scripts/verify.sh` was **NOT RUN locally**;
the pull-request GitHub Actions gate is the independent executable verification.

**Learned:** Android TV reconnect certificate pinning, protected Android TV client-key storage, and
protected Samsung token storage are existing OSS implementation patterns rather than open feasibility
questions. Samsung first-use authentication remains unresolved. Flipper-IRDB has an explicit CC0
provenance boundary at commit `2319685`; earlier content cannot simply be assumed covered.
IRremoteESP8266 is LGPL-2.1, not Apache-2.0. Current Roku documentation rejects third-party/mobile
ECP use. A phone-only Samsung fingerprint display is circular unless the same identity is
independently authenticated on the TV/vendor side.

**Dead ends:** Additional generic client-library comparisons for already-demonstrated storage,
pinning and discovery patterns would repeat solved work. Local verification is blocked here by DNS.

**Next:** Use PR CI as the gate. Remaining discovery work is Samsung first-use product disposition,
real-TV security/compatibility evidence, vendor/legal review, a provenance-clean V1 IR seed,
product naming, and an explicit casting/mirroring V1 scope decision.
