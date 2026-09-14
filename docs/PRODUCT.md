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
- V1 will support **two evidence-selected smart-TV ecosystems** (selection based on current evidence, not uploaded research packs alone). **Approved V1 ecosystem direction** per ADR-0005 **accepted** and explicit product-owner approval on Issue #7 (anthracite-labs, 2026-09-14T16:08:57Z): **Android TV / Google TV (Remote v2 protocol, reverse-engineered)** and **Samsung Tizen (2016+ WebSocket API, community reverse-engineered)**. This is **product-direction approval only**, subject to documented PARTIAL security/legal validation and remaining discovery exit criteria. It is not shipping readiness. Security validation for both ecosystems remains PARTIAL pending trustworthy device identity analysis and hardware validation.
- Built-in phone IR is a V1 capability when the Android device exposes suitable hardware (fixed product capability). **Candidate/evaluated data sources** for IR codes are IRDB (custom permission license per primary LICENSE.md, not CC0, with obligations) and LIRC remotes DB (licensing unresolved) per `docs/research/2026-09-14-ir-dataset.md` — **not fixed shipping dependencies** while legal acceptability remains conditional/unresolved. Product-owner/legal acceptance of IRDB obligations still required and was **not** part of Issue #7 approval. LIRC remains unresolved.
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

- **two evidence-selected smart-TV ecosystems — approved V1 direction:** **Android TV / Google TV** and **Samsung Tizen** per ADR-0005 **accepted** and Issue #7 product-owner approval (defer LG webOS as immediate next candidate, reject Roku ECP for V1 due to security model and primary docs restriction; security validation remains PARTIAL pending trustworthy identity analysis; product-direction approval is not shipping readiness, security clearance, or legal clearance);
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

**Approved V1 ecosystems (product-direction):** Android TV/Google TV and Samsung Tizen per ADR-0005 **accepted** and Issue #7 product-owner approval. This is product-direction approval, not shipping readiness. Security validation remains PARTIAL; legal/terms validation remains PARTIAL; hardware matrix is not yet executed. LG webOS deferred as immediate next candidate. Roku ECP explicitly **Unsupported for V1** due to security model failure (no authentication, no secret, any LAN device can control if enabled), per business rule "Security wins over popularity" and primary docs restriction. This is honest compatibility, not hidden.

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

## Research still required in discovery (update 2026-09-14 — Issue #7 product-owner approval reconciliation)

- [x] Choose the first two smart-TV ecosystems using current evidence — **COMPLETE at product-direction level:** Android TV/Google TV + Samsung Tizen approved on Issue #7 (anthracite-labs, 2026-09-14T16:08:57Z) and ADR-0005 **accepted**. Evidence in `docs/research/2026-09-14-ecosystem-evidence.md`. Distinction preserved: product-direction approval is complete; unresolved security/legal/hardware validation remain separate open/PARTIAL items below and are **not** cleared by this checkbox.
- [ ] Resolve vendor terms/legal restrictions on third-party remote control — **PARTIAL:** preliminary evidence gathered in ecosystem research doc; official vendor-term validation remains unavailable due to egress allowlist blocking developer.samsung.com, developer.lge.com, Google partner SDK portal; Roku primary docs now reachable (developer.roku.com) showing Control by mobile apps must be Enabled as of OS 14.1 and restriction "ECP commands may not be sent from 3rd-party platforms". Full legal validation requires human access, recorded as reduced coverage, not marked complete. Issue #7 approval did not complete legal validation.
- [ ] Validate protocol stability, pairing flows, security identities, wake behavior, app launch/text input/casting/voice capabilities, and maintenance burden — **PARTIAL:** stability/pairing/wake/app launch/text input documented via primary sources (samsungtvws v3.0.6 2026-09-11, LGWebOSRemote, kud/androidtv-remote reverse-engineered), but security identities validation PARTIAL due to CERT_NONE/rejectUnauthorized:false in reference implementations; Samsung first-use MITM resistance unresolved (token flow not shown to cryptographically bind TLS cert), Android TV PIN pairing hash includes cert material but first-use trust and persistence need verification — see research doc separated trust analysis; casting PARTIAL (Samsung not via WS API, requires Smart View SDK/DIAL/Google Cast separate validation); voice NOT VERIFIED (Samsung VoiceControl is on-TV Web API, Android remote-manager.ts notes voice not implemented); maintenance risk for Android TV reassessed as reverse-engineered not official public API, kud lib very new June 2026. Pinning/TOFU is candidate mitigation, not demonstrated solution. Issue #7 approval did not complete security, casting, or voice validation.
- [ ] Validate IR dataset provenance, quality, matching strategy, and device coverage — **PARTIAL:** IRDB licensing corrected to custom permission (notify via issue, attribution notice, up to 3 copies on request) per primary LICENSE.md, not CC0; LIRC database licensing unresolved (license null, cannot infer from GPL) and **not approved as shipping source**; coverage/size stats (500k+ codes, 10k brands, 80-85%) are secondary estimates from infishark blog, not verified from primary IRDB repo, removed from fixed constraints; matching strategy documented; V1 IR capability fixed, but IRDB/LIRC are candidate/evaluated data sources until legal acceptability resolved; IRDB requires product-owner/legal acceptance of obligations still pending and **not** part of Issue #7 approval; runtime CDN access recommended by README for updateability but does not remove license obligations because license also covers network access.
- [ ] Define the release hardware matrix once the two ecosystems are selected — **criteria defined** in ecosystem research doc (minimum 2 models/firmware per ecosystem, Tested vs Expected vs Unsupported); execution requires real hardware in future session; pinning stability across reboots/firmware must be validated. Product-direction approval does not execute or complete the hardware matrix.
- [ ] Finish product naming — **criteria and candidates researched** in `docs/research/2026-09-14-naming.md`; final name/slug decision deferred to product owner, PROJECT_NAME/SLUG remain blank per ADR-0001.

Remaining discovery exit criteria: finalize security pinning/TOFU analysis with hardware validation separating Samsung vs Android TV first-use trust; complete legal/terms validation with human portal access; obtain separate product-owner/legal acceptance of IRDB obligations if IRDB is to be a shipping source; keep LIRC unresolved until database/config licensing is established; finalize naming; execute hardware matrix with real devices; and review PRODUCT.md before moving to architecture. Product-direction approval on Issue #7 is complete and is **not** shipping readiness, security clearance, legal clearance, IR-source approval, naming, or an architecture transition. Do not move to architecture. Do not weaken PRODUCT.md or DOMAIN.md security requirements.

## Research provenance rule

Uploaded universal-remote and harvest-matrix ZIPs are useful research evidence only. They do not override repository governance and do not become product requirements unless the product owner explicitly adopts an idea during discovery.

## Lifecycle boundary

This document does not select the application stack. Discovery must finish and be reviewed before moving to `architecture`. Application code remains blocked by the no-stack guard until an accepted application-stack ADR and the later implementation transition satisfy the repository lifecycle gate.
