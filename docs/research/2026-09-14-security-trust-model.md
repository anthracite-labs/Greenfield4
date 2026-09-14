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
| `xchwarze/samsung-tv-ws-api` (`samsungtvws`) | tag `v3.0.6`, published 2026-09-11T22:40:25Z | Primary, Samsung Tizen — **client side only** |
| `kud/androidtv-remote` | commit `5a05d73eb477` (2026-07-10, release `0.1.2`) | Primary, Android TV Remote v2 — **client side** |
| `tronikos/androidtvremote2` | commit `b09f21432ba3` (2026-09-07) | Primary cross-check, Android TV Remote v2 — **client side** |
| `platform/external/google-tv-pairing-protocol` (AOSP) | commit `7c99785` | **Primary, Android TV pairing protocol — both roles.** `PoloChallengeResponse.java` blob `81095fd`; `PairingSession.java` blob `8baccf4`; `cpp/.../pairingsession.cc` blob `011c913` |

The three GitHub sources were fetched with `gh api repos/<owner>/<repo>/contents/<path>?ref=<ref>`
and decoded in this session. The AOSP sources were read from
`android.googlesource.com/platform/external/google-tv-pairing-protocol/+/7c99785/...`.

**Evidence-class split used throughout.** Three different questions get three different labels,
and they are never collapsed into one:

- **[VERIFIED — protocol]** what the AOSP reference implementation of the pairing protocol does,
  including the server/output-device role. This is protocol semantics, not device behaviour.
- **[VERIFIED — client]** what a specific client library does.
- **[HARDWARE-REQUIRED]** whether a *contemporary* Android TV / Google TV device's shipping Remote
  Service firmware conforms to the reference behaviour. The AOSP source predates current devices and
  does not answer this. Never inferred from it.

No server-side implementation was available for Samsung; that asymmetry between the two ecosystems is
a fact about the evidence, not a conclusion about the products.

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

**A verified detail with security consequence:** the token is attached **only when
`ssl and token is not None`** [VERIFIED — client, `connection.py`]. On plaintext port 8001 the
reference client sends **no persistent token credential at all**.

That is a statement about **the client**, and it is scoped that way deliberately. It does **not**
establish that the TV firmware leaves 8001 unauthenticated: the TV could still require per-session
on-TV approval, or apply some other server-side rule that this client simply never exercises,
because the client never opens that path with a token. **[INFERRED]** the practical consequence is
that a Greenfield4 client which uses 8001 gets no token-based authorisation;
**[HARDWARE-REQUIRED]** whether the TV independently authenticates or gates 8001.

The safe product rule follows regardless of which is true: do not assume that "a token exists"
implies the transport is authenticated, and do not use the plaintext port.

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
- Within the client-observable protocol path, the token therefore behaves as a **bearer
  credential**: it is presented as an opaque value, and no certificate, public key, or device
  identity is bound into it. **[VERIFIED — client]** no cryptographic binding exists in the
  client-side flow.

**Scoping note — what this does and does not prove.** "Whoever holds it can act as the paired
client" is a statement about **server-side authorisation**, and the open-source client cannot prove
it. The TV firmware may associate the token with the client name, a device/session record, or a
network address, and may require re-approval under conditions this client never triggers. So:

- **[VERIFIED — client]** no cryptographic binding of the token to a certificate, public key, or
  device identity appears anywhere in the client-side exchange.
- **[INFERRED]** a copied token is therefore likely sufficient to act as the paired client, because
  the client proves nothing beyond presenting it. This is the consequence the product must defend
  against, and it should be treated as the planning assumption.
- **[HARDWARE-REQUIRED]** whether target Samsung firmware actually enforces any additional
  server-side association that would defeat a copied token.
- **[UNRESOLVED]** the TV's full server-side token/device association rule. No server-side
  implementation or Samsung specification was available; unlike the Android TV track, there is no
  AOSP-equivalent source here.

