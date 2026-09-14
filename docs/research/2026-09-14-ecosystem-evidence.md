# Research — V1 Smart-TV Ecosystem Evidence

**Date:** 2026-09-14
**Phase:** discovery
**Sources:** web_search (secondary), GitHub repos (primary), pypi.org (primary for package metadata)
**Status:** evidence for ADR-0005, not a stack decision

This document establishes verifiable evidence for selecting the first two V1 ecosystems per `docs/PRODUCT.md` research requirements: legal/terms, security model, protocol stability, pairing, wake, capabilities, maintenance risk, user reach.

## Market reach baseline

- **Q4 2024 vendor shipments (TechInsights, via web_search):** Samsung 16.9%, TCL 13.9%, Hisense 12.8%, LG 11.1% [1](https://www.techinsights.com/blog/smart-tv-vendor-and-os-market-share-q4-2024-region)
- **Q4 2024 OS shipments:** Android/Google TV >24%, Tizen 16.9%, webOS 11.8%, Roku 9% [1](https://www.techinsights.com/blog/smart-tv-vendor-and-os-market-share-q4-2024-region)
- **Global OS installed base 2024 (CTVMA via MediaPost):** Tizen 12.9% of 120M sets, VIDAA 7.8%, webOS 7.4%, Roku 6.4%, Fire TV 6.4%, Android TV 5.9% [5](https://www.mediapost.com/publications/article/396751/samsung-smart-tv-os-tops-in-2024-hisense-lg-and.html) — note discrepancy between shipment vs installed base methodology; shipment data is more current for V1 planning.
- **US CTV market share Q1 2025 (Pixalate):** Roku 38%, Fire TV 18%, Apple TV 13%, Samsung 12%, LG 5% [2](https://www.pixalate.com/blog/q1-2025-connected-tv-ctv-device-market-share-report)

[VERIFIED web_search depth 2] market figures are secondary evidence; primary shipment reports are paywalled, so figures are cited as reported by secondary summary.

Interpretation: Maximum reach with two ecosystems is Android/Google TV + Samsung Tizen (~40.9% shipment share). Samsung + LG = 28% vendor, LG + Android = ~35.9%.

## Candidate deep dives

### 1. Samsung Tizen (2016+)

**Protocol:**
- WebSocket API on `ws://<tv>:8001/api/v2/channels/samsung.remote.control` and `wss://<tv>:8002/...` with token auth [7](https://forum.logicmachine.net/printthread.php?tid=4606)
- Primary implementation: `samsungtvws` PyPI package v3.0.5, LGPL-3.0, Python >=3.9, supports sync/async, encrypted v1 for older Orsay, CLI [1](https://pypi.org/project/samsungtvws/)
- GitHub: `xchwarze/samsung-tv-ws-api` — active, README documents CLI examples: wol, power, apps, device-info, art-mode [2](https://github.com/xchwarze/samsung-tv-ws-api)
- Pairing flow: first connection triggers on-TV popup "Allow device?"; TV returns token; client must store token and send as `?token=TOKEN` on subsequent connections [7](https://forum.logicmachine.net/printthread.php?tid=4606)
- Token persistence: library handles token storage; token is secret per DOMAIN.md "Paired device" invariants.

**Security:**
- TokenAuthSupport = true per device info JSON [7](https://forum.logicmachine.net/printthread.php?tid=4606)
- No global "ignore security" — pairing required. Meets PRODUCT.md "Security wins over popularity" and "If a paired device’s security identity changes unexpectedly, fail safely and require re-pairing".
- Transport: WS (port 8001) unencrypted local, WSS (8002) TLS; token is bearer secret. Must treat token like password, exclude from logs.
- Legal: Samsung SmartThings Terms prohibit reverse engineering of Accessed Developer Tools [5](https://developer.smartthings.com/termsofservice), but Tizen local WS API is not documented as part of SmartThings developer tools; it is reverse-engineered community protocol. Third-party remote apps widely exist but not Knox certified [4](https://electronics.alibaba.com/question/samsung-tv-remote-app-best-options-setup-guide). Risk: Samsung could change protocol or assert terms; mitigation: document as community protocol, not official SDK.

**Capabilities (from samsungtvws README + awesome-smart-tv list):**
- Power on via Wake-on-LAN (`samsungtv --host ... wol`) [1](https://pypi.org/project/samsungtvws/)
- Power toggle, volume, channel, directional keys (KEY_VOLDOWN etc) [10](https://github.com/Ape/samsungctl)
- App listing (`apps`), app launch (`app-run <id>`), open browser [1](https://pypi.org/project/samsungtvws/)
- Art Mode for Frame TVs [1](https://pypi.org/project/samsungtvws/)
- Text input via IME? Tizen supports `ImeSyncedSupport` true [7](https://forum.logicmachine.net/printthread.php?tid=4606) — implies keyboard input possible.
- Casting: not directly via this API; casting is separate (SmartThings or DIAL). V1 scope says casting/mirroring included — need separate validation.

**Stability/Maintenance:**
- Tizen OS since 2015, protocol stable 2016+; library supports Orsay H/J series (2014-2015) via encrypted API [2](https://github.com/xchwarze/samsung-tv-ws-api)
- Maintenance: samsungtvws 3.0.5 released 2026-05-28 [1](https://pypi.org/project/samsungtvws/), active.
- Wake: WoL works per community reports [7](https://forum.logicmachine.net/printthread.php?tid=4606)

**User journey impact:** Meets <2 min target if WoL + token stored; first pairing requires TV popup, similar to PRODUCT.md step 5.

### 2. LG webOS (2012+)

**Protocol:**
- WebSocket on port 3000 (newer firmware) and legacy REST on 8080 (closed on newer) [6](https://github.com/klattimer/LGWebOSRemote)
- Primary implementation: `LGWebOSRemote` Python CLI — commands: `scan`, `auth`, `on`, `off`, `listApps`, `setVolume`, `openAppWithPayload`, `openYoutubeId`, `sendButton`, etc. [6](https://github.com/klattimer/LGWebOSRemote)
- Auth: `lgtv --ssl auth <ip> MyTV` triggers on-TV pairing request; stores pairing material [6](https://github.com/klattimer/LGWebOSRemote)
- Community Android app `heroslender/lg-remote` — ad-free, controls webOS via Wi-Fi [5](https://github.com/heroslender/lg-remote)

**Security:**
- Pairing requires on-screen authorization; pairing material stored as secret.
- SSL mode (`--ssl`) supported; token/certificate must be protected.
- Legal: LG webOS forum thread asks "Do I need LG permission to publish remote app?" — answer indicates compatibility naming ("for LG TV") without logos is allowed if no official logos/graphics/restricted SDKs used [3](https://forum.webostv.developer.lge.com/t/do-i-need-lg-permission-to-publish-a-remote-control-app-for-lg-tvs-on-galaxy-store/28119). No explicit prohibition found in reachable terms, but must report reduced coverage: official LG developer terms portal not reachable via allowlisted egress.
- Meets security product requirement.

**Capabilities (from LGWebOSRemote + Play Store listings):**
- Pointer trackpad (Magic Remote emulation), full nav pad, playback, volume/channel with live level, power on/off via Wake-on-LAN, on-screen keyboard, app launcher, channel list, inputs, settings [2](https://play.google.com/store/apps/details?id=dev.niamor.webosremote&hl=en_US)
- App launch, browser open, YouTube open via URL/ID [6](https://github.com/klattimer/LGWebOSRemote)
- Keyboard input: Play Store app lists on-screen keyboard [2](https://play.google.com/store/apps/details?id=dev.niamor.webosremote&hl=en_US)
- Casting: LG supports Miracast, but remote protocol focuses on control.

**Stability/Maintenance:**
- webOS since 2014, protocol stable; Python tool tested on 3.9-3.14, Windows/Linux/macOS [6](https://github.com/klattimer/LGWebOSRemote)
- Community maintained, but less active than Samsung; still viable.
- Wake: WoL supported, plus `on` command requires MAC [6](https://github.com/klattimer/LGWebOSRemote)

**User reach:** 11.1% shipments, premium OLED leadership, strong EU presence.

### 3. Android TV / Google TV Remote v2 (current)

**Protocol:**
- Pairing port 6467 TLS, remote port 6466 TLS, certificate-based [1](https://github.com/kud/androidtv-remote)
- Pairing flow: client generates self-signed cert (OpenSSL), sends PAIRING_REQUEST (type 10), OPTIONS (20), CONFIGURATION (30), then SECRET (40) with hashed PIN (4-char code shown on TV). Server returns SECRET_ACK (41) [8](https://stackoverflow.com/questions/57809046/is-there-an-api-or-sdk-to-create-a-remote-control-application-on-smartphone-for)
- Libraries:
  - `kud/androidtv-remote` TypeScript/ESM, MIT, modern implementation, features: first-class pairing, native text input via IME, full key control, state events (powered, volume, current_app, unpaired) [1](https://github.com/kud/androidtv-remote)
  - `drosoCode/atvremote` Go, supports v1 (old Android TV Remote app) and v2 (Google TV app, remote service >=5) [3](https://pkg.go.dev/github.com/drosocode/atvremote)
  - `farshid616/Android-TV-Remote-Controller-Python` — simple Python, references Aymkdn wiki [9](https://github.com/farshid616/Android-TV-Remote-Controller-Python)
- Text input: `sendText()` uses IME injection, no keycode mapping [1](https://github.com/kud/androidtv-remote)
- App launch: `sendAppLink(link)` deep-link URI [1](https://github.com/kud/androidtv-remote)

**Security:**
- Certificate private key + cert must be treated as secret, persisted for reconnect; PIN is ephemeral.
- TLS, certificate pinning, no global bypass.
- Meets PRODUCT.md security requirements fully.
- Legal: Anymote + Pairing protocols were originally Google open source (code.google.com/p/google-tv-pairing-protocol, anymote-protocol) [2](https://stackoverflow.com/questions/4662236/how-android-remote-control-works-with-google-tv) [5](https://github.com/NineWorlds/google-tv-remote). Official Google TV partner SDK is closed behind partner portal [8](https://stackoverflow.com/questions/57809046/is-there-an-api-or-sdk-to-create-a-remote-control-application-on-smartphone-for) — comment: "if you are a Google Partner (and only then) you can download jar". Reverse-engineered implementations are tolerated and widely used in Home Assistant etc. Risk low but note partner docs not reachable.

**Capabilities:**
- Full Android keycodes (HOME, BACK, DPAD_UP/DOWN/LEFT/RIGHT, MEDIA_PLAY_PAUSE, VOLUME_UP/DOWN, etc.) [1](https://github.com/kud/androidtv-remote)
- Power toggle (`sendPower()`), volume state events, current app tracking [1](https://github.com/kud/androidtv-remote)
- App launch via deep-link, text injection.
- Casting: Google Cast is separate but TV supports Cast natively; remote can launch cast-enabled apps.
- Voice: if TV supports voice, keycode may trigger assistant.

**Stability/Maintenance:**
- Protocol v2 deployed since Google TV app update Sept 2021 (remote service v5+ incompatible with legacy) [8](https://stackoverflow.com/questions/57809046/is-there-an-api-or-sdk-to-create-a-remote-control-application-on-smartphone-for)
- Library `kud/androidtv-remote` modern TypeScript rewrite, ESM, lean deps, derived from louis49 MIT [1](https://github.com/kud/androidtv-remote)
- Maintenance risk low — protocol is Google's own, stable, used by first-party Google TV app.

**User reach:** >24% OS shipment share, plus powers Sony Bravia, TCL, Hisense, Xiaomi, NVIDIA Shield, Chromecast with Google TV — broadest single protocol coverage.

**Android-first alignment:** Product constraint "Android first, while preserving credible later iPhone path" — Android TV protocol aligns with Android phone development, certificate handling similar to Android Keystore patterns.

### 4. Roku ECP (External Control Protocol)

**Protocol:**
- HTTP REST on port 8060, no auth, SSDP discovery on 239.255.255.250:1900 [4](https://dev.to/hisuperdev/i-built-an-in-browser-roku-tv-remote-with-80-lines-of-typescript-heres-how-rokus-ecp-api-56ni)
- Endpoints: `/keypress/<key>`, `/query/apps`, `/launch/<appId>`, `/query/active-app` etc. [3](https://adtechmadness.wordpress.com/2020/11/01/attacking-roku-sticks-for-fun-and-profit/)
- No pairing, no token, no TLS.

**Security assessment — fails PRODUCT.md bar:**
- No authentication: "Anything inside the LAN could use it to issue ECP commands" [3](https://adtechmadness.wordpress.com/2020/11/01/attacking-roku-sticks-for-fun-and-profit/)
- PRODUCT.md: "Security wins over popularity or convenience. A popular ecosystem is not worth shipping through an unsafe trust model." + "If a secure connection cannot be established, refuse the unsafe connection. There is no global 'ignore security' mode." + "Pairing secrets are treated like passwords."
- Roku ECP cannot meet these: there is no secret, no identity, no secure connection. Browser mixed-content issues also noted [4](https://dev.to/hisuperdev/i-built-an-in-browser-roku-tv-remote-with-80-lines-of-typescript-heres-how-rokus-ecp-api-56ni) — HTTPS browser cannot call HTTP TV without proxy.
- Therefore Roku must be explicitly deferred from V1, not because of popularity but because it fails security invariants defined in DOMAIN.md "Paired device" and business rules.

**Legal note:** Roku docs quote: "ECP commands may not be sent from 3rd-party platforms (for example, mobile applications)" — this appears in context of channels including ECP code, not external remote apps [2](https://www.reddit.com/r/esp32/comments/13avuyz/homemade_roku_remote/). However Roku's official mobile app exists; third-party remote apps exist widely (hiremote.app example [4](https://dev.to/hisuperdev/i-built-an-in-browser-roku-tv-remote-with-80-lines-of-typescript-heres-how-rokus-ecp-api-56ni)). Legal risk moderate, but security failure is primary blocker.

**Capabilities:** keypress full vocabulary, direct app launch via channel ID, query apps (XML) [4](https://dev.to/hisuperdev/i-built-an-in-browser-roku-tv-remote-with-80-lines-of-typescript-heres-how-rokus-ecp-api-56ni) [8](https://dev.to/hisuperdev/every-roku-tv-ships-with-a-documented-rest-api-heres-how-to-use-it-from-curl-or-js-1jb1)

**User reach:** 38% US CTV, 25% UK, 73% Mexico [2](https://www.pixalate.com/blog/q1-2025-connected-tv-ctv-device-market-share-report) — high, but security bar prevents V1 inclusion.

### 5. Other notes

- **Fire TV:** Android-based, ADB or Fire OS remote; not researched deeply due to Amazon terms and limited protocol docs reachable; deferred.
- **VIDAA (Hisense):** 7.8% global share [5](https://www.mediapost.com/publications/article/396751/samsung-smart-tv-os-tops-in-2024-hisense-lg-and.html), but protocol not well documented in allowlisted sources; deferred.
- **Titan OS (Philips):** 4.9% share, newer, insufficient evidence.

## Capability matrix vs PRODUCT.md V1 scope

| Capability | Samsung Tizen | LG webOS | Android TV | Roku ECP |
| :--- | :--- | :--- | :--- | :--- |
| Discovery | SSDP + IP, plus WoL | SSDP + scan, plus manual IP | mDNS + SSDP, plus IP | SSDP, simple |
| Pairing | Popup + token | Popup + pairing material | PIN + cert TLS | None (fails security) |
| Power on | WoL | WoL | Power toggle (WoL via separate) | PowerOff only (TVs may not WoL) |
| Power off | Yes | Yes | Yes | Yes |
| Volume | Yes | Yes + live level | Yes + volume events | Yes (TV only) |
| D-pad nav | Yes | Yes + pointer trackpad | Yes | Yes |
| Touchpad/swipe | Possible via touchPad support flag | Yes (Magic Remote) | Limited | No |
| Text input | IME synced | Keyboard | Native IME injection | No |
| App launch | Yes via ID | Yes | Yes via deep-link | Yes via channel ID |
| App list | Yes | Yes listApps | Yes via current_app events | Yes /query/apps |
| Casting/mirroring | Separate (SmartThings/DIAL) | Miracast separate | Google Cast native | DIAL |
| Voice | VoiceSupport flag true [7](https://forum.logicmachine.net/printthread.php?tid=4606) | Possible | Keycode assistant | No |

## Maintenance burden ranking (lowest to highest)

1. Android TV — Google-maintained protocol, stable v2 since 2021, multiple language libs, TLS cert model well understood.
2. Samsung Tizen — community library samsungtvws active 2026, but token rotation observed ("TV keeps asking to accept connection, token changing" [7](https://forum.logicmachine.net/printthread.php?tid=4606)) — requires robust re-pair handling.
3. LG webOS — stable but smaller community, Python lib tested across OSes but less frequent releases.
4. Roku — simplest code, but security debt and mixed-content browser limitation increase UX cost.

## Legal/terms summary (reduced coverage disclosure)

- **Egress allowlist** blocked direct fetch of vendor developer portals (developer.samsung.com, developer.lge.com docs, Google partner SDK). Evidence relies on GitHub primary sources and secondary web_search summaries.
- **Samsung:** SmartThings developer ToS prohibits reverse engineering of Accessed Developer Tools [5](https://developer.smartthings.com/termsofservice); local WS protocol not explicitly listed as Accessed Tool, but risk noted.
- **LG:** Forum guidance says compatibility naming without logos is allowed [3](https://forum.webostv.developer.lge.com/t/do-i-need-lg-permission-to-publish-a-remote-control-app-for-lg-tvs-on-galaxy-store/28119); official trademark policy not reachable.
- **Google/Android TV:** Original Anymote/Pairing protocols were open source [2](https://stackoverflow.com/questions/4662236/how-android-remote-control-works-with-google-tv); current partner SDK closed behind partner login [8](https://stackoverflow.com/questions/57809046/is-there-an-api-or-sdk-to-create-a-remote-control-application-on-smartphone-for) — reverse-engineered impl tolerated.
- **Roku:** ECP documented at developer.roku.com (secondary reference [4](https://dev.to/hisuperdev/i-built-an-in-browser-roku-tv-remote-with-80-lines-of-typescript-heres-how-rokus-ecp-api-56ni)); terms about 3rd-party platforms ambiguous [2](https://www.reddit.com/r/esp32/comments/13avuyz/homemade_roku_remote/).

No legal opinion provided; this is research evidence only, per PRODUCT.md research provenance rule.

## Recommendation for V1 (to be recorded in ADR-0005)

**Primary:** Android TV / Google TV + Samsung Tizen

Rationale:
- Maximizes user reach (~40.9% shipment) with two distinct stacks (OS + vendor).
- Both meet security bar (certificate/token pairing, no bypass mode).
- Both support WoL/power, volume, D-pad, text input, app launch — required for capability-driven remote.
- Android-first aligns with product constraint and preserves iPhone path (protocol is platform-agnostic, libraries exist for both Android and iOS via JS/TS).
- Maintenance burden lowest for Android, moderate for Samsung, both with active 2026 libraries.
- Casting: Android TV gives Google Cast natively, satisfying V1 casting/mirroring without extra cloud relay.

**Deferred:** LG webOS — strong candidate for third ecosystem immediately after V1, due to premium positioning, pointer trackpad, and WoL support. Security model meets bar. Reason for deferral: reach slightly lower than Samsung, and pairing two ecosystems first keeps V1 scope tight.

**Rejected for V1:** Roku ECP — fails security invariants (no auth, no secret, no secure connection) per DOMAIN.md business rule "Security wins over popularity". Must not ship through unsafe trust model. Documented as honest Unsupported/Deferred with rationale, not hidden.

## Hardware matrix criteria (for release)

Per PRODUCT.md compatibility promise (Tested / Expected / Unsupported):

- **Tested:** Verified on real hardware/model/firmware in release test matrix. Minimum for V1 release:
  - Samsung: at least 2 models across 2 firmware generations (e.g., Frame 2022 Tizen 6.5 + QLED 2024 Tizen 8.0) — proves integration not tied to one model.
  - Android TV: at least 2 distinct OEMs (e.g., Sony Bravia Google TV + TCL Google TV or Chromecast with Google TV) across 2 Android TV Remote Service versions (>=5).
  - Each Tested entry records: model, firmware/OS version, protocol version (Tizen version or Android TV Remote Service version), pairing method, wake method, capabilities verified (power, volume, D-pad, text, app launch).
- **Expected:** Predicted from verified family/protocol evidence but not represented by specific tested device. Example: Samsung Crystal UHD series expected to work if same Tizen version as Tested QLED, but not yet in matrix.
- **Unsupported:** Known not to work or intentionally not supported. Roku ECP = Unsupported for V1 due to security model, with explicit rationale, not merely offline.

One physical TV can prove first engineering integration, but release requires deliberate matrix covering more than one model/firmware generation so "works on my TV" is not mistaken for product support (per PRODUCT.md).

## Next steps

- Record selection as ADR-0005 with alternatives and consequences.
- Update PRODUCT.md V1 scope to name selected ecosystems, referencing this research.
- Validate IR dataset provenance in separate doc.
- Define naming research.
