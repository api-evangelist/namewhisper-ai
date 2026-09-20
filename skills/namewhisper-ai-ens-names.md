---
name: ens-names-data
description: How ENS names actually work as data — ENSIP-15 normalization, labelhash vs namehash, the two token identities (BaseRegistrar ERC-721 vs NameWrapper ERC-1155), reading ownership without lying, subgraph field semantics, primary names, text records, subnames, and fuses. Use when validating labels, resolving names, or reading ENS state from the subgraph or onchain.
---

# ENS Names & Data — reading ENS without lying

Most wrong ENS answers are not RPC failures — they are correct reads of the
wrong field. This skill is the map of which field means what.

## Normalization (ENSIP-15) — your intuitions are wrong

Validity is defined by ENSIP-15, implemented by `@adraffy/ens-normalize`.
**Never hand-roll label validation with an ASCII regex** — verified surprises:

- **Accents are valid and preserved**: `beyoncé.eth` is real and *distinct*
  from `beyonce.eth`. Normalization case-folds and NFC-normalizes but does not
  strip diacritics.
- **Apostrophes are valid**: ASCII `'` maps to curly `’` — `o'brien` →
  `o’brien.eth`, registrable.
- **Spaces are invalid.** Multi-word names must concatenate.
- Leading `_` is valid (mid-label `_` is not). Leading/trailing `-` and even
  `--` are valid; only the punycode-reserved `xx--` pattern at positions 3–4
  is rejected. `$degen` is valid.
- Emoji are first-class labels. Variation selectors (FE0F) are stripped during
  normalization, so visually-identical emoji strings can be the same name.

**Length = Unicode code points on the normalized label.** Not JS `.length`
(UTF-16 code units — double-counts astral-plane emoji), not grapheme clusters
(ZWJ sequences count per code point). This drives price tiers and length-club
membership (999 Club, 10k Club, …).

## Wrapping, in one paragraph

Every skill in this set branches on "wrapped vs unwrapped", so know what it
means: **wrapping** deposits a name's original ERC-721 into the **NameWrapper**
contract, which issues the holder an ERC-1155 token for the same name.
Wrapping is what enables **fuses** (irreversibly burning permissions — see
below) and subnames with real onchain expiry. Unwrapping reverses it, unless
the `CANNOT_UNWRAP` fuse was burned. Names become wrapped either by being
registered through a wrapping controller path or by an explicit wrap
transaction later — so **never assume a name's wrap state; read it** (is the
BaseRegistrar owner the NameWrapper contract?). Length clubs referenced
throughout ("999 Club" = the 1,000 three-digit names `000`–`999`, "10k Club"
= the 10,000 four-digit names) are cohorts defined by this label space.

## Hashing and token identities

- `labelhash = keccak256(label)` — one label, e.g. `vitalik`.
- `namehash` — recursive hash of the full name, e.g. `vitalik.eth`.
- **Unwrapped .eth 2LD:** ERC-721 on the **BaseRegistrar**,
  `tokenId = uint256(labelhash)`.
- **Wrapped name:** ERC-1155 on the **NameWrapper**,
  `id = uint256(namehash)`. The NameWrapper holds the ERC-721; the user holds
  the ERC-1155.

Same name, two different (contract, tokenId) pairs depending on wrap state.
Any code that matches names to NFTs — trades, transfers, event decoding —
must check both identities. Recovering a label from a tokenId is one keccak
away if you have a candidate list; there is no onchain reverse function.

**ENS v2 will change this** (L1-only; no mainnet date announced):
registries become per-name ERC-1155 contracts and token IDs are deliberately
*mutable* (they regenerate on role changes and re-registrations). Do not build
new long-lived systems that assume `tokenId = labelhash` is forever.

## Reading ownership — the #1 source of ghost data

There are three "owner-shaped" fields and none of them is simply "the holder":

- **Registry owner (subgraph `domain.owner`)** = the *controller*. For a
  wrapped name this is the **NameWrapper contract itself** — store it as the
  holder and the name vanishes from every owner-keyed query. For ~10% of
  unwrapped names the controller is a different wallet from the registrant —
  store it and you file the name into a wallet that never owned it.
- **Registrant (subgraph `registration.registrant`, `BaseRegistrar.ownerOf`)**
  = the true holder of an unwrapped name. `ownerOf` **reverts once expired**.
