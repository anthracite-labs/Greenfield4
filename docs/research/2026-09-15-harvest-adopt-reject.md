# Harvest / Adopt / Reject Reconciliation — TV Control and Remaining Discovery Gaps

**Date:** 2026-09-15  
**Issue:** #14  
**Phase:** discovery  
**Status:** reconciliation complete at source level; hardware/legal/product-owner gates remain where explicitly listed

## Purpose

This document reconciles the accepted Greenfield4 discovery/security record with a targeted second
pass over maintained open-source implementations and authoritative upstream/vendor sources.

It is deliberately **not** a new ecosystem survey. ADR-0005 already fixes the V1 direction as
Android TV / Google TV plus Samsung Tizen. The goal here is to stop spending research time on
engineering mechanics that credible open-source projects already implement, while preserving
Greenfield4's stricter trust and product invariants.

The working strategy is:

- **HARVEST** — treat a proven external implementation pattern as sufficient engineering evidence
  to stop broad exploratory research. Harvesting a pattern does not mean copying source code,
  accepting its security posture, or adopting it as a dependency.
- **ADOPT** — carry the pattern or rule into Greenfield4 because it satisfies the product/domain
  invariants, with any Greenfield-specific hardening stated explicitly.
- **REJECT** — do not carry the external shortcut into Greenfield4 because it weakens security,
  creates unnecessary setup, has unresolved licensing/terms, or conflicts with an accepted product
  requirement.

A row can therefore be **HARVEST → ADOPT** (reuse the solution), **HARVEST → ADOPT WITH
HARDENING**, or **HARVEST → REJECT** (learn from the implementation, but deliberately do not copy
its compromise).

## 1. Reconciliation: what remains green

Nothing below is reopened by this pass.

| Recorded Greenfield4 finding | Reconciliation |
| :-- | :-- |
| V1 ecosystem direction is Android TV / Google TV + Samsung Tizen | **KEEP.** ADR-0005 remains accepted for product direction only. |
| Both ecosystems' common reference/community clients disable normal server-certificate authentication | **KEEP.** Current OSS still demonstrates this clearly. |
| Samsung stock WebSocket pairing has no client-observable cryptographic binding to a trusted TV identity at first use | **KEEP.** The deeper OSS/vendor pass found no stock-TV first-use identity channel that changes this conclusion. |
| Samsung TOFU/pinning can protect reconnects but cannot authenticate the first connection | **KEEP.** Custom certificate-validation hooks make reconnect hardening more concrete, not first-use stronger. |
| Android TV pairing binds certificate key material into the pairing challenge, and the AOSP output-device role verifies the full Secret before SecretAck | **KEEP.** Rechecked against Google AOSP at pinned commit `7c99785`. |
| Android TV's deployed six-symbol flow provides an 8-bit out-of-band alpha-prefix gate per **independent pairing trial**, not 256 bits of cross-leg first-use authentication | **KEEP.** No source found invalidates the existing derivation. Practical independent-trial availability remains hardware-required. |
| Neither ecosystem is security-cleared merely because an OSS app works | **KEEP.** Interoperability evidence and Greenfield security evidence remain separate. |
| IRDB is custom-permission licensed; runtime CDN access does not remove its obligations | **KEEP.** Rechecked at pinned IRDB commit `11aa5eb3`. |
| LIRC remotes database/config licensing is unresolved and cannot be inferred from LIRC software GPL | **KEEP.** The source data repository still supplies no clear database/config license. |
| Hardware matrix is unexecuted; no row may be called Tested | **KEEP.** OSS evidence reduces research duplication, not Greenfield release verification. |
| Vendor terms/legal and the Samsung first-use residual-risk decision are separate gates | **KEEP.** OSS prevalence is not legal clearance or product-owner acceptance. |
| Repository remains in discovery; no app stack may be introduced | **KEEP.** `PROJECT_PHASE=discovery`, `ALLOW_APP_STACK=0`, `STACK_DECISION_ADR=`. |

## 2. Source set for the second pass

### 2.1 Open-source implementations — pinned primary source

