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

## 2026-09-15 evidence-reuse policy

Issue #14 reconciled this catalogue against maintained open-source implementations and upstream
sources in [2026-09-15-harvest-adopt-reject.md](2026-09-15-harvest-adopt-reject.md).

The rows remain recorded and remain `NOT RUN`. The reconciliation changes only whether a row is
an open research question:

- established discovery, pairing, command-framing, and client credential-lifecycle mechanics are
  **HARVESTED**; later Greenfield runs are integration/release conformance checks rather than another
  literature survey;
- Greenfield-specific server-identity pinning and identity-change handling are **ADOPTED** and still
  require our own physical evidence;
- common compatibility shortcuts that remove peer verification or store pairing material
  unprotected are **REJECTED** as Greenfield baselines;
- device-side facts that source cannot establish remain **HARDWARE-REQUIRED**, including
  contemporary Android TV Secret enforcement, certificate behavior across update/reset, Samsung
  first-use behavior, and Android independent-trial/retry/session controls.

Open-source evidence can close a research unknown; it cannot mark a Greenfield hardware row Passed
or Tested.

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
| PROXY-1 | Laptop running a TLS interception proxy (e.g. mitmproxy) on the same LAN | Substituted-certificate tests (ATV-17c, SAM-17) |
| CLIENT-INST | An **instrumented Greenfield4 test client** under our control that can log the TLS peer-certificate fingerprint, the local gamma-check outcome, the exact Secret bytes transmitted, and the response class — and that can be told to transmit a deliberately corrupted Secret | **ATV-17a, ATV-17b, ATV-17d (which repeats both on ATV-B), and the local-prefix-loop tests ATV-21, ATV-22 and ATV-29** — used wherever the local gamma-check outcome or the exact transmitted Secret must be observed. This is what makes the decisive test possible without any interception: the client already holds its own key and talks straight to the TV, so injecting a bad Secret needs no MITM |
| LAB-NET | An isolated lab network we own, carrying only the test devices | All interception tests; keeps ATV-17c and SAM-17 off any network used by other people |

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

> **Why ATV-17 was split into four tests (corrected 2026-09-14 after independent review).** The
> original ATV-17 pointed a terminating MITM proxy at pairing and treated "pairing failed" as
> evidence that the TV verifies the Secret. **That inference is invalid.** The AOSP reference
> (`PairingSession.doPairingPhase()`, `SetSecret()`) shows the client runs a **local `checkGamma`
> before transmitting anything**, so a generic terminated TLS session can fail entirely on the
> phone and never reach the TV. Reading that as server-side rejection would be a false positive.
>
> The redesigned test separates the concerns:
>
> - **ATV-17a** proves the harness works and captures a clean baseline.
> - **ATV-17b** is decisive and needs **no interception at all**: our own client sends a
>   deliberately corrupted Secret directly to the TV. Because the client already holds its key and
>   is talking to the real TV, the local check is bypassed by design and the TV's response
>   unambiguously reveals whether it verifies.
> - **ATV-17c** keeps the substituted-certificate scenario but only ever **classifies the failure
>   point**; it never infers server verification from a generic failure.
> - **ATV-17d** repeats the baseline on the second device generation before 17b is treated as an
>   ecosystem result.
>
> **Safety.** ATV-17a/17b/17d involve no interception whatsoever. Only ATV-17c and SAM-17 use a
> proxy, and only on LAB-NET — an isolated network we own.
>
> **Per-device scope.** ATV-17b runs on ATV-A and ATV-17d repeats both the baseline and the
> corrupted-Secret test on ATV-B. A result on one device is a **per-device / per-firmware**
> result and must never be reported as ecosystem-wide contemporary-firmware conformance.
> Conformance claims require the corrupted-Secret test to pass on **every** device in the
> release matrix.

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
| ATV-17a | **Harness validation — correct Secret.** Instrumented client pairs normally against ATV-A: records TLS peer cert fingerprint, local `checkGamma` outcome, the exact Secret bytes sent, and the TV's response. Sends the **correct** alpha. | SecretAck received. | **PASS only if** SecretAck is received **and** the log shows the correct alpha was sent. This proves the harness and the baseline before any fault is injected. If this fails, 17b/17c are void. | Client log: peer fingerprint, local-check result, Secret bytes, response class; TV-observed pairing result |
| ATV-17b | **Decisive — deliberately mismatched Secret.** Using the same harness, complete the handshake and read gamma from the TV, but **transmit an intentionally corrupted Secret** (e.g. flip one byte of the computed alpha). No proxy, no interception: the client holds its own key and talks directly to the TV. | Unknown — this is the decisive observation. | **Record the TV's response class, do not pre-label it.** <br>• TV returns an error / closes without SecretAck → **server-side secret verification CONFIRMED on this firmware.** <br>• TV returns SecretAck → **server-side verification ABSENT on this firmware — critical finding, stop and escalate.** <br>Do **not** interpret any other outcome as confirmation. | Exact Secret bytes sent; TV response bytes/state; whether SecretAck observed; firmware build |
| ATV-17c | **Substituted certificate during pairing (terminating proxy, owned lab LAN only).** Proxy terminates 6467 with its own cert; complete the code flow. | Unknown. | **Classify the failure point — never infer server verification from a generic failure.** Explicitly distinguish: (1) TLS/mTLS handshake failure; (2) client-side gamma/check-byte rejection **before** Secret transmission; (3) Secret actually sent; (4) TV rejection after receiving a mismatched Secret; (5) SecretAck / pairing success. Only outcome (4) evidences server-side verification. | Proxy cert fingerprint; client log showing whether Secret was sent; the classified rejection point |
| ATV-17d | **Second device generation — baseline *then* corrupted Secret.** Repeat **ATV-17a** on **ATV-B** (different vendor / OS build), then repeat **ATV-17b** on **ATV-B**. | 17a: SecretAck. 17b: rejected (error / no SecretAck) if ATV-B enforces server verification. | **Per-device result, recorded separately from ATV-A.** 17a must pass or 17b on ATV-B is void. 17b on ATV-B: error/no-SecretAck → server verification confirmed **on ATV-B**; SecretAck → absent **on ATV-B**, escalate. **A clean baseline alone proves only harness portability, NOT server verification.** | Same artefacts as 17a/17b, tagged per device and per firmware build |
| ATV-18 | **Substituted certificate on reconnect (MITM).** Proxy terminates 6466 with its own cert after a successful pairing. | With pinning: rejected. | **Must fail closed.** Without pinning, record that it is accepted — which is the verified weakness in the reference posture. | Proxy cert fingerprint; client behaviour |
| ATV-19 | **Wrong code.** Enter an incorrect 6-symbol code during pairing. | Pairing rejected; no credential issued. | Pairing fails; client surfaces an error and does not fall back to an insecure path. | Error surfaced |
| ATV-20 | **Independent second phone.** Pair PHONE-2 to the same TV without touching PHONE-1. | Both work independently; PHONE-1's credentials unaffected. | Both control the TV; no credential sharing between phones. | Two client-cert fingerprints; both functional |