- **Wrapped holder (`NameWrapper.ownerOf(namehash)`, subgraph
  `wrappedDomain.owner`)** = the true holder of a wrapped name. Answers until
  registrar expiry + 90d, then returns zero — a wrapper-lapsed name reads as
  "unwrapped" to naive checks, hiding exactly the worst cases.

Resolution rule: determine wrap state first, then read the matching field, and
treat the NameWrapper address and `0x0` as "not a holder" everywhere.

Two more ownership traps:

- **Expired names keep their last owner in most caches and indexers** (expiry
  emits no Transfer event). A naive "how many names does this wallet hold"
  counts every name it *ever* held — a wallet with zero active names can read
  as a thousands-of-names whale. Holder counts need an active + grace filter.
- **Long-lived references should store the address or labelhash, never the
  resolved name.** A saved link or subscription keyed on a name silently
  follows whoever owns the name *later* — sell the name, and the reference
  points at a stranger.

## Not everything ENS-shaped is a mainnet .eth 2LD

ENS-style names now come from multiple systems: mainnet .eth, DNS-imported
names (`foo.com`), subnames, and lookalike systems on other chains (Coinbase
Basenames et al.). Data pipelines that assume ".eth 2LD" corrupt themselves
quietly. The reliable guard is to count the labels in the **full name**: a
.eth 2LD has exactly two (`vitalik` + `eth`); anything with more is a subname
and anything not ending in `.eth` is a different system, and neither has a
BaseRegistrar registration. When in doubt,
`BaseRegistrar.nameExpires(labelhash) == 0` is the clean "this was never a
mainnet .eth registration" test.

## Subgraph field semantics (the traps)

- `Domain.expiryDate` **includes the 90-day grace period** for .eth 2LDs; so
  does `WrappedDomain.expiryDate`. Registrar truth: `Registration.expiryDate`.
- `domain.owner` is the controller (above).
- Subgraph transfer events carry **no block timestamp** — if you need event
  times, estimate from block numbers or read receipts.
- The Domain entity is a *derived view*, not a mirror of the registrar. When a
  subgraph field's name suggests it means X, verify before trusting it.

## Primary names (reverse resolution)

A wallet's "primary name" is a reverse record (`addr.reverse`) that the wallet
itself sets, plus a **forward check**: the name must also resolve back to the
address, or clients must ignore it. Never display a reverse record without
verifying the forward resolution — anyone can point a reverse record at any
name. A name being someone's primary says nothing about it being their only
name. To *set* one: the wallet calls `setName` on the reverse registrar (the
official app exposes this as "set primary name") — it must be sent from the
address being named, and the forward record must already point home.

## Text records, subnames, fuses

- **Text records** are arbitrary key-value pairs on the resolver (`avatar`,
  `url`, `com.twitter`, agent records — see the
  [agents skill](https://namewhisper.ai/skills/agents/SKILL.md)). Reading them
  requires the name's resolver; names with no resolver have no records.
- **Subnames** (`sub.name.eth`) are minted by the parent. Wrapped subnames have
  their own wrapper expiry with **no grace baked in** (unlike 2LDs);
  unwrapped subnames have no expiry of their own at all.
- **Fuses** (NameWrapper) are permissions burned onto wrapped names.
  The two you will actually meet: `PARENT_CANNOT_CONTROL` (65536, marks an
  emancipated name) and `IS_DOT_ETH` (131072). A typical wrapped 2LD carries
  196608 = both. `CANNOT_TRANSFER` exists and makes a name soulbound — check
  fuses before assuming a wrapped name is tradable.

## Acting on names & data

Name Whisper MCP tools (`https://namewhisper.ai/mcp`):

- `search_ens_names` — natural-language + semantic search over 3.5M names.
- `get_name_details` — normalized label, wrap state, owner, expiry, records.
- `get_primary_name` — verified reverse resolution.
- `get_wallet_portfolio` — owner-keyed holdings with the ghost-data traps
  handled.
- `set_ens_records` / `manage_fuses` / `mint_subnames` — unsigned record and
  wrapper transactions.

Related: [lifecycle](https://namewhisper.ai/skills/lifecycle/SKILL.md) for
expiry semantics. Index:
[https://namewhisper.ai/SKILL.md](https://namewhisper.ai/SKILL.md)
