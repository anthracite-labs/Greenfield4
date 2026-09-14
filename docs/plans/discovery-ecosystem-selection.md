# Plan — Discovery continuation: ecosystem selection and naming research

Requirement: Continue the Greenfield4 product discovery from the committed state (PROJECT_PHASE=discovery, PRODUCT.md v1 definition, DOMAIN.md vocabulary). Establish verifiable evidence for the first two smart-TV ecosystems using the evidence bar in PRODUCT.md, validate legal/terms, security, protocol stability, pairing, wake, capabilities, maintenance risk, and user reach. Validate IR dataset provenance/quality. Define hardware matrix criteria. Research product naming constraints and candidates. Record durable findings in repository, run verification, open PR for independent review, without introducing implementation-stack artifacts.

Acceptance (derived from docs/PRODUCT.md Research still required + docs/ROADMAP.md exit condition + docs/MEMORY.md Next):
- [ ] Evidence for ecosystem candidates gathered via repository-first, then GitHub/package registries, then web search, with pinned sources.
- [ ] Current legal/terms constraints for third-party remote control evaluated for top candidates.
- [ ] Protocol stability, pairing flows, security identities, wake behavior, app launch/text input/casting/voice capabilities, maintenance burden evaluated.
- [ ] IR dataset provenance, quality, matching strategy, device coverage validated.
- [ ] First two ecosystems selected using evidence, not uploaded ZIPs alone; rationale recorded as ADR with real alternatives and why not.
- [ ] Release hardware matrix criteria defined (what counts as Tested vs Expected, minimum models/firmware).
- [ ] Product naming criteria and candidates researched; PROJECT_NAME/SLUG intentionally left open until explicit product-owner decision, but research recorded.
- [ ] Findings recorded durably: docs/research/ artifact(s) + ADR + update to docs/PRODUCT.md compatibility promise section if needed.
- [ ] scripts/verify.sh passes (PASS) in this session; selftest not required but gate must not be weakened.
- [ ] PR opened from session branch arena/01a0a053-greenfield4 with verification output, Closes reference if issue exists, and known limitations.

Approach: Research-first workflow per .ecc/skills/research.md, then planning, then implementation as docs-only changes (allowed in discovery, no_app_stack guard). Create:
1. docs/research/2026-09-14-ecosystem-evidence.md — structured evidence table per candidate (Samsung Tizen, LG webOS, Android TV/Google TV, Roku ECP, plus Fire TV/Vidaa notes), with citations, security assessment, legal notes, capabilities matrix.
2. docs/research/2026-09-14-ir-dataset.md — IR provenance (IRDB, LIRC), licensing, coverage estimate, matching strategy.
3. docs/research/2026-09-14-naming.md — naming constraints (phone-first universal remote, premium, no account, local-first), trademark search considerations, candidate shortlist.
4. docs/decisions/0005-v1-ecosystem-selection.md — ADR recording selection of Android TV + Samsung Tizen as V1, with alternatives considered (LG webOS, Roku, Fire TV) and why deferred/rejected, including security wins over popularity rationale.
5. Update docs/PRODUCT.md — replace generic "two major smart-TV ecosystems, selected by evidence" with explicit selected ecosystems and reference to research/ADR, without introducing stack decisions.
6. Update docs/MEMORY.md entry after verification.
Risks/unknowns:
- Egress allowlist blocks most vendor docs; web search is secondary evidence, must be cross-checked with primary GitHub repos where possible (samsungtvws, LGWebOSRemote, androidtv-remote).
- Legal terms for Samsung/LG/Google may be behind partner portals not reachable; must report reduced coverage rather than invent.
- No real hardware available in sandbox; "Tested" claims cannot be made, only "Expected" from protocol evidence.
- Naming trademark search limited to web search, not USPTO primary; must mark as preliminary.
- No open GitHub issue for discovery continuation; PR will not have Closes #N but will reference discovery state.

Out of scope:
- Selecting or implementing application framework, language, database, auth scheme, hosting, UI (blocked by lifecycle).
- Changing PROJECT_PHASE, ALLOW_APP_STACK, STACK_DECISION_ADR.
- Creating src/, package.json, or any no_app_stack forbidden artifacts.
- Merging PR (ChatGPT independent review required).
- Definitive trademark clearance or legal opinion.

Phases:
1. Research — web_search for market share, protocol docs, IRDB, legal terms; record sources with [VERIFIED command] or [INFERRED].
   Verify: sources listed in research docs, no invented claims.
2. Draft research artifacts — ecosystem evidence, IR dataset, naming.
   Verify: markdown links resolve (links check), no secrets, docs present.
3. ADR 0005 — ecosystem selection decision with alternatives and consequences.
   Verify: ADR follows template, indexed in docs/decisions/README.md, links resolve.
4. PRODUCT.md update — explicit V1 ecosystems, reference research/ADR.
   Verify: PRODUCT.md still respects discovery boundary (no stack).
5. Verification gate — bash scripts/verify.sh must PASS.
   Verify: command output recorded, 0 failures.
6. MEMORY.md + PR — append session entry, commit, push, gh pr create.
   Verify: git diff main...HEAD reviewed, no scratch files.

References:
- docs/PRODUCT.md lines 40-45 fixed V1 constraints, lines 140-170 research still required, compatibility promise.
- docs/DOMAIN.md entities and business rules (security wins, pairing secrets, Tested/Expected/Unsupported).
- config/project.env PROJECT_PHASE=discovery, ALLOW_APP_STACK=0.
- scripts/verify.sh check_no_app_stack forbids stack artifacts.