Conclusion, scoped: the approval/token flow is **not shown to bind anything beyond possession of
the issued token**. That is weaker than "the token is unconditionally a pure bearer credential",
and it is the honest limit of the evidence. It is still enough to drive the first-use finding in
§1.10, which rests on the absence of an *authenticated TV identity* at first connection — a fact
that holds no matter what the server does with the token afterwards.

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

`kud/androidtv-remote` and `tronikos/androidtvremote2` are independent codebases, but **both derive
from the same reverse-engineered lineage**. Their agreement is therefore corroboration of the
*client-side construction only* — it is **not** independent proof of what the TV does. That
distinction is load-bearing and is preserved below.

Accordingly, this document does **not** use the two client libraries as evidence of server
behaviour at all. Server/output-device behaviour is evidenced **only** by the pinned AOSP
`google-tv-pairing-protocol` source at `7c99785`, which implements both roles. Where the deployed
clients and the AOSP reference differ (see §2.6), the difference is recorded and reconciled on
hardware rather than resolved by preferring one source.

The AOSP reference is authoritative for **protocol semantics**; it is not evidence about any
**specific shipping device**. Those two are kept separate everywhere below.

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

**Correction (2026-09-14, after independent review).** An earlier version of this document called
the byte split in `kud` a "genuine defect". **That claim was wrong and is withdrawn.** For a valid
six-hex-symbol code such as `1A2B3C`, the layout is correct Polo, not a bug: the AOSP reference
(`PoloChallengeResponse.getGamma()`, blob `81095fd`) defines gamma as an **alpha prefix followed by
the nonce** — `result = new byte[nonce.length * 2]`, copying `alpha[0..nonce.length]` then
`nonce` — and the client's job is to check that prefix and then transmit the full alpha. Comparing
the first byte of the digest to the first byte of the code while hashing the remainder is exactly
that structure. [VERIFIED — protocol, AOSP `7c99785`]

What remains is a **narrower finding about malformed input, not about protocol handling**:

- `0x1A2B3C` is **eight characters and is not a valid six-symbol pairing code**, so it is out of
  contract. Behaviour on it is a **malformed-input handling / input-validation weakness**, not
  evidence that valid-protocol handling is broken.
- `kud` applies `hexStringToBytes` to the raw `code` string while the digest uses `code.slice(2)`,
  with no up-front validation that the input is exactly six hex symbols. Confirmed empirically in
  this session with Node 22: `hexStringToBytes("0x1A2B3C")` yields `[NaN, 26, 43, 60]`, so an
  out-of-contract `0x`-prefixed value makes the one-byte check unsatisfiable and pairing aborts with
  "Bad Code" — a confusing failure rather than a clear validation error.
- `androidtvremote2` validates first (`len(pairing_code) != 6`, then `bytes.fromhex(...)`) and
  slices as `pairing_code[0:2]` / `pairing_code[2:]`, so it rejects the same input cleanly.
  [VERIFIED — client]

**Narrowed finding:** `kud` lacks explicit six-hex-symbol input validation and handles malformed
`0x…` input poorly. This is an API ergonomics/validation issue in one library. It is **not** a
protocol defect, and it is not presented as one.

**One genuine open question, stated without a diagnosis.** The deployed clients use a **1-byte**
alpha prefix with a 2-byte nonce (3-byte / 6-symbol gamma), while AOSP `getGamma()` builds gamma
from `nonce.length` **bytes** of alpha prefix. The *structure* (alpha prefix ‖ nonce) is confirmed
by AOSP; the prefix **width** in deployed Android TV Remote v2 differs from this 2009-era reference.
This is recorded as an **[UNRESOLVED]** reconciliation item to confirm on hardware, **not** as a
defect: both peers of a working pairing evidently agree, and the security property derives from both
sides computing alpha over the same nonce and the same two certificates, not from the prefix width.
The AOSP arithmetic here is itself not internally consistent about units — `PairingSession.java`
computes `nonceLength = symbolLength / 2` and then `bytesNeeded = nonceLength /
mEncoder.symbolsPerByte()`, which mixes symbol and byte units — so no conclusion is drawn from it.

