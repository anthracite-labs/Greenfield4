# Hardware Validation Matrix — Samsung Tizen and Android TV / Google TV Remote v2

**Date:** 2026-09-14
**Phase:** discovery
**Issue:** #11
**Status: NOT EXECUTED.** No test below has been run. No device was available in the sandbox.
**Companion:** [2026-09-14-security-trust-model.md](2026-09-14-security-trust-model.md)

## Why this document exists

The protocol-level trust model is documented from primary source code, but several questions in it
are **not answerable without a physical TV**: whether certificates are stable, whether the Android TV
verifies the pairing digest, how tokens rotate, and what reconnect actually does across standby,
reboot, and network change. Those questions are turned into tests here rather than being left as
vague "needs hardware validation" notes.

**Nothing here is a result.** A test is only "passed" once it has been executed against a real
device and the recorded evidence exists. Until then every row is `NOT RUN`.

## Required environment

### Devices

| ID | Requirement | Rationale |
| :-- | :-- | :-- |
| SAM-A | Samsung Tizen TV, 2016+ (e.g. a current mid-range UHD model) | Baseline Tizen target |
| SAM-B | Samsung Tizen TV, **different model year and firmware generation** from SAM-A | ADR-0005 requires ≥2 models/firmware per ecosystem; one TV proves an integration, not an ecosystem |
| ATV-A | Android TV / Google TV device with Remote v2 (e.g. NVIDIA SHIELD or a Google TV Sony/TCL model) | Baseline Android TV target |
| ATV-B | Android TV / Google TV device, **different vendor and OS build** from ATV-A (e.g. Chromecast with Google TV) | Same ≥2-generation requirement |
| PHONE-1 | Android phone, the Greenfield4 test client | Primary client |
| PHONE-2 | Second Android phone | Distinguishes client-side from TV-side state; needed for the independent-pairing tests |
| AP-1 | Consumer router/AP with a configurable DHCP pool | Network/IP-change and DHCP tests |
| PROXY-1 | Laptop running a TLS interception proxy (e.g. mitmproxy) on the same LAN | Substituted-certificate tests |

### Required firmware/OS versions

Recorded per run, not assumed. For every device, capture **before** testing:

- vendor, model code, and retail model name;
- firmware / OS build (Tizen version; Android TV OS build and security patch level);
- the remote-control service version where the ecosystem exposes one (Android TV: remote service
  ≥5 for Remote v2);
- network: Wi-Fi band or Ethernet, and whether the TV uses Wi-Fi power saving.

Firmware must be recorded because it is the primary suspected cause of certificate change, and the
matrix must be repeatable at a stated version.

## General rules

- Run every Samsung test against **SAM-A first**, then repeat on **SAM-B**. Same for Android TV
  with ATV-A then ATV-B. A result on one device does not transfer.
- Perform pairing tests from a **clean client state** (no stored identity) unless the test says
  otherwise.
- Capture the same evidence for every test (see below). Absence of evidence is a fail, not a pass.
- Where a test can destroy a pairing, run it last in its group or re-pair afterwards.
- Substituted-certificate tests must be run **only on a lab network you own**. Do not test MITM
  against a network carrying other people's traffic.

## Evidence to record for every test

1. Device IDs, firmware/OS versions, date and time, and network (SSID/band, client IP, TV IP).
2. The Greenfield4 client build or commit under test.
3. Client log at debug level, **redacted**: no pairing secret, token, private key, or full
   certificate chain in the artefact. Record fingerprints, not secrets.
4. Where a certificate is involved: SHA-256 fingerprint of the DER certificate, and separately the
   SHA-256 of the SubjectPublicKeyInfo, plus subject and issuer.
5. Packet capture, if taken, stored with the same redaction rule.
6. Observed vs expected, and the pass/fail verdict.

---

# Samsung Tizen tests

