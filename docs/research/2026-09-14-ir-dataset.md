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

- **Obligations for Greenfield4 V1:**
  1. Open issue on irdb repo before product use — low effort, compatible with open development.
  2. Include attribution notice in product (e.g., About screen) — compatible with premium UX if placed in settings/about.
  3. Provide up to three fully licensed copies/units on request — for free app, providing Play Store access or APK satisfies; for future hardware, shipping cost must be budgeted. Not onerous but must be tracked as legal obligation.

- **Access recommendation from primary README:** Do not bundle whole DB, access dynamically at runtime via CDN (e.g., `https://cdn.jsdelivr.net/gh/probonopd/irdb@master/codes/...`) to benefit from updates [VERIFIED gh api readme content].

- **Acceptability for V1:** **Conditionally acceptable** with obligations tracked. Not CC0, but custom permissive with attribution and notification. Requires product owner to approve obligations and ensure attribution UI and issue-notification process exists. Alternative: avoid bundling, use runtime CDN access to reduce license surface.

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

## Matching strategy (for V1) — unchanged, but licensing-aware

Per PRODUCT.md power/failure and DOMAIN.md:

1. **Phone capability detection:** Check ConsumerIrManager; if absent, hide IR.
2. **Candidate profiles:** If using IRDB, prefer runtime CDN access per primary README recommendation, not full bundling, to benefit from updates and reduce license bundling surface. Filter by brand/model, OEM cross-reference.
3. **Verification flow:** Ask user to point phone at TV, transmit low-risk command (Power/Volume), ask "Did TV respond?", verify 2-3 more commands, compute coverage score, select best demonstrated coverage, store as IR profile entity with source + verified coverage.
4. **No blind transmit:** Never iterate every known code automatically.
5. **Hybrid control:** One visible remote may combine transports.

## Device coverage for V1 — revised

- Target: Samsung and LG TVs via IR fallback for power when WoL fails.
- **Previous claim:** IRDB contains Samsung and LG codes verified via LIRC list [SECONDARY]. Now marked as secondary estimate: LIRC remotes list includes lg, samsung entries via gist referencing remotecentral, lirc-remotes, remotecodelist, harctoolbox [SECONDARY], not primary verification of IRDB contents.
- **Revised:** Expected coverage for Samsung/LG via IRDB is **inferred** from brand popularity, not verified from primary index. Must be validated via real hardware or direct CDN index check (`https://cdn.jsdelivr.net/gh/probonopd/irdb@master/codes/index`).

## Licensing and provenance rule compliance — corrected

- **IRDB:** Custom permission, not CC0. Obligations: notify via GitHub issue, include attribution notice, provide up to 3 copies on request. Must be tracked as legal obligation, not public domain.
- **LIRC DB:** Licensing unresolved; do not treat as legally cleared via GPL. Leave unresolved.
- **Uploaded ZIPs:** Evidence only per PRODUCT.md provenance rule.
- IR codes are public facts, not credentials, but database compilation has licensing.

## Release criteria for IR

- **Tested:** IR profile verified on real hardware with user confirmation flow.
- **Expected:** Profile from IRDB (accessed via CDN) that matches brand family but not yet verified via user flow.
- **Unsupported:** No profile found or phone lacks IR hardware.

## Open questions

- IRDB: Does CDN runtime access count as "accessing database over network" under license? Yes, license explicitly allows accessing over network, with same obligations.
- LIRC: Need primary license file for config data.
- Coverage stats need primary authoritative source or hardware validation.

## References

- IRDB primary LICENSE.md [VERIFIED fetch_page raw.githubusercontent.com/probonopd/irdb/master/LICENSE.md]
- IRDB primary README [VERIFIED gh api repos/probonopd/irdb/readme]
- LIRC remotes mirror README [VERIFIED fetch_page raw.githubusercontent.com/probonopd/lirc-remotes/master/README.md], license null [VERIFIED gh api]
- LIRC software license GPL-2.0-or-later example [VERIFIED web_search lirc.org/html/lirc-config-tool.html]
- Previous secondary source that gave incorrect CC0 claim: infishark blog [SECONDARY ESTIMATE]
