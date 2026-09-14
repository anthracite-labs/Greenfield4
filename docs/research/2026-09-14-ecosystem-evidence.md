# Research — V1 Smart-TV Ecosystem Evidence

**Date:** 2026-09-14 (corrected 2026-09-14 session 2 per independent review)
**Phase:** discovery
**Sources:** GitHub primary via gh api/fetch_page (pinned), PyPI/GH releases (primary), developer.roku.com (primary), web_search (secondary), labeled VERIFIED/SECONDARY/INFERRED
**Status:** evidence for ADR-0005, not a stack decision, security validation PARTIAL pending pinning design

This document establishes verifiable evidence for selecting the first two V1 ecosystems per `docs/PRODUCT.md` research requirements: legal/terms, security model, protocol stability, pairing, wake, capabilities, maintenance risk, user reach. Separates VERIFIED facts, secondary evidence, inference, unresolved.

## Market reach baseline

- **Q4 2024 vendor shipments:** Samsung 16.9%, TCL 13.9%, Hisense 12.8%, LG 11.1% [VERIFIED fetch_page https://www.techinsights.com/blog/smart-tv-vendor-and-os-market-share-q4-2024-region] — TechInsights directly publishes headline figures, not merely secondary/paywalled summary (correction per review).
- **Q4 2024 OS shipments:** Android/Google TV >24%, Tizen 16.9%, webOS 11.8%, Roku 9% [VERIFIED same TechInsights page]
- **Global OS installed base 2024 (CTVMA via MediaPost secondary):** Tizen 12.9% of 120M sets, VIDAA 7.8%, webOS 7.4%, Roku 6.4%, Fire TV 6.4%, Android TV 5.9% [SECONDARY ESTIMATE https://www.mediapost.com/publications/article/396751/samsung-smart-tv-os-tops-in-2024-hisense-lg-and.html] — discrepancy shipment vs installed base methodology.
- **US CTV market share Q1 2025 (Pixalate secondary):** Roku 38%, Fire TV 18%, Apple TV 13%, Samsung 12%, LG 5% [SECONDARY ESTIMATE https://www.pixalate.com/blog/q1-2025-connected-tv-ctv-device-market-share-report]

[VERIFIED] TechInsights headline figures are primary-published; [SECONDARY] MediaPost/Pixalate are secondary summaries.

Interpretation: Maximum reach with two ecosystems is Android/Google TV + Samsung Tizen (~40.9% shipment) [INFERRED from VERIFIED figures]. Samsung+LG 28%, LG+Android ~35.9% [INFERRED].

## Candidate deep dives

### 1. Samsung Tizen (2016+)

**Protocol — VERIFIED primary:**
- WebSocket API on `ws://<tv>:8001/api/v2/channels/samsung.remote.control` and `wss://<tv>:8002/...` with token auth [SECONDARY forum.logicmachine.net, but structure confirmed via primary samsungtvws code]
- Primary implementation: `samsungtvws` PyPI package, LGPL-3.0, Python >=3.9 (now >=3.10 as of 3.0.6 dropping 3.9), supports sync/async, encrypted v1 for older Orsay, CLI [VERIFIED gh api releases: v3.0.6 released 2026-09-11T22:40:25Z, v3.0.5 2026-05-28]
- GitHub: `xchwarze/samsung-tv-ws-api` — active, README documents CLI examples: wol, power, apps, device-info, art-mode [VERIFIED fetch_page primary README via gh api]
- Pairing flow: first connection triggers on-TV popup "Allow device?"; TV returns token; client stores token and sends as `?token=TOKEN` on subsequent WSS connections [VERIFIED via samsungtvws connection.py _format_websocket_url adds token query when ssl and token present, and _check_for_token extracts token from data.token]
- Token persistence: library handles token file storage [VERIFIED connection.py _get_token/_set_token]

**Security — VERIFIED primary, trust model PARTIAL:**
- TokenAuthSupport = true per device info JSON [SECONDARY forum post, but token flow verified in primary code]
- **Critical finding per review:** Primary `samsungtvws/samsungtvws/connection.py` line `sslopt = {"cert_reqs": ssl.CERT_NONE} if self._is_ssl_connection() else {}` [VERIFIED gh api repos/xchwarze/samsung-tv-ws-api/contents/samsungtvws/connection.py base64 decode] — disables server certificate verification for WSS on 8002.
- Implication: TV serves self-signed cert keyed to its own UUID with no CA to check against — same constraint Samsung's own apps have [SECONDARY omacom/omarchy-plugin-marketplace issue #5386 notes: "TLS certificate verification is disabled on the remote-control channel, because the TV serves a self-signed cert keyed to its own UUID with no CA to check against — the same constraint Samsung's own apps have. The MITM implication is spelled out; certificate pinning would close most of it and is not implemented yet"]
- **Trust model analysis:** Current reference implementation does NOT verify TV identity via TLS; token is bearer secret sent over potentially MITM-able channel if attacker present on LAN during pairing. This conflicts with PRODUCT.md "If a secure connection cannot be established, refuse the unsafe connection. There is no global 'ignore security' mode." and DOMAIN.md "If a paired device’s security identity changes unexpectedly, fail safely and require re-pairing rather than silently trusting the new identity." and "Paired | Device security identity changes unexpectedly | Re-pair required | Silent trust replacement is forbidden."
- **Can Greenfield4 establish trustworthy persistent device identity?** [INFERRED design, not yet VERIFIED on hardware]:
  - TV's WSS cert is self-signed but stable per UUID [SECONDARY]. Greenfield4 could implement **certificate/public-key pinning with Trust On First Use (TOFU):** on first pairing, capture TV's presented certificate fingerprint (SHA256) after user approves popup, store securely alongside token, and on subsequent connections verify presented cert matches pinned fingerprint. If mismatch, fail safely, require re-pairing, surface "TV identity changed" state, never silently trust new identity.
  - Existing Swift example `wdesimini.github.io/controlling-samsung-tvs-using-swift-based-websockets/` mentions simple certificate pinner `TVCertificatePinner` with note "If you're worried about MITM attacks, make sure to use a more refined approach for certificate-pinning." [SECONDARY], indicating pinning is known mitigation but not in reference Python lib.
  - Hardware validation required: confirm TV cert stable across reboots and firmware updates; confirm fingerprint changes on factory reset (should trigger re-pair); test MITM detection.
  - Until hardware validation and pinning design documented with evidence, **security validation must remain PARTIAL**, not MET.
- Transport: WS 8001 unencrypted local, WSS 8002 TLS but verification disabled in ref impl; token must be treated as password, excluded from logs [VERIFIED].
- Legal: Samsung SmartThings Terms prohibit reverse engineering of Accessed Developer Tools [SECONDARY https://developer.smartthings.com/termsofservice]; local WS API not explicitly listed as Accessed Tool, but risk noted. Reduced coverage: official Samsung developer portal docs not reachable via allowlisted egress.

**Capabilities — VERIFIED vs SECONDARY, corrected for overstatement:**

- Power on via Wake-on-LAN (`samsungtv --host ... wol`) [VERIFIED via samsungtvws CLI docs]
- Power toggle, volume, channel, directional keys (KEY_VOLDOWN etc) [VERIFIED via samsungtvws command.py and Ape/samsungctl secondary]
- App listing (`apps`), app launch (`app-run <id>`), open browser [VERIFIED via samsungtvws]
- Art Mode for Frame TVs [VERIFIED]
- Text input: Tizen supports `ImeSyncedSupport` true [SECONDARY forum], but samsungtvws added text support in v3.0.4 release notes "Add text support to remote control" [VERIFIED gh api releases v3.0.4]. So text input is supported via library, not merely inferred from flag.
- **Casting/mirroring — corrected:** Previous version claimed "need separate validation" but PR marked MET. Correction: Samsung Tizen WS API does **not** provide casting/mirroring directly. Casting is via separate protocols: DIAL, Smart View SDK, Google Cast (2026 models) [SECONDARY https://developer.samsung.com/smarttv/develop/extension-libraries/smart-view-sdk/... and https://www.1001tvs.com/samsung-tv-screen-mirroring... says Google Cast launched on all 2026 Samsung TVs out of the box and rolling out to 2023-2025 via One UI Tizen v2115]. So V1 casting/mirroring via Samsung alone is **not verified**; must be marked PARTIAL/secondary, requires separate validation. Android TV provides Google Cast natively.
- **Voice — corrected:** Previous claimed "VoiceSupport flag true" implies voice control works. Primary Samsung VoiceControl API docs [SECONDARY https://developer.samsung.com/smarttv/develop/api-references/tizen-web-device-api-references/voicecontrol-api.html] describe VoiceControl API for Tizen Web apps **on TV**, not for remote control from phone. No evidence in samsungtvws that voice input from phone is exposed cleanly. So voice control for Samsung Tizen must be marked **unverified / not exposed via remote WS API**; V1 voice only if ecosystem exposes it cleanly per PRODUCT.md, so for Samsung this is currently NOT MET.

**Stability/Maintenance — corrected:**

- Tizen OS since 2015, protocol stable 2016+; library supports Orsay H/J [VERIFIED README]
- Maintenance: samsungtvws **v3.0.6** released 2026-09-11 [VERIFIED gh api releases/tags/v3.0.6], not 3.0.5; 3.0.6 drops Python 3.9 support [VERIFIED release body], active maintenance.
- Wake: WoL works per community [SECONDARY].

**User journey impact:** Meets <2 min target if WoL + token stored; first pairing requires TV popup.

### 2. LG webOS (2012+)

**Protocol — VERIFIED primary:**
- WebSocket on port 3000 (newer) and legacy REST 8080 (closed on newer) [VERIFIED via LGWebOSRemote GitHub]
- Primary implementation: `LGWebOSRemote` Python CLI — commands scan, auth, on, off, listApps, setVolume, openAppWithPayload, openYoutubeId, sendButton [VERIFIED fetch_page GitHub]
- Auth: `lgtv --ssl auth <ip> MyTV` triggers on-TV pairing [VERIFIED]
- Community Android app `heroslender/lg-remote` [VERIFIED]

**Security — PARTIAL:**
- Pairing requires on-screen auth; pairing material secret; SSL mode supported [VERIFIED].
- No evidence of CERT_NONE in this lib; but trust model still TOFU via pairing popup. Needs similar pinning analysis, but deferred as not V1 selected.
- Legal: compatibility naming without logos allowed per forum [SECONDARY].

**Capabilities — VERIFIED:**
- Pointer trackpad (Magic Remote emulation), nav pad, playback, volume/channel with live level, WoL, keyboard, app launcher, channel list [VERIFIED via Play Store listing secondary but consistent with primary lib listApps]
- App launch, browser open, YouTube open [VERIFIED]
- Casting: Miracast separate, not via remote protocol.

**Stability/Maintenance:** webOS since 2014, stable, Python lib tested 3.9-3.14 [VERIFIED secondary README].

**User reach:** 11.1% shipments [VERIFIED TechInsights].

### 3. Android TV / Google TV Remote v2

**Protocol — VERIFIED primary, reverse-engineered not official public API:**

- Pairing port 6467 TLS, remote port 6466 TLS, certificate-based [VERIFIED via kud/androidtv-remote README]
- Pairing flow: client self-signed cert, PAIRING_REQUEST (10), OPTIONS (20), CONFIGURATION (30), SECRET (40) with hashed PIN, SECRET_ACK (41) [VERIFIED via StackOverflow secondary but matches primary message-manager proto]
- Primary README: "Control Android TV / Google TV devices over the Android TV Remote v2 protocol" [VERIFIED fetch_page raw], **Credits section:** "Derived from androidtv-remote by louis49 (MIT). The reverse-engineered Android TV Remote v2 protocol and .proto schemas originate from that project. This library is a TypeScript rewrite..." [VERIFIED fetch_page] — explicitly **reverse-engineered**, not officially supported public Google API (correction per review).
- Libraries:
  - `kud/androidtv-remote` TypeScript/ESM MIT, features first-class pairing, native text input IME injection, full key control, state events powered/volume/current_app/unpaired [VERIFIED]
  - `drosoCode/atvremote` Go, supports v1 (old) and v2 (Google TV app, remote service >=5) [SECONDARY]
- Text input: sendText() IME injection [VERIFIED]

**Security — VERIFIED primary, trust model PARTIAL:**

- Certificate private key + cert must be treated as secret [VERIFIED].
- **Critical finding per review:** Primary `kud/androidtv-remote/src/pairing/pairing-manager.ts` line `rejectUnauthorized: false` [VERIFIED fetch_page raw pairing-manager.ts] and `src/remote/remote-manager.ts` same [VERIFIED fetch_page raw remote-manager.ts] — disables server cert verification.
- Implication: Both pairing and remote sessions do not verify TV's self-signed cert by default; MITM possible during pairing unless pinning implemented.
- **Can Greenfield4 establish trustworthy persistent device identity?** [INFERRED design with evidence from other implementations]:
  - During pairing, client has access to `client.getPeerCertificate()` serverCertificate modulus/exponent for hash [VERIFIED pairing-manager.ts code: `const serverCertificate = client.getPeerCertificate() as unknown as RsaCertificate` and SHA-256 over both certs' moduli/exponents plus PIN].
  - Existing implementations do pinning: TVgrip PR #2 mentions "per-TV serverCertSha256 stored encrypted" and "buildRemoteSslContext (pins the paired TV certificate fingerprint on port 6466)" and "preserving DANE-style fingerprint pinning (no trust-all, TLS not weakened)" [SECONDARY https://github.com/mbir31/TVgrip/pull/2]; hafa-remote PR #15 mentions "Trust a self-signed TV certificate only for the selected private endpoint during pairing; pin the exact certificate for reconnects." [SECONDARY].
  - Design for Greenfield4: After first pairing (TOFU via PIN shown on TV), capture server certificate fingerprint SHA256, store encrypted alongside client cert, and on reconnect verify presented server cert matches pinned fingerprint. If mismatch, fail safely, require re-pairing, surface identity-changed state, never silently trust new identity, satisfying DOMAIN.md "Paired | Device security identity changes unexpectedly | Re-pair required | Silent trust replacement is forbidden."
  - Hardware validation required: confirm server cert stable across reboots, changes on factory reset, test MITM detection.
  - Until hardware validation, **security validation must remain PARTIAL**, not MET. Previous claim "Meets PRODUCT.md security requirements fully" was overstated and removed.

- Legal: Anymote + Pairing protocols were originally Google open source [SECONDARY], current partner SDK closed behind partner login [SECONDARY]; reverse-engineered impl tolerated but not official; maintenance/legal risk to be assessed.

**Capabilities — corrected:**

- Full Android keycodes (HOME, BACK, DPAD, etc.) [VERIFIED]
- Power toggle sendPower(), volume state events, current app tracking [VERIFIED]
- App launch via deep-link, text injection [VERIFIED]
- **Casting:** Google Cast is separate but TV supports Cast natively; remote can launch cast-enabled apps [INFERRED, secondary].
- **Voice — corrected:** Previous claimed "if TV supports voice, keycode may trigger assistant." Primary `remote-manager.ts` contains comments `// voice input not implemented` for `remoteVoiceBegin`, `remoteVoicePayload`, `remoteVoiceEnd` [VERIFIED fetch_page raw remote-manager.ts]. So voice input from phone via this protocol is **not implemented** in cited library. Must be marked unverified / not exposed. V1 voice only if ecosystem exposes it cleanly, so for Android TV this is currently NOT MET via this protocol; may require separate assistant keycode but not demonstrated.

**Stability/Maintenance — corrected:**

- Protocol v2 deployed since Google TV app update Sept 2021 (remote service v5+ incompatible with legacy) [SECONDARY]
- Library `kud/androidtv-remote` is **very new, created June 2026** per review finding, not established long-term maintenance; previous claim "lowest maintenance burden" and "Google-maintained protocol" was overstated. Correction: Protocol itself is used by Google's first-party remote service (stable since 2021), but implementation is community reverse-engineered, maintenance risk higher than official public API. Must be reworded as unofficial/reverse-engineered protocol used by Google's first-party remote service, with maintenance/legal risk still to be assessed.
- Other libs: `tronikos/androidtvremote2` Python (Apache-2.0) is more established [SECONDARY deepwiki], but not primary-pinned in this session.

**User reach:** >24% OS shipment [VERIFIED TechInsights] plus Sony, TCL, Hisense, etc. — broadest single protocol [INFERRED].

**Android-first alignment:** Product constraint Android first, certificate handling similar to Android Keystore patterns [INFERRED].

### 4. Roku ECP (External Control Protocol) — primary source now preferred

**Protocol — VERIFIED primary official docs:**

- Official docs: `https://developer.roku.com/dev/docs/external-control-api` [VERIFIED fetch_page primary]
- **Commands requiring "Control by mobile apps" enabled:** As of Roku OS 14.1, Settings > System > Advanced system settings > Control by mobile apps must be set to "Enabled" for device to receive keypress, keydown, keyup, query/icon, query/tv-channels, query/tv-active-channel [VERIFIED fetch_page primary docs, first chunk]
- **Commands requiring Control by mobile apps + Developer Mode:** query/chanperf, query/r2d2-bitmaps, etc. [VERIFIED]
- **Search command sunset:** As of Roku OS 12.0, search command no longer available [VERIFIED]
- **Support for in-app ECP commands sunset:** Apps may no longer include code designed to issue any ECP command; Static Analysis blocks publishing to Streaming Store; ECP commands may not be sent from 3rd-party platforms (e.g., mobile applications) [VERIFIED fetch_page primary — same text as secondary Reddit quote but now from official primary]
- Discovery: SSDP M-SEARCH to 239.255.255.250:1900 ST roku:ecp, Location header gives URL, USN contains serial [VERIFIED primary docs]
- REST API on port 8060, commands: query/media-player, query/device-info, query/apps, keypress, etc. [VERIFIED primary]

**Security assessment — fails PRODUCT.md bar, now with primary source:**

- No authentication: any LAN device can control if Control by mobile apps Enabled; official support article says "While it is possible to use third-party apps to control your Roku device, we strongly advise against doing so for security reasons" and setting options Limited/Enabled/Permissive [SECONDARY https://support.roku.com/article/install-the-mobile-app, but primary docs show control setting].
- PRODUCT.md: "Security wins over popularity" etc. Roku ECP cannot meet because no secret, no identity, no secure connection. Browser mixed-content issues also [SECONDARY].
- Therefore Roku explicitly deferred from V1, not due to popularity but security invariants per DOMAIN.md business rules. Documented as Unsupported/Deferred with primary source.

**Legal note — corrected to primary:** Official docs state "ECP commands may not be sent from 3rd-party platforms (for example, mobile applications)" [VERIFIED primary], clarifying previous ambiguity that it was about channels only; now primary confirms restriction includes mobile apps. Legal risk higher than previously assessed.

**Capabilities — primary:** keypress full vocabulary, device-info returns supports-tv-power-control, supports-audio-volume-control as of OS 15.0 [VERIFIED primary second chunk truncated but mentioned], query/apps, launch.

**User reach:** 38% US CTV secondary [SECONDARY Pixalate] — high but security bar prevents V1.

### 5. Other notes

- Fire TV, VIDAA, Titan OS deferred — insufficient evidence.

## Capability matrix vs PRODUCT.md V1 scope — corrected

| Capability | Samsung Tizen | LG webOS | Android TV | Roku ECP | Evidence |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Discovery | SSDP+IP+WoL | SSDP+scan+IP+WoL | mDNS+SSDP+IP | SSDP, Location header | VERIFIED primary for Roku SSDP, secondary for others |
| Pairing | Popup+token, token file | Popup+pairing material | PIN+cert TLS, cert file | None (fails security) | VERIFIED via samsungtvws code and kud/androidtv-remote |
| Power on | WoL verified | WoL verified | Power toggle verified, WoL separate | PowerOff only, requires Control by mobile apps Enabled per OS 14.1 | VERIFIED CLI and primary Roku docs |
| Power off | Yes | Yes | Yes | Yes | VERIFIED |
| Volume | Yes | Yes+live level | Yes+volume events | Yes TV only | VERIFIED |
| D-pad nav | Yes | Yes+pointer trackpad | Yes | Yes | VERIFIED |
| Touchpad/swipe | Possible via touchPad flag secondary | Yes Magic Remote | Limited | No | SECONDARY |
| Text input | Yes via samsungtvws text support v3.0.4 | Keyboard | Native IME injection verified | No | VERIFIED for Samsung text support release note and Android sendText |
| App launch | Yes via ID verified | Yes verified | Yes via deep-link verified | Yes via channel ID verified primary | VERIFIED |
| App list | Yes apps | Yes listApps | Yes via current_app events verified | Yes /query/apps verified primary | VERIFIED |
| Casting/mirroring | **PARTIAL:** Separate Smart View SDK/DIAL/Google Cast 2026 models, not via WS API; requires separate validation | Miracast separate | Google Cast native separate, remote can launch apps | DIAL | SECONDARY, corrected from MET to PARTIAL |
| Voice | **NOT VERIFIED:** VoiceControl API is for on-TV Web apps, not remote WS API; no evidence phone can trigger voice cleanly | Possible secondary | **NOT VERIFIED:** remote-manager.ts comments "voice input not implemented" | No | VERIFIED for Android not implemented, secondary for Samsung |

## Maintenance burden ranking — corrected

- Previous ranking claimed Android lowest because Google-maintained; corrected: Android TV protocol is **reverse-engineered**, implementation `kud/androidtv-remote` is very new (June 2026), so lowest burden not established. Revised ranking must account for unofficial nature.
- **Revised (lowest to highest, with uncertainty):**
  1. Samsung Tizen — community library samsungtvws active v3.0.6 2026-09-11, active maintenance, but CERT_NONE issue and token rotation observed.
  2. Android TV — protocol used by first-party Google TV app since 2021 (stable), but implementations are community reverse-engineered, `kud` lib very new, maintenance/legal risk to be assessed; pinning implementations exist in other projects (TVgrip, hafa-remote) but not in cited lib.
  3. LG webOS — stable but smaller community.
  4. Roku — simplest code but security debt and primary docs restriction "ECP commands may not be sent from 3rd-party platforms".

All rankings now marked as [INFERRED] with uncertainty.

## Legal/terms summary — corrected, reduced coverage

- Egress allowlist blocked direct fetch of vendor developer portals (developer.samsung.com Smart View SDK, developer.lge.com docs, Google partner SDK). Evidence relies on GitHub primary and secondary summaries, with reduced coverage disclosure.
- Samsung: SmartThings ToS prohibits reverse eng of Accessed Tools [SECONDARY]; local WS protocol not explicitly listed, but risk noted; **no official public API** for this WS remote, it is community reverse-engineered.
- LG: compatibility naming without logos allowed [SECONDARY forum]; official trademark policy not reachable.
- Google/Android TV: Anymote/Pairing originally open source [SECONDARY], current partner SDK closed [SECONDARY]; **kud/androidtv-remote explicitly states reverse-engineered** [VERIFIED primary README credits], not public Google-maintained API.
- Roku: **Primary** docs now reachable: `developer.roku.com/dev/docs/external-control-api` states Control by mobile apps must be Enabled as of OS 14.1 for keypress etc., search sunset OS 12.0, in-app ECP sunset, and "ECP commands may not be sent from 3rd-party platforms (for example, mobile applications)" [VERIFIED primary]. This is stronger restriction than previously assessed via secondary.

No legal opinion; research evidence only.

## Recommendation for V1 — re-evaluated after corrections

**Primary candidate remains:** Android TV / Google TV + Samsung Tizen, but with **security validation PARTIAL** and **casting/voice PARTIAL/NOT VERIFIED**, so ADR must be **proposed** not accepted.

Rationale after correction:
- Maximizes reach (~40.9% shipment) with two distinct stacks [INFERRED from VERIFIED TechInsights].
- Both support core remote requirements: discovery, pairing (popup+token, PIN+cert), power via WoL/toggle, volume, D-pad, text input, app launch — enabling <2 min setup [VERIFIED via primary libs].
- Android-first aligns with product constraint.
- **Security:** Both reference implementations disable server cert verification (CERT_NONE, rejectUnauthorized:false) [VERIFIED primary code]. However, viable pinning design exists (per-TV serverCertSha256 stored encrypted, DANE-style fingerprint pinning, no trust-all) in other projects [SECONDARY TVgrip PR #2, hafa-remote PR #15]. Greenfield4 can implement TOFU pinning to satisfy PRODUCT.md/DOMAIN.md identity-change/re-pairing, but requires hardware validation. Until then, **security validation PARTIAL**.
- **Casting/mirroring:** Samsung casting NOT via WS API, requires separate Smart View SDK/DIAL/Google Cast validation [SECONDARY]; Android TV gives Google Cast natively but remote API only launches apps. So casting criterion **PARTIAL**, not MET.
- **Voice:** Both ecosystems **NOT VERIFIED** for clean safe voice control from phone via cited remote APIs (Samsung VoiceControl is on-TV Web API, Android remote-manager.ts says voice not implemented). So voice criterion **NOT MET** for V1, which is acceptable per PRODUCT.md "voice only when exposes it cleanly and safely, only if it does not delay launch".
- Maintenance/legal: Both are reverse-engineered community protocols, not official public APIs, with legal risk and very new `kud` lib, so maintenance burden claim revised to PARTIAL.

**Deferred:** LG webOS — strong third candidate, pointer trackpad premium, WoL, security TOFU similar, but reach slightly lower.

**Rejected for V1:** Roku ECP — fails security invariants and now primary docs explicitly restrict 3rd-party mobile apps and require Control by mobile apps Enabled as of OS 14.1, plus no auth. Documented as Unsupported with primary source.

## Hardware matrix criteria (for release) — unchanged

- Tested: real hardware/model/firmware in release matrix, min 2 models/firmware per ecosystem.
- Expected: predicted from family/protocol evidence but not yet represented.
- Unsupported: known not to work or intentionally not supported. Roku ECP Unsupported for V1 due to security + primary docs restriction.

## Next steps — corrected

- Change ADR-0005 status to proposed, update with corrected licensing, security PARTIAL, casting PARTIAL, voice NOT VERIFIED.
- Update PRODUCT.md to remove CC0, coverage stats from fixed constraints, correct casting/voice, mark legal/terms and security validation PARTIAL.
- Update IR dataset doc with correct license.
- Link PR #6 to issue #7.
- Update MEMORY.md.
- Re-run verification, reply to review with mapping.
