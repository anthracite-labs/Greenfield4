# ADR-0005: V1 Smart-TV Ecosystems — Android TV/Google TV + Samsung Tizen

**Date:** 2026-09-14
**Status:** accepted
**Deciders:** Discovery research (session arena/01a0a053-greenfield4), product owner via docs/PRODUCT.md evidence bar

## Context

PRODUCT.md V1 is a finished consumer product targeting two major smart-TV ecosystems selected by evidence, not by uploaded ZIPs alone, with explicit validation of legal/terms, security model, protocol stability, pairing, wake, capabilities, and maintenance risk. Market data Q4 2024 shows Android/Google TV >24% OS shipments, Samsung Tizen 16.9%, LG webOS 11.8%, Roku 9% (TechInsights secondary summary). Security product requirements state security wins over popularity, pairing secrets treated like passwords, no global ignore-security mode, and DOMAIN.md defines Paired device invariants. Research in docs/research/2026-09-14-ecosystem-evidence.md evaluated Samsung Tizen (samsungtvws v3.0.5, WebSocket token auth), LG webOS (LGWebOSRemote, WebSocket port 3000, pairing), Android TV Remote v2 (kud/androidtv-remote TypeScript, TLS cert + PIN), and Roku ECP (port 8060 HTTP no auth). IR dataset validated as IRDB CC0/public domain 500k+ codes and LIRC remotes DB. Naming remains open per ADR-0001.

## Decision

Select **Android TV / Google TV (Remote v2 protocol)** and **Samsung Tizen (2016+ WebSocket API)** as V1 ecosystems. Defer LG webOS as immediate next candidate. Reject Roku ECP for V1 due to security model failure. Record hardware matrix criteria (Tested/Expected/Unsupported) requiring at least two models/firmware per ecosystem for release.

## Alternatives considered

### Alternative: Samsung Tizen + LG webOS

- **Pros:** Two largest single vendors by brand recognition (Samsung 16.9%, LG 11.1% shipments), both support Wake-on-LAN, pointer/trackpad, premium OLED/QLED positioning, clear marketing "Works with Samsung and LG".
- **Cons:** Misses Android/Google TV OS share >24% which covers Sony, TCL, Hisense, Xiaomi, Shield — broadest single protocol. Android-first product constraint favors Android TV. Both vendor-specific, not OS-wide.
- **Why not:** Maximizing reach with two ecosystems favors Android TV + Samsung (40.9% combined shipment) over Samsung + LG (28%). Android TV also gives native Google Cast for V1 casting/mirroring scope and lowest maintenance burden (Google-maintained protocol). LG remains strong but deferred to keep V1 tight.

### Alternative: LG webOS + Android TV

- **Pros:** Good reach (35.9%), both have strong security (pairing + token/cert), both support keyboard/text input, LG brings Magic Remote pointer trackpad which is premium UX differentiator, Android brings broad OEM coverage.
- **Cons:** Misses Samsung vendor leadership (16.9% shipments, 12% US CTV per Pixalate) and Samsung Frame Art Mode etc. Samsung's Tizen library samsungtvws is more active (3.0.5 released 2026-05-28) than LG community lib. Product would launch without largest vendor.
- **Why not:** Samsung's market leadership and active maintenance outweigh LG's pointer advantage for V1. LG is better as third ecosystem after V1 proves two.

### Alternative: Roku ECP + Samsung or Android TV

- **Pros:** Roku has 38% US CTV market share Q1 2025, 25% UK, 73% Mexico — huge popularity, especially US. Protocol is simplest HTTP REST, no pairing, 80 lines TypeScript remote possible.
- **Cons:** Security model fails PRODUCT.md and DOMAIN.md invariants: no authentication, no secret, any LAN device can control, documented as "Anything inside the LAN could use it to issue ECP commands". PRODUCT.md states "Security wins over popularity or convenience. A popular ecosystem is not worth shipping through an unsafe trust model." and "If a secure connection cannot be established, refuse the unsafe connection. There is no global 'ignore security' mode." Roku ECP has no secure connection. Also browser mixed-content limitation (HTTPS page cannot call HTTP TV). Legal terms ambiguous about 3rd-party mobile apps.
- **Why not:** Explicitly rejected for V1 due to security bar failure. Must be honest about Unsupported status rather than hiding limitation. Deferred with documented rationale, not hidden as "offline".

### Alternative: Include Fire TV / VIDAA / Titan OS in V1

- **Pros:** Additional reach (Fire TV 18% US, VIDAA 7.8% global, Titan 4.9%).
- **Cons:** Insufficient evidence in allowlisted sources, protocol docs not reachable, maintenance burden high for V1. PRODUCT.md says two ecosystems for V1, not five.
- **Why not:** Out of scope for V1, would violate scope discipline and increase risk.

## Consequences

### Positive

- V1 maximizes user reach with two distinct stacks (OS + vendor) while meeting security invariants (certificate + token pairing, no bypass).
- Both ecosystems support core capability-driven remote requirements: discovery, pairing, power via WoL/toggle, volume, D-pad, text input via IME, app launch, app list — enabling <2 min setup target.
- Android TV gives Google Cast natively, satisfying V1 casting/mirroring without cloud relay, keeping local-first promise.
- Maintenance burden minimized: Android TV protocol is Google-maintained stable v2 since 2021 with modern TS lib; Samsung lib active 2026.
- Clear honest compatibility story: Tested vs Expected vs Unsupported per PRODUCT.md, with Roku explicitly documented as Unsupported for V1 due to security, not as "not found".
- Leaves room for LG webOS as third ecosystem with minimal rework, preserving credible iPhone path (both protocols platform-agnostic).

### Negative

- V1 will not support Roku TVs at launch despite high US popularity — must communicate clearly to avoid user disappointment; need "found but not supported yet" UX per first-run journey step 4.
- Samsung token rotation observed in community ("TV keeps asking to accept connection, token changing") requires robust re-pair handling and safe storage.
- Legal coverage reduced: vendor developer portals blocked by egress allowlist, so terms evaluation relies on secondary summaries and GitHub primary sources; not a legal opinion, must be re-validated by human with access to official terms.
- Hardware matrix not yet executed — no real devices in sandbox, so all claims remain Expected, not Tested. Release still requires deliberate real-hardware matrix.
- IR fallback still requires phone hardware detection and user verification flow; IRDB coverage 80-85% means 15-20% long-tail TVs remain uncovered.

### Follow-ups

- Update docs/PRODUCT.md V1 scope to name selected ecosystems and reference this ADR and research docs.
- Implement IR dataset matching strategy per docs/research/2026-09-14-ir-dataset.md in architecture phase.
- Define release hardware matrix: Samsung at least 2 models/firmware (e.g., Frame 2022 Tizen 6.5 + QLED 2024 Tizen 8.0), Android TV at least 2 OEMs (Sony Bravia + TCL/Google TV) across Remote Service versions >=5, with recorded model/firmware/protocol/capabilities.
- Complete product naming via separate research ADR, then set PROJECT_NAME/SLUG in config/project.env.
- Re-validate vendor terms/legal with human access to official portals before architecture.
- Document Roku ECP as Unsupported for V1 in compatibility view with security rationale, per honest compatibility promise.
- Do not transition to architecture until PRODUCT.md review completes and naming decision recorded.
