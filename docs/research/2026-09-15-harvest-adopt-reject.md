# Research — Harvest / Adopt / Reject reconciliation and deeper upstream pass

**Date:** 2026-09-15  
**Issue:** #13  
**Phase:** discovery  
**Input:** uploaded `harvest-matrix-remote.zip`, SHA-256 `7281920fb5585d048a13341c95e78a22d22536d2eb601616d11449c526576dbc`  
**Status:** reconciliation complete; load-bearing OSS patterns re-checked; remaining gates narrowed to product/legal/hardware evidence

## Purpose

This record reconciles the green / HARVEST entries in the supplied research pack with Greenfield4's
current product constraints and the evidence already committed in this repository. It then performs
a second pass only where a load-bearing claim could change V1 scope, security posture, licensing, or
the amount of discovery work still required.

The uploaded pack is evidence, not governance. Its architecture stack, accepted ADRs, monetization,
distribution, storage-library and telemetry choices do not override `PROJECT_PHASE=discovery`,
`ALLOW_APP_STACK=0`, PRODUCT.md, DOMAIN.md, or accepted repository ADRs.

## Decision vocabulary

- **HARVEST** — the external implementation proves a useful pattern, protocol shape, UX technique,
  test vector, or compatibility fact. Carry the knowledge forward; do not blindly copy code.
- **ADOPT** — the pattern is compatible with Greenfield4's already-approved product direction and is
  recommended as the discovery-level choice. Architecture-specific implementation remains deferred.
- **REJECT** — do not carry the supplied recommendation forward in its current form because it
  conflicts with Greenfield4 scope/security, lacks provenance, is contradicted upstream, or is a
  premature architecture/implementation decision.

A HARVEST result is deliberately enough to stop repetitive discovery research when the engineering
question is already solved by multiple working implementations and upstream does not contradict it.
It is **not** enough to skip Greenfield4's release hardware matrix, legal review, or security
invariants.

## Executive reconciliation

