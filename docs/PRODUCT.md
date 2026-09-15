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

## Research still required in discovery (updated 2026-09-15 — Issue #14 Harvest / Adopt / Reject reconciliation)

- [x] Choose the first two smart-TV ecosystems using current evidence — **DONE (direction approved):** Android TV/Google TV + Samsung Tizen per ADR-0005 **accepted** and `docs/research/2026-09-14-ecosystem-evidence.md` corrected; selection re-evaluated after fixing licensing/security overstatements and then **explicitly approved by the product owner on [Issue #7](https://github.com/anthracite-labs/Greenfield4/issues/7#issuecomment-5666976043)** (2026-09-14, issue closed as completed). This closes the **selection** item only. The approval is explicitly "subject to the documented PARTIAL security/legal validation and remaining discovery exit criteria", so security validation and legal validation for both ecosystems stay **PARTIAL** and are tracked by the separate open items below. Distinction preserved between product-direction approval (given) and unresolved security/legal/hardware validation (still PARTIAL/open).
- [ ] Resolve vendor terms/legal restrictions on third-party remote control — **PARTIAL, targeted upstream pass complete:** the 2026-09-15 Harvest / Adopt / Reject reconciliation reached current Samsung Developer documentation and terms. Samsung's documented Smart View path is a sender + receiver application model with TLS; targeted official searches did **not** surface documentation for the reverse-engineered stock-TV `samsung.remote.control` / `ms.remote.control` endpoint. The historical Google AOSP pairing implementation is authoritative for protocol/reference semantics but is not a modern public Remote-v2 API contract. This narrows the remaining work to **human/legal review of the exact V1 protocol use**, not another broad OSS survey. OSS prevalence is not legal clearance.
- [ ] Validate protocol stability, pairing flows, security identities, wake behavior, app launch/text input/casting/voice capabilities, and maintenance burden — **PARTIAL:** stability/pairing/wake/app launch/text input documented via primary sources (samsungtvws v3.0.6 2026-09-11, LGWebOSRemote, kud/androidtv-remote reverse-engineered), but security identities validation remains PARTIAL. Analysis sharpened 2026-09-14 in `docs/research/2026-09-14-security-trust-model.md` (issue #11), which documents the two trust models separately from pinned primary sources and **does not upgrade either ecosystem**: reference clients disable server authentication on both ecosystems and on both phases (Samsung: `CERT_NONE` on the sync *and* async paths; Android TV: `rejectUnauthorized:false` / `CERT_NONE` on both ports, in two independent implementations); Samsung's **client-observable** pairing path binds **nothing** — the token is an opaque value with no certificate or public-key binding, and the reference client sends no token on plaintext port 8001 (the TV's server-side token/device association rule is **[UNRESOLVED]**; there is no server-side source for Samsung comparable to the AOSP source available for Android TV); Android TV's pairing protocol binds client certificate, server certificate and nonce into `alpha`, and the **pinned AOSP reference implementation (`google-tv-pairing-protocol` @ `7c99785`) has the output device verify the full 32-byte alpha server-side**, withholding SecretAck on mismatch — but in the deployed six-symbol format the **user-transferred out-of-band authenticator is only an 8-bit alpha prefix plus a 16-bit nonce**, so against a terminating active MITM the first-use binding is **8 bits per independent pairing
trial**, not 256; the full alpha verifies the leg it is sent on and does not bind two separately terminated TLS legs. Remaining **[HARDWARE-REQUIRED]**: target-firmware conformance, and pairing-attempt controls that
bound how many *independent* trials an attacker can obtain — attempts per code, whether a failure
rotates the code or session, retry delay, rate limit, lockout, code lifetime, and whether the TV even
observes a locally rejected attempt (255/256 of failures happen at the phone before any Secret is
sent); neither reference client persists any *server* identity, so reconnect is unprotected in both ecosystems as shipped by them. The targeted 2026-09-15 OSS pass in `docs/research/2026-09-15-harvest-adopt-reject.md` now supplies concrete implementation precedent for the mitigation: ScreenCast @ `7e66bbe7` captures the Android TV server certificate only after successful pairing and enforces its SHA-256 pin on 6466 reconnect, while Samsung.SmartTv.Client @ `ceb07002` exposes an application-controlled peer-certificate validator. Greenfield therefore treats reconnect pinning as a **HARVEST → ADOPT** engineering pattern rather than an open literature-research question; it still needs Greenfield hardware proof of certificate stability and fail-closed behavior. One structural conflict remains: **Samsung first-use MITM resistance is not achievable by TOFU alone**, because there is no prior authenticated fingerprint at first pairing. Casting PARTIAL (Samsung not via the stock WS remote API; Samsung's official Smart View path is a separate sender/receiver-app model); voice NOT VERIFIED; maintenance/legal status remains distinct because both V1 remote protocols are not established here as modern public vendor APIs.
- [ ] Resolve Samsung first-use MITM exposure — **OPEN CONFLICT, product-owner decision required; broad protocol research STOPPED:** Samsung's client-observable stock-remote pairing path establishes no cryptographic binding, so **no authenticated TV identity exists at the first connection** and TOFU cannot protect that first connection. The deeper OSS/vendor pass found no stock-TV mechanism that changes this. A fingerprint shown only by the phone is **not** an out-of-band solution because an active peer can supply the value the phone displays; only an independently authenticated TV-side/vendor channel would change the conclusion, and none was found for the stock remote path. The honest choices are therefore: accept and document the residual first-use risk, introduce a genuinely independent authenticated channel if one later exists without breaking the product setup model, or decline/defer Samsung for V1.
- [ ] Finalize V1 IR data-source compliance — **PARTIAL, research narrowed:** IRDB @ `11aa5eb3` is the preferred **conditional** candidate: its primary license permits app inclusion/network access subject to prior project notification, the required notice, and up to three fully licensed copies/units on request. Runtime CDN access does not remove those obligations. Product-owner/legal acceptance is still required before adoption. LIRC remotes data is **REJECTED for V1 shipping unless explicit primary data/config permission is later obtained**; Greenfield will not spend V1 schedule trying to infer a database license from adjacent GPL software. Matching/profile verification remains a Greenfield hardware/product-quality task, not a licensing-research gap.
- [ ] Execute the release hardware matrix — **written, NOT EXECUTED; research-vs-test scope reconciled:** `docs/research/2026-09-14-hardware-validation-matrix.md` retains the full Samsung/Android catalogue and the minimum two-model/firmware rule. Per Issue #14, ordinary discovery/pairing/credential-lifecycle mechanics that mature OSS already implements are no longer treated as open literature-research gaps; they become Greenfield integration/release verification. Physical evidence remains load-bearing for Greenfield's security delta and device-side facts: certificate/SPKI stability across reboot/update/reset, fail-closed substituted/wrong-identity behavior, contemporary Android TV Secret enforcement, and the retry/session controls that bound the practical 8-bit first-use risk. **No hardware row has been run or passed.**
- [ ] Finish product naming — **criteria and candidates researched** in `docs/research/2026-09-14-naming.md`; final name/slug decision deferred to product owner, PROJECT_NAME/SLUG remain blank per ADR-0001.

Remaining discovery exit criteria: resolve the Samsung first-use residual-risk conflict (product-owner decision); obtain the targeted real-hardware security evidence required by the existing matrix before making shipping-security claims; complete human/legal review of the exact V1 protocol use; accept or reject IRDB's explicit license obligations (LIRC is rejected for V1 unless newly licensed); finalize naming; and review PRODUCT.md before moving to architecture. The protocol mechanics and common implementation patterns have now been reconciled in `docs/research/2026-09-15-harvest-adopt-reject.md`; **do not restart broad research for a gap already harvested from credible OSS unless new contradictory primary evidence appears.** Product-owner approval of the ecosystem direction is already recorded on Issue #7. Do not move to architecture until security validation is resolved or explicitly accepted as PARTIAL by the product owner, and legal/terms validation is completed.

**Approval states are separate and must stay separate.** Product-direction approval (given, Issue #7) is not shipping readiness, not security validation, not legal validation, and not hardware validation — each of those is tracked independently above and none of them is complete.

## Research provenance rule

Uploaded universal-remote and harvest-matrix ZIPs are useful research evidence only. They do not override repository governance and do not become product requirements unless the product owner explicitly adopts an idea during discovery.

**Research reuse rule (Issue #14):** use **Harvest → Adopt → Reject**. Once a maintained OSS implementation demonstrably solves an engineering mechanic and no load-bearing primary source contradicts it, pin that evidence and stop broad exploratory research. Adopt only the parts that satisfy Greenfield4 invariants; explicitly reject compatibility shortcuts such as global certificate-verification bypass. Hardware-only, legal, and product-owner questions stay open rather than being converted into more web research. See `docs/research/2026-09-15-harvest-adopt-reject.md`.

## Lifecycle boundary

This document does not select the application stack. Discovery must finish and be reviewed before moving to `architecture`. Application code remains blocked by the no-stack guard until an accepted application-stack ADR and the later implementation transition satisfy the repository lifecycle gate.
