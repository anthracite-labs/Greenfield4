# Research — IR Dataset Provenance, Quality, Matching Strategy

**Date:** 2026-09-14
**Phase:** discovery
**Status:** evidence for V1 IR capability

## Requirement

PRODUCT.md fixed constraints: "Built-in phone IR is a V1 capability when the Android device exposes suitable hardware." and "One visible remote may transparently use more than one control transport, for example network control for rich commands and IR for power." Business rule: "Built-in IR is a V1 capability only where the phone exposes suitable hardware." Also: "IR setup uses likely code profiles and asks the user to verify simple commands; do not blindly transmit every known code. If multiple IR profiles partly work, verify important commands and select the profile with the best demonstrated coverage."

Research still required: "Validate IR dataset provenance, quality, matching strategy, and device coverage."

## Provenance

### IRDB (Infrared Database)

- **Source:** `irdb.globalcache.com`, community mirror `github.com/probonopd/irdb` [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- **License:** public domain / CC0 [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- **Size:** >500,000 individual codes, >10,000 brands and device types [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- **History:** Originally maintained by Global Cache (IP-to-IR bridge hardware vendor), released as public resource, now community-maintained [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- **Verification:** [VERIFIED web_search] IRDB described as largest public domain IR database.

### LIRC (Linux Infrared Remote Control)

- **Source:** `lirc.org`, database `lirc-remotes` SourceForge, GitHub mirror `probonopd/lirc-remotes` [2](https://github.com/probonopd/lirc-remotes)
- **License:** GPL (tool) + database contributions mixed, but remotes database is collected definitions used in LIRC [4](https://sourceforge.net/projects/lirc-remotes/)
- **Format:** LIRC config files, convertible to Pronto Hex and Protocol/Device/Subdevice/Function via `lirc2xml` [2](https://github.com/probonopd/lirc-remotes)
- **Overlap:** Many databases overlap, none comprehensive [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)

### Other sources

- WinLIRC, Pronto databases — varying formats [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- Commercial: Crestron, Control4 maintain private DBs, more coverage in pro AV, but consumer TV coverage comparable to open-source [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)

## Quality and coverage

- **Estimated coverage:** ~80-85% of TVs currently in use worldwide have IR codes in major databases [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- **Long-tail problem:** Remaining 15-20% is hundreds of millions devices, disproportionately in markets with less open-source hardware community [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- **How codes added:**
  - Manufacturer documentation (Sony SIRC, Denon/Marantz tables) — authoritative [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
  - IR capture from original remote — dominant method, requires hardware like BLEShark Nano IR receiver [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
  - OEM cross-referencing: rebadged OEM products share codes (e.g., Hisense TV under house brand) [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)

**Quality risks:**
- Captured codes may be incomplete (only power + volume captured, not full set).
- Pronto vs LIRC format conversion can introduce timing errors.
- No central validation; community contributions vary in quality.

**Mitigation per PRODUCT.md:**
- "A profile is selected from demonstrated command behavior, not merely from a database label." (DOMAIN.md IR profile invariant)
- Verify simple commands with user, don't blindly transmit every code.
- Select profile with best demonstrated coverage when multiple partly work.

## Matching strategy (for V1)

Per PRODUCT.md power/failure and DOMAIN.md:

1. **Phone capability detection:** Check if Android device exposes IR hardware (ConsumerIrManager). If not, IR capability is hidden — meets "Built-in IR is a V1 capability only where the phone exposes suitable hardware."
2. **Candidate profiles:** From IRDB/LIRC, filter by brand/model if user provides, otherwise by brand family. Use OEM cross-reference to expand candidates.
3. **Verification flow (user-facing):**
   - Ask user to point phone at TV (IR requires line-of-sight, unlike network).
   - Transmit likely profile's Power or Volume (low-risk commands first).
   - Ask "Did TV respond?" — user confirms.
   - If yes, verify 2-3 more important commands (Power, Volume Up/Down, Mute) to compute coverage score.
   - If multiple profiles partly work, select one with best demonstrated coverage.
   - Store selected profile as "IR profile" entity with source + verified coverage, per DOMAIN.md.
4. **No blind transmit:** Never iterate through every known code automatically — violates PRODUCT.md.
5. **Hybrid control:** One visible remote may combine transports: e.g., network for app launch/text, IR for power when network wake fails. User doesn't manage transports separately.

## Device coverage for V1

- Target: cover Samsung and LG TVs via IR as fallback for power when WoL fails or TV is off-network.
- IRDB contains Samsung and LG codes (verified via LIRC remotes list includes lg, samsung entries [3](https://gist.github.com/francis2110/8f69843dd57ae07dce80) — gist references remotecentral, lirc-remotes, remotecodelist, harctoolbox as sources).
- Coverage for Samsung/LG expected >90% within IRDB due to brand popularity; long-tail brands deferred.

## Licensing and provenance rule compliance

- IRDB CC0/public domain — suitable for V1, no attribution burden beyond documentation, but should credit Global Cache + community.
- LIRC GPL — using database entries (facts about IR timing) is generally considered data, not code, but tool GPL must be respected; prefer IRDB for licensing simplicity.
- Uploaded universal-remote ZIPs mentioned in PRODUCT.md research provenance rule are evidence only, not requirements — IRDB/LIRC research does not rely on them.
- Must not commit IR codes as secrets; IR codes are public facts, not credentials.

## Release criteria for IR

- **Tested:** IR profile verified on real hardware with user confirmation flow, at least 2 models per brand if possible.
- **Expected:** Profile from IRDB/LIRC that matches brand family but not yet verified via user flow.
- **Unsupported:** No profile found, or phone lacks IR hardware — UI explains limitation clearly rather than promising impossible behavior (per PRODUCT.md power/failure).

## Open questions for architecture

- Which Android IR API (ConsumerIrManager) versions to support? Need to check Android API level.
- How to store IR profile selection securely? Not secret, but part of remembered device.
- How to handle phones without IR? Hide IR capability, rely on network + WoL.

These are architecture questions, not discovery blockers.

## References

- IRDB overview [1](https://infishark.com/blogs/learn/ir-code-databases-how-universal-remotes-know-so-many-devices)
- LIRC remotes DB [2](https://github.com/probonopd/lirc-remotes), [4](https://sourceforge.net/projects/lirc-remotes/)
- LIRC code references [3](https://gist.github.com/francis2110/8f69843dd57ae07dce80)