| Supplied layer | Green item | Disposition | Greenfield4 consequence |
| :-- | :-- | :-- | :-- |
| L1 IR transmit | Android `ConsumerIrManager` + runtime emitter check | **HARVEST → ADOPT** | Already a fixed V1 capability. Android upstream explicitly exposes `hasIrEmitter()`, carrier ranges and synchronous `transmit()`. No more discovery research is needed on whether built-in Android IR is technically available. |
| L1 IR transmit | USB IR dongles / learn hardware | **HARVEST → REJECT V1** | Useful future precedent, but external IR hubs/dedicated hardware are an explicit V1 non-goal. Do not widen V1 because another app supports it. |
| L1 IR transmit | honest no-IR state + network fallback | **HARVEST → ADOPT** | Already required by PRODUCT.md capability honesty. |
| L2 IR data | Flipper-IRDB as CC0 seed | **HARVEST → ADOPT AS PROVENANCE-CONSTRAINED CANDIDATE** | Current repo is CC0-1.0, but its own README says commits before `2319685` are not covered. A V1 seed may use only files whose relevant content is demonstrably covered by the post-cutover CC0 policy (or separately cleared). Reject the supplied “whole corpus, no licensing risk ever” claim. |
| L2 IR data | LIRC / IRPLUS import | **HARVEST format; REJECT LIRC corpus as shipping source** | Importing a user-supplied LIRC/IRPLUS file is an interoperability feature; bundling the LIRC remote database remains unlicensed on current primary evidence. Keep those two questions separate. |
| L2 IR data | USB learn + raw timing | **HARVEST → REJECT V1 hardware dependency** | Preserve file/learning concepts for later; external learner hardware is outside V1. |
| L3 IR codecs | IRremoteESP8266 timing/protocol knowledge | **HARVEST, do not port on supplied license claim** | It is a strong protocol/test-vector reference, but the supplied matrix incorrectly calls it Apache-2.0. GitHub identifies current `IRremoteESP8266` as **LGPL-2.1**. Any later implementation must review reuse boundaries instead of treating it as Apache code. |
| L3 IR codecs | Pronto/raw timing primitives | **HARVEST** | Useful interchange/escape-hatch concept; implementation belongs in architecture/implementation, not discovery. |
| L4 discovery | SSDP, Android NSD/mDNS, on-demand discovery | **HARVEST → ADOPT concept** | Android upstream confirms `NsdManager` is DNS-SD/mDNS. Exact manifest permissions are not frozen: Android 17/API 37 introduces `ACCESS_LOCAL_NETWORK` and service-scoped picker flows. Reject the uploaded pack's fixed permission recipe as a timeless architecture decision. |
| L4 discovery | Wake-on-LAN | **HARVEST → ADOPT where ecosystem/device proves support** | Already consistent with PRODUCT.md safe wake behavior. MAC/device metadata must never become a substitute for cryptographic identity. |
| L5 adapters | Roku ECP | **REJECT V1** | The supplied “build first” recommendation is contradicted by current Roku primary documentation: ECP commands may not be sent from third-party platforms such as mobile apps. It also conflicts with Greenfield4's security model. Roku remains Unsupported for V1. |
| L5 adapters | Samsung Tizen WSS/token | **HARVEST → ADOPT ecosystem, REJECT trust-all posture** | Ecosystem already approved. OSS solves discovery/control/token persistence mechanics. Common `CERT_NONE` / trust-all handling is compatibility precedent, not acceptable Greenfield security. |
| L5 adapters | LG webOS | **HARVEST → DEFER** | Good immediate next ecosystem candidate, already recorded, but not one of the approved two V1 targets. |
| L5 adapters | Android TV Remote v2 | **HARVEST → ADOPT ecosystem and hardened reconnect pattern** | Ecosystem already approved. Multiple clients solve pairing/control; ScreenCast proves exact server-certificate verification on reconnect (its pin store is host-keyed), while TVgrip binds the fingerprint to a persisted TV record. Greenfield must adopt the mechanism **without** copying host/IP as identity. Stop researching pinning feasibility; hardware stability and first-use attempt controls remain. |
| L5 adapters | Sony BRAVIA | **HARVEST → DEFER** | Useful future candidate; not a reason to widen V1 beyond the approved two ecosystems. |
| L6 abstraction | adapter boundary + canonical capabilities/keys | **HARVEST product pattern; ADOPT domain concept** | PRODUCT.md already requires a capability-driven, brand-agnostic remote. Do not accept the uploaded Kotlin `TvAdapter` interface/module layout during discovery. |
| L7 mirroring | omit from V1 | **HARVEST rationale; NOT AUTO-ADOPTED** | Greenfield4 currently records casting/mirroring as PARTIAL V1 scope. Omitting it would be a product-scope change, not a research correction. Route separately to the product owner if desired. |
| L8 media casting | DLNA/UPnP-AV | **HARVEST → DEFER** | Potential later transport, not required to close current two-ecosystem remote-control discovery. Do not expand research now. |
| L9 local data | local persistence, secure secrets, export/import | **HARVEST → ADOPT behavior; REJECT Room as current decision** | Local-first and protected pairing material are already requirements. Database/library choice waits for architecture. |
| L10 backend | no core backend/cloud | **HARVEST → ADOPT** | Already fixed in PRODUCT.md. Optional data refresh must not make core control internet-dependent. |
| L11 monetization | one-time premium/donations/F-Droid parity | **REJECT as current decision** | PRODUCT.md says V1 is free and monetization can be revisited later. This is not a discovery blocker. |
| L12 diagnostics | opt-in crash reports/local health panel | **HARVEST behavior; REJECT ACRA dependency now** | Privacy-safe, redacted diagnostics already allowed. Exact telemetry library/hosting is architecture/implementation work. |
| L13 distribution | F-Droid/GitHub/Play/reproducible builds; GPL/Apache choice | **HARVEST future criteria; REJECT current license/stack ADR** | Useful release considerations, but the repository has not chosen an app stack or application license. |
| L14 honesty | runtime capability matrix + public Tested/Expected/Unsupported + explicit consent | **HARVEST → ADOPT** | Already matches PRODUCT.md/DOMAIN.md and should remain a release rule. |

## Second pass — Samsung Tizen

### What OSS has solved

**Control mechanics are solved enough to harvest.** `xchwarze/samsung-tv-ws-api` at
`e48d6377faede37db1f034d726a079b9d8034fac` remains an active reference for the 2016+ WebSocket
channel, token acquisition, key commands, app operations and wake behavior. The fact that it writes
tokens to a normal token file is evidence of protocol mechanics, not a storage recommendation.

**Secure mobile token persistence is also solved.** `sturlese/tvremote-app` at
`7880e2c026c102053ccfbfbf6a05b452fafa310d` separates ordinary TV metadata from pairing tokens and
stores tokens with `expo-secure-store` (Keychain/Keystore-backed platform storage). Greenfield4 does
not need more discovery research to establish that Samsung tokens can be protected on a phone.

