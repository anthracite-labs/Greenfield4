# Product

**Status: discovery in progress.**

This repository is the product repository for a phone-first universal TV remote. The final product name is intentionally undecided. This definition records product-owner decisions made in discovery; it does not select an application stack or authorize implementation.

## Problem and user

The first user is an ordinary person whose physical TV remote is lost, broken, has dead batteries, is inconvenient, or is simply worse than using the phone already in their hand.

The product should let that person find a supported TV, connect, and gain useful control very quickly without needing to understand networking, protocols, IP addresses, pairing credentials, or transport details.

The baseline to beat is the physical remote: it is immediate, familiar, dependable, and requires almost no thought.

## Product promise

The product is a **phone-first universal TV remote**, designed so other living-room devices can be added later without changing the simple user-facing promise.

“Universal” means one app can honestly support multiple TV technologies behind one consistent experience, with a clear compatibility list. It does **not** mean claiming every television works.

Reliability and premium visual quality are joint priorities. Broad device-count claims never justify unreliable control.

## Fixed V1 constraints

- Android first, while preserving a credible later iPhone path.
- Core remote control is local-first and must not require our cloud or an internet connection.
- No user account is required for core use.
- V1 will support **two evidence-selected smart-TV ecosystems** (selection based on current evidence, not uploaded research packs alone). The **approved V1 ecosystem direction** per ADR-0005 **accepted** and research in `docs/research/2026-09-14-ecosystem-evidence.md` is **Android TV / Google TV (Remote v2 protocol, reverse-engineered)** and **Samsung Tizen (2016+ WebSocket API, community reverse-engineered)**, approved by the product owner on [Issue #7](https://github.com/anthracite-labs/Greenfield4/issues/7#issuecomment-5666976043). That approval covers the **product direction only** — it explicitly leaves the documented PARTIAL security/legal validation and the remaining discovery exit criteria open. Security validation for both ecosystems is **PARTIAL** pending trustworthy device identity analysis and hardware validation, and neither ecosystem is shipping-ready.
- Built-in phone IR is a V1 capability when the Android device exposes suitable hardware (fixed product capability). **Candidate/evaluated data sources** for IR codes are IRDB (custom permission license per primary LICENSE.md, not CC0, with obligations) and LIRC remotes DB (licensing unresolved) per `docs/research/2026-09-14-ir-dataset.md` — **not fixed shipping dependencies** while legal acceptability remains conditional/unresolved. Product-owner/legal acceptance of IRDB obligations still required.
- One visible remote may transparently use more than one control transport, for example network control for rich commands and IR for power.
- Product name and repository slug remain open during discovery; naming criteria and candidates in `docs/research/2026-09-14-naming.md`.
- The product owner prefers a Rust core with a switchable UI layer, but that is an **architecture preference only** and is not accepted in discovery.

## First-run journey

1. Show a short welcome/explanation, then begin automatic discovery.
2. Before Android permission prompts, explain in ordinary language why local-network/device access is needed.
3. Show friendly device cards using trustworthy discovered information; keep protocol/IP details elsewhere.
4. When practical, show unsupported discovered TVs as “found but not supported yet” rather than pretending nothing was found.
5. If a TV requires a pairing code, simply ask the user to enter the code shown on the TV.
6. Pair once, store the resulting credentials securely, and reconnect automatically later.
7. If the TV’s IP changes, rediscover the remembered TV without exposing IP-address management to the user.
8. If automatic discovery fails, explain likely causes first and offer a manual fallback where the TV technology supports one.
9. Prefer smart/network control; use built-in IR when it helps or when network control cannot solve a required function.
10. After successful setup, immediately show the capability-driven remote.

The hard setup target for a supported device is **install/open → find → pair → working control in under two minutes**. Faster is better where the TV permits it.

## Device and household behavior

- Remember multiple TVs.
- Let users give friendly names such as Lounge or Bedroom.
- Reopen the last-used TV by default.
- On later launches, reopen the last remote and reconnect quietly in the background.
- Additional household phones pair independently in V1; pairing secrets are not shared through an account or cloud service.
- After a router/Wi-Fi change, rediscover remembered devices and repair the connection with as little user involvement as the device permits.
- If phone and TV are on different local networks, explain that clearly rather than hiding the limitation behind a cloud relay.

## Capability-driven remote

The connected device is the truth. The product should discover what that actual device can do and present useful controls from real capabilities rather than hardcoding one generic button set.

- Familiar enough to understand immediately, but designed for a phone rather than visually copying a plastic remote.
- Support directional-button navigation and touchpad/swipe navigation when the TV supports them; let the user choose.
- Use subtle haptic feedback on remote presses by default, with an option to disable it.
- While the remote is active, Android physical volume buttons control TV volume by default, with an option to disable that behavior.
- If the TV supports text input, use the normal Android keyboard.
- If installed apps are discoverable and launchable, show the actual supported apps rather than hardcoded app buttons.
- Keep everyday controls on the main remote and less-used supported controls under “More”.
- Allow simple rearranging/favourites in V1; a full custom remote designer is not V1.
- Keep core controls stable while allowing relevant extra controls to surface contextually.
- Make one-handed use a first-class requirement.
- Accessibility is first-class: screen-reader support, scalable text, strong contrast, large touch targets, and clear labels.
- Show simple states such as Connected, Reconnecting, or TV offline; keep technical connection details away from the main remote.
- If a capability becomes unavailable, reflect that honestly rather than leaving a dead control.
- V1 visual quality should feel premium; decorative animation must not reduce reliability or delay critical behavior.

## Power, failure, and recovery

- For a sleeping/off TV, try safe supported wake methods automatically: network wake first, then built-in IR when available and appropriate.
- If the hardware cannot be powered on remotely, explain the limitation clearly rather than promising impossible behavior.
- Connection drops trigger quiet reconnection with a small Reconnecting state, not repeated error dialogs.
- Model/firmware assumptions never override what the actual device proves it can do.
- Unsupported TVs may offer a manual “Request support” action where the user explicitly chooses what device information to send.
- IR setup uses likely code profiles and asks the user to verify simple commands; do not blindly transmit every known code.
- If multiple IR profiles partly work, verify important commands and select the profile with the best demonstrated coverage.

## Security and privacy product requirements

Security wins over popularity or feature coverage.

- Pairing credentials, tokens, and keys are treated like passwords: protect them on-device and exclude them from logs/analytics.
- If a paired device’s security identity changes unexpectedly, fail safely and require re-pairing rather than silently trusting the new identity.
- If a secure connection cannot be established, refuse the unsafe connection. There is no global “ignore security” mode.
- Core product data stays local by default and collection is minimized.
- V1 has **no behavioral usage analytics**.
- Anonymous crash/error reporting is acceptable only when remote commands, pairing secrets, Wi-Fi names, IP addresses, and identifiable TV details are excluded unless the user explicitly sends a diagnostic report.
- Diagnostic logs are minimal, temporary, and redacted; exporting diagnostics is an explicit user action.

## V1 scope

V1 is a finished consumer product, not a protocol demo. It includes:

- **two evidence-selected smart-TV ecosystems — approved direction, not yet shipping-ready:** **Android TV / Google TV** and **Samsung Tizen** per ADR-0005 **accepted** (LG webOS deferred as the immediate next candidate; Roku ECP rejected for V1 due to its security model and the primary-docs restriction; security validation remains **PARTIAL** pending trustworthy identity analysis and hardware validation, so ecosystem approval is not a shipping or security clearance);
- built-in phone IR where available as fixed capability; candidate data sources IRDB (custom permission license with attribution/notification/copy obligations requiring product-owner/legal acceptance) and LIRC (licensing unresolved, not approved as shipping source) evaluated per research, not fixed dependencies;
- automatic discovery, pairing, remembered devices, reconnection, and failure recovery;
- capability-driven controls and premium accessible phone-native remote UX;
- multiple remembered TVs;
- casting/mirroring — **PARTIAL:** Google Cast native via Android TV separate from remote protocol, Samsung casting via Smart View SDK/DIAL/Google Cast 2026 models requires separate validation, not via WS remote API;
- voice control only when the connected ecosystem exposes it cleanly and safely, and only if it does not delay launch — **currently NOT VERIFIED** for Samsung Tizen (VoiceControl is on-TV Web API) and Android TV (remote-manager.ts notes voice not implemented), so V1 may ship without voice;
- honest compatibility information distinguishing Tested / Expected / Unsupported;
- privacy-safe crash/error reporting as defined above.

## Explicit V1 non-goals

- Macros/scenes/automation.
- External IR hubs or dedicated external control hardware.
- Soundbars, receivers, and other non-TV living-room devices; the product should leave room for them later.
- A full drag-and-drop remote designer.
- Required cloud accounts or cloud relay for core control.
- Advertising.
- A paid/freemium/subscription requirement at launch; V1 is free and the business model can be revisited after product value is proven.
- Selecting or implementing an application framework, language, database, auth scheme, hosting target, or accepted architecture during discovery.

## Compatibility promise

Compatibility must be specific and honest. Publish a supported-device view that distinguishes at least:

- **Tested** — verified on real hardware/model/firmware in the release test matrix.
- **Expected** — expected to work from verified protocol/family evidence but not yet represented by a specific tested device.
- **Unsupported** — known not to work or intentionally not supported.

A good open-source implementation is sufficient to **shortlist** an ecosystem for deeper investigation, not to ship it.

Before an ecosystem ships, validate current legal/terms constraints, security model, protocol stability, pairing behavior, maintenance risk, user reach, and real-device behavior — evidence recorded in `docs/research/2026-09-14-ecosystem-evidence.md`.

**Approved V1 ecosystem direction:** Android TV/Google TV and Samsung Tizen per ADR-0005 **accepted** (direction approved by the product owner on Issue #7; security validation remains **PARTIAL**, so approval of the direction is not a statement that either ecosystem can ship today). LG webOS deferred as immediate next candidate. Roku ECP explicitly **Unsupported for V1** due to security model failure (no authentication, no secret, any LAN device can control if enabled), per business rule "Security wins over popularity" and primary docs restriction. This is honest compatibility, not hidden.

One physical TV can prove the first engineering integration for an ecosystem. Release requires a deliberate hardware matrix covering more than one model/firmware generation so “works on my TV” is not mistaken for product support. Matrix criteria defined in ecosystem research doc: at least 2 models/firmware per ecosystem, recording model, firmware/OS, protocol version, pairing method, wake method, capabilities verified.

If a vendor changes a protocol and previously supported devices break, treat that as a high-priority regression: restore support where feasible or clearly update the affected compatibility status.

## Success criteria

Headline success metric:

> A user with a supported device can install/open the app, find the device, pair it, and reach working control in under two minutes.

Release quality also requires:

- command reliability that feels comparable to a physical remote; failures should be rare enough that an ordinary user does not think about them;
- finished setup/recovery/security/privacy behavior, not only happy-path control;
- a deliberate real-hardware compatibility matrix for each shipping ecosystem;
- premium, accessible UI quality without sacrificing reliability.

## Research still required in discovery (update 2026-09-14 — reconciled with Issue #7 product-owner approval)

- [x] Choose the first two smart-TV ecosystems using current evidence — **DONE (direction approved):** Android TV/Google TV + Samsung Tizen per ADR-0005 **accepted** and `docs/research/2026-09-14-ecosystem-evidence.md` corrected; selection re-evaluated after fixing licensing/security overstatements and then **explicitly approved by the product owner on [Issue #7](https://github.com/anthracite-labs/Greenfield4/issues/7#issuecomment-5666976043)** (2026-09-14, issue closed as completed). This closes the **selection** item only. The approval is explicitly "subject to the documented PARTIAL security/legal validation and remaining discovery exit criteria", so security validation and legal validation for both ecosystems stay **PARTIAL** and are tracked by the separate open items below. Distinction preserved between product-direction approval (given) and unresolved security/legal/hardware validation (still PARTIAL/open).
- [ ] Resolve vendor terms/legal restrictions on third-party remote control — **PARTIAL:** preliminary evidence gathered in ecosystem research doc; official vendor-term validation remains unavailable due to egress allowlist blocking developer.samsung.com, developer.lge.com, Google partner SDK portal; Roku primary docs now reachable (developer.roku.com) showing Control by mobile apps must be Enabled as of OS 14.1 and restriction "ECP commands may not be sent from 3rd-party platforms". Full legal validation requires human access, recorded as reduced coverage, not marked complete.
- [ ] Validate protocol stability, pairing flows, security identities, wake behavior, app launch/text input/casting/voice capabilities, and maintenance burden — **PARTIAL:** stability/pairing/wake/app launch/text input documented via primary sources (samsungtvws v3.0.6 2026-09-11, LGWebOSRemote, kud/androidtv-remote reverse-engineered), but security identities validation remains PARTIAL. Analysis sharpened 2026-09-14 in `docs/research/2026-09-14-security-trust-model.md` (issue #11), which documents the two trust models separately from pinned primary sources and **does not upgrade either ecosystem**: reference clients disable server authentication on both ecosystems and on both phases (Samsung: `CERT_NONE` on the sync *and* async paths; Android TV: `rejectUnauthorized:false` / `CERT_NONE` on both ports, in two independent implementations); Samsung's **client-observable** pairing path binds **nothing** — the token is an opaque value with no certificate or public-key binding, and the reference client sends no token on plaintext port 8001 (the TV's server-side token/device association rule is **[UNRESOLVED]**; there is no server-side source for Samsung comparable to the AOSP source available for Android TV); Android TV's pairing protocol binds client certificate, server certificate and nonce into `alpha`, and the **pinned AOSP reference implementation (`google-tv-pairing-protocol` @ `7c99785`) has the output device verify the full 32-byte alpha server-side**, withholding SecretAck on mismatch — but in the deployed six-symbol format the **user-transferred out-of-band authenticator is only an 8-bit alpha prefix plus a 16-bit nonce**, so against a terminating active MITM the first-use binding is **8 bits per independent pairing
trial**, not 256; the full alpha verifies the leg it is sent on and does not bind two separately terminated TLS legs. Remaining **[HARDWARE-REQUIRED]**: target-firmware conformance, and pairing-attempt controls that
bound how many *independent* trials an attacker can obtain — attempts per code, whether a failure
rotates the code or session, retry delay, rate limit, lockout, code lifetime, and whether the TV even
observes a locally rejected attempt (255/256 of failures happen at the phone before any Secret is
sent); neither reference client persists any *server* identity, so reconnect is unprotected in both ecosystems as shipped by them; fail-closed pinning is a viable design that requires no global ignore-security mode, but is a candidate mitigation, not a demonstrated solution. One structural conflict is recorded for decision: **Samsung first-use MITM resistance is not achievable by TOFU alone**, because there is no prior fingerprint at first pairing. Casting PARTIAL (Samsung not via WS API, requires Smart View SDK/DIAL/Google Cast separate validation); voice NOT VERIFIED (Samsung VoiceControl is on-TV Web API, Android remote-manager.ts notes voice not implemented); maintenance risk for Android TV reassessed as reverse-engineered not official public API, kud lib very new June 2026.
- [ ] Resolve Samsung first-use MITM exposure — **OPEN CONFLICT, product-owner decision required (not decidable by more research):** Samsung's client-observable pairing path establishes no cryptographic binding, so **no authenticated TV identity exists at the first connection** and first-use MITM resistance cannot be achieved by trust-on-first-use pinning alone. Options recorded in the security trust model research doc include accepting a documented residual first-use risk, adding out-of-band fingerprint confirmation at pairing, or declining Samsung for V1. This is a non-negotiable-invariant conflict and is routed to the product owner rather than resolved by relaxing the requirement or by reversing ADR-0005.
- [ ] Validate IR dataset provenance, quality, matching strategy, and device coverage — **PARTIAL:** IRDB licensing corrected to custom permission (notify via issue, attribution notice, up to 3 copies on request) per primary LICENSE.md, not CC0; LIRC database licensing unresolved (license null, cannot infer from GPL) and **not approved as shipping source**; coverage/size stats (500k+ codes, 10k brands, 80-85%) are secondary estimates from infishark blog, not verified from primary IRDB repo, removed from fixed constraints; matching strategy documented; V1 IR capability fixed, but IRDB/LIRC are candidate/evaluated data sources until legal acceptability resolved; IRDB requires product-owner/legal acceptance of obligations still pending; runtime CDN access recommended by README for updateability but does not remove license obligations because license also covers network access.
- [ ] Define the release hardware matrix once the two ecosystems are selected — **matrix written, NOT EXECUTED:** `docs/research/2026-09-14-hardware-validation-matrix.md` defines **17 Samsung and 30 Android TV tests** (33 rows counting the ATV-17a–d split) covering initial pairing, fingerprint capture, app/phone restart, standby/wake, power cycle, reboot, network and DHCP change, reinstall, credential and certificate persistence, firmware update, factory reset, unpair/re-pair, unexpected identity change, substituted-certificate simulation, **and a dedicated pairing-attempt-controls section (ATV-21 … ATV-30) that measures the local prefix-mismatch retry loop — whether the same gamma/session survives a local failure, whether the next trial is cryptographically independent, whether the TV observes local failures at all, and how many independent trials one user-mediated session yields** — each with required devices/firmware, expected observation, pass/fail criterion, and evidence to record. Criteria from the ecosystem research doc still apply (minimum 2 models/firmware per ecosystem, Tested vs Expected vs Unsupported) and pinning stability across reboots/firmware is still unvalidated. **No test has been run**; none may be reported as passed until executed on physical hardware.
- [ ] Finish product naming — **criteria and candidates researched** in `docs/research/2026-09-14-naming.md`; final name/slug decision deferred to product owner, PROJECT_NAME/SLUG remain blank per ADR-0001.

Remaining discovery exit criteria: resolve the recorded Samsung first-use MITM conflict (product-owner decision); execute the hardware validation matrix on real devices to establish certificate/SPKI stability across reboot and firmware update, MITM detection, and token rotation; complete legal/terms validation with human portal access; resolve IR data-source licensing (IRDB obligation acceptance and LIRC database licensing); finalize naming; and review PRODUCT.md before moving to architecture. The protocol-level trust analysis itself is now written up per ecosystem in `docs/research/2026-09-14-security-trust-model.md` — what remains is hardware evidence and the one product-owner decision, not further protocol reading. **Product-owner approval of the ecosystem direction is no longer an outstanding exit criterion** — it is recorded on Issue #7. Do not move to architecture until security validation is resolved or explicitly accepted as PARTIAL by the product owner, and legal/terms validation is completed. Do not accept an ADR merely to make a PR easier to merge.

**Approval states are separate and must stay separate.** Product-direction approval (given, Issue #7) is not shipping readiness, not security validation, not legal validation, and not hardware validation — each of those is tracked independently above and none of them is complete.

## Research provenance rule

Uploaded universal-remote and harvest-matrix ZIPs are useful research evidence only. They do not override repository governance and do not become product requirements unless the product owner explicitly adopts an idea during discovery.

## Lifecycle boundary

This document does not select the application stack. Discovery must finish and be reviewed before moving to `architecture`. Application code remains blocked by the no-stack guard until an accepted application-stack ADR and the later implementation transition satisfy the repository lifecycle gate.