## 2.7 Does the TV-displayed code cryptographically bind the server identity?

**Corrected 2026-09-14 after independent review.** The previous version of this section said the
TV's enforcement "is not something a client can prove" and that no server-side implementation was
available. **Both statements were wrong and are withdrawn.** A pinned AOSP primary source
implements *both* roles of the pairing protocol, including the output-device (TV) side. What
remains genuinely open is narrower: whether **contemporary shipping firmware** conforms to that
reference behaviour.

Sources: `platform/external/google-tv-pairing-protocol` at commit `7c99785` —
`java/src/com/google/polo/pairing/PoloChallengeResponse.java` (blob `81095fd`),
`java/src/com/google/polo/pairing/PairingSession.java` (blob `8baccf4`), and
`cpp/src/polo/pairing/pairingsession.cc` (blob `011c913`).

### 2.7.1 What the reference protocol defines

- **`alpha`** = `SHA-256(clientModulus ‖ clientExponent ‖ serverModulus ‖ serverExponent ‖ nonce)`,
  with leading null bytes stripped from each RSA component. The `PoloChallengeResponse.getAlpha()`
  docstring states it directly: *"From the Polo design document, `alpha` is the value
  h(K_a | K_b | R_a)"*, listing the client key modulus, client public exponent, server modulus,
  server exponent, and the random nonce, and the code updates the digest in exactly that order.
  [VERIFIED — protocol]
- **`gamma`** = alpha prefix ‖ nonce, and gamma is what the output device displays. `getGamma()`
  allocates `new byte[nonce.length * 2]`, copies `alpha[0 .. nonce.length]` then `nonce`.
  [VERIFIED — protocol]
- **Roles.** The output device generates the nonce, computes gamma, and displays it; the input
  device receives the user's typed gamma. [VERIFIED — protocol, `PairingSession.doPairingPhase()`]

### 2.7.2 The input-device (phone) path

1. Wait for the user's typed gamma.
2. `boolean match = mChallenge.checkGamma(userGamma);` and
   `if (match != true) throw new BadSecretException("Secret failed local check.");`
3. `extractNonce(userGamma)`, then `getAlpha(userNonce)`.
4. Send `SecretMessage(genAlpha)`.
5. Wait for `SecretAck`.

[VERIFIED — protocol] The C++ `SetSecret()` mirrors this: `if (!challenge().CheckGamma(secret))`
logs *"Secret failed local check"* and returns false **before** anything is transmitted.

**This is the fact that invalidates a naive MITM test** (see §2.7.5): a client can reject locally
and never send a Secret at all.

### 2.7.3 The output-device (TV) path — the decisive evidence

`PairingSession.doPairingPhase()`, output branch [VERIFIED — protocol]:

1. Generate a random nonce from the negotiated encoding's symbol length.
2. `gamma = mChallenge.getGamma(nonce)`, then `onPerformOutputDeviceRole(this, gamma)` — display it.
3. Wait for `SecretMessage`.
4. **`localAlpha = mChallenge.getAlpha(nonce); inbandAlpha = secretMessage.getSecret();
   matched = Arrays.equals(localAlpha, inbandAlpha); if (!matched) throw new BadSecretException(
   "Inband secret did not match. Expected [...], got [...]");`**
5. Only then `sendMessage(new SecretAckMessage(inbandAlpha))`.

The C++ cross-check is explicit and separately confirms it [VERIFIED — protocol]:

```cpp
void PairingSession::OnSecretMessage(const message::SecretMessage& message) {
  ...
  if (!VerifySecret(message.secret())) {
    wire()->SendErrorMessage(kErrorInvalidChallengeResponse);
    listener()->OnError(kErrorInvalidChallengeResponse);
    return;
  }
  ...
  wire()->SendSecretAckMessage(ack);
  listener()->OnPairingSuccess();
}

bool PairingSession::VerifySecret(const Alpha& secret) const {
  ...
  const Alpha* gen_alpha = challenge().GetAlpha(*nonce_);
  bool valid = (secret == *gen_alpha);
  if (!valid) { LOG(ERROR) << "Inband secret did not match. Expected [...], got [...]"; }
  return valid;
}
```

