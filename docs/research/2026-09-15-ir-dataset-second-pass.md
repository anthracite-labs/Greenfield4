# Research — IR dataset second pass: Harvest / Adopt / Reject

**Date:** 2026-09-15  
**Phase:** discovery  
**Issue/PR:** #14 / #16  
**Scope:** targeted second pass after Arena review; no application implementation.

## Executive conclusion

The Arena pass materially improves the IR decision, but its headline conclusion needs one correction.

Greenfield should **not** adopt the entire `flipperdevices/IRDB` repository as a clean MIT dataset. The
repository has a top-level MIT license, but a large 2024-07-08 import is explicitly traceable to
`ysard/mi_remote_database`. At the import commit, affected files carried an
`AGPL-3.0 license, Copyright (C) 2021-2023 Ysard` notice; a later parsing commit converted many
signals from raw to parsed form and removed that notice from the file text while preserving the IR
content. A downstream top-level MIT file therefore does not, by itself, establish that the imported
subset is cleanly MIT-redistributable.

The community `Lucaslhm/Flipper-IRDB` repository is also not a broad clean substitute. Its README
does clearly state that contributions are CC0-1.0, but explicitly excludes **commits prior to
`2319685`**. Comparing the tree immediately before that boundary with the pinned current tree shows
only **143 IR files first introduced after the boundary, of which 20 are TV files**. That post-boundary
TV subset includes 2 Samsung files and no LG files. It is useful, but not sufficient as the sole V1
database.

Therefore the V1 decision is:

- **HARVEST** Flipper's formats, normalization, parsed protocol representation, button vocabulary,
  contribution workflow, and matching/test mechanics.
- **ADOPT only provenance-clean profiles**, tracked at file/source level. Do not ingest a whole
  repository merely because its top-level license is permissive.
- **DEFER probonopd/IRDB** as a long-tail fallback. Its license is known and usable if the product
  owner/legal later accepts its notification, notice, and up-to-three-copy/unit obligations.
- **REJECT LIRC for V1 shipping** while its remote-data license remains unresolved.
- Do not make IRDB acceptance a discovery blocker. Greenfield can proceed with a small curated V1
  profile set and grow it from verified, provenance-clean contributions.

This is intentionally narrower than "find the biggest IR database." V1 only needs enough clean IR
coverage to support the product's hybrid-control fallback on phones with IR hardware; rich control
continues to use the selected network ecosystems.

## Pinned primary evidence

| Source | Pin | Primary finding |
|---|---:|---|
| `flipperdevices/IRDB` | `f7b15366521cc81ba11b341538f0097bddbb748b` | Top-level MIT; 3,051 current TV `.ir` files across 596 TV brand directories. |
| `flipperdevices/IRDB` import | `992b64a9adfb7b3c5d8a6a9ea13e4759368cd2cb` | Commit message `add mi_remote_database`; 2,808 TV paths already present at this point. |
| `flipperdevices/IRDB` parse conversion | `2e1b88ce8f73f6ab94f11b878d3d9af36dac8566` | `add parsed infrared files (#3)`; converted imported raw signals to parsed form. |
| `ysard/mi_remote_database` | `8a60b419506e8c407f2b756ec8ab4b4bc446678d` | Repository license is AGPL-3.0. |
| `Lucaslhm/Flipper-IRDB` | `d126fb1b6f1e114c52b4a8c19839ea65e3a9c24d` | README + LICENSE available through GitHub; CC0-1.0 contribution policy with explicit boundary. |
| CC0 boundary | `2319685f2cbf0cd3f809609622cade14d24fb819` | 2025-08-07 commit: `feat: add LICENSE and add license note to README (#960)`; README says commits before it are not covered. |
| IRDB | `11aa5eb3ad9fec9e5c03f170c29c1467733d9f3e` | Custom permission license; known obligations. |
| LIRC mirror | `e4a758048908b7e1a571dc2a89c409a33398f2f6` | No explicit database/config-data license established. |

## Critical provenance correction: official Flipper IRDB is mixed

The official repository's top-level `LICENSE` is MIT. That is useful evidence for material for which
Flipper or its contributors actually hold/licensed the relevant rights, but it is not enough to erase
more specific upstream provenance.

A representative imported Samsung TV file at the original import commit begins:

- `from Mi Remote DB <https://github.com/ysard/mi_remote_database>`
- `AGPL-3.0 license, Copyright (C) 2021-2023 Ysard`

The same path after commit `2e1b88ce` contains the parsed Samsung32 address/command representation,
but the upstream license/provenance comment is no longer present in the file body.

That history is load-bearing. Greenfield must treat files whose history traces to the
`992b64a9` Mi Remote import as **mixed/AGPL-marked provenance**, not silently as MIT-only.

This does not require a legal conclusion about whether every IR fact is copyrightable in every
jurisdiction. It simply means Greenfield does not have enough evidence to call that imported subset
cleanly permissive for a proprietary V1 shipping artifact.

### Current-vs-import TV counts

At `992b64a9` the repository already contained **2,808 TV `.ir` paths**. At pinned current head it
contains **3,051**. Using path presence as a conservative first filter leaves **245 TV paths added
after the bulk import**, across **66 brands**.

Major-brand counts in this post-import path set:

| Brand | TV paths added after import |
|---|---:|
| Samsung | 43 |
| LG | 28 |
| Sony | 20 |
| Panasonic | 15 |
| Philips | 15 |
| Vizio | 11 |
| Hisense | 8 |
| Toshiba | 8 |
| Sharp | 6 |
| TCL | 3 |