---

## Pairing-attempt controls (bounding practical first-use attackability)

**Why this section exists.** The Android TV first-use assessment in
[`2026-09-14-security-trust-model.md`](2026-09-14-security-trust-model.md) §2.7.6/§2.7.7 concludes
that, against a terminating active MITM, the out-of-band binding is **8 bits per independent pairing
trial** — an attacker clears the alpha-prefix gate with probability ~1/256 per independent trial and
then recovers the 16-bit nonce from at most **65,536 candidate hashes**, which is computationally
small as an offline search. (No timing is claimed; this was not benchmarked.)

How exploitable that is depends on **how many independent trials an attacker can obtain** — and
"independent" is load-bearing: re-entering the same gamma against unchanged key material is
deterministic, not a fresh chance (§2.7.7). None of this is visible in any source read in this
research, so it must be measured on hardware.

**Two failure paths, measured separately.** In 255/256 of trials the phone rejects the prefix
**locally and transmits no Secret at all**; the TV may see nothing. Only in the remaining 1/256 does
the attacker reach the TV with a Secret it can reject. ATV-21 … ATV-23 and ATV-29 … ATV-30 measure
the **local** loop; ATV-24 … ATV-28 measure **TV-visible** throttling. **TV-visible throttling must
not be credited as mitigation for the local loop unless ATV-23 shows the TV actually observes local
failures.**

**No expected protections are invented here.** Every row records what the device *actually* does;
several rows are deliberately pass/fail-free because the safe answer is not known in advance. All
are **NOT EXECUTED**.