| ID | Test | Expected observation | Pass criterion | Evidence |
| :-- | :-- | :-- | :-- | :-- |
| SAM-01 | **Initial pairing.** Clean client, connect to `wss://<tv>:8002`, approve the on-TV popup. | Popup appears on TV; client receives a token; control works. | Token received **and** a certificate fingerprint is captured at pairing time. | Screenshot of popup; fingerprint; redacted log |
| SAM-02 | **Fingerprint capture.** After SAM-01, record cert DER SHA-256 and SPKI SHA-256. | Two stable-looking values. | Both values captured and written to the run sheet. | Fingerprints, subject, issuer |
| SAM-03 | **App restart / reconnect.** Force-stop and relaunch the client. | Client reconnects using the stored token; no new popup. | Reconnect succeeds **and** presented cert matches SAM-02 fingerprint. | Fingerprint comparison |
| SAM-04 | **Phone restart.** Reboot the phone, open the app. | Reconnect with stored identity. | Same as SAM-03. | Fingerprint comparison |
| SAM-05 | **TV standby / wake.** Put the TV in standby, wake it, reconnect. | Token still valid; cert unchanged. | Reconnect succeeds, fingerprint matches. | Fingerprint comparison; time-to-reconnect |
| SAM-06 | **TV power cycle.** Disconnect mains for ≥60 s, restore, reconnect. | Token still valid; cert unchanged. | Fingerprint matches. | Fingerprint comparison |
| SAM-07 | **TV reboot.** Reboot from the TV menu, reconnect. | Token still valid; cert unchanged. | Fingerprint matches. | Fingerprint comparison |
| SAM-08 | **Network change (Wi-Fi → Ethernet, or different SSID).** | Client rediscovers the TV at its new address. | Reconnect succeeds; **fingerprint unchanged**; product hides the IP change from the user. | Old/new IP; fingerprint |
| SAM-09 | **DHCP address change.** Force a new DHCP lease for the TV, then reconnect. | Reconnect at the new address. | Fingerprint unchanged across the address change — this is the test that proves identity is not the IP. | Old/new IP; fingerprint |
| SAM-10 | **App removal / reinstall.** Uninstall the client, reinstall, re-pair. | Old token does **not** survive. | New pairing required; a new token is issued; fingerprint matches SAM-02 (same TV, same cert). | Token-change note; fingerprint |
| SAM-11 | **Pairing-token persistence.** Reboot phone, kill app, cold start, reconnect over 7 days. | Token remains valid. | No re-prompt within the observation window. | Timestamped reconnect log |
| SAM-12 | **Token rotation observation.** Leave paired and observe for ≥30 days, logging every re-pair prompt. | Unknown — this is what the test establishes. | **No pass/fail**: record the rotation interval and whether the fingerprint changes when rotation occurs. | Prompt timestamps; fingerprints before/after |
| SAM-13 | **Certificate persistence across firmware update.** Record fingerprint, apply a TV firmware update, reconnect. | Unknown. | Fingerprint either stable (**pinning viable**) or changed (**pinning breaks on update → re-pair required**). Either is a usable result; "unknown" is not. | Pre/post firmware; pre/post fingerprint |
| SAM-14 | **Factory reset.** Factory reset the TV, then attempt reconnect with the stored identity. | TV rejects the token and/or presents a new certificate. | Connection **fails closed** and the client requires re-pairing — it must not silently accept a new identity. | Pre/post fingerprint; failure mode observed |
| SAM-15 | **Explicit unpair / re-pair.** Unpair from the TV's device list, then pair again. | Old token rejected; new pairing issues a new token. | Old token rejected; new token works. | Old/new token validity |
| SAM-16 | **Unexpected identity change.** Point the client at a different Samsung TV (SAM-B) while holding SAM-A's stored identity. | Certificate differs from the pinned value. | Client **refuses**, surfaces "TV identity changed", requires re-pair. Silent acceptance is a **FAIL**. | Pinned vs presented fingerprint; UI state |
| SAM-17 | **Substituted certificate (MITM).** On the lab LAN, interception proxy presents its own cert for the TV's address. Repeat (a) with no prior pairing and (b) with a prior pairing. | (a) Unknown — hypothesised to be vulnerable at first use. (b) With pinning, must fail closed. | (b) **must fail closed**. (a) records whether first-use MITM is real. Both results recorded honestly, including a result that confirms the vulnerability. | Proxy cert fingerprint; client behaviour |

---

# Android TV / Google TV Remote v2 tests

