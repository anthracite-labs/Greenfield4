# Research — Security Trust Model: Samsung Tizen and Android TV / Google TV Remote v2

**Date:** 2026-09-14
**Phase:** discovery
**Issue:** #11
**Status:** protocol-level trust model documented; **no ecosystem is security-cleared**; all
hardware-dependent behaviour remains untested
**Base:** `main` at `92730e6` (PR #10 merged). ADR-0005 `accepted` for product direction only.

## Purpose and non-goals

This document answers, per ecosystem: what authenticates the TV to the client; what authenticates
the client to the TV; what the user physically verifies during pairing; what cryptographic
identity is bound; what identity can be persisted for reconnect; what happens when that identity
changes; whether first-use MITM resistance is established; and whether pinning is viable.

It does **not** select an architecture, write or propose application code, change the accepted
ecosystem direction, or weaken any requirement in `docs/PRODUCT.md` or `docs/DOMAIN.md`. Where a
protocol cannot satisfy an invariant, that conflict is recorded rather than smoothed over.

## Evidence labelling used throughout

| Label | Meaning |
| :-- | :-- |
| **[VERIFIED]** | Read in this session from a primary source at a pinned ref, with file path and line context. |
| **[SECONDARY]** | A source that is not the protocol or implementation itself — blog, forum, issue comment, third-party summary. Never promoted to primary. |
| **[INFERRED]** | Reasoning from verified facts. Not evidence. |
| **[PROPOSED MITIGATION]** | A design Greenfield4 could adopt. Not demonstrated here. |
| **[HARDWARE-REQUIRED]** | Cannot be established without a physical TV. Not claimed. |
| **[UNRESOLVED]** | Investigated and not answerable from available evidence. Recorded as a gap, not filled. |

## Sources read in this session (pinned)

| Source | Ref | Role |
| :-- | :-- | :-- |
| `xchwarze/samsung-tv-ws-api` (`samsungtvws`) | tag `v3.0.6`, published 2026-09-11T22:40:25Z | Primary, Samsung Tizen |
| `kud/androidtv-remote` | commit `5a05d73eb477` (2026-07-10, release `0.1.2`) | Primary, Android TV Remote v2 |
| `tronikos/androidtvremote2` | commit `b09f21432ba3` (2026-09-07) | Primary cross-check, Android TV Remote v2 |

All file contents were fetched with `gh api repos/<owner>/<repo>/contents/<path>?ref=<ref>` and
decoded, in this session. Line references are to those fetched files.

---

# Part 1 — Samsung Tizen

## 1.1 Transport

- Endpoint shape: `ws://<host>:8001/api/v2/channels/samsung.remote.control` and
  `wss://<host>:8002/api/v2/channels/samsung.remote.control`, with the remote-control channel
  `samsung.remote.control` [VERIFIED `samsungtvws/remote.py`, `REMOTE_ENDPOINT`].
- A REST variant exists: `http(s)://<host>:<port>/api/v2/<route>` [VERIFIED
  `samsungtvws/connection.py`, `_format_rest_url`].
- SSL is selected **purely by port number**: `_is_ssl_connection()` returns `self.port == 8002`
  [VERIFIED `connection.py`]. There is no capability negotiation and no upgrade decision.

## 1.2 Certificate validation in the reference client

Both code paths disable server authentication:

- Sync: `sslopt = {"cert_reqs": ssl.CERT_NONE} if self._is_ssl_connection() else {}`, passed to
  `websocket.create_connection` [VERIFIED `connection.py`].
- Async: `get_ssl_context()` builds `ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)` and sets
  `check_hostname = False` **and** `verify_mode = ssl.CERT_NONE` [VERIFIED `helper.py`].

So on the TLS port the reference client validates neither the certificate chain nor the hostname.
The previous session's research recorded only the sync `connection.py` line; the async path is
additionally affected, so the finding is broader than previously documented.

The library's `README.md` contains no occurrence of `token`, `pair`, `cert`, or `ssl`
[VERIFIED — grep across the 246-line README at `v3.0.6`]. There is no documented security guidance
or pinning facility in the reference client.

## 1.3 What authenticates the TV to the client

**Nothing, in the reference posture.** The TLS certificate is not verified, so the client accepts
any certificate, including one presented by a LAN attacker. The TV is identified only by the IP
address the client chose to connect to.

## 1.4 What authenticates the client to the TV

Possession of the bearer token, plus the fact that the user previously pressed "Allow" on the TV.

## 1.5 The token flow, as implemented

1. Client opens the channel with `?name=<serialised client name>`
   [VERIFIED `connection.py`, `_format_websocket_url`].
2. The TV returns a `ms.channel.connect` event; the client extracts
   `response["data"]["token"]` and stores it [VERIFIED `_check_for_token`].
3. Storage is either a caller-supplied in-memory value or a plaintext file
   [VERIFIED `_set_token`]. There is no encryption or platform keystore in the library.
4. On later connections the token is appended as a query parameter: `?token=<token>`
   [VERIFIED `_format_websocket_url`].

**A newly verified detail with security consequence:** the token is attached **only when
`ssl and token is not None`** [VERIFIED `connection.py`]. On the plaintext port 8001 the reference
client never sends a token at all. The library therefore treats 8001 as an unauthenticated
endpoint. Greenfield4 must not assume that "a token exists" implies the transport is authenticated.

Because the token travels as a URL query parameter, it is inside the TLS payload on 8002 — but
URL-shaped secrets are a well-known leakage vector into proxy and server logs, which matters for
the PRODUCT.md rule that pairing secrets are treated like passwords.

## 1.6 What the user physically verifies during pairing

The user approves a popup rendered **on the TV** ("Allow device?"). This is a proximity signal: it
proves the user is in front of the display. It is **not** a cryptographic binding — the approval
carries no fingerprint, no code, and no confirmation of what the client observed.

## 1.7 What cryptographic identity is bound during pairing

**None that primary evidence shows.** Specifically:

- The token is received as an opaque JSON string [VERIFIED `_check_for_token`]. No TLS certificate
  material, TV public key, device serial, or nonce appears in the exchange.
- There is no PIN, no shared-secret derivation, and no digest construction anywhere in the
  Samsung remote-control path [VERIFIED — absence across `connection.py`, `async_connection.py`,
  `remote.py`, `helper.py` at `v3.0.6`].
- The token is therefore a **pure bearer credential**: whoever holds it can act as the paired
  client, and it is unbound to any transport identity.

Conclusion: the approval/token flow binds **nothing beyond possession of the issued bearer token.**

## 1.8 Token lifetime, rotation, and re-pair behaviour

[UNRESOLVED] Nothing in the primary code establishes a token lifetime, rotation interval, or
revocation signal. The library stores whatever the TV sends and never invalidates it. Community
reports that Samsung TVs periodically re-prompt and reissue tokens are [SECONDARY] and are not
treated as evidence here. Establishing rotation behaviour requires a TV on the bench.

## 1.9 Can a client capture a stable certificate fingerprint?

Technically yes: the peer certificate is available to any TLS client, so a Greenfield4 client could
capture a SHA-256 fingerprint of the DER certificate or of the SubjectPublicKeyInfo during the
user-approved pairing and persist it. Nothing in the protocol prevents this
[PROPOSED MITIGATION].

But two things are **not** established:

- Whether the presented certificate is **stable** across TV reboot, standby/wake, power cycle,
  firmware update, network change, and app reinstall — **[HARDWARE-REQUIRED]**.
- Whether it changes on factory reset, which is the behaviour a re-pair trigger would depend on —
  **[HARDWARE-REQUIRED]**.

## 1.10 First-use MITM resistance

**NOT ESTABLISHED.** From verified facts: the transport is unauthenticated (§1.2) and the token is
unbound (§1.7). An attacker on the LAN who can intercept the first connection can present their own
certificate, relay the popup approval, and capture the issued token. The user's on-TV approval is
the only thing that happened, and it binds nothing the attacker cannot replay.

This is recorded as unresolved-with-a-known-gap rather than as a risk estimate: the attack is
consistent with the verified code, but no attack was executed in this session.

## 1.11 Would TOFU/pinning fix it?

TOFU pinning would make **reconnects** fail closed against a substituted certificate, which is
exactly the DOMAIN.md "identity changes → re-pair" behaviour. It would **not** make the first
connection secure: at first use there is no prior fingerprint to compare against, so the attacker's
certificate simply becomes the pinned one.

Do not read "pinning protects reconnects" as "first-use MITM is solved". It is not solved.
Closing first use for Samsung would require an out-of-band verification step — for example,
surfacing the captured fingerprint for the user to confirm against a value the TV displays — which
is a product and UX trade-off, not a protocol feature, and is not decided here.

**Risk boundary:** under TOFU, the security of the persisted identity degrades to the security of
the network at the moment of first pairing. That is a real residual risk and must be visible to
whoever approves shipping.

## 1.12 Samsung Tizen assessment

| Concern | State |
| :-- | :-- |
| TLS transport validation | **UNSAFE / FAILS REQUIREMENT** in the reference posture (`CERT_NONE`, both paths) [VERIFIED] |
| Pairing authentication | **PARTIAL** — on-TV approval only; no cryptographic binding [VERIFIED] |
| Persistent device identity | **NOT VERIFIED** — no identity is captured or persisted by the reference client [VERIFIED] |
| First-use trust | **NOT VERIFIED / UNRESOLVED** — MITM resistance not established [from VERIFIED facts] |
| Reconnect trust | **UNSAFE / FAILS REQUIREMENT** as-is; **PARTIAL** if TOFU pinning is adopted, subject to hardware proof of stability |
| Certificate/public-key pinning viability | **PARTIAL** — capturing a fingerprint is feasible [PROPOSED MITIGATION]; stability **[HARDWARE-REQUIRED]** |

---

# Part 2 — Android TV / Google TV Remote v2

## 2.1 A note on corroboration

The two implementations below are independent codebases, but both derive from the same
reverse-engineered lineage, so agreement is **corroboration of the client-side construction**, not
independent proof of what the TV does. That distinction is load-bearing and is preserved below.

## 2.2 Certificates and ports

- Ports: pairing on **6467**, remote control on **6466**
  [VERIFIED `kud/androidtv-remote` `README.md` options table].
- The client generates a **self-signed RSA-2048 certificate** locally
  [VERIFIED `src/certificate/certificate-generator.ts`]. The source comment is explicit about its
  role: *"The cert is identity-bearing: the TV ties pairing to it, so reconnects must reuse the
  exact same cert."* [VERIFIED]
- Mutual TLS: the client presents that certificate on both ports
  [VERIFIED `pairing-manager.ts`, `remote-manager.ts`, `tls.ConnectionOptions` `key`/`cert`].

## 2.3 What `rejectUnauthorized: false` means

Both reference implementations disable server authentication, in both phases:

- `kud/androidtv-remote`: `rejectUnauthorized: false` in `pairing-manager.ts`
  **and** `remote-manager.ts` [VERIFIED both files].
- `tronikos/androidtvremote2`: `_load_cert_chain` sets `check_hostname = False` and
  `verify_mode = ssl.VerifyMode.CERT_NONE` [VERIFIED `androidtv_remote.py`].

Meaning, precisely: the client does not check the TV's certificate against any trust anchor, does
not check the hostname, and will complete the TLS handshake with **any** peer certificate —
including an attacker's. What it *does* still provide: encryption of the channel against a passive
eavesdropper, and client authentication to the TV via the client certificate.

It does **not** mean "no security at all". It means **the TV is not authenticated to the client at
the transport layer.**

## 2.4 The pairing code

- The protocol negotiates the code format during pairing:
  `ENCODING_TYPE_HEXADECIMAL` with `symbolLength: 6`
  [VERIFIED `src/pairing/pairing-message-manager.ts`, `createPairingOption` and
  `createPairingConfiguration`].
- `androidtvremote2` enforces the same shape client-side: the code must be exactly 6 characters and
  must parse as hex [VERIFIED `pairing.py`, `async_finish_pairing`].
- The code is displayed **on the TV** and typed into the phone
  [VERIFIED `README.md`: "The TV shows a pairing PIN — read it from stdin and submit"].

**Correction to the previous session's description.** ADR-0005 and the ecosystem research describe
this as a "PIN" and the library's own example passes `"123456"`. That reads as a decimal PIN. It is
not: the protocol negotiates **hexadecimal** symbols, and both implementations treat the code as a
hex string. The example `"123456"` is simply six characters that happen to be valid hex. This
matters because the byte-level digest below is defined over hex-parsed bytes, and any Greenfield4
input handling must match that, not a decimal interpretation.

## 2.5 Exactly what goes into the pairing digest

Both implementations construct the same SHA-256 input, in this order:

1. client certificate RSA modulus (big-endian bytes)
2. client certificate exponent, rendered as hex with a leading `"0"`
3. **server (peer) certificate RSA modulus**
4. **server (peer) certificate exponent, rendered as hex with a leading `"0"`**
5. the pairing code with its **first byte removed**

[VERIFIED `kud/androidtv-remote` `src/pairing/pairing-manager.ts` (`sha256.update` sequence) and,
independently, `tronikos/androidtvremote2` `src/androidtvremote2/pairing.py`
(`hashlib.sha256` / `h.update` sequence). The two agree.]

The server certificate is obtained via `client.getPeerCertificate()` — that is, **the certificate
as observed by the client on this connection** [VERIFIED], not a certificate fetched from a trust
store. This is the fact that makes the digest a channel-binding construction.

## 2.6 The client-side check byte, and what it is not

Both implementations compare **one byte** before sending:

- `kud`: `if (hashArray[0] !== codeBytes[0]) { client.destroy(new Error("Bad Code")); return false }`
- `androidtvremote2`: `if hash_result[0] != int(pairing_code[0:2], 16): raise InvalidAuth(...)`

[VERIFIED both.]

Read together with §2.5, the code layout is: **byte 0 is an 8-bit check byte derived from the
digest, and bytes 1–2 are the secret input.** The client recomputes the digest and confirms its
first byte matches the displayed check byte before transmitting.

Two things follow, and both matter:

- This check is a **cheap client-side sanity check, not the security mechanism.** Eight bits is
  trivially guessable at 1/256. Anyone treating it as the binding is mistaken.
- The security mechanism is the **full 32-byte digest** sent to the TV as
  `pairingSecret.secret` [VERIFIED both], which the TV is expected to verify against its own
  computation.

**An implementation detail worth recording.** `kud`'s `hexStringToBytes` is applied to the raw
`code` string while the digest uses `code.slice(2)`. Confirmed empirically in this session with
Node 22: `hexStringToBytes("0x1A2B3C")` yields `[NaN, 26, 43, 60]`, so if the code is passed with a
`0x` prefix the one-byte check can never succeed and pairing aborts with "Bad Code"; if passed
without the prefix, the check compares against a byte that is not the byte the digest seeds from.
`androidtvremote2` handles the same input cleanly by using `pairing_code[0:2]` and
`pairing_code[2:]` on a validated 6-character hex string. This is a **reference-implementation
defect**, not a protocol flaw, but it is a concrete warning: the byte-level handling around this
digest is easy to get wrong, and a Greenfield4 implementation must be tested against a real TV
rather than ported on faith.

## 2.7 Does the TV-displayed code cryptographically bind the server identity?

**The construction does. Whether the TV enforces it is not something a client can prove.**

- **[VERIFIED]** The digest the client sends includes the server certificate's modulus and
  exponent as observed on that connection (§2.5).
- **[INFERRED — high confidence]** The TV recomputes the same digest from the client certificate it
  received, its own certificate, and the code it displayed, and accepts only on match. If it does,
  then a MITM that terminates TLS and presents its own certificate produces a client digest that
  cannot match the TV's, and pairing fails.
- **[UNRESOLVED]** The TV's verification logic lives in closed firmware and was not read. The two
  client implementations agree on what the *client* sends; neither can show what the *TV* checks.
  The existence of the code is strong circumstantial evidence — a code that nothing verifies would
  be pointless — but circumstantial evidence is not a proof, and this document does not upgrade it.

This is the single most important open question on the Android TV track, and it is testable: run a
MITM proxy during pairing and observe whether pairing succeeds. See test **ATV-MITM-01** in the
hardware matrix.

## 2.8 Does successful pairing compensate for disabled PKI validation?

**Partly, and only for the pairing phase.**

During pairing, if §2.7 holds, the digest provides an authentication property that does not depend
on PKI: the TV's identity (its public key) is bound into a value that the user's physical act of
reading the code helps protect. That is a genuine compensation for `rejectUnauthorized: false`
**at pairing time**.

It does **not** extend to the remote session. On reconnects over port 6466 there is no code, no
digest, and no user action — just `rejectUnauthorized: false` / `CERT_NONE` (§2.3). So the
compensation is phase-specific, and the reconnect phase inherits the full weakness.

## 2.9 What identity can be persisted after pairing

| Candidate | Verdict |
| :-- | :-- |
| **Client certificate + private key** | **[VERIFIED] the intended persistent identity.** `certificate-generator.ts` states the TV ties pairing to it; the README documents persisting it to skip pairing; `getCertificate()` exists for that purpose. This authenticates **Greenfield4 to the TV**. |
| **Server certificate fingerprint (SHA-256 of DER)** | **[PROPOSED MITIGATION]** Capturable via `getPeerCertificate` / `getpeercert(True)`; both implementations already obtain the DER bytes for the digest, so capture costs nothing extra. Pins the exact certificate. |
| **SPKI / public-key fingerprint** | **[PROPOSED MITIGATION]** Pins the key rather than the certificate, so it survives certificate re-issue under the same key. Arguably the better pin, but stability under firmware update is **[HARDWARE-REQUIRED]**. |
| **Name + MAC from the TV certificate subject** | **[VERIFIED present, not authenticated].** `androidtvremote2` parses these from the certificate subject, documenting forms such as `CN=atvremote/darcy/darcy/SHIELD Android TV/XX:XX:XX:XX:XX:XX` and dnQualifier `fugu/fugu/Nexus Player/CN=atvremote/XX:XX:XX:XX:XX:XX` [VERIFIED `_parse_name_and_mac` docstring]. Useful as a **human-readable device label** and as a secondary identity signal. It is **not** an authentication mechanism on its own: the value is asserted by the certificate itself, so whoever presents a forged certificate also chooses the MAC. Using it as an identity *check* would be circular. |
| **Token** | Not applicable — there is no token in this protocol. |

**Neither reference implementation persists any server identity.** There is no pinning code in
either `remote-manager.ts` or `androidtv_remote.py` [VERIFIED]. The reconnect path re-connects with
`CERT_NONE` every time. So "persist the server identity" is something Greenfield4 would have to add;
it is not available for free from the ecosystem's reference clients.

## 2.10 What happens when identity changes

- **Client identity revoked or unknown to the TV:** the TV resets the connection; `kud` maps
  `ECONNRESET` to an `unpaired` event [VERIFIED `remote-manager.ts`], and `androidtvremote2` maps
  `ssl.SSLError` to `InvalidAuth("Need to pair again")` [VERIFIED]. So the TV→client direction has a
  real fail-closed signal.
- **Server identity changes:** **nothing happens.** With server verification disabled, a different
  TV certificate is accepted silently. There is no detection, no `unpaired`, no error
  [VERIFIED — no code path compares a stored server identity in either implementation]. This is the
  direct conflict with DOMAIN.md "Paired | Device security identity changes unexpectedly |
  Re-pair required | Silent trust replacement is forbidden."
- **Reconnect impersonation, precisely stated:** an attacker on the LAN can present any certificate
  and impersonate the TV *to the client*. The client's commands and keystrokes then go to the
  attacker instead of the TV. The attacker **cannot** relay them to the real TV, because client
  authentication requires the client's private key, which the attacker does not hold. So the impact
  is interception, deception, and loss of confidentiality of what the user types — not remote
  control of the TV by the attacker. Stating this precisely matters: it is a real failure of
  TV→client authentication and a real privacy problem, but it is **not** the same as full
  compromise, and conflating the two would be its own inaccuracy.

## 2.11 Can reconnect be made fail-closed without a global trust-all mode?

**Yes — and this is the key positive finding.**

Because the TV's certificate is self-signed and the client already obtains it during pairing, the
client can treat **the paired certificate as its own trust anchor** for that one endpoint. The
standard mechanism is to build a per-device TLS context that loads the pinned certificate as the
CA (`load_verify_locations` with the pinned cert) with `check_hostname` left off, instead of using
`CERT_NONE`. That yields:

- strict verification of the TV on every reconnect;
- no global "ignore security" flag anywhere in the product;
- a mismatch producing a hard failure, which is exactly where a re-pair flow belongs.

**[PROPOSED MITIGATION]** — not implemented or demonstrated here. Two other projects are reported
to apply per-TV certificate pinning to this same protocol [SECONDARY: TVgrip PR #2, hafa-remote
PR #15]. That is secondary evidence that the pattern is viable on this protocol; it is not evidence
that it works on Greenfield4's target devices, and it is not generalised from the Samsung analysis.

## 2.12 First-use MITM resistance

**PARTIAL — stronger than Samsung by design, not yet proven end to end.**

- The digest binds the observed server public key to a code the user reads off the physical TV
  [VERIFIED]. That is a real cryptographic binding, and it is the thing Samsung entirely lacks.
- Enforcement depends on the TV verifying the digest [UNRESOLVED, §2.7].
- The user must genuinely read the code from the TV and enter it into the phone; the protocol
  cannot detect a user who enters a code an attacker supplied.

## 2.13 Android TV / Google TV assessment

| Concern | State |
| :-- | :-- |
| TLS transport validation | **UNSAFE / FAILS REQUIREMENT** in the reference posture (`rejectUnauthorized:false` / `CERT_NONE` on both ports, both implementations) [VERIFIED] |
| Pairing authentication | **PARTIAL** — digest binds client cert, server cert, and code [VERIFIED]; TV-side enforcement **[UNRESOLVED]** |
| Persistent device identity | **PARTIAL** — client identity is well-defined and persisted [VERIFIED]; **server identity is persisted by neither reference client** [VERIFIED], so Greenfield4 must add it |
| First-use trust | **PARTIAL** — cryptographic binding designed and verified client-side; end-to-end resistance **[HARDWARE-REQUIRED]** |
| Reconnect trust | **UNSAFE / FAILS REQUIREMENT** as-is; **viable fail-closed design exists** [PROPOSED MITIGATION], pending hardware proof |
| Certificate/public-key pinning viability | **PARTIAL** — mechanism is straightforward and does not require a trust-all mode [PROPOSED MITIGATION]; stability **[HARDWARE-REQUIRED]** |

---

# Part 3 — Security decision output

## 3.1 The two trust models are not the same

| | Samsung Tizen | Android TV Remote v2 |
| :-- | :-- | :-- |
| Authenticates TV → client | Nothing (cert unverified) | Nothing at TLS layer; digest binds server key **during pairing only** |
| Authenticates client → TV | Bearer token | Client certificate (mutual TLS) |
| User physically verifies | On-TV "Allow" popup — binds nothing | 6 hex symbols shown on TV — bound into the digest |
| Cryptographic binding at pairing | **None found** | Client cert + server cert + code → SHA-256 |
| Persistable identity | Token (bearer) + capturable cert fingerprint | Client cert (verified) + capturable server cert/SPKI |
| On identity change | Undetected | Client→TV: TV resets, `unpaired`. TV→client: undetected without pinning |
| First-use MITM | **Not established** | **Partial — binding designed, enforcement unproven** |
| Reconnect protection | None as-is | None as-is; fail-closed pinning is viable |

The ecosystems must not be treated as one trust model. Android TV has a genuine cryptographic
binding at pairing that Samsung lacks; Samsung is materially weaker on first use. Both are equally
weak on reconnect until Greenfield4 adds server-identity persistence.

## 3.2 The six questions

**1. Can Greenfield4 establish a trustworthy device identity?**
- Samsung: **NO, not from the protocol.** The token is unbound and the certificate is unverified. A
  capturable certificate fingerprint is a candidate identity, but it inherits whatever the network
  was at first pairing. **[HARDWARE-REQUIRED]** for stability.
- Android TV: **PARTIALLY.** The client identity is solid [VERIFIED]. A server identity is
  capturable and the pairing digest gives it a stronger basis than Samsung's. Trustworthiness of
  that server identity depends on the unproven TV-side verification.

**2. Can it safely persist that identity?**
- Samsung: **[PROPOSED MITIGATION]** — pinning is feasible; the reference client offers no storage
  security at all (tokens written to plaintext files [VERIFIED]), so Greenfield4 must supply
  platform-secure storage. Stability **[HARDWARE-REQUIRED]**.
- Android TV: **YES for the client identity** [VERIFIED — that is the documented reconnect
  mechanism]. Server pinning is a **[PROPOSED MITIGATION]** on top.

**3. Can it detect unexpected identity changes?**
- Samsung: **NO as-is.** With pinning: **YES for reconnects**, never for first use.
- Android TV: **NO as-is** (server unverified). With pinning: **YES**. The reverse direction —
  the TV rejecting the client — is already detected [VERIFIED `unpaired` / `InvalidAuth`].

**4. Can it fail closed and require re-pairing?**
- Both: **NO as-is.** Both: **YES once server identity is pinned**, which both ecosystems permit
  because both present a self-signed certificate the client can capture. Neither requires a global
  trust-all mode to do it. **[PROPOSED MITIGATION]**, pending hardware validation.

**5. Is first-use MITM resistance demonstrated?**
- Samsung: **NO.** Not established by any available evidence. Recorded as unresolved, not as
  mitigated.
- Android TV: **NO — not demonstrated.** The binding is verified as constructed; enforcement by the
  TV is unproven; no attack or non-attack was executed. **PARTIAL**, deliberately not upgraded.

**6. What remains dependent on hardware evidence?**
Everything in `docs/research/2026-09-14-hardware-validation-matrix.md` — certificate stability
across reboot/firmware/reset, MITM detection, token rotation, and reconnect behaviour under network
change. None of it was executed.

## 3.3 Mapping against existing invariants

| Invariant (source) | Samsung | Android TV |
| :-- | :-- | :-- |
| "If a paired device's security identity changes unexpectedly, fail safely and require re-pairing rather than silently trusting the new identity." (PRODUCT.md) | **FAILS** as-is; met only with pinning | **FAILS** as-is; met only with pinning |
| "If a secure connection cannot be established, refuse the unsafe connection. There is no global 'ignore security' mode." (PRODUCT.md) | **FAILS** as-is | **FAILS** as-is |
| "Pairing credentials, tokens, and keys are treated like passwords." (PRODUCT.md) | **FAILS** in reference client (plaintext token file) **[must not be replicated]** | Client private key must be stored as a secret; reference clients persist PEM files |
| DOMAIN.md "Paired → Device security identity changes unexpectedly → Re-pair required; Silent trust replacement is forbidden." | **FAILS** as-is | **FAILS** as-is (server direction) |
| DOMAIN.md Paired-device invariant "Device identity plus protected pairing material." | Identity absent | Identity present for client, absent for server |
| DOMAIN.md Device invariant "Stable device identity where the ecosystem exposes one; never just the current IP address." | IP-only as-is | IP-only as-is |
| DOMAIN.md "Security wins over popularity or convenience." | Blocks shipping until resolved | Blocks shipping until resolved |

**No invariant above is weakened by this document.** Where a protocol cannot meet one, the mismatch
is recorded, and the path to meeting it (pinning + hardware validation) is a change to
Greenfield4's client, not a relaxation of the requirement.

## 3.4 Conflict requiring a decision, not a documentation edit

One conflict is structural and cannot be closed by pinning:

> **Samsung first-use MITM resistance is not achievable through TOFU alone.** PRODUCT.md requires
> refusing an unsafe connection with no global ignore-security mode. On first pairing there is no
> prior fingerprint, so the connection is unauthenticated by construction. Pinning protects every
> connection *after* the first; it cannot protect the first.

Resolving this needs a product decision among options that include: accepting a documented residual
first-use risk; adding out-of-band fingerprint confirmation at pairing; or declining Samsung for V1.
That is a product-owner decision on a non-negotiable invariant and is **not** made here. Per the
repository decision process it is recorded as an open conflict, and ADR-0005's accepted
product-direction status is **not** reversed on these grounds.

## 3.5 Reduced coverage

- `developer.samsung.com` and Google/ Android TV partner portals are outside the sandbox egress
  allowlist and were not reached. No vendor documentation is cited as primary anywhere above.
- Only client-side implementations were read. No TV firmware, no vendor specification, and no
  official protocol document was available.
- No MITM, no TLS interception, and no traffic capture was performed. Every attack described is
  reasoned from code, not executed.
- Vendor/legal validation remains open and is untouched by this document.

## Related

- [PRODUCT.md](../PRODUCT.md) — security requirements this maps against
- [DOMAIN.md](../DOMAIN.md) — state transitions and business rules
- [2026-09-14-ecosystem-evidence.md](2026-09-14-ecosystem-evidence.md) — ecosystem selection evidence
- [2026-09-14-hardware-validation-matrix.md](2026-09-14-hardware-validation-matrix.md) — untested hardware tests
- [../decisions/0005-v1-ecosystem-selection.md](../decisions/0005-v1-ecosystem-selection.md) — accepted direction
