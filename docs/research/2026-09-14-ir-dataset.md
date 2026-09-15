# Research — IR Dataset Provenance, Quality, Matching Strategy

**Date:** 2026-09-14 (corrected 2026-09-14 session 2)
**Phase:** discovery
**Status:** evidence for V1 IR capability, licensing corrected per primary source

## Requirement

PRODUCT.md fixed constraints: "Built-in phone IR is a V1 capability when the Android device exposes suitable hardware." and "One visible remote may transparently use more than one control transport, for example network control for rich commands and IR for power." Business rule: "Built-in IR is a V1 capability only where the phone exposes suitable hardware." Also: "IR setup uses likely code profiles and asks the user to verify simple commands; do not blindly transmit every known code. If multiple IR profiles partly work, verify important commands and select the profile with the best demonstrated coverage."

Research still required: "Validate IR dataset provenance, quality, matching strategy, and device coverage."

## Provenance

### IRDB (Infrared Database) — Primary source licensing

- **Source:** `github.com/probonopd/irdb` [VERIFIED gh api repos/probonopd/irdb], mirror of `irdb.globalcache.com` historically [INFERRED from README]
- **Primary README:** "One of the largest crowd-sourced, manufacturer-independent databases of infrared remote control codes on the web" [VERIFIED gh api repos/probonopd/irdb/readme]
- **License file primary:** `https://github.com/probonopd/irdb/blob/master/LICENSE.md` [VERIFIED fetch_page raw.githubusercontent.com/probonopd/irdb/master/LICENSE.md]
- **Actual license text (verbatim from primary):**
  > irdb Copyright (c) 2013-15 Simon Peter and contributors
  > You may include this database and derivative works with your software (e.g., app) and/or access this database over network from your commercial or non-commercial software (e.g., app) or embedded hardware (subsequently called "your product") provided that:
  > 1. Prior to using this database in your product, you will inform the irdb project about your product by opening an issue on https://github.com/probonopd/irdb/issues
  > 2. The following notice shall be included in your product: `Contains/accesses irdb by Simon Peter and contributors, used under permission. For licensing details and for information on how to contribute to the database, see https://github.com/probonopd/irdb`
  > 3. You will make available up to three fully licensed copies/units of your product to the irdb team, represented by Simon Peter, free of charge (including shipping and handling) upon request.
  > If you fail to comply, permission is revoked. AS IS, no warranty.

- **Previous incorrect claim removed:** Earlier version described IRDB as "CC0 / public domain" — **incorrect**, removed everywhere per review finding. Source of incorrect claim was secondary blog `infishark.com` [2](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices) which described IRDB as public domain; primary LICENSE.md contradicts that.

- **Obligations for Greenfield4 V1 (state only what license requires, no interpretation):**
  1. Prior to using database in product, inform irdb project by opening an issue on https://github.com/probonopd/irdb/issues — verbatim from LICENSE.md.
  2. Include notice in product: `Contains/accesses irdb by Simon Peter and contributors, used under permission. For licensing details and for information on how to contribute to the database, see https://github.com/probonopd/irdb` — verbatim.
  3. Make available up to three fully licensed copies/units of product to irdb team, represented by Simon Peter, free of charge (including shipping and handling) upon request — verbatim. No interpretation that Play Store access or APK necessarily satisfies; state only license text. Operational consequence to be confirmed by product-owner/legal, not assumed.
  4. If fail to comply, permission revoked; AS IS no warranty.

- **Access recommendation from primary README:** Do not bundle whole DB, access dynamically at runtime via CDN (e.g., `https://cdn.jsdelivr.net/gh/probonopd/irdb@master/codes/...`) to benefit from updates [VERIFIED gh api readme content]. **Clarification:** Runtime CDN access is recommended by README for updateability, but does not remove license obligations because LICENSE.md explicitly covers both "include this database" and "access this database over network" under same conditions — network access still requires issue, attribution, and copy provision.

- **Acceptability for V1:** **Conditionally acceptable as candidate data source** with obligations tracked, not fixed shipping dependency while legal acceptability unresolved. Not CC0, but custom permissive with attribution and notification. Requires product-owner/legal acceptance of obligations still pending and explicit approval before treating as approved V1 shipping source. Alternative: avoid bundling, use runtime CDN access (still with obligations).

### LIRC (Linux Infrared Remote Control) remote database