| ID | Test | Expected observation | Pass criterion | Evidence |
| :-- | :-- | :-- | :-- | :-- |
| ATV-01 | **Initial pairing.** Clean client (no stored cert) to port 6467; TV displays a 6-hex-symbol code; enter it. | `secret_ack` / `pairingSecretAck` received. | Pairing completes **and** the client captures the server cert DER and SPKI fingerprints at pairing time. | Code format as displayed (confirm 6 hex symbols); fingerprints |
| ATV-02 | **Fingerprint capture.** After ATV-01, record server cert DER SHA-256, SPKI SHA-256, and the parsed name/MAC from the subject. | Stable-looking values. | All recorded. | Fingerprints; subject string |
| ATV-03 | **App restart / reconnect.** Relaunch with the saved client cert; connect to 6466. | No code; reconnect succeeds. | Reconnect succeeds **and** presented server cert matches ATV-02. | Fingerprint comparison |
| ATV-04 | **Phone restart.** Reboot, open app. | Reconnect with saved client cert. | Same as ATV-03. | Fingerprint comparison |
| ATV-05 | **TV standby / wake.** | Reconnect succeeds. | Fingerprint matches. | Fingerprint comparison; reconnect latency |
| ATV-06 | **TV power cycle.** Mains off ≥60 s. | Reconnect succeeds. | Fingerprint matches. | Fingerprint comparison |
| ATV-07 | **TV reboot.** | Reconnect succeeds; client cert still accepted. | Fingerprint matches; no `unpaired`. | Fingerprint comparison |
| ATV-08 | **Network change.** Move the TV between SSIDs/bands. | Rediscovery at the new address. | Fingerprint unchanged; user not exposed to IP details. | Old/new IP; fingerprint |
| ATV-09 | **DHCP address change.** | Reconnect at the new address. | Fingerprint unchanged — proves identity is not the IP. | Old/new IP; fingerprint |
| ATV-10 | **App removal / reinstall.** | Client cert is gone; TV does not recognise a new client. | Re-pairing required; a new code is shown. | New code; new client-cert fingerprint |
| ATV-11 | **Pairing-credential persistence.** Client cert survives app restart, phone reboot, and 7 days of use. | No re-pair. | No re-pair prompt within the window. | Timestamped log |
| ATV-12 | **Client-certificate persistence across firmware update.** Update the TV, reconnect with the same client cert. | Unknown. | Either the same client cert is still accepted, or the TV demands re-pair. Record which. | Pre/post firmware; accepted/rejected |
| ATV-13 | **Server-certificate persistence across firmware update.** Record fingerprints, update TV firmware, reconnect. | Unknown. | Fingerprint stable (pinning viable) or changed (pinning breaks → re-pair). Record which. | Pre/post firmware; pre/post fingerprint |
| ATV-14 | **Factory reset.** Reset the TV, reconnect with the stored client cert and pinned server cert. | TV rejects the client and/or presents a new cert. | **Fails closed**; re-pair required. Silent acceptance is a **FAIL**. | Pre/post fingerprint; failure mode |
| ATV-15 | **Explicit unpair / re-pair.** Remove the client from the TV's accessory list, then re-pair. | TV rejects the old client cert; new pairing issues a new code. | Old cert rejected (`unpaired` / `InvalidAuth`); new pairing works. | Error surfaced; new code |
| ATV-16 | **Unexpected identity change.** Point the client at ATV-B while holding ATV-A's pinned identity. | Server cert differs. | Client **refuses**, surfaces "TV identity changed", requires re-pair. Silent acceptance is a **FAIL**. | Pinned vs presented fingerprint; UI state |
| ATV-17 | **Substituted certificate during pairing (MITM).** Interception proxy terminates 6467 with its own cert; complete the code flow. | Unknown — this is the decisive test. | **No pass/fail.** Record whether pairing **succeeds** (TV does not verify the digest → the binding is not enforced) or **fails** (TV verifies → first-use MITM resisted). | Proxy cert fingerprint; pairing outcome |
| ATV-18 | **Substituted certificate on reconnect (MITM).** Proxy terminates 6466 with its own cert after a successful pairing. | With pinning: rejected. | **Must fail closed.** Without pinning, record that it is accepted — which is the verified weakness in the reference posture. | Proxy cert fingerprint; client behaviour |
| ATV-19 | **Wrong code.** Enter an incorrect 6-symbol code during pairing. | Pairing rejected; no credential issued. | Pairing fails; client surfaces an error and does not fall back to an insecure path. | Error surfaced |
| ATV-20 | **Independent second phone.** Pair PHONE-2 to the same TV without touching PHONE-1. | Both work independently; PHONE-1's credentials unaffected. | Both control the TV; no credential sharing between phones. | Two client-cert fingerprints; both functional |

---

## Priority order

The decisive tests — the ones whose outcome would change a conclusion in the trust model — are:

1. **ATV-17** — determines whether the Android TV pairing binding is actually enforced. This is the
   single largest open question on either track.
2. **ATV-18** — determines whether the fail-closed pinning design works on reconnect.
3. **SAM-17** — determines the real-world shape of the Samsung first-use exposure.
4. **SAM-13 / ATV-13** — determine whether pinning survives firmware updates, which decides whether
   pinning is shippable or merely theoretical.
5. **SAM-14 / ATV-14** — confirm the factory-reset re-pair trigger that the DOMAIN.md state machine
   depends on.

## What this matrix does not cover

- Capability, wake, casting, and voice behaviour — separate discovery items.
- IR profile validation.
- Vendor legal/terms review.
- Performance and latency beyond the reconnect observations noted above.

## Related

- [2026-09-14-security-trust-model.md](2026-09-14-security-trust-model.md) — the trust model these tests validate
- [2026-09-14-ecosystem-evidence.md](2026-09-14-ecosystem-evidence.md) — ecosystem selection evidence
- [../PRODUCT.md](../PRODUCT.md) — release hardware-matrix requirement
