# Plan — Discovery correction session 2: fix licensing, security PARTIAL, casting/voice per review #6

Requirement: Address CHANGES_REQUESTED on PR #6 per independent review. Fix IRDB CC0/public domain claim, LIRC license null, coverage stats secondary, security fully-met claim (CERT_NONE/rejectUnauthorized:false), casting verified claim (Samsung no casting via WS API), voice verified claim (VoiceControl on-TV only, android remote-manager voice not implemented), Roku primary docs missing, samsungtvws version outdated, ADR status accepted prematurely, PRODUCT.md legal checkbox done prematurely, PR #6 not linked to issue #7.

Acceptance (from review):
- [ ] `docs/research/2026-09-14-ir-dataset.md` corrected: primary LICENSE.md custom permission obligations (notify via issue, attribution notice, up to 3 copies), LIRC license null unresolved, coverage 500k+/10k/80-85% marked SECONDARY ESTIMATE from infishark blog, runtime CDN access recommendation, remove CC0.
- [ ] `docs/research/2026-09-14-ecosystem-evidence.md` corrected: security PARTIAL with primary code refs CERT_NONE and rejectUnauthorized:false verified, pinning design TOFU capture server cert SHA256/fingerprint store encrypted verify on reconnect fail-safe re-pair per DOMAIN.md, casting PARTIAL (Samsung Smart View SDK/DIAL/Google Cast 2026 separate), voice NOT VERIFIED, provenance reverse-engineered, Roku primary developer.roku.com OS14.1 Control by mobile apps Enabled + search sunset + in-app ECP sunset + restriction, samsungtvws v3.0.6 2026-09-11, maintenance revised kud lib very new June 2026.
- [ ] `docs/PRODUCT.md` corrected: remove CC0, add obligations, revert legal checkbox to PARTIAL, mark security PARTIAL, casting PARTIAL, voice NOT VERIFIED, coverage removed from fixed constraints.
- [ ] `docs/decisions/0005-v1-ecosystem-selection.md` status accepted→proposed with PARTIAL validation and pinning, consequences updated, date corrected session 2.
- [ ] `docs/decisions/README.md` index updated to proposed.
- [ ] PR #6 linked to #7 (Closes #7 in body), product-owner approval noted as not existing, removed overstated MET.
- [ ] `scripts/verify.sh` PASS re-run, spec/security reviews via skills.
- [ ] `docs/MEMORY.md` appended with correction entry.
- [ ] Reply mapping finding→change/evidence for re-review.

Approach:
1. Verify primary sources via gh api and fetch_page for IRDB LICENSE.md, web_search for pinning implementations TVgrip PR #2 and hafa-remote PR #15.
2. Rewrite research docs with corrected claims and proper [VERIFIED] tags, preserving links.
3. Edit PRODUCT.md and ADR-0005 as above.
4. Update README index.
5. Run verification gate.
6. Commit, push, update PR #6 body, comment mapping.
7. Append MEMORY.md.

Out of scope:
- Implementing pinning code (discovery docs only).
- Changing PROJECT_PHASE, ALLOW_APP_STACK.
- Merging PR.
- Introducing stack artifacts.

Phases:
1. Source verification — gh api readme, fetch_page, web_search pinning.
2. Docs correction — IR dataset, ecosystem evidence.
3. Product/ADR correction — PRODUCT.md, ADR-0005, README index.
4. Verification — scripts/verify.sh + reviews.
5. PR update — body + link to issue #7 + comment.

References:
- PR #6 review comments
- Issue #7 discovery issue
- Primary: https://github.com/probonopd/irdb LICENSE.md, https://developer.roku.com/dev/docs/external-control-api
- Secondary: TVgrip PR #2, hafa-remote PR #15, omarchy issue #5386, wdesimini Swift example
- DOMAIN.md Paired device invariants, PRODUCT.md security wins over popularity
