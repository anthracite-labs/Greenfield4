# ADR-0005: V1 Smart-TV Ecosystems — Android TV/Google TV + Samsung Tizen

**Date:** 2026-09-14 (corrected 2026-09-14 session 2 per independent review #6)
**Status:** proposed
**Deciders:** Discovery research (session arena/01a0a053-greenfield4), product owner via docs/PRODUCT.md evidence bar, independent review feedback

## Context

PRODUCT.md V1 targets two major smart-TV ecosystems selected by evidence, not by uploaded ZIPs alone, with explicit validation of legal/terms, security model, protocol stability, pairing, wake, capabilities, and maintenance risk. Market data Q4 2024 from TechInsights primary: Samsung 16.9% vendor, LG 11.1%, Android/Google TV >24% OS shipments, Roku 9%. Security product requirements: security wins over popularity, pairing secrets treated like passwords, no global ignore-security mode, identity changes require safe re-pairing per DOMAIN.md. Research in docs/research/2026-09-14-ecosystem-evidence.md evaluated Samsung Tizen (samsungtvws v3.0.6 released 2026-09-11, WebSocket token auth, but connection.py uses ssl.CERT_NONE), LG webOS (LGWebOSRemote), Android TV Remote v2 (kud/androidtv-remote TypeScript, reverse-engineered not official public API, pairing-manager.ts and remote-manager.ts use rejectUnauthorized:false, voice not implemented), and Roku ECP (official primary docs developer.roku.com: Control by mobile apps must be Enabled as of OS 14.1, search sunset OS 12.0, in-app ECP sunset, "ECP commands may not be sent from 3rd-party platforms"). IR dataset licensing corrected: IRDB custom permission (notify via issue, attribution notice, up to 3 copies on request) per primary LICENSE.md https://github.com/probonopd/irdb/blob/master/LICENSE.md, not CC0; LIRC database licensing unresolved (license null). Coverage/size stats are secondary estimates from infishark blog, not verified from primary IRDB repo.

## Decision

Propose **Android TV / Google TV (Remote v2 protocol, reverse-engineered)** and **Samsung Tizen (2016+ WebSocket API, community reverse-engineered)** as V1 ecosystems, with security validation **PARTIAL** pending trustworthy persistent device identity pinning design and hardware validation. Defer LG webOS as immediate next candidate. Reject Roku ECP for V1 due to security model failure (no authentication, no secret, any LAN device can control if enabled) and primary docs restriction on 3rd-party mobile apps. Record hardware matrix criteria requiring at least two models/firmware per ecosystem for release, including pinning stability validation.

Status is **proposed** not accepted because security trust model for both selected ecosystems disables server certificate verification in reference implementations and requires additional evidence-backed pinning design to satisfy PRODUCT.md/DOMAIN.md identity-change/re-pairing requirements.

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
- **Security validation PARTIAL:** Both reference implementations disable server cert verification (Samsung ssl.CERT_NONE VERIFIED via connection.py, Android rejectUnauthorized:false VERIFIED via pairing-manager.ts and remote-manager.ts). MITM risk during pairing and reconnect unless pinning implemented. Requires TOFU pinning design: capture TV certificate fingerprint after first pairing (user-approved popup/PIN), store securely, verify on reconnect, fail safely and require re-pairing if identity changes, never silently trust new identity, per DOMAIN.md. Existing pinning implementations exist in other projects: TVgrip PR #2 "per-TV serverCertSha256 stored encrypted" and "DANE-style fingerprint pinning (no trust-all)" SECONDARY, hafa-remote PR #15 "pin exact certificate for reconnects" SECONDARY, indicating viability but not yet VERIFIED on Greenfield4 hardware.
- Legal coverage reduced: vendor developer portals blocked by egress allowlist, so terms evaluation relies on secondary summaries and GitHub primary sources; full legal validation requires human with portal access. Android TV is reverse-engineered, not official public Google API, so legal risk higher.
- Hardware matrix not yet executed — no real devices in sandbox, so all claims remain Expected, not Tested. Release still requires deliberate real-hardware matrix including pinning stability across reboots/firmware.
- IR fallback licensing: IRDB custom permission requires notifying via GitHub issue, including attribution notice, providing up to 3 copies on request — acceptable for free app but must be tracked; LIRC database licensing unresolved, cannot be treated as legally cleared; coverage/size stats are secondary estimates, not verified from primary IRDB repo, removed from fixed constraints.
- Voice control: Samsung VoiceControl is on-TV Web API for Web apps, not remote WS API [SECONDARY Samsung docs]; Android remote-manager.ts comments "voice input not implemented" [VERIFIED primary code]; so V1 voice criterion NOT VERIFIED, may ship without voice per PRODUCT.md conditional.

### Follow-ups

- Implement and document pinning design for Samsung Tizen (capture self-signed cert keyed to UUID, pin fingerprint) and Android TV (capture server cert via getPeerCertificate, store SHA256, verify on reconnect) with hardware validation: confirm cert stable across reboots, changes on factory reset triggers re-pair, MITM detection test.
- Update docs/PRODUCT.md V1 scope to reflect PARTIAL security, PARTIAL casting, NOT VERIFIED voice, corrected IR licensing.
- Update research docs with primary sources and corrected claims.
- Link PR #6 to issue #7 (discovery issue with acceptance criteria) and record that product-owner approval does not yet exist in GitHub/project state.
- Re-validate vendor terms/legal with human access to official portals before architecture; re-evaluate Roku primary docs restriction.
- Document Roku ECP as Unsupported for V1 in compatibility view with primary source and security rationale.
- Do not transition to architecture until security validation resolved or explicitly accepted as PARTIAL with ADR proposed, and legal/terms validation completed.
- Complete product naming via separate research ADR, then set PROJECT_NAME/SLUG in config/project.env.