So the reference implementation **recomputes alpha locally and equality-compares it with the
received in-band secret**. On mismatch it raises `kErrorInvalidChallengeResponse` (C++) or
`BadSecretException` (Java), and **does not send SecretAck**. Both language implementations agree.

### 2.7.4 What SecretAck means

**[VERIFIED — protocol]** `SecretAck` is sent by the output device **only after** its local alpha
comparison succeeded. It is therefore a *success signal for server-side secret verification*, not a
generic acknowledgement. Observing a SecretAck is evidence that the peer accepted the Secret.

Two caveats, both from the source, that a test must respect:

- The **client's** verification of the alpha echoed *inside* SecretAck is **disabled by default**:
  `private static final boolean VERIFY_SECRET_ACK = false;` with the comment *"One implementation
  does not send the secret back in the SecretAck … it is not essential that we verify it, since
  *any* acknowledgment from the sender is enough to indicate protocol success."* So the ack's
  *presence*, not its payload, is the signal.
- Verifying the ack's payload is supported (`kVerifySecretAck`) but off by default, so a test must
  not depend on the ack containing a checkable secret.

### 2.7.5 Consequence, and the correct evidence classification

Because both peers compute alpha over the **same nonce** plus **both certificates as each peer
sees them**, an attacker who terminates TLS and substitutes a certificate changes the client's
alpha but not the TV's, and the two cannot match. Under the reference protocol, first-use MITM is
**resisted by construction**, and specifically by a **server-side** check — not merely by the
client's local check.

| Claim | Classification |
| :-- | :-- |
| alpha/gamma are constructed as above, over both certificates and the nonce | **[VERIFIED — protocol]** AOSP `7c99785`, blobs `81095fd`, `8baccf4`, `011c913` |
| The output device recomputes alpha and rejects a mismatched in-band Secret | **[VERIFIED — protocol]** `PairingSession.doPairingPhase()`; `OnSecretMessage` → `VerifySecret` |
| SecretAck is sent only after that comparison succeeds | **[VERIFIED — protocol]** both implementations |
| The client can reject locally before sending (so a generic MITM failure is ambiguous) | **[VERIFIED — protocol]** `checkGamma` / `CheckGamma` gate in `SetSecret` |
| **Contemporary Android TV / Google TV firmware on target devices enforces this** | **[HARDWARE-REQUIRED]** — see below |

**What is still not proven.** The AOSP tree is a pairing-protocol reference implementation; it is
not the shipping Android TV Remote Service on a 2026 device, and the Java files carry a 2009
copyright while the C++ carries 2012. Nothing here proves that current firmware — or every
vendor's build of it — retained this verification path. Firmware conformance is therefore
**[HARDWARE-REQUIRED]**, and is the subject of the redesigned **ATV-17** in the hardware matrix.

The previous inference — "the TV probably verifies, because a code that nothing verifies would be
pointless" — is **removed**. It is no longer needed: the verification is now directly documented in
primary source. What replaces it is a narrower and testable question about device conformance,
which is a real remaining gap but a much smaller one.

**Correcting a test-design error.** The earlier text pointed at "ATV-MITM-01" and said "run a MITM
proxy during pairing and observe whether pairing succeeds". That inference is invalid: because the
client's local `checkGamma` runs **before** transmission, a generic terminating proxy can produce a
pairing failure that never reaches the TV, which would be misread as server-side rejection. ATV-17
has been redesigned to separate the rejection points. See
[`2026-09-14-hardware-validation-matrix.md`](2026-09-14-hardware-validation-matrix.md).

## 2.8 Does successful pairing compensate for disabled PKI validation?

**Partly, and only for the pairing phase.**