**Application-controlled certificate validation is technically available.**
`Perun85/Samsung.SmartTv.Client` at `ceb0700282298c2f7be072d9a8bade005bdf6232` exposes
`IRemoteCertificateValidator` to both registration and remote-control clients. Its default
`AlwaysValidRemoteCertificateValidator` returns true for every certificate, but the hook proves a
client can substitute a strict per-TV validator. That is enough to stop researching whether the
client can inspect/control certificate acceptance.

**Broad compatibility projects still use unsafe trust.** Current openHAB
(`openhab/openhab-addons` at `cc5aa9383f49f417df0dd1dea34a8b519e201961`) stores the Samsung
WebSocket token and supports many TV generations, but its Samsung
`SamsungTvTlsTrustManagerProvider` returns `TrustAllTrustManager`. Its README explicitly documents
that even the stricter legacy-cipher option does not establish normal certificate trust. This is
strong evidence that “accept any self-signed TV cert” is a common compatibility workaround — and
equally strong evidence that Greenfield4 must **not** mistake popularity for authentication.

### What upstream Samsung does and does not settle

Samsung's official Smart View SDK documentation is now reachable. It documents a supported
**sender-app + TV receiver-app** model: sender applications identify a receiver application by App
ID, the TV-side receiver participates in a channel, and the SDK can enable TLS/WSS security mode.
That is useful authoritative evidence for future casting/app-to-app communication.

It is **not documentation for the stock `samsung.remote.control` WebSocket used by generic remote
apps**, and it does not provide an authenticated first-use identity for that channel. Requiring a
custom TV receiver application would also materially change Greenfield4's under-two-minute consumer
setup. Therefore Smart View SDK is HARVEST for future casting, not a substitute for the V1 remote
protocol.

### Samsung conclusion

No more discovery research is needed for token storage, command transport, or whether certificate
inspection can be implemented. The remaining questions are deliberately narrower:

1. **Product decision:** accept or reject Samsung's residual unauthenticated first-use trust model.
2. **Hardware evidence:** certificate/SPKI stability across reboot/firmware/reset and actual token
   behavior on target models.
3. **Vendor/legal:** whether shipping a third-party client for the reverse-engineered stock remote
   channel is acceptable under current vendor terms.

The first-use issue is **not solved by common OSS trust-all behavior**, and no upstream source found
in this pass changes that conclusion.

## Second pass — Android TV / Google TV Remote v2

### What is now implementation precedent rather than a proposal

Greenfield4's earlier record correctly established the pairing construction and the bounded
first-use authenticator. The new result is about **reconnect hardening**.

`ddagunts/ScreenCast` at `7e66bbe7ae5cc64c012bbe4987940be67925d137` implements the pattern in
source:

- a persistent per-install RSA client certificate/private key;
- encrypted client material using AndroidX `EncryptedFile` with a Keystore-derived AES-256-GCM
  master key;
- capture of the TV leaf certificate SHA-256 during the pairing connection;
- persistence of that server fingerprint after successful pairing;
- on port 6466, exact comparison of the presented server leaf SHA-256 with the stored value;
- socket close + `SecurityException` on mismatch, directing the user to re-pair after a TV reset;
- reconnect logic continues to use the stored pin.

The TLS context in that implementation accepts the self-signed certificate at handshake time, then
performs the exact pin check immediately after `startHandshake()` and before the application remote
channel is used. That is not a general CA-validation design, but for a paired self-signed endpoint it
demonstrates the core mechanism: **a subsequent session can verify the exact certificate captured at
pairing and fail closed on change without a global process-wide ignore switch.**

**Do not copy ScreenCast's association key.** `AndroidTvCertStore.kt` explicitly stores
`getServerPin(host)` / `pinServer(host, ...)` in a per-host preference map, while
`AndroidTvPersistence.kt` separately maintains a more stable paired-device key (BLE MAC when
available, otherwise name) and last-known host. Therefore ScreenCast proves certificate-checking
feasibility, **not** that host/IP is an acceptable security identity. Greenfield must attach the
certificate/SPKI pin to its own paired-device record; rediscovery may update a route/IP only after
the presented certificate proves continuity with that record. mDNS names, MAC-like TXT fields and
IP addresses remain discovery/routing hints, never the cryptographic identity.

