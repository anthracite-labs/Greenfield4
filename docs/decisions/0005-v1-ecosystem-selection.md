# ADR-0005: V1 Smart-TV Ecosystems — Android TV/Google TV + Samsung Tizen (proposed candidates)

**Date:** 2026-09-14 (corrected 2026-09-14 session 3 governance fix per independent review #6 second review)
**Status:** proposed
**Deciders:** Discovery research (session arena/01a0a053-greenfield4) — primary evidence gathering and analysis; independent review feedback (ChatGPT independent reviewer on PR #6) — security/legal corrections. **Explicit product-owner approval does NOT exist in durable GitHub record as of this session:** Issue #7 created by arena-ai-coding-agent[bot], open, zero comments/reactions, no approval; no other approval record found. Product owner not listed as decider for this proposed state.

## Context

PRODUCT.md V1 will support two evidence-selected smart-TV ecosystems, with current proposed/leading candidates evaluated per evidence bar (legal/terms, security, stability, pairing, wake, capabilities, maintenance, reach), not fixed/accepted. This ADR proposes candidates, not fixed requirements, because product-owner approval is pending and security validation is PARTIAL.

Market data Q4 2024 from TechInsights primary: Samsung 16.9% vendor, LG 11.1%, Android/Google TV >24% OS shipments, Roku 9%. Security product requirements: security wins over popularity, pairing secrets treated like passwords, no global ignore-security mode, identity changes require safe re-pairing per DOMAIN.md.

Research in docs/research/2026-09-14-ecosystem-evidence.md evaluated Samsung Tizen (samsungtvws v3.0.6 released 2026-09-11, WebSocket token auth, but connection.py uses ssl.CERT_NONE, first-use MITM resistance unresolved), LG webOS (LGWebOSRemote), Android TV Remote v2 (kud/androidtv-remote TypeScript, reverse-engineered not official public API, pairing-manager.ts and remote-manager.ts use rejectUnauthorized:false, PIN pairing hash includes cert material but first-use trust and persistence need verification, voice not implemented), and Roku ECP (official primary docs developer.roku.com: Control by mobile apps must be Enabled as of OS 14.1, search sunset OS 12.0, in-app ECP sunset, "ECP commands may not be sent from 3rd-party platforms").

IR dataset licensing corrected: IRDB custom permission (notify via issue, attribution notice, up to 3 copies on request) per primary LICENSE.md https://github.com/probonopd/irdb/blob/master/LICENSE.md, not CC0; product-owner/legal acceptance of obligations still required; runtime CDN access recommended by README for updateability but does not remove obligations because license covers network access. LIRC database licensing unresolved (license null), not approved as shipping source. Coverage/size stats are secondary estimates from infishark blog, not verified from primary IRDB repo.

Governance note per BOOTSTRAP.md hard rule 4: durable product requirements require approved issue + ADR. Issue #7 exists but lacks explicit product-owner approval in GitHub record, so this ADR remains proposed and PRODUCT.md must describe ecosystems as proposed candidates, not fixed V1 constraints.

## Decision

Propose **Android TV / Google TV (Remote v2 protocol, reverse-engineered)** and **Samsung Tizen (2016+ WebSocket API, community reverse-engineered)** as current proposed/leading V1 ecosystem candidates, with security validation **PARTIAL** pending trustworthy persistent device identity analysis (separated per ecosystem) and hardware validation, and pending explicit product-owner approval. Defer LG webOS as immediate next candidate. Reject Roku ECP for V1 due to security model failure (no authentication, no secret, any LAN device can control if enabled) and primary docs restriction on 3rd-party mobile apps. Record hardware matrix criteria requiring at least two models/firmware per ecosystem for release, including pinning stability validation.

Status is **proposed** not accepted because: (1) explicit product-owner approval does not exist in durable GitHub record (Issue #7 open, no approval), (2) security trust model for both candidates disables server certificate verification in reference implementations and requires additional evidence-backed analysis to satisfy PRODUCT.md/DOMAIN.md identity-change/re-pairing requirements, including first-use MITM resistance. Do not accept merely to make PR easier to merge.

## Alternatives considered

### Alternative: Samsung Tizen + LG webOS

- **Pros:** Two largest single vendors by brand recognition (Samsung 16.9%, LG 11.1% shipments VERIFIED TechInsights), both support WoL, pointer/trackpad, premium OLED/QLED positioning, clear marketing "Works with Samsung and LG". Both have pairing popup, token/pairing material.
- **Cons:** Misses Android/Google TV OS share >24% VERIFIED which covers Sony, TCL, Hisense, Xiaomi, Shield — broadest single protocol. Android-first product constraint favors Android TV. Both vendor-specific, not OS-wide. Samsung casting/mirroring not via WS API (requires Smart View SDK/DIAL/Google Cast separate validation) — PARTIAL. Voice not verified for Samsung (VoiceControl is on-TV Web API).
- **Why not:** Maximizing reach with two ecosystems favors Android TV + Samsung (40.9% combined shipment INFERRED) over Samsung + LG (28%). Android TV gives Google Cast natively separate from remote protocol, satisfying V1 casting/mirroring partially without cloud relay, though requires separate validation. LG remains strong but deferred to keep V1 tight. However, this alternative has similar security PARTIAL issue (Samsung CERT_NONE).

### Alternative: LG webOS + Android TV

- **Pros:** Good reach (35.9% INFERRED), both support keyboard/text input, LG brings Magic Remote pointer trackpad premium UX, Android brings broad OEM coverage. Both have pairing.
- **Cons:** Misses Samsung vendor leadership (16.9% shipments VERIFIED, 12% US CTV secondary Pixalate) and Frame Art Mode. Samsung's Tizen library samsungtvws is more active (v3.0.6 2026-09-11 VERIFIED) than LG community lib. Samsung token rotation observed ("TV keeps asking to accept connection, token changing" SECONDARY forum) requires robust re-pair handling. Voice not verified for either.
- **Why not:** Samsung's market leadership and active maintenance outweigh LG's pointer advantage for V1. LG better as third ecosystem after V1 proves two. Security for Android TV still PARTIAL due to rejectUnauthorized:false.

### Alternative: Roku ECP + Samsung or Android TV

- **Pros:** Roku has 38% US CTV market share Q1 2025 secondary Pixalate, 25% UK, 73% Mexico — huge popularity, especially US. Protocol is simplest HTTP REST, 80 lines TypeScript remote possible (secondary Dev.to).
- **Cons:** Security model fails PRODUCT.md/DOMAIN.md invariants: no authentication, no secret, any LAN device can control if Control by mobile apps Enabled [VERIFIED primary developer.roku.com: as of OS 14.1, keypress etc require Control by mobile apps Enabled]. Official primary docs now explicitly state "ECP commands may not be sent from 3rd-party platforms (for example, mobile applications)" [VERIFIED primary https://developer.roku.com/dev/docs/external-control-api] and "While it is possible to use third-party apps to control your Roku device, we strongly advise against doing so for security reasons" [SECONDARY support.roku.com]. No secure connection, no identity, violates "If a secure connection cannot be established, refuse the unsafe connection. There is no global 'ignore security' mode." Also browser mixed-content limitation.
- **Why not:** Explicitly rejected for V1 due to security bar failure and primary docs restriction. Must be honest about Unsupported status rather than hiding limitation. Documented as Unsupported with primary source.

### Alternative: Include Fire TV / VIDAA / Titan OS in V1

- **Pros:** Additional reach (Fire TV 18% US secondary, VIDAA 7.8% global secondary, Titan 4.9% secondary).
- **Cons:** Insufficient evidence in allowlisted primary sources, protocol docs not reachable, maintenance burden high for V1. PRODUCT.md says two ecosystems for V1, not five.
- **Why not:** Out of scope for V1, would violate scope discipline.

## Consequences

### Positive

- V1 candidate maximizes user reach with two distinct stacks (OS + vendor) while documenting security gaps and pinning path, not claiming full compliance prematurely.
- Both ecosystems support core capability-driven remote requirements: discovery, pairing, power via WoL/toggle, volume, D-pad, text input (Samsung text support added in v3.0.4 VERIFIED, Android sendText() IME injection VERIFIED), app launch, app list — enabling <2 min setup target [VERIFIED via primary libs].
- Android TV gives Google Cast natively separate from remote protocol, partially satisfying V1 casting/mirroring without cloud relay, though Samsung casting requires separate Smart View SDK/DIAL/Google Cast 2026 models validation (PARTIAL).
- Maintenance: samsungtvws active v3.0.6 2026-09-11 VERIFIED; Android TV protocol used by first-party Google TV app since 2021 (stable) but implementation kud/androidtv-remote is very new June 2026 and explicitly reverse-engineered per README credits [VERIFIED], so maintenance/legal risk to be assessed, not lowest burden.
- Clear honest compatibility story: Tested vs Expected vs Unsupported per PRODUCT.md, with Roku explicitly Unsupported for V1 due to security + primary docs restriction, with primary source.
- Leaves room for LG webOS as third ecosystem with minimal rework, preserving credible iPhone path (both protocols platform-agnostic).

### Negative

- V1 will not support Roku TVs at launch despite high US popularity — must communicate clearly with "found but not supported yet" UX per first-run journey step 4; now primary docs require user to manually enable Control by mobile apps as of OS 14.1.
- **Security validation PARTIAL — separated trust analysis, candidate mitigation not demonstrated:**
  - **Samsung Tizen:** Primary `connection.py` `ssl.CERT_NONE` VERIFIED. Token flow: first connection triggers on-TV popup, TV returns bearer token in `data.token` [VERIFIED]. No cryptographic binding of observed TLS certificate to token or to physical TV identity is shown in primary code — token is bearer secret sent over WSS where server cert verification is disabled. Therefore first-use MITM resistance is **unresolved**: attacker on LAN during first pairing could present different self-signed cert, intercept token, and impersonate TV. TOFU certificate pinning (capture cert fingerprint SHA256 after first pairing, store encrypted alongside token, verify on reconnect, fail-safe re-pair) is **candidate mitigation**, not demonstrated satisfying design. Requires hardware validation: cert stable across reboots/firmware, changes on factory reset → re-pair, MITM detection. Until proven, does not satisfy PRODUCT.md "If a secure connection cannot be established, refuse" and DOMAIN.md identity-change → re-pair.
  - **Android TV Remote v2:** Primary `pairing-manager.ts` `rejectUnauthorized:false` VERIFIED, `remote-manager.ts` same. However pairing computation **does** cryptographically bind cert material: `client.getCertificate()` and `client.getPeerCertificate()` moduli/exponents plus PIN are SHA-256 hashed, first byte checked against code, then secret hash sent [VERIFIED primary pairing-manager.ts lines]. This provides stronger first-use binding than Samsung because PIN is shown on physical TV and hash includes server cert material — MITM would need to present different cert and still produce hash that matches PIN-derived check. But first-use trust still depends on user verifying PIN on physical TV and no prior trust anchor; persistence of server cert identity for reconnect (pinning) must be verified. TOFU pinning (capture server cert fingerprint after pairing, store encrypted, verify on reconnect) is **candidate mitigation**, not demonstrated, and must not be generalized from Samsung evidence or third-party projects (TVgrip PR #2 per-TV serverCertSha256 encrypted, hafa-remote PR #15 pin exact cert are secondary evidence of viability, not proof for this protocol).
  - Both ecosystems remain PARTIAL until mechanism proven against PRODUCT.md/DOMAIN.md including unexpected identity change → fail safe and re-pair, and first-use MITM resistance explicitly evaluated.
- Legal coverage reduced: vendor developer portals blocked by egress allowlist, so terms evaluation relies on secondary summaries and GitHub primary sources; full legal validation requires human with portal access. Android TV is reverse-engineered, not official public Google API, so legal risk higher.
- Hardware matrix not yet executed — no real devices in sandbox, so all claims remain Expected, not Tested. Release still requires deliberate real-hardware matrix including pinning stability across reboots/firmware.
- IR fallback licensing: IRDB custom permission requires notifying via GitHub issue, including attribution notice, providing up to 3 fully licensed copies/units on request per primary LICENSE.md — state only what license requires, do not interpret Play Store/APK as satisfying unless confirmed; product-owner/legal acceptance still required; runtime CDN access recommended by README for updateability but does not remove obligations because license explicitly covers network access. LIRC database licensing unresolved (license null), not approved as shipping source, cannot be treated as legally cleared; coverage/size stats are secondary estimates, not verified from primary IRDB repo, removed from fixed constraints.
- Voice control: Samsung VoiceControl is on-TV Web API for Web apps, not remote WS API [SECONDARY Samsung docs]; Android remote-manager.ts comments "voice input not implemented" [VERIFIED primary code]; so V1 voice criterion NOT VERIFIED, may ship without voice per PRODUCT.md conditional.
- Governance: No explicit product-owner approval exists in durable GitHub record (Issue #7 open, no approval), so ecosystem choice must remain proposed candidates, not fixed V1 requirements. Accepting ADR merely to make PR easier to merge is prohibited.

### Follow-ups

- **Governance:** Obtain explicit product-owner approval on Issue #7 (or other durable approval record) before treating Android TV + Samsung Tizen as fixed V1 constraints; until then keep as proposed/leading candidates in PRODUCT.md and ADR.
- **Security — Samsung Tizen:** Determine from primary protocol evidence whether on-TV approval/token flow cryptographically binds observed TLS certificate or another stable device identity; if not, record first-use MITM resistance as unresolved; evaluate TOFU cert pinning as candidate mitigation with hardware validation (cert stable across reboots, changes on factory reset triggers re-pair, MITM detection).
- **Security — Android TV Remote v2:** Verify from primary protocol/code evidence (pairing-manager.ts SHA-256 over client/server cert moduli/exponents + PIN) whether PIN pairing computation cryptographically binds cert material and what identity can safely be persisted for reconnect; document first-use trust analysis separately from Samsung; evaluate TOFU pinning as candidate mitigation with hardware validation, not generalized from other ecosystem.
- Keep security validation PARTIAL until mechanism satisfies PRODUCT.md/DOMAIN.md including unexpected identity change → fail safe and re-pair.
- Update docs/PRODUCT.md V1 scope to reflect proposed candidates, PARTIAL security, PARTIAL casting, NOT VERIFIED voice, candidate IR data sources.
- Update research docs with primary sources and corrected claims, separating VERIFIED facts, secondary evidence, inference, proposed design, unresolved questions, product-owner decisions.
- Link PR #6 to issue #7 and record that product-owner approval does not yet exist in GitHub/project state — do not invent approval.
- Re-validate vendor terms/legal with human access to official portals before architecture; re-evaluate Roku primary docs restriction.
- Document Roku ECP as Unsupported for V1 in compatibility view with primary source and security rationale.
- Do not transition to architecture until security validation resolved or explicitly accepted as PARTIAL with ADR proposed, legal/terms validation completed, and product-owner approval exists.
- Complete product naming via separate research ADR, then set PROJECT_NAME/SLUG in config/project.env.