During pairing the digest provides an authentication property that does not depend on PKI: the TV's
public key is bound into alpha, which is compared by the output device itself
**[VERIFIED — protocol, §2.7.3]**, and the user's physical act of reading gamma off the TV is what
protects the nonce. Under the reference protocol this is a genuine compensation for
`rejectUnauthorized: false` **at pairing time** — and, importantly, it is a *server-side* check, so
it does not depend on the client's local one-byte check.

The remaining uncertainty is not "does the protocol verify?" but "does the device?" — see the
[HARDWARE-REQUIRED] row in §2.7.5.

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

**PARTIAL — but the *nature* of the residual gap changed materially with the AOSP evidence.**

Corrected split:

- **[VERIFIED — protocol]** The reference protocol resists first-use MITM *by construction and on
  the server side*: alpha binds both peers' RSA public keys and the nonce, and the output device
  recomputes and equality-compares it, rejecting a mismatch with
  `kErrorInvalidChallengeResponse` and withholding SecretAck (§2.7.3). This is a real cryptographic
  binding, and it is the thing Samsung entirely lacks.
- **[HARDWARE-REQUIRED]** Whether **contemporary target firmware** still enforces that check. The
  AOSP tree is a protocol reference implementation (Java files © 2009, C++ © 2012), not the
  shipping Remote Service on a 2026 device. Conformance is **not** assumed, and is the subject of
  the redesigned **ATV-17**.
- **[VERIFIED — protocol]** The user must genuinely read gamma off the physical TV and enter it into
  the phone. The protocol cannot detect a user who types a code an attacker supplied, and the
  client-side `checkGamma` is only a local sanity check (§2.6).

**Not over-corrected.** "The protocol verifies" is now proven; "the device verifies" is not. This
document does **not** upgrade Android TV to VERIFIED overall, because three things remain
unproven and none of them are answered by source code: current-device conformance, reconnect-time
server-identity persistence, and real-hardware behaviour (§2.9, §2.10, §2.11).

## 2.13 Android TV / Google TV assessment

| Concern | State |
| :-- | :-- |
| TLS transport validation | **UNSAFE / FAILS REQUIREMENT** in the reference posture (`rejectUnauthorized:false` / `CERT_NONE` on both ports, both implementations) [VERIFIED] |
| Pairing authentication | **PARTIAL** — alpha binds client cert, server cert and nonce, and the output device verifies it **[VERIFIED — protocol]** (§2.7); current-device conformance **[HARDWARE-REQUIRED]** (ATV-17) |
| Persistent device identity | **PARTIAL** — client identity is well-defined and persisted [VERIFIED — client]; **server identity is persisted by neither reference client** [VERIFIED — client], so Greenfield4 must add it |
| First-use trust | **PARTIAL** — protocol-level resistance now **[VERIFIED — protocol]**; whether target firmware enforces it is **[HARDWARE-REQUIRED]**; user must read gamma off the physical TV |
| Reconnect trust | **UNSAFE / FAILS REQUIREMENT** as-is; **viable fail-closed design exists** [PROPOSED MITIGATION], pending hardware proof |
| Certificate/public-key pinning viability | **PARTIAL** — mechanism is straightforward and does not require a trust-all mode [PROPOSED MITIGATION]; stability **[HARDWARE-REQUIRED]** |

---

# Part 3 — Security decision output

## 3.1 The two trust models are not the same

| | Samsung Tizen | Android TV Remote v2 |
| :-- | :-- | :-- |
| Authenticates TV → client | Nothing (cert unverified) | Nothing at TLS layer; during pairing the **output device verifies alpha server-side** **[VERIFIED — protocol]** |
| Authenticates client → TV | Opaque token presented by client; **server-side association rule [UNRESOLVED]** | Client certificate (mutual TLS) |
| User physically verifies | On-TV "Allow" popup — binds nothing | 6 hex symbols (gamma) shown on TV — nonce bound into alpha |
| Cryptographic binding at pairing | **None found in the client-observable path** | Client cert + server cert + nonce → SHA-256 (alpha), **verified by the output device** |
| Persistable identity | Token (opaque; bearer semantics **[INFERRED]**) + capturable cert fingerprint | Client cert (verified) + capturable server cert/SPKI |
| On identity change | Undetected | Client→TV: TV resets, `unpaired`. TV→client: undetected without pinning |
| First-use MITM | **Not established** | **Protocol-verified; target-device conformance unproven** |
| Reconnect protection | None as-is | None as-is; fail-closed pinning is viable |