This is a **candidate provenance-clean subset, not automatically an approved subset**. Before a
specific file is shipped, Greenfield should retain its introducing commit/PR and verify that it does
not itself import from another uncleared source. That is bounded per-profile provenance work, not a
reason to reopen broad ecosystem research.

## Community Flipper-IRDB CC0 boundary

The Arena pass reported the community repository as unavailable. GitHub's repository/file API can
currently serve the pinned README and CC0 `LICENSE`, so the source is usable as evidence even though
clone/search availability has been inconsistent.

The README states:

- contributors agree to CC0-1.0; and
- **commits prior to `2319685` are not covered.**

For a conservative shipping subset, Greenfield should not treat an old pre-boundary file as clean
merely because it was later touched. The strongest automated rule is: file path first appears after
the pre-boundary parent tree, then retain the introducing commit as provenance.

Tree comparison:

- pre-boundary IR files: **8,778**
- pinned current IR files: **8,901**
- paths first introduced after the CC0 boundary: **143**
- TV paths first introduced after the boundary: **20**

Post-boundary TV additions include:

- Samsung: 2
- Sanyo: 2
- Bush: 2
- one each for several other brands including Vizio, Philips, JVC, Toshiba, Hitachi and others
- LG: **0**

Conclusion: the CC0 delta is genuinely clean-looking and worth harvesting, but it is **not broad
enough to replace a V1 IR source by itself**.

## Harvest / Adopt / Reject matrix

| Source/pattern | Evidence | Disposition | Greenfield rule |
|---|---|---|---|
| Flipper `.ir` format and parsed protocol/address/command representation | Mature official/community datasets and firmware ecosystem | **HARVEST** | Reuse representation ideas when architecture is allowed; canonicalize in Greenfield rather than depend on an upstream runtime format. |
| Flipper button vocabulary / normalization / contribution workflow | Mature OSS | **HARVEST** | Reuse normalization and validation ideas. |
| Entire `flipperdevices/IRDB` as "MIT dataset" | Top-level MIT conflicts with file history showing Mi Remote/AGPL-marked bulk import | **REJECT as blanket adoption** | Never classify the whole repository as one clean license bucket. |
| Flipper official profiles first introduced after the Mi Remote import | 245 current TV paths across 66 brands, including useful major-brand coverage | **ADOPT CANDIDATE, per-file provenance required** | Ship only profiles whose introducing commit/source is retained and does not trace to an uncleared import. |
| Mi Remote-derived Flipper paths from `992b64a9` | Original files explicitly identify Mi Remote DB + AGPL-3.0/Ysard | **HARVEST for knowledge; REJECT for proprietary V1 shipping unless legal approves** | Do not strip or ignore provenance because a later transform removed comments. |
| `Lucaslhm/Flipper-IRDB` post-`2319685` first-introduced files | README + CC0 license + explicit boundary | **ADOPT CANDIDATE** | Only post-boundary new files, with introducing commit retained. Too small to be sole source. |
| Pre-`2319685` community Flipper content | README expressly excludes it from CC0 | **REJECT for V1 shipping** | No inference from current repository CC0 file. |
| `probonopd/irdb` | Clear custom permission license | **DEFER / FALLBACK** | Use only if a real coverage gap justifies accepting the known obligations. |
| LIRC remotes | No explicit data/config license found | **REJECT for V1 shipping** | May be used as engineering reference only. |
| User confirmation of low-risk command + multi-command verification | Product requirement; common mature remote setup pattern | **ADOPT** | Persist only demonstrated profiles; no blind transmit sweep. |

## Recommended V1 IR strategy

1. Keep **IR capability** fixed where Android hardware exposes an IR emitter.
2. Do **not** bundle an entire third-party IR database in V1 merely for theoretical breadth.
3. Create a small Greenfield manifest of provenance-clean profiles actually needed for supported
   V1 device families.
4. A profile may enter that manifest only with:
   - source repository;
   - exact source commit;
   - source file/path;
   - applicable license/provenance class;
   - normalized protocol/address/command or raw timing payload;
   - hardware/user-verification state.
5. Prefer:
   - post-boundary CC0 contributions; then
   - official Flipper profiles with clean post-import provenance; then
   - other primary/manufacturer or independently captured sources whose rights are clear.
6. Treat Mi Remote-derived/AGPL-marked Flipper material as excluded from the proprietary shipping
   manifest unless human/legal explicitly approves the intended use.
7. Keep `probonopd/irdb` as a fallback. Accept its obligations only if the clean profile set has a
   material coverage gap that affects a supported V1 device family.
8. Matching UX remains: shortlist by brand/family → low-risk test → user confirmation → verify more
   controls → persist only the demonstrated profile.
9. Grow coverage through provenance-clean contributions after V1 rather than importing historical
   unknown-license corpora wholesale.

## What further research should stop

Stop broad searches for:

- "largest IR database";
- generic IR parsing libraries;
- generic remote setup UX;
- whether Flipper formats can represent Samsung/NEC-family commands;
- LIRC licensing unless a primary explicit data license newly appears;
- IRDB licensing text (already known).

Remaining work is bounded:

1. When selecting an actual V1 profile, verify that file's introducing commit/source once.
2. Hardware-verify the chosen profile before calling it Tested.
3. Use IRDB only if a concrete supported-device gap remains and then make the known obligation
   decision.

## Reconciliation with prior Greenfield research

This supersedes only one earlier recommendation: **IRDB is no longer the preferred conditional V1
candidate.** It becomes a deferred long-tail fallback.

Prior findings remain valid:

- IRDB's custom license obligations are accurately recorded.
- LIRC remains uncleared for shipping.
- IR profile matching must be verified on real hardware.
- IR data-source licensing is distinct from the fixed product capability.

The product does not need another broad IR-dataset research cycle.