- **Source:** `lirc-remotes` SourceForge, GitHub mirror `probonopd/lirc-remotes` [VERIFIED gh api repos/probonopd/lirc-remotes]
- **Primary README:** "The LIRC remote configurations project" — imported from SourceForge, manages config files, no license stated in README [VERIFIED fetch_page raw.githubusercontent.com/probonopd/lirc-remotes/master/README.md]
- **GitHub license field:** `null` [VERIFIED gh api repos/probonopd/lirc-remotes --jq .license] — no explicit license detected via API.
- **LIRC software license:** GPL-2.0-or-later for tools like lirc-config-tool [VERIFIED web_search lirc.org/html/lirc-config-tool.html] and for `aldebaran/lirc` GPL-2.0 [VERIFIED gh api search]. However, **database/config files licensing cannot be inferred from software license** per review finding.
- **Config file header example (secondary):** Example config from lirc docs contains "# Please make this file available to others by sending it to ..." suggesting intent to share, but not a formal license [VERIFIED web_search lirc.readthedocs.io].
- **Status:** **Unresolved** — no authoritative license file found for the database/config data itself in primary sources reachable via allowlisted egress. Must remain unresolved until primary license file located or contributor terms clarified. Do not use as legally cleared.

### Other sources

- WinLIRC, Pronto databases — varying formats, licensing not verified in this session [INFERRED from earlier secondary source, now marked secondary].
- Commercial: Crestron, Control4 private DBs — more coverage in pro AV, but licensing commercial, not evaluated [SECONDARY ESTIMATE from infishark blog, not primary].

## Quality and coverage — secondary estimates

- **Previous claim:** ">500,000 codes, >10,000 brands, 80-85% TV coverage" — source was secondary blog `infishark.com` [2](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices), not primary IRDB repo. Primary IRDB README describes itself as "one of the largest" but does not list exact counts [VERIFIED gh api readme].
- **Correction:** Treat size/coverage numbers as **secondary estimates with uncertainty**, not verified facts. Label as [SECONDARY ESTIMATE] and do not embed in fixed PRODUCT.md constraints.
- **How codes added (secondary estimate from same blog):** manufacturer docs (Sony SIRC, Denon/Marantz) authoritative, IR capture dominant, OEM cross-referencing rebadged products [SECONDARY ESTIMATE].
- **Quality risks (inference):** captured codes may be incomplete, format conversion timing errors, community contributions vary. Mitigation per PRODUCT.md: verify via user flow, select profile with best demonstrated coverage.

## Matching strategy (for V1) — licensing-aware, candidate source

Per PRODUCT.md power/failure and DOMAIN.md:

1. **Phone capability detection:** Check ConsumerIrManager; if absent, hide IR.
2. **Candidate profiles:** IRDB is candidate/evaluated data source (not fixed shipping dependency while legal acceptability unresolved). If using IRDB, primary README recommends runtime CDN access per primary README recommendation, not full bundling, to benefit from updates [VERIFIED]. This does NOT remove license obligations because LICENSE.md covers network access too. Filter by brand/model, OEM cross-reference.
3. **Verification flow:** Ask user to point phone at TV, transmit low-risk command (Power/Volume), ask "Did TV respond?", verify 2-3 more commands, compute coverage score, select best demonstrated coverage, store as IR profile entity with source + verified coverage.
4. **No blind transmit:** Never iterate every known code automatically.
5. **Hybrid control:** One visible remote may combine transports.

## Device coverage for V1 — revised

- Target: Samsung and LG TVs via IR fallback for power when WoL fails.
- **Previous claim:** IRDB contains Samsung and LG codes verified via LIRC list [SECONDARY]. Now marked as secondary estimate: LIRC remotes list includes lg, samsung entries via gist referencing remotecentral, lirc-remotes, remotecodelist, harctoolbox [SECONDARY], not primary verification of IRDB contents.
- **Revised:** Expected coverage for Samsung/LG via IRDB is **inferred** from brand popularity, not verified from primary index. Must be validated via real hardware or direct CDN index check (`https://cdn.jsdelivr.net/gh/probonopd/irdb@master/codes/index`).

## Licensing and provenance rule compliance — corrected session 3

- **IRDB:** Custom permission, not CC0. Obligations verbatim: notify via issue, include attribution notice, make up to 3 fully licensed copies/units available on request (no interpretation that Play Store/APK satisfies). Must be tracked as legal obligation requiring product-owner/legal acceptance still pending. Runtime CDN access recommended by README for updateability but does not remove obligations because license explicitly covers network access. Candidate/evaluated data source, not fixed shipping dependency while acceptability unresolved.
- **LIRC DB:** Licensing unresolved (license null); do not treat as legally cleared via GPL. Leave unresolved and **not approved as shipping source** until database/config licensing established.
- **Uploaded ZIPs:** Evidence only per PRODUCT.md provenance rule.
- IR codes are public facts, not credentials, but database compilation has licensing.

## Release criteria for IR

- **Tested:** IR profile verified on real hardware with user confirmation flow.
- **Expected:** Profile from IRDB (accessed via CDN) that matches brand family but not yet verified via user flow.
- **Unsupported:** No profile found or phone lacks IR hardware.