| ID | Observation | Pass/fail | Evidence |
| :-- | :-- | :-- | :-- |
| ATV-21 | **Local prefix-mismatch retry loop — the dominant 255/256 path.** Using CLIENT-INST, enter a wrong code so the **local alpha-prefix check fails and no Secret is transmitted**. Record the full cycle: (a) did the failure stay local — log shows prefix mismatch and **zero bytes of Secret sent**; (b) do the same displayed gamma **and the same TV pairing session** remain usable afterwards; (c) can the same gamma be re-entered; (d) is the phone-side TLS connection destroyed or reusable; (e) exactly what user action (on TV, on phone) precedes another trial. | **No pass/fail — record the complete per-trial cycle and its cost.** This row bounds the attack loop that §2.7.6/§2.7.7 depend on. **Note:** a plain wrong-code entry is a *local* event and may never reach the TV, so it is not by itself a measure of attempts the TV permits. | Client log showing prefix-match result and Secret-bytes-sent = 0; gamma before/after; TV session id before/after; connection state; per-trial user-action log |
| ATV-22 | **Is the next trial cryptographically independent?** Repeat the ATV-21 local failure and determine whether the following trial is a *new independent* trial per §2.7.7 — i.e. whether the nonce `N`, the TV session, or any other alpha-digest input actually changed. Then deliberately vary the **attacker-controlled phone-facing key `M1`** with the TV session and displayed gamma held fixed, and confirm the local prefix result changes accordingly (a trial that now matches where it previously did not proves a fresh independent draw was obtained **without** forcing a new TV session). | **No pass/fail — record whether independence is achievable at all, and what producing it costs.** Do **not** assume each retry yields a fresh nonce: AOSP generates the nonce once per output-device pairing phase, and nothing read establishes that a rejected attempt regenerates it. | Nonce before/after per trial (instrumented client); `M1` fingerprint per trial; prefix-match outcome per trial; whether TV session id or gamma rotated |
| ATV-23 | **Is the local failure visible to the TV at all?** With TV-side observation in place, cause a local prefix mismatch (wrong code, **no Secret sent**) and determine whether the TV registers **any** event: a failed-pairing record, a counter increment, a log entry, a UI change, or nothing whatsoever. Measure any imposed retry delay and whether it grows. | **No pass/fail — record exactly what the TV can and cannot see, plus any delay.** **If the TV observes nothing, the TV-side throttling measured by ATV-24 … ATV-26 cannot be credited as mitigation for the 255/256 local loop**, and the write-up must say so explicitly rather than implying the local loop is rate-limited. | TV-observed pairing result; any counter/log/UI change; captured frames during the attempt; timestamps and measured delay |
| ATV-24 | **Rate limiting on repeated pairing sessions.** Start many pairing sessions in succession and measure whether the TV throttles or refuses them. | **No pass/fail — record the observed limit.** | Session start times; accept/refuse outcomes |
| ATV-25 | **Lockout / backoff after N failures.** Continue failing until the TV refuses, then observe the lockout duration and whether it persists across TV standby/wake and reboot. | **No pass/fail — record threshold, duration, and persistence.** | Failure count to lockout; duration; persistence across power states |
| ATV-26 | **Can a pairing session be initiated remotely without fresh user action?** Determine whether an unauthenticated LAN peer can cause the TV to display a new code at will. | **PASS (for security) only if** a new code cannot be produced without on-device user action. Otherwise record the exposure — this determines whether an attacker can farm attempts unattended. | Whether a code appears without user action; captured session initiation |
| ATV-27 | **Is user approval required to begin pairing each time?** Observe what the user must do on the TV to start a pairing session. | **No pass/fail — record the required action.** | Description/screenshot of required on-TV action |
| ATV-28 | **Code lifetime.** Measure how long a displayed gamma remains valid, and whether it expires on its own. | **No pass/fail — record the lifetime.** | Display time to expiry; TV UI state |
| ATV-29 | **Independent trials obtainable per user-mediated session.** With one live TV pairing session and one displayed gamma, present a sequence of distinct phone-facing TLS keys (`M1`) and count how many trials complete before something ends the run — session rotation, code expiry, TV refusal, or nothing at all. | **No pass/fail — record the count and identify what limited it.** This yields *independent trials per user-mediated pairing session*, the exact quantity §2.7.7 leaves open. | Distinct `M1` fingerprints attempted; session id and code state per trial; what terminated the run |
| ATV-30 | **Total cost of many independent trials.** End to end, measure wall-clock time and the exact number of on-TV and on-phone user actions needed to obtain 10 independent trials; extrapolate to the ~177 trials that give roughly even cumulative odds in §2.7.7. | **No pass/fail — record the cost.** This is what converts §2.7.7's conditional probability into a practical statement; report the extrapolation as an extrapolation, not a measurement. | Timestamps; per-trial user-action log; stated extrapolation and its assumptions |

**Combining the result.** Practical first-use attackability is bounded by the product of: attempts
per code (ATV-21/22), TV visibility of local failure and delay (ATV-23), rate limiting (ATV-24),
lockout threshold and persistence (ATV-25), and whether attempts can be generated unattended
(ATV-26/27). Record each on **ATV-A and
ATV-B separately** — throttling behaviour is per-firmware and must not be generalised from one
device.

---

## Priority order

The decisive tests — the ones whose outcome would change a conclusion in the trust model — are:

1. **ATV-17b** — determines whether contemporary target firmware enforces the server-side secret
   verification that the AOSP reference implementation documents. This is the single largest open
   question on either track. **ATV-17a must pass first**, or 17b's result is meaningless.
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
