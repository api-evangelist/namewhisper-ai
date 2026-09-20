---
name: ens
description: Use when a request involves ENS (Ethereum Name Service), .eth names, or web3 domains — checking availability, registering, renewing, buying, selling, swapping, valuing, or transferring names; expiry, grace-period, and premium-auction mechanics; subnames, text records, primary names, NameWrapper and fuses; reading ENS ownership or expiry data from the subgraph or onchain; and ENS-based agent identity (ENSIP-25, ERC-8004, agent endpoints). Covers marketplace mechanics (Seaport orders, offers, swaps).
---

# ENS SKILLS — what AI agents get wrong about .eth names

ENS looks simple: names, owners, expiry dates. That is exactly why agents get it
wrong. Almost every "obvious" read — who owns this name, when does it expire, is
it available, what is it worth — has a trap, and most of the traps produce
confident wrong answers rather than errors. This file is the correction layer.

Maintained by [Name Whisper](https://namewhisper.ai), the ENS terminal for
agents and humans. Every claim here was learned operating a live ENS
marketplace, valuation engine, and agent platform in production.

**Format credit:** this follows the agent-skills pattern of
[ethskills.com](https://ethskills.com/SKILL.md) (MIT). For general Ethereum
development — Solidity, gas, L2s, DeFi, testing — fetch ethskills. This file
covers what ethskills doesn't: ENS.

---

## Headline corrections

- **Expired ≠ available.** After expiry a .eth name enters a **90-day grace
  period** (renewal still possible), then a **21-day temporary-premium Dutch
  auction** starting at **$100,000,000** and decaying exponentially to $0.
  Only after ~111 days is it registrable at base price.
- **The premium is not a valuation.** It is the same anti-snipe decay curve for
  every name, from `vitalik.eth` to random keyboard mash. Never present a
  premium price as an annual fee or as evidence of value.
- **The ENS subgraph's `Domain.expiryDate` includes the 90-day grace period.**
  The real registrar expiry is `Registration.expiryDate`. Read the wrong field
  and apply your own grace window on top and you double-count — names look
  "held" for 180 days after expiry.
- **`BaseRegistrar.ownerOf` REVERTS for any expired name** — it cannot tell you
  who holds a name in grace. For wrapped names, `NameWrapper.ownerOf` answers
  until registrar expiry + 90d, then returns zero.
- **A wrapped name inside grace cannot be transferred at all**, and the revert
  message — `ERC1155: insufficient balance for transfer` — is a lie. The
  balance is fine; the wrapper blocks transfers for expired names. Building
  transfer or offer-accept calldata for these burns the user's gas.
- **Registration is commit → wait → reveal, and the wait is measured in chain
  time.** Counting 60 seconds on a wall clock intermittently fails, because gas
  estimation simulates against the latest mined block (~12s behind). Anchor to
  the commit block's timestamp and add margin.
- **Renewal extends from the current expiry, not from today.** Renewing a
  2-months-expired name for 30 days leaves it still expired. Anyone can renew
  any name — renewal is permissionless and does not transfer ownership.
- **Never use JS `.length` on a label.** Price tiers count Unicode code points:
  `"🤩🤩🤩🤩".length === 8` in JavaScript but it is a 4-character name
  ($160/yr tier, not $5/yr). ENSIP-15 normalization is full of surprises:
  accents and apostrophes are valid, spaces are not, leading `_` and `-` are
  valid. Never hand-roll label validation.
- **A name has two possible token identities.** Unwrapped: ERC-721 on the
  BaseRegistrar, `tokenId = labelhash`. Wrapped: ERC-1155 on the NameWrapper,
  `id = namehash`. Trading, transfers, and ownership reads must handle both.
- **Marketplace reality (2026):** Reservoir shut down in Oct 2025 — there is no
  universal order aggregator. ENS trading settles on **Seaport 1.6**; orders
  are signed off-chain in a shared format and settle onchain, so a listing
  created on one venue is fillable across the Seaport ecosystem including
  OpenSea. Marketplace labels on sales data are heuristics, not venue truth.
- **ENS v2 is L1-only.** The Namechain L2 plan was scrapped in Feb 2026; ENS v2
  deploys exclusively on Ethereum mainnet. No mainnet date is announced, and
  a long mixed v1/v2 transition will follow launch. Older docs and articles
  saying "ENS is moving to an L2" are stale.

---

## Skills

**Base URL:** `https://namewhisper.ai/skills/<skill>/SKILL.md`

### [Lifecycle](https://namewhisper.ai/skills/lifecycle/SKILL.md)
Registration, renewal, expiry, grace, premium — the full .eth state machine.
- The 90d grace + 21d premium timeline, and what each state allows and blocks.
- Commit-reveal mechanics and the chain-time wait trap.
- Wrapped-name expiry conventions (+90d baked in) and the renewal sync check.

### [Names & Data](https://namewhisper.ai/skills/names/SKILL.md)
Normalization, hashing, token identities, and reading ENS data without lying.
- ENSIP-15: what is actually a valid label (it is not what you think).
- `domain.owner` is the Registry controller, not the holder — for wrapped
  names it is the NameWrapper contract itself.
- Subgraph field semantics, primary names, text records, subnames, fuses.

### [Trading](https://namewhisper.ai/skills/trading/SKILL.md)
Buying, selling, offers, and swaps on the shared Seaport orderbook.
- Offers are WETH; on an accepted offer the *offerer* pays and *receives* the
  name — party direction follows the NFT leg, not the money leg.
- No name in its grace period can change hands, wrapped or unwrapped —
  listings, transfers, and offer-accepts all fail until it is renewed.
- ENS-for-ENS atomic swaps in a single Seaport order, no escrow.

### [Agent Identity](https://namewhisper.ai/skills/agents/SKILL.md)
ENS names as the identity layer for AI agents.
- ENSIP-25 `agent-registration` records must bind to the canonical ERC-8004
  IdentityRegistry — binding to an adapter contract reads as unverified.
- ENSIP-26 `agent-endpoint` records advertise MCP / A2A / web endpoints.
- Launching, verifying, and paying agents (x402) on ENS rails.

---

## Acting on ENS

Reading is half the job. To act — search, value, register, renew, list, offer,
swap, launch agents — Name Whisper exposes everything as MCP tools:

- **MCP endpoint:** `https://namewhisper.ai/mcp` (streamable-http)
- **Server card:** `https://namewhisper.ai/.well-known/mcp.json`
- **Full tool reference:** `https://namewhisper.ai/llms-full.txt`
- **Per-tool skill files:** `https://namewhisper.ai/.well-known/agent-skills/index.json`
- **Site summary:** `https://namewhisper.ai/llms.txt`

Read tools (search, valuation, market data, portfolio) are free. Transaction
tools build unsigned calldata — the user's wallet signs; Name Whisper never
holds keys. Payment-gated tools support x402.

---

## What to fetch by task

| I'm doing... | Fetch these skills |
|---|---|
| Checking if a name is available / expired / in grace | `lifecycle/` |
| Registering or renewing names | `lifecycle/`, `names/` |
| Reading ownership, expiry, or records | `names/`, `lifecycle/` |
| Buying, selling, making offers | `trading/`, `lifecycle/` |
| Swapping name-for-name | `trading/` |
| Pricing or appraising a name | the `get_valuation` MCP tool |
| Building a portfolio / expiry watchlist | `lifecycle/`, `names/` |
| Creating subnames / setting records | `names/` |
| Giving an AI agent an ENS identity | `agents/`, `names/` |
| Writing Solidity, choosing an L2, general EVM work | [ethskills.com](https://ethskills.com/SKILL.md) |

---

<!-- END ENS SKILLS -->