## Open questions — session 3

- IRDB: Does CDN runtime access count as "accessing database over network" under license? Yes, license explicitly allows accessing over network, with same obligations — CDN does not reduce obligations. Operational interpretation of "up to three fully licensed copies/units" for free app (Play Store/APK vs physical shipment) requires product-owner/legal confirmation, not assumed.
- LIRC: Need primary license file for config data; remains unresolved, not approved as shipping source.
- Coverage stats need primary authoritative source or hardware validation.
- Governance: Product-owner/legal acceptance of IRDB obligations still required before treating as approved V1 shipping source; IR capability fixed, data sources remain candidate/evaluated until resolved.

## References

- IRDB primary LICENSE.md [VERIFIED fetch_page raw.githubusercontent.com/probonopd/irdb/master/LICENSE.md]
- IRDB primary README [VERIFIED gh api repos/probonopd/irdb/readme]
- LIRC remotes mirror README [VERIFIED fetch_page raw.githubusercontent.com/probonopd/lirc-remotes/master/README.md], license null [VERIFIED gh api]
- LIRC software license GPL-2.0-or-later example [VERIFIED web_search lirc.org/html/lirc-config-tool.html]
- Previous secondary source that gave incorrect CC0 claim: infishark blog [SECONDARY ESTIMATE]


---

## 2026-09-15 harvest reconciliation addendum (issue #13)

This addendum updates the **candidate-source set**, not the fixed product capability. It follows the
Harvest / Adopt / Reject pass in
[`2026-09-15-harvest-adopt-reject.md`](2026-09-15-harvest-adopt-reject.md).

### Flipper-IRDB — new preferred low-obligation candidate, with a hard provenance boundary

**Primary source:** `Lucaslhm/Flipper-IRDB` at
`d126fb1b6f1e114c52b4a8c19839ea65e3a9c24d`.

- GitHub repository metadata identifies the current repository license as **CC0-1.0**.
- The root `LICENSE` contains the CC0 1.0 legal text.
- The README says contributors agree to license submissions under CC0-1.0 **and explicitly states
  that commits before `2319685` are not covered**.
- Commit `2319685f2cbf0cd3f809609622cade14d24fb819` is dated 2025-08-07 and is titled
  `feat: add LICENSE and add license note to README (#960)`.

**Consequence:** the earlier candidate set was too narrow (IRDB + LIRC only), but the uploaded
harvest pack's opposite claim — “Flipper-IRDB is public domain, no database licensing risk ever” —
is also too broad.

**HARVEST:** Flipper `.ir` format, naming conventions, post-cutover CC0 contribution policy, and
the project as a source of candidate profiles.

**ADOPT candidate:** prefer a curated V1 seed whose relevant file content can be shown to originate
under the post-`2319685` CC0 policy (or is separately cleared). This can avoid IRDB's custom
notification/attribution/copy obligations if coverage is adequate.

**REJECT:** treating an unchanged pre-cutover file as CC0 merely because it exists in a repository
whose root now contains a CC0 license. The project's own README prevents that inference.

### LIRC — separate import format from bundled corpus

The unresolved LIRC database license remains unresolved. The harvest pass found good precedent for
**parsing user-supplied LIRC files**, but that does not license Greenfield4 to bundle the LIRC
database.

Therefore:
- LIRC may remain a future/user-import **format**.
- LIRC is still **not approved as a bundled V1 data source**.
- Greenfield4 **does not need to solve LIRC corpus licensing as a discovery exit criterion** if a
  provenance-clean Flipper-IRDB subset (or another approved source) provides sufficient V1 coverage.

### IRRemoteESP8266 license correction

The supplied matrix called `crankyoldgit/IRremoteESP8266` “Apache-2.0”. Current GitHub repository
metadata instead identifies it as **LGPL-2.1**, and the repository carries `LICENSE.txt`.

Its protocol behavior, supported-protocol catalogue and test-vector ideas remain useful **HARVEST**
material, but any later code reuse/port must be reviewed against the actual LGPL terms. Discovery
does not select a Kotlin codec module or authorize copying implementation code.

### Updated V1 source order

1. **First candidate:** provenance-clean Flipper-IRDB CC0 subset; validate TV coverage and command
   quality on the hardware matrix.
2. **Fallback candidate:** IRDB under its documented custom obligations, if explicitly accepted.
3. **Do not bundle:** LIRC database until a primary license is established.
4. **Interoperability only:** LIRC / IRPLUS / Flipper formats may be parsed for user-provided files
   independently of whether their upstream corpora are bundled.

This narrows the remaining IR work to **provenance filtering + coverage validation + source
selection**. Another broad IR-database survey is not required unless these candidates fail coverage.