| Project | Pinned revision | Why it matters |
| :-- | :-- | :-- |
| [xchwarze/samsung-tv-ws-api](https://github.com/xchwarze/samsung-tv-ws-api/tree/e48d6377faede37db1f034d726a079b9d8034fac) | `e48d6377` | Mature Samsung WebSocket mechanics; explicitly uses `CERT_NONE`; token/file persistence and command flow. |
| [sturlese/tvremote-app](https://github.com/sturlese/tvremote-app/tree/7880e2c026c102053ccfbfbf6a05b452fafa310d) | `7880e2c0` | End-to-end Samsung mobile app: Zeroconf/IP discovery, TV Allow/Deny, token reuse, OS SecureStore; also demonstrates an overly broad iOS ATS bypass. |
| [Perun85/Samsung.SmartTv.Client](https://github.com/Perun85/Samsung.SmartTv.Client/tree/ceb0700282298c2f7be072d9a8bade005bdf6232) | `ceb07002` | Exposes an application-controlled `IRemoteCertificateValidator`; proves per-connection Samsung certificate validation is technically feasible even though its default accepts all certs. |
| [tronikos/androidtvremote2](https://github.com/tronikos/androidtvremote2/tree/b09f21432ba33e42536215a8f41641d801cf6a2c) | `b09f2143` | Widely reused Remote v2 protocol implementation; persists client cert/key, reads the server cert for pairing computation, but uses `CERT_NONE`. |
| [Home Assistant core](https://github.com/home-assistant/core/tree/ec2a00de1ea269004463a710648079ead65c24b2) | `ec2a00de` | Production integration using `androidtvremote2`, corroborating the client-certificate lifecycle as established interoperability practice. |
| [SebghatYusuf/android_remote](https://github.com/SebghatYusuf/android_remote/tree/130badada23d0332fe6aa8d44cec21e1c3f44801) | `130badad` | Android TV mobile implementation using `FlutterSecureStorage` for client cert/key and capturing the peer cert; still accepts bad/self-signed certs unconditionally. |
| [ddagunts/ScreenCast](https://github.com/ddagunts/ScreenCast/tree/7e66bbe7ae5cc64c012bbe4987940be67925d137) | `7e66bbe7` | Strongest reusable Android pattern found: Android Keystore-backed encrypted client credential, server-cert SHA-256 captured only after successful pairing, strict pin on 6466 reconnect, fail closed on mismatch. |
| [probonopd/irdb](https://github.com/probonopd/irdb/tree/11aa5eb3ad9fec9e5c03f170c29c1467733d9f3e) | `11aa5eb3` | Primary IRDB license and runtime-access guidance. |
| [probonopd/lirc-remotes](https://github.com/probonopd/lirc-remotes/tree/e4a758048908b7e1a571dc2a89c409a33398f2f6) | `e4a75804` | Primary converted LIRC remote-data repository; README identifies provenance but does not provide a database/config license. |

### 2.2 Authoritative upstream/vendor sources

- Google AOSP, [Google TV Pairing Protocol @ `7c99785`](https://android.googlesource.com/platform/external/google-tv-pairing-protocol/+/7c99785):
  `PairingSession.java` and `pairingsession.cc` independently show the input-device local
  `checkGamma` before transmitting Secret and the output-device full Secret verification before
  SecretAck. This confirms the directionality already recorded in the Greenfield trust model.
- Samsung Developer, [Smart View SDK — Getting Started](https://developer.samsung.com/smarttv/develop/extension-libraries/smart-view-sdk/getting-started.html):
  Samsung's documented Smart View model is **sender + receiver application**, with TLS support.
- Samsung Developer, [Receiver Apps](https://developer.samsung.com/smarttv/develop/extension-libraries/smart-view-sdk/receiver-apps.html):
  the documented secure channel is an app-to-app receiver model; it is not documentation for the
  reverse-engineered stock-TV `samsung.remote.control` WebSocket.
- Samsung Developer, [Terms of Use](https://developer.samsung.com/terms?location=us), effective
  2026-01-29: current developer-site terms were reachable in this pass. They do not by themselves
  grant or deny use of the undocumented stock-TV remote endpoint; that legal interpretation remains
  a human/legal task.
- Targeted searches of `developer.samsung.com` for `samsung.remote.control`,
  `ms.remote.control`, and the port-8002 stock-TV remote path returned no official documentation.
  **This is absence from the searched public documentation, not proof of prohibition.**

The AOSP repository is authoritative for the historical pairing-protocol implementation, but it is
not contemporary firmware evidence. The second pass did not find a modern public Google API
specification for the deployed Android TV Remote v2 service. That leaves maintenance/terms and
firmware conformance separate from the protocol mechanics.

## 3. Harvest / Adopt / Reject matrix

| Area | What OSS/upstream already establishes | Strategy | Greenfield4 disposition | More research? |
| :-- | :-- | :-- | :-- | :-- |
| **Samsung discovery** | Maintained apps combine Zeroconf/service discovery with IP-scan/fallback and hide normal DHCP movement from users. | **HARVEST → ADOPT** | Use automatic local discovery. Treat IP as routing state only; never as the security identity. | **Stop broad research.** Verify our integration on release hardware. |
| **Samsung Allow/Deny + token pairing flow** | Multiple independent implementations agree on `wss://:8002/api/v2/channels/samsung.remote.control`, TV approval, `ms.channel.connect`, token reuse. | **HARVEST → ADOPT** | Reuse the established flow and failure states. | **Stop broad research.** Hardware smoke test only. |
| **Samsung token storage** | `tvremote-app` demonstrates OS SecureStore; `samsung-tv-ws-api` also demonstrates the common but weaker plaintext token-file pattern. | **HARVEST → ADOPT WITH HARDENING** | Store the token in Android platform-secure storage; redact from logs/backups/analytics. | No protocol research. Verify storage behavior in our app. |
| **Samsung plaintext token files** | Common and interoperable. | **HARVEST → REJECT** | Never use normal files/preferences for pairing tokens. | None. |
| **Samsung TLS certificate bypass** | `samsung-tv-ws-api` uses `CERT_NONE`; `tvremote-app` uses broad `NSAllowsArbitraryLoads`; `Samsung.SmartTv.Client` defaults to an always-valid validator. | **HARVEST → REJECT** | No global or reconnect-time “accept any certificate” mode. Compatibility is not authentication. | None; already sufficiently evidenced. |
| **Samsung reconnect pinning** | `Samsung.SmartTv.Client` exposes the peer certificate to a custom validator, proving app-controlled validation is feasible. | **HARVEST → ADOPT** | Capture DER SHA-256 (and record SPKI SHA-256 for diagnostics) during approved first pairing; subsequent sessions must match the stored security identity or fail closed and require explicit re-pair. | Only **hardware stability** across reboot/update/reset remains. |
| **Samsung first-use identity** | Every stock-remote OSS implementation found must initially accept a self-signed/unverified peer; none supplies an independently authenticated TV fingerprint. Samsung's official Smart View TLS applies to a sender/receiver-app model, not this stock remote endpoint. | **HARVEST → REJECT FALSE CLOSURE** | Do not claim TOFU, the Allow popup, or a fingerprint displayed only by the phone authenticates the first connection. | **No more broad protocol research.** Product-owner residual-risk decision remains; only a genuinely independent authenticated TV-side channel would change it. |
| **Samsung companion receiver app as workaround** | Samsung officially supports signed receiver apps and secure Smart View channels. | **HARVEST → REJECT FOR V1 CORE REMOTE** | It is a different product/setup model, not a transparent fix for stock-TV pairing. Keep as a future option, not a V1 dependency. | None unless product direction changes. |
| **Android TV discovery** | `_androidtvremote2._tcp` mDNS is implemented across OSS; ScreenCast also consumes model/MAC-like TXT hints. | **HARVEST → ADOPT** | Use mDNS for routing discovery. Ecosystem identifiers are lookup hints only; certificate pin is the security identity. | Stop broad research; smoke-test target devices. |
| **Android client certificate lifecycle** | `androidtvremote2`, Home Assistant and mobile implementations generate/persist a client cert/key and reuse it after pairing. | **HARVEST → ADOPT** | One protected client identity per install/profile; losing it means re-pair. | Stop broad research. |
| **Android private-key storage** | ScreenCast uses Android Keystore-derived encrypted storage; another mobile implementation uses platform secure storage. | **HARVEST → ADOPT** | Use Android platform-backed secret storage; never plain PEM files in normal app storage as the desired production posture. | No protocol research. |
| **Android pairing mechanics** | `androidtvremote2` and other clients implement 6467 pairing; AOSP `7c99785` confirms the protocol roles and Secret verification semantics. | **HARVEST → ADOPT** | Reuse protocol shape; preserve strict input validation and explicit pairing errors. | **Stop protocol research.** Contemporary TV conformance remains hardware-required. |
| **Android reconnect server identity** | ScreenCast captures TV server-cert SHA-256 on successful pairing and enforces the exact pin on 6466; mismatch closes the socket. | **HARVEST → ADOPT WITH HARDENING** | Adopt fail-closed pinning. Persist both DER/SPKI evidence. Do **not** key the security record by host/IP: store the pin in the paired-device record and let rediscovery update only the route after the presented cert proves the record. | Hardware proof for cert stability/update/reset only. |
| **Android trust-all TLS on reconnect** | `androidtvremote2` and other clients use `CERT_NONE`/`onBadCertificate => true`. | **HARVEST → REJECT** | Unacceptable after pairing because it discards the identity Greenfield can persist. | None. |
| **Android self-signed TLS on the pairing port** | ScreenCast scopes a trust-all manager to the pairing SSL context, captures the cert, completes the OOB pairing, and only then persists the pin. | **HARVEST → ADOPT WITH CONSTRAINT** | A scoped first-contact TLS context may accept the self-signed cert **only as transport**, never as authenticated identity; persist the pin only after the pairing protocol succeeds. No global trust override. | First-use strength is governed by the OOB protocol, already analyzed; practical trial availability remains hardware-required. |
| **Android first-use OOB strength** | AOSP confirms the local-gamma/full-Secret roles; current Greenfield analysis establishes 8 bits of OOB alpha-prefix binding per independent trial in the deployed six-symbol format. | **HARVEST → ADOPT CURRENT MODEL** | Keep the bounded-risk classification; do not reinterpret full 256-bit alpha as 256-bit cross-leg first-use authentication. | No more source reading unless new protocol/firmware evidence appears. Execute the targeted retry/session hardware tests. |
| **Android contemporary server-side Secret enforcement** | Historical AOSP says the output role rejects a bad Secret before SecretAck. OSS clients assume compatible deployed behavior. | **HARVEST, BUT NOT ENOUGH TO ADOPT AS FACT** | Keep `[HARDWARE-REQUIRED]`; execute the existing corrupted-Secret test on each release firmware generation. | **Hardware only.** |
| **Official Flipper IRDB** | Top-level MIT, but 2,808 TV paths already existed in the 2024-07-08 `mi_remote_database` import. Original imported files explicitly carried AGPL-3.0/Ysard notices; later parsing removed those comments from many transformed files. | **HARVEST → REJECT BLANKET ADOPTION; ADOPT ONLY PER-FILE CLEAN PROVENANCE** | Harvest format, parsing, normalization and post-import contributions. Never treat the whole corpus as one MIT bucket; retain source commit/path/license for every shipped profile. | Broad research stops; per-profile provenance is a bounded implementation/release check. |
| **Community Flipper-IRDB** | README applies CC0-1.0 to contributions but expressly excludes commits prior to `2319685`. Only 20 TV paths are first-introduced after that boundary at pinned head. | **HARVEST → ADOPT POST-BOUNDARY NEW FILES ONLY** | Use only clearly post-boundary first-introduced contributions with retained commit provenance. Too small to be the sole V1 source. | Stop broad research; consume individual clean additions if useful. |
| **IRDB source** | Primary license explicitly allows inclusion/network access subject to notification, notice, and up-to-three-copy obligations; README recommends runtime access but points back to the same license. | **HARVEST → DEFER / FALLBACK** | Do not make IRDB acceptance a discovery gate. Use only if the provenance-clean V1 manifest has a concrete supported-device coverage gap worth accepting the obligations for. | License research is complete; revisit only against a real gap. |
| **LIRC remotes data** | Primary converted-data repo identifies provenance but no explicit data/config license was found; software GPL is not a database license. | **HARVEST → REJECT FOR V1 SHIPPING** | Do not use as a shipping data source unless explicit primary permission/license is obtained later. | Stop research for V1. |
| **Vendor/API legal status** | OSS demonstrates widespread technical interoperability. Samsung public docs demonstrate a supported Smart View app-to-app path, but targeted official search does not document the raw stock remote endpoint; no modern public Remote-v2 API spec was found in the targeted Google pass. | **HARVEST → REJECT “OSS USE = LEGAL CLEARANCE”** | Keep legal/terms as a human gate. Present counsel/owner the exact protocols and current vendor terms rather than continuing broad web research. | **Human/legal review**, not more OSS surveying. |
| **Hardware validation** | OSS can establish that the mechanics work somewhere; it cannot prove Greenfield's release build, fail-closed pinning, firmware stability, or target-device conformance. | **HARVEST → ADOPT DELTA TESTING** | Stop treating ordinary discovery/pairing/command mechanics as open research questions. Keep physical tests for Greenfield-specific trust boundaries and support claims. | Targeted hardware execution remains. |
| **Naming** | Existing repo research already supplies criteria/candidates; no external technical gap remains. | **HARVEST → ADOPT EXISTING RESEARCH** | Stop naming research until product owner selects/finalizes a name and conflict check is run. | Owner decision, then focused clearance. |

## 4. Greenfield design rules harvested from the pass

These are the reusable rules that should flow into architecture **after discovery exits**:

1. **Route is not identity.** IP, hostname, mDNS instance and MAC-like TXT values can locate a TV;
   they cannot replace a cryptographic identity. A paired-device record owns the pin; rediscovery
   proposes a new route and the pinned certificate proves it.
2. **Secrets use platform-secure storage.** Samsung token and Android client private key are
   password-equivalent pairing material.
3. **First contact and reconnect are different trust states.** A self-signed certificate may have
   to be accepted as unauthenticated transport during pairing, but the app must never carry that
   posture into an already-paired reconnect.
4. **Pin only after the user-mediated pairing protocol succeeds.** Merely observing a certificate
   on the wire is not sufficient.
5. **Fail closed on identity change.** Do not silently replace a stored pin. Surface identity
   change and require explicit re-pair.
6. **No global certificate bypass.** Do not copy `CERT_NONE`,
   `NSAllowsArbitraryLoads=true`, a global bad-certificate callback, or an always-valid validator
   as the product's steady-state security posture.
7. **Compatibility precedent is not security proof.** A mature OSS app can end broad research into
   command framing, discovery or pairing UX; it cannot certify Greenfield's first-use trust,
   firmware behavior or legal status.
8. **Do not key a pin by IP.** ScreenCast's reconnect-pinning concept is strong, but its
   host-keyed pin store is intentionally not adopted. Greenfield must bind the pin to its own
   paired-device record.

## 5. Research stop rule and hardware delta

The full 2026-09-14 hardware matrix remains a useful release-test catalogue and **none of its rows
is retroactively marked passed**. This reconciliation changes what counts as an *open research
question*, not what must be tested before claiming broad support.

### Stop researching and move to implementation verification later

The following are now harvested engineering patterns, not discovery unknowns:

- Samsung and Android TV discovery mechanics;
- Samsung Allow/Deny/token flow and command framing;
- Android TV Remote-v2 client-certificate lifecycle and pairing message flow;
- OS-backed storage of pairing material;
- reconnect certificate pinning as an implementable client-side mechanism;
- fail-closed identity-change handling as the Greenfield target posture.

When architecture/implementation begins, these become ordinary integration/unit/release tests rather
than invitations for another literature survey.

### Hardware remains load-bearing where source cannot answer the device

The full matrix remains mandatory **before release support is labelled Tested**, but Issue #14
separates that release gate from the smaller set that must be known before discovery can close.

**Decision-critical discovery probes:**

- **Android TV:** ATV-17a/17b on ATV-A and ATV-17d on ATV-B — establish whether contemporary target
  firmware performs the server-side full-Secret verification documented by historical AOSP.
- **Android TV:** ATV-21/22/23 — establish the dominant local prefix-failure loop, whether the next
  trial can be made cryptographically independent, and whether the TV observes that local failure.
- **Android TV:** ATV-26 — establish whether an unauthenticated LAN peer can cause a fresh pairing
  code/session without on-device user action.
- **Android TV:** ATV-29 — count independent trials obtainable inside one user-mediated pairing
  session.
- **Conditional:** run ATV-24/25 as discovery probes only if ATV-23 shows the TV observes local
  failures and can therefore plausibly throttle that loop. Run ATV-27/28 only if the initiation or
  code-lifetime facts remain necessary to bound the result.

Samsung's structural first-use conclusion does **not** depend on running SAM-17(a): the source-level
problem is the absence of an independently authenticated TV identity on first contact. Its discovery
gate was therefore the product-owner residual-risk disposition, which was **accepted on
2026-09-15**. No further Samsung first-use research is required unless contradictory primary
evidence appears. Samsung certificate stability, substituted-cert handling, token/recovery
behaviour, and the remaining Android reconnect/persistence rows remain important implementation and
release-conformance work.

**Release hardware still establishes:**

- Samsung certificate/SPKI stability across reboot/firmware/reset, token/recovery behaviour,
  different-TV/substituted-cert fail-closed behavior, and broad model coverage;
- Android server-certificate stability across firmware/reset, substituted-cert fail-closed behavior,
  credential/recovery behaviour, and broad model coverage;
- minimum two model/firmware generations per ecosystem before broad “Tested” claims.

Long-duration observations (for example token rotation) may continue as release/field evidence; they
must not be replaced with an OSS author's anecdote.

## 6. Legal/compliance routing after this pass

More broad OSS research is unlikely to improve the legal decision.

- **IRDB:** the license question is sufficiently answered. Route to product-owner/legal acceptance,
  then comply or reject the source.
- **LIRC data:** reject for V1 unless explicit data permission appears; do not spend V1 schedule
  trying to infer a license from adjacent GPL software.
- **Samsung/Android remote protocols:** present the exact undocumented/reverse-engineered endpoints,
  the official public alternatives, and the current vendor terms to the human/legal reviewer.
  Technical prevalence is supporting context only.

## 7. Net effect on discovery

This pass **reduces research scope without changing the lifecycle or weakening a security
requirement**.

Resolved as engineering-pattern questions:
- discovery, pairing mechanics, secure credential storage, and reconnect pinning are reusable;
- Android reconnect hardening has a working OSS precedent;
- Samsung custom certificate validation is demonstrably implementable;
- IR datasets no longer need broad licensing research. Use the provenance-clean profile strategy in
  `2026-09-15-ir-dataset-second-pass.md`; IRDB is a fallback, not a discovery blocker.

Still genuinely open:
1. targeted physical-hardware evidence for the Android TV device-side/security facts listed above;
2. human vendor terms/legal review;
3. final product name/slug and focused clearance.

**Samsung product-owner disposition (2026-09-15): ACCEPT.** Samsung remains in V1 with the
documented stock-remote first-use MITM residual risk consciously accepted. This acceptance is narrow:
it does not authorize global certificate bypass, insecure fallback, silent identity replacement, or
weaker reconnect handling. Greenfield must pin the paired TV identity after the approved pairing
flow and fail closed on unexpected identity changes.

No application-stack decision is made here. Architecture remains locked until the discovery exit
criteria are deliberately reconciled and approved.