A second implementation precedent exists in merged TVgrip PR #2
(`mbir31/TVgrip`, merge commit `42a1c151de6fb6b86713ab30ec83962cd14e8cec`), whose recorded
change centralizes pairing/remote TLS, keeps AndroidKeyStore client identity, captures the TV cert
during pairing, and builds the remote TLS context around a per-TV certificate fingerprint.

These projects are young and are not evidence that every 2026 TV firmware keeps a stable
certificate. They **are** enough evidence that the client-side engineering pattern is solved.

### What authoritative AOSP still says

The pinned AOSP `google-tv-pairing-protocol` reference at `7c99785` remains the authoritative
protocol source available for both roles. The output-device path verifies the in-band Secret against
its locally recomputed alpha and returns an invalid-challenge error on mismatch; SecretAck follows
successful verification. The input-device path checks the user-transferred gamma before sending its
Secret.

Nothing in this second pass overturns Greenfield4's existing first-use analysis: the deployed
six-symbol Remote-v2 format carries an 8-bit alpha prefix plus a 16-bit nonce, so the user-mediated
cross-leg authenticator is bounded at **8 bits per independent pairing trial**. Full-alpha checking
authenticates each terminated leg; it does not magically bind two attacker-terminated TLS legs.

### Android TV conclusion

**Stop researching:**
- whether Remote v2 can be discovered and controlled from third-party clients;
- whether client credentials can be stored securely on Android;
- whether a TV server certificate can be captured during pairing;
- whether subsequent 6466 sessions can be pinned and failed closed.

**Still measure on physical hardware:**
- whether target 2026 firmware conforms to the AOSP Secret verification behavior;
- whether the server certificate/SPKI is stable across standby, reboot and firmware update;
- reset/unpair identity behavior;
- the independent-trial/retry/rate-limit questions in ATV-21…ATV-30.

Those are hardware compatibility/security measurements, not invitations to keep reading more client
libraries.

## IR data — corrected path

### Flipper-IRDB

