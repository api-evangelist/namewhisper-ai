---
name: ens-trading
description: Buying, selling, offering on, and swapping .eth names — Seaport orderbook mechanics, listings vs offers, party direction on accepted offers, grace-period trading traps, ENS-for-ENS swaps, bulk operations, and reading sales data (wash trades, marketplace labels). Use for any ENS marketplace action or when interpreting ENS sales/listings data.
---

# ENS Trading — the shared Seaport orderbook

## The 2026 landscape

- **Reservoir shut down in October 2025.** There is no universal NFT order
  aggregator anymore; anyone claiming to aggregate through it is running on
  stale information.
- ENS trading settles on **Seaport 1.6**. A listing or offer is a signed
  Seaport order on a shared onchain orderbook — which means liquidity is
  pooled across the Seaport ecosystem: a name listed on Name Whisper is
  fillable on OpenSea and vice versa. "Which marketplace" is largely a UI
  question, not a liquidity question.
- **Marketplace labels on sales data are heuristics.** Venues share Seaport
  contracts, so attribution by contract address collapses everything to one
  label, and API self-reports disagree. Never treat a venue label as truth.

## Listings and offers — mechanics that bite

- **Listings** are seller-signed orders, priced in ETH. Fulfillment transfers
  name and payment atomically.
- **Offers (bids) are WETH**, not ETH — Seaport offers need an ERC-20 the
  contract can pull. An offer without WETH balance + approval behind it is
  noise. To check an offer is real: the offerer's WETH `balanceOf` must cover
  the amount, and their WETH `allowance` to the order's conduit (or Seaport
  itself for no-conduit orders) must too. Both can be revoked after signing,
  so re-check at accept time.
- **Party direction on an accepted offer follows the NFT leg, not the money
  leg.** The *offerer* pays WETH and **receives the name**; the *accepter*
  receives WETH and gives up the name. Marketplace APIs that report buyer and
  seller by following the money get accepted-offer sales **backwards**. When
  accuracy matters, derive parties from the onchain token transfer
  (`to_address` = buyer) — and skip the burn/mint legs: re-registering a
  lapsed name burns the old token and mints the new one in the same tx, so
  "first transfer in the tx" picks the burn and reports the buyer as `0x0`.
- Cancelling an order is an onchain transaction (or signature invalidation
  via counter-increment). An order can also die silently: the seller transfers
  the name away, the listing expires, or WETH approval is revoked. And venues
  support **off-chain cancellation** — an order can be dead at the venue while
  `getOrderStatus` onchain still reports it live. **A listing you can see is
  not necessarily a listing you can fill** — validate against current
  ownership, approvals, and the venue before building fulfillment.
- Venue quirks worth knowing: OpenSea's API **rejects orders expiring more
  than 6 months out** (long listings cross-post as silent 400s), and its
  events API names the token field **per event type** — offers carry `asset`,
  sales carry `nft`. Read the wrong one and 100% of that event type silently
  drops. When an integration returns suspiciously few results, suspect field
  shape before rate limits.

## Lifecycle × trading (where users lose gas)

Cross-check every trade against the
[lifecycle skill](https://namewhisper.ai/skills/lifecycle/SKILL.md):

- **Never build a sale, transfer, or offer-accept for any name in its grace
  period** — wrapped or unwrapped, it reverts and the user pays gas (the
  lifecycle skill has the per-case mechanics; the wrapped revert message,
  `ERC1155: insufficient balance for transfer`, misleads everyone
  downstream). At any given time a meaningful share of open bids sit on
  grace names: this is not an edge case.
- Listings on names that then expire become phantoms. If you index listings,
  invalidate on expiry; if you fill them, re-check expiry first.
- Making an offer on a grace name is legal and rational (bid → owner renews →
  accepts), but the accept can only settle after renewal. **To sell a name
  that is in grace:** renew it first (long enough to clear the lapse), verify
  the wrapper expiry synced if it's wrapped, and only then list or accept.
- A name's tokenId identity can change out from under an order: unwrap/wrap
  moves it between ERC-721 and ERC-1155 identities
  ([names](https://namewhisper.ai/skills/names/SKILL.md) owns this concept),
  killing orders signed against the other one.
- **ENS names carry no protocol royalties.** The cost components of a trade
  are price + venue fee + gas — don't budget for creator fees that don't
  exist.

## ENS-for-ENS swaps

Seaport supports true atomic barter: name(s) for name(s), optional WETH on
either side, in **one order, no escrow**. Both sides settle or neither does —
which removes the counterparty risk of the "sell yours, then buy theirs"
dance.

## Reading sales data

- **Check the currency before you read the number.** ENS sales settle in ETH,
  WETH, DAI, USDC, and USDT, and raw log values carry the *token's* decimals —
  DAI is 18 (indistinguishable from wei by magnitude), USDC/USDT are 6. A
  pipeline that scans receipt values as wei without a token check will record
  a 40 DAI sale as 40 ETH and manufacture phantom whales.
- **In multi-name sweep transactions, attribute payment at the order level,
  never by summing the receipt.** One tx can settle several orders in
  different currencies — the USDC in the receipt may have paid for a
  *different* name than the one you're pricing.
- **Wash trading is endemic in ENS sales feeds.** Patterns to filter: the same
  name recycling at the same price, buyer/seller wallets that share funding
  history, self-sales through fresh wallets, offer-choreography (place bid,
  accept from alt). Naive "last sale" comps inherit all of it.
- Some sales are private-listing mirrors or bulk-sale legs — a single tx can
  carry many names, and per-name price attribution inside a bundle is
  approximate.
- Registration cost ≠ market price. Premium-auction registrations mix a
  time-decaying surcharge into the paid amount
  ([lifecycle](https://namewhisper.ai/skills/lifecycle/SKILL.md)).

## Fees (Name Whisper)

Stated plainly because agents should be able to account for costs:

| Action | Fee |
|---|---|
| Sell via NW listing | 1% (seller-paid, baked into the order) |
| Register / renew via NW | 1% |
| ENS-for-ENS swap | flat $5 in ETH, paid by the accepter |
| Buy, search, value, read data | free |

## Acting on the market

Name Whisper MCP tools (`https://namewhisper.ai/mcp`):

- `get_market_activity` — recent sales, premium registrations, and swaps (no listings or standing offers; it makes no wash judgment — pair with `wash_check`).
- `purchase_name` / `batch_purchase` — fulfill listings (unsigned txs).
- `create_listing` / `batch_create_listings` — create listings fillable
  across Seaport venues.
- `make_offer` / `accept_offer` / `cancel_offer` — WETH offers with
  funding checks and grace-period guards.
- `make_swap` — atomic ENS-for-ENS swap orders.
- `find_alpha` — screener for mispriced listings.

Every transaction tool returns **unsigned calldata** — the user's wallet
signs; Name Whisper never holds keys.

Before pricing anything, get a data-backed estimate from `get_valuation`.
Index: [https://namewhisper.ai/SKILL.md](https://namewhisper.ai/SKILL.md)