The ecosystems must not be treated as one trust model, and the difference between them is now
sharper than in the previous version of this document. Android TV has a **protocol-level,
server-side cryptographic binding** at pairing — documented in pinned AOSP source, not inferred —
which Samsung entirely lacks in its client-observable path. Samsung is materially weaker on first
use, and that gap is now a *protocol* gap rather than merely an *evidence* gap.

What has **not** changed: both ecosystems are equally weak on reconnect until Greenfield4 adds
server-identity persistence, and Android TV's protocol-level guarantee is still subject to
**[HARDWARE-REQUIRED]** confirmation that target firmware enforces it. A protocol guarantee on
paper is not a device guarantee in the hand.

## 3.2 The six questions

**1. Can Greenfield4 establish a trustworthy device identity?**
- Samsung: **NO, not from the protocol.** The token is unbound and the certificate is unverified. A
  capturable certificate fingerprint is a candidate identity, but it inherits whatever the network
  was at first pairing. **[HARDWARE-REQUIRED]** for stability.
- Android TV: **PARTIALLY, with a firmer basis than Samsung.** The client identity is solid
  [VERIFIED — client]. A server identity is capturable, and at pairing the protocol binds both
  certificates into alpha and has the **output device verify it** **[VERIFIED — protocol]**. What
  remains open is narrower than before: whether target firmware conforms, which is
  **[HARDWARE-REQUIRED]** (ATV-17), not a gap in the protocol design.

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
- Android TV: **NO — not demonstrated on any device, but now for a much narrower reason.** The
  protocol-level resistance is **[VERIFIED — protocol]**: the output device recomputes alpha and
  rejects a mismatch before sending SecretAck (§2.7.3). What is unproven is whether *contemporary
  target firmware* enforces it. That is **[HARDWARE-REQUIRED]** and is the explicit question ATV-17
  now answers. No interception has been executed in this session. **PARTIAL**, deliberately not
  upgraded to VERIFIED.

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

- `developer.samsung.com` and the Google / Android TV partner portals are outside the sandbox
  egress allowlist and were not reached. No vendor documentation is cited as primary anywhere above.
- **Android TV: the AOSP pairing-protocol reference implementation was reachable and is cited at
  `7c99785`.** It establishes **protocol and reference-implementation semantics**, including the
  output-device role. It is **not** evidence about any shipping device: the Java sources carry a
  2009 copyright and the C++ a 2012 copyright, and contemporary Android TV Remote Service firmware
  was not read and is not available to read. That gap is carried as [HARDWARE-REQUIRED] throughout.
- **Samsung: no server-side implementation or specification was available.** Unlike the Android TV
  track there is no AOSP-equivalent source, so every statement about what the TV does with a token
  is [INFERRED] or [HARDWARE-REQUIRED]. This asymmetry between the ecosystems is a fact about the
  available evidence and is not presented as a conclusion about either product.
- No MITM, no TLS interception, and no traffic capture was performed. Every attack described is
  reasoned from source, not executed.
- Vendor/legal validation remains open and is untouched by this document.

## Related

- [PRODUCT.md](../PRODUCT.md) — security requirements this maps against
- [DOMAIN.md](../DOMAIN.md) — state transitions and business rules
- [2026-09-14-ecosystem-evidence.md](2026-09-14-ecosystem-evidence.md) — ecosystem selection evidence
- [2026-09-14-hardware-validation-matrix.md](2026-09-14-hardware-validation-matrix.md) — untested hardware tests
- [../decisions/0005-v1-ecosystem-selection.md](../decisions/0005-v1-ecosystem-selection.md) — accepted direction