`Lucaslhm/Flipper-IRDB` at `d126fb1b6f1e114c52b4a8c19839ea65e3a9c24d` reports
`CC0-1.0` in repository metadata and carries the CC0 legal text. The README, however, states that
submissions are CC0 **and that commits before**
`2319685f2cbf0cd3f809609622cade14d24fb819` **are not covered**. That cutover commit is dated
2025-08-07 and is titled “feat: add LICENSE and add license note to README (#960)”.

Therefore:

- **HARVEST:** Flipper `.ir` format, naming conventions, community-maintained post-cutover data and
  CC0 contribution policy.
- **ADOPT candidate:** build a curated V1 seed only from content with provenance showing it was added
  or otherwise validly licensed under the post-cutover CC0 policy.
- **REJECT:** “the current checkout is entirely CC0 merely because the repository root has a CC0
  file.” Unchanged content inherited from before the cutover is not made safe by that statement.
- **No need to solve LIRC licensing for V1** if a provenance-clean Flipper subset plus user-import
  support gives adequate coverage. LIRC can remain an import format without becoming a bundled
  corpus.

### Existing IRDB and LIRC candidates

The existing `probonopd/irdb` custom permission remains usable only with its notification,
attribution and copy-provision obligations. It is no longer the only plausible seed candidate.
`probonopd/lirc-remotes` still has no primary database license established in our evidence and
therefore remains rejected as a shipping corpus.

### IR protocol reference correction

The supplied pack calls `crankyoldgit/IRremoteESP8266` an Apache-2.0 reference. Current GitHub
repository metadata identifies it as **LGPL-2.1**, with `LICENSE.txt` at the root. Harvest its
protocol coverage, behavior and test-vector ideas; do not plan a code port based on the incorrect
Apache claim.

## Android platform upstream — discovery/permission implications

Android's current `ConsumerIrManager` documentation directly supports the V1 runtime capability
gate: the system exposes whether an IR emitter exists, supported carrier ranges and transmission of
alternating microsecond patterns.

Android's current `NsdManager` documentation confirms DNS-SD/mDNS local service discovery. It also
shows why the uploaded pack's fixed manifest recipe should not become a discovery ADR: beginning
with API 37, local network access is restricted and apps targeting that API generally need
`ACCESS_LOCAL_NETWORK`, while newer NSD discovery can use a system picker to grant service-specific
access without the broad permission. Greenfield4 should preserve the product requirement (“explain
local-device access before the platform prompt”) and decide the exact permission strategy against
the chosen target SDK during architecture.

## Roku — authoritative contradiction, research closed

Roku's current official ECP documentation still describes SSDP/HTTP ECP behavior, but now explicitly
states that ECP commands may not be sent from third-party platforms such as mobile applications.
That directly contradicts the uploaded matrix's green recommendation to build Roku first.

**REJECT for V1 remains final under current evidence.** There is no value in further technical ECP
research unless Roku changes that official restriction or Greenfield4's product/security scope
changes.

## Uploaded ADR / security-spec reconciliation

The package marks nine ADRs as accepted. Those statuses are not imported into Greenfield4: they
belong to a different proposed product architecture and are research input only.

| Supplied decision | Greenfield4 disposition |
| :-- | :-- |
| Local-first, no backend, no account | **HARVEST → already adopted product requirement.** |
| Apache-2.0 core / GPL module boundaries | **REJECT as current decision.** Application license and module boundaries wait for architecture; the package also mislabels IRremoteESP8266 as Apache-2.0. |
| One TvAdapter interface / module per brand | **HARVEST abstraction principle; reject the concrete Kotlin/module contract during discovery.** |
| IR honesty policy | **HARVEST → already adopted.** Runtime hardware/capability gating is fixed product behavior. |
| Flipper-IRDB seed + imports + learn | **PARTIAL ADOPT.** Prefer a provenance-clean post-license Flipper subset; imports are useful; USB learning remains outside V1. |
| TOFU + visible prompts + Keystore | **PARTIAL ADOPT / PARTIAL REJECT.** Protected secrets and fail-closed reconnect identity are adopted. A phone-only fingerprint display does not authenticate Samsung first use without an independent TV-side value. |
| Premium IAP + donations | **REJECT as current decision.** PRODUCT.md keeps V1 free and defers business model. |
| Omit mirroring from V1 | **Product decision required.** The rationale is useful, but current Greenfield scope still lists casting/mirroring as PARTIAL V1. |
| ACRA self-hosted crash reporting | **HARVEST privacy posture; reject dependency choice now.** Exact telemetry library/endpoint waits for architecture. |
| Fixed Android permission list | **REJECT as frozen manifest.** Current Android upstream changes local-network permission behavior at API 37; exact manifest belongs to target-SDK architecture. |
| Keystore for persisted secrets | **HARVEST → adopt behavior.** Exact Android API remains architecture work. |
| No listening ports in V1 | **Consistent with core remote scope**, but must be revisited if casting/mirroring is explicitly retained and requires a phone-side server. |
| F-Droid / reproducible build / no tracker rules | **HARVEST release criteria**, not discovery exit gates. |

## Authoritative upstream checkpoints

The second pass re-checked primary platform/vendor sources where they can override OSS conclusions:

- Android ConsumerIrManager confirms runtime emitter detection, carrier-range queries, and IR pattern
  transmission. Built-in Android IR feasibility is closed.
- Android NsdManager confirms DNS-SD over mDNS. Current API 37 guidance also introduces local-network
  access controls and service-scoped picker flows, so the package's fixed permission manifest is not
  adopted.
- The AOSP google-tv-pairing-protocol source at `7c99785` remains the available authoritative
  reference for both pairing roles and Secret verification.
- Samsung Smart View SDK documentation describes a sender + receiver-app model and TLS/security
  mode for that SDK. It does not document the stock `samsung.remote.control` WebSocket used by
  generic remote clients, so it does not resolve Samsung first-use trust or legal-clear that channel.
- Current Roku ECP documentation explicitly says ECP commands may not be sent from third-party
  platforms such as mobile applications. This authoritatively rejects the package's “Roku build
  first” recommendation for Greenfield4.
- Current Google developer material found in this pass documents Cast / Cast Connect sender-receiver
  APIs, not a supported public API for third-party mobile apps to use the reverse-engineered Android
  TV Remote-v2 control channel. OSS feasibility must not be described as vendor API support.

## Research compression rule for the next phase

The following are now treated as **solved engineering patterns**, not open discovery research:

- Android built-in IR capability detection/transmit API.
- Android local service discovery via NSD/mDNS.
- local-first/no-cloud core-control model.
- Samsung token acquisition/control mechanics.
- secure phone-side storage of Samsung pairing tokens.
- Android TV Remote-v2 client certificate persistence.
- Android TV server-certificate capture and fail-closed reconnect pinning.
- capability-driven UI/domain separation from transport adapters.
- Tested / Expected / Unsupported compatibility reporting.

Do **not** spend another discovery cycle comparing more apps on these topics unless an authoritative
upstream source contradicts the pattern.

The remaining work is evidence that OSS cannot substitute for:

| Remaining question | Evidence needed |
| :-- | :-- |
| Samsung first-use unauthenticated identity | **Product-owner risk disposition**; no amount of another client-library survey creates an OOB authenticator. |
| Samsung / Android TV identity stability and reset behavior | **Physical hardware**, minimum release matrix. |
| Android TV practical first-use attackability | **Physical hardware** for independent trial/retry/rate-limit behavior. |
| Reverse-engineered vendor-protocol terms | **Current vendor/legal review**. |
| V1 IR seed | **Provenance-filtered corpus + coverage check**; choose clean Flipper subset or explicitly accept another source's obligations. |
| Product name | **Product-owner decision / conflict check**. |
| Casting/mirroring V1 inclusion | **Product-scope decision**, not more remote-protocol research. |

## Pinned OSS sources

| Source | Ref used | What this pass uses it for |
| :-- | :-- | :-- |
| `xchwarze/samsung-tv-ws-api` | `e48d6377faede37db1f034d726a079b9d8034fac` | Samsung WebSocket/token mechanics; trust-all anti-pattern |
| `sturlese/tvremote-app` | `7880e2c026c102053ccfbfbf6a05b452fafa310d` | secure Samsung token storage; local-first session pattern |
| `openhab/openhab-addons` | `cc5aa9383f49f417df0dd1dea34a8b519e201961` | broad Samsung compatibility; explicit TrustAll manager |
| `Perun85/Samsung.SmartTv.Client` | `ceb0700282298c2f7be072d9a8bade005bdf6232` | custom TV certificate-validator hook; trust-all default |
| `tronikos/androidtvremote2` | `b09f21432ba33e42536215a8f41641d801cf6a2c` | established Remote-v2 client behavior |
| `kud/androidtv-remote` | `5a05d73eb477688fa04117961d9c7596fce31828` | independent Remote-v2 client behavior |
| `ddagunts/ScreenCast` | `7e66bbe7ae5cc64c012bbe4987940be67925d137` | exact server-cert reconnect check + encrypted client credential; **pin is host-keyed, so Greenfield must not copy that association model** |
| `mbir31/TVgrip` PR #2 | merge `42a1c151de6fb6b86713ab30ec83962cd14e8cec` | second Android TV pinning / AndroidKeyStore implementation precedent |
| `SebghatYusuf/android_remote` | `130badada23d0332fe6aa8d44cec21e1c3f44801` | mobile secure client-key storage precedent; trust-all server anti-pattern |
| `Lucaslhm/Flipper-IRDB` | `d126fb1b6f1e114c52b4a8c19839ea65e3a9c24d` | CC0 policy + pre-`2319685` exclusion |
| `iodn/android-ir-blaster` | `3546eb19e71f002d0471515f05dafe47c4061026` | user import / external-learning interoperability patterns |
| `crankyoldgit/IRremoteESP8266` | current `master`, metadata checked 2026-09-15 | protocol coverage; license correction to LGPL-2.1 |
| `probonopd/lirc-remotes` | current primary README/metadata | LIRC corpus provenance gap |

## Authoritative upstream sources

- Android Developers: `ConsumerIrManager` API reference.
- Android Developers: `NsdManager` API reference, including Android 17/API 37 local-network access.
- AOSP / Git at Google: `platform/external/google-tv-pairing-protocol` at `7c99785`,
  `PairingSession.java` and `pairingsession.cc`.
- Samsung Developer: Smart View SDK sender/receiver and TLS/security-mode documentation; authoritative
  for that SDK, **not** for the stock remote WebSocket.
- Roku Developer: current External Control Protocol documentation, including the third-party/mobile
  restriction.

## Net result for discovery

The supplied green matrix is valuable, but it is not adopted wholesale. The highest-value parts are
now durable Greenfield evidence: capability honesty, platform-native IR/NSD, local-first operation,
adapter/capability separation, secure secret storage, and hardened Android TV reconnect pinning.

The second pass **reduces**, rather than expands, the research backlog. The project should not keep
surveying remote apps for already-demonstrated engineering patterns. Discovery now needs decisions,
legal clearance, provenance selection, and real-device evidence — not another generic OSS comparison.
