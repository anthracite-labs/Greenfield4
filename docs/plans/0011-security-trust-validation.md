# Plan — Issue #11: validate Samsung Tizen and Android TV Remote v2 security trust models

**Requirement:** Determine, from primary evidence wherever possible and separately for each
approved V1 ecosystem, what the pairing trust model actually establishes — what authenticates
each side, what the user physically verifies, what identity can be persisted for reconnect, and
what happens when that identity changes — then separate what is provable from what needs
real-hardware validation, and convert the hardware-dependent questions into explicit tests.

**Issue:** #11 (this task). **ADR-0005:** accepted, product-direction only, unchanged by this work.
**Phase:** discovery. **Base:** `main` at `92730e6` (PR #10 merged).

## Acceptance criteria

Copied verbatim from Issue #11; the full list is in the issue body. Condensed here to the
criteria that shape the approach:

1. Each ecosystem documented separately across five concerns: TLS transport validation,
   pairing authentication, persistent device identity, first-use trust, reconnect trust.
2. Per ecosystem: what authenticates TV→client; what authenticates client→TV; what the user
   physically verifies; what cryptographic identity is bound; what can safely be persisted;
   what happens on identity change; whether first-use MITM resistance is established; whether
   pinning is viable.
3. Every load-bearing claim cites a pinned primary source, or is labelled secondary/inference.
4. Unprovable first-use binding recorded as **unresolved**, not filled with inference.
5. TOFU/pinning described as candidate mitigation only, never as solving first use.
6. A reproducible hardware-validation matrix with defined devices, firmware, expected
   observation, pass/fail, and evidence to record.
7. No hardware test reported as passed.
8. Six security decision questions answered per ecosystem.
9. Findings mapped against named PRODUCT.md / DOMAIN.md invariants.
10. No PRODUCT.md / DOMAIN.md security requirement weakened.
11. ADR-0005 direction not reversed; no PARTIAL upgraded to VERIFIED; no shipping-ready claim.
12. Lifecycle unchanged: `discovery`, `ALLOW_APP_STACK=0`, `STACK_DECISION_ADR=` empty.
13. Inaccessible vendor material reported as reduced coverage.
14. `scripts/verify.sh` exits 0, skips named; `scripts/selftest.sh` run and reported.
15. Code review, spec review, and mandatory security review performed and recorded.
16. One PR per issue, session branch only, left open, not self-merged.

## Approach

Research-only. No application code, no stack, no lifecycle change. The method is:

1. **Read primary protocol implementations at pinned refs** rather than trusting blog posts
   or the previous session's summary. Two independent implementations were used per ecosystem
   where one exists, so a shared upstream mistake is less likely to be mistaken for protocol truth.
2. **Label every claim.** `VERIFIED` requires a file path plus a pinned version/commit/tag read
   in this session. Everything else is secondary, inference, proposed mitigation, or unresolved.
3. **Separate first-use from reconnect.** This is the distinction the previous research blurred,
   and it is where TOFU is most often oversold.
4. **Stop at the boundary of provability.** Where the TV's own behaviour is closed firmware,
   say so and turn it into a hardware test.

## Phases

1. **Preflight** — verify repository/GitHub state from tools; close the obsolete PR #9; search
   for an existing issue; create Issue #11 before making durable conclusions.
   *Verify:* `gh pr view`, `gh issue list`, `config/project.env`.
2. **Samsung Tizen primary evidence** — read `samsungtvws` at tag `v3.0.6`: transport, both the
   sync and async TLS paths, token issue/storage/use, and whether anything binds the token to a
   certificate or device identity.
   *Verify:* `gh api` file fetch at `?ref=v3.0.6`, base64 decode, grep.
3. **Android TV Remote v2 primary evidence** — read `kud/androidtv-remote` at `5a05d73eb477`
   and, as a cross-check, `tronikos/androidtvremote2` at `b09f21432ba3`: certificate generation,
   TLS options on both ports, the pairing digest, the code format, and reconnect behaviour.
   *Verify:* `gh api` file fetch at pinned commits; `node -e` to confirm the hex-parsing behaviour.
4. **Synthesise the trust model** into a focused research document with per-ecosystem states and
   explicit first-use vs reconnect separation.
5. **Write the hardware-validation matrix** for everything that cannot be proven without TVs.
6. **Update MEMORY.md** append-only, and PRODUCT.md only where the evidence state genuinely
   changed. Leave ADR-0005's status and decision alone.
7. **Review** — code review, spec review against Issue #11, mandatory security review.
8. **Verify** — `scripts/verify.sh`, `scripts/selftest.sh`, full `main...HEAD` diff inspection,
   then open the PR and leave it open.

## Risks and unknowns

- **Egress allowlist.** `developer.samsung.com`, `developer.android.com` and Google partner
  portals are not reachable from this sandbox. Reported as reduced coverage; not worked around.
- **TV firmware is closed.** Whether the Android TV verifies the pairing digest against its own
  computation cannot be read from any client implementation. Recorded as inference plus a
  hardware test, not as a fact.
- **Two client implementations can share one upstream error.** Both Android TV libraries derive
  from the same lineage, so agreement is corroboration, not independent proof of TV behaviour.
- **No hardware in the sandbox.** Every behavioural claim about reboots, firmware updates,
  factory reset, and certificate stability is necessarily untested.
- **Scope creep risk.** This work must not become architecture selection or a stack decision.

## Out of scope

- Selecting a language, framework, or architecture.
- Writing application code or designing the pinning implementation in code.
- Vendor legal/terms validation (separate open item; needs human portal access).
- Executing any hardware test.
- Changing ADR-0005's decision or status, or reversing the accepted ecosystem direction.
- Moving out of `discovery` or touching `config/project.env`.
