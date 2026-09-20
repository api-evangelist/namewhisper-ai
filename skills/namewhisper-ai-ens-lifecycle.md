---
name: ens-lifecycle
description: The full .eth name state machine — registration (commit-reveal), renewal, expiry, the 90-day grace period, the 21-day premium Dutch auction, and re-registration. Use when checking availability, registering or renewing names, watching expiries, or reasoning about what an expired name's holder can and cannot do.
---

# ENS Lifecycle — registration to expiry and back

The single most common ENS mistake agents make is treating expiry as a boolean.
A .eth name moves through five states, and each one allows and blocks different
actions:

```
ACTIVE ──expiry──▶ GRACE (90d) ──▶ PREMIUM (21d) ──▶ AVAILABLE ──register──▶ ACTIVE
   ▲                  │ renew (base price, no premium; extends from OLD expiry)
   └──────────────────┘
```

## The timeline, precisely

- **Active:** registered, owner controls it. Anyone may renew it (renewal is
  permissionless and never transfers ownership).
- **Grace (expiry → +90 days):** the name is expired but NOT available. Renewal
  still works and restores the name with full ownership intact. Nobody else can
  register it. **No grace name can change hands, wrapped or unwrapped** —
  listings on grace names are phantoms that cannot settle:
  - **Unwrapped:** the BaseRegistrar's transfer path checks ownership through
    `ownerOf`, which reverts for any expired name — so transfers and sale
    fulfillments revert.
  - **Wrapped:** the NameWrapper blocks transfers of an *emancipated* name
    (one whose `PARENT_CANNOT_CONTROL` fuse, 65536, is burned — which is
    every normally-wrapped 2LD) once it is in grace, and the revert message —
    `ERC1155: insufficient balance for transfer` — is misleading; the real
    cause is grace. Detect it from `NameWrapper.getData(namehash)`:
    `registrarExpiry = wrapperExpiry − 90d`; blocked when that is past and
    fuse 65536 is burned.
  - Either way: never build transfer, sale-fulfillment, or offer-accept
    calldata for a grace name — it reverts and the user pays gas.
- **Premium (grace end → +21 days):** the name becomes registrable by anyone,
  but with a **temporary premium** added to the base price: starts at
  **$100,000,000**, halves every day, and is offset so it reaches exactly
  **$0** at day 21. The premium is an anti-snipe mechanism — the same curve
  for every name. Never describe a premium price as an annual fee
  ("455 ETH/year" is a classic agent hallucination — it is a one-time,
  decaying surcharge), and never read it as a value estimate. To quote it
  correctly: the controller's `rentPrice(name, duration)` returns **base and
  premium as separate components** — present the base as the annual price and
  the premium as a one-time surcharge that shrinks daily.
- **Available:** registrable at base price.

**Re-registration burns and mints in the same transaction.** A fresh
registration of a lapsed name emits a transfer to `0x0` (burn of the old token)
and a mint to the new owner in one tx. Naive event readers see the burn leg and
report "owner: 0x0" for a name that was just registered.

## Registration mechanics

Two transactions, commit → reveal, to prevent frontrunning:

1. `commit(commitment)` — a hash of name + owner + secret.
2. Wait at least **60 seconds** (max 24 hours, or the commitment expires).
3. `register(...)` with the same parameters + secret. Minimum registration
   duration is 28 days. Whether the resulting name is wrapped or unwrapped
   depends on which controller path registered it — verify with
   `NameWrapper.getData` afterwards rather than assuming (see
   [names](https://namewhisper.ai/skills/names/SKILL.md) for what wrapping is).

**Contract addresses:** deliberately not printed here — hardcoded addresses in
agent-facing docs go stale and propagate. Resolve the current controller,
BaseRegistrar, and NameWrapper addresses from the canonical deployments table
at [docs.ens.domains](https://docs.ens.domains/learn/deployments), and never
from model memory.

**The 60-second wait is chain time, not wall-clock time.** Wallets run
`eth_estimateGas` before sending, and that simulates against the latest *mined*
block, whose timestamp trails your clock by up to a full block (~12s). Count 60
wall-clock seconds and the reveal intermittently fails with `CommitmentTooNew`
(selector `0x74480cc9` on the current controller; older controllers use a
1-arg variant, `0x5320bcf9`) — after the user already paid for the commit. Anchor
the wait to the commit block's timestamp, poll `getBlock('latest')`, and add
~15s of margin. This bug is intermittent by nature: it depends on where in the
block cycle the commit landed, so it passes tests and fails in production.

**Pricing** (ENS v1, paid in ETH at oracle rate — always verify live via
`rentPrice`, never from memory):

| Label length (code points) | Price / year |
|---|---|
| 3 characters | $640 |
| 4 characters | $160 |
| 5+ characters | $5 |

1- and 2-character .eth names are not registrable. Count length in **Unicode
code points on the normalized label** — never JS `.length` (UTF-16 units), and
not grapheme clusters either. `🤩🤩🤩🤩` is a 4-character, $160/yr name; JS
`.length` says 8 and prices it at $5. Pricing changes for ENS v2 are moving
through DAO governance, so re-verify prices rather than caching them.

**Testing registration flows:** Sepolia's v1 registrar stopped accepting
registrations permanently in July 2026 (the ENS v2 migration closed it) — a
bare revert from a Sepolia registration is that, not your bug. Test
registration end-to-end on a mainnet fork instead.

## Renewal mechanics

- **Renewal extends from the current expiry, not from today.** A name that
  expired 2 months ago and gets renewed for 30 days is *still expired*. To
  rescue a grace name, renew long enough to clear the lapse.
- Anyone can renew any name. Renewing does not grant ownership.
- **Wrapped names: verify the wrapper synced.** The current
  ETHRegistrarController's `renew()` advances the BaseRegistrar expiry but does
  not write the NameWrapper, so a wrapped name renewed through a naive path can
  end up paid-up on the registrar while the wrapper still thinks it is
  expiring — and at wrapper expiry an emancipated name's wrapper ownership is
  lost until repaired. Wrapper-aware renewal paths exist (the official app
  routes wrapped renewals through one; Name Whisper's bulk renewal contract
  does the same). After renewing a wrapped name, check
  `NameWrapper.getData(namehash).expiry == registrarExpiry + 90d`. A renewal
  that leaves the name still inside grace won't touch the wrapper on *any*
  path — a follow-up renewal that clears grace repairs it.

## Reading expiry correctly

- The ENS subgraph's `Domain.expiryDate` **already includes the 90-day grace
  period** for .eth 2LDs. The registrar expiry is `Registration.expiryDate`
  (= `Domain.expiryDate − 90d`). The NameWrapper's stored expiry uses the same
  +90d convention. Apply your own grace window on top of either and you
  double-count — names read as "held" for 180 days.
- `BaseRegistrar.nameExpires(labelhash)` is the onchain source of truth. Note
  it **retains the last expiry after a name is fully released** — a lapsed,
  re-registrable name still reports its old (past) expiry. `nameExpires == 0`
  means *never registered*, not "available".
- **Never truncate an expiry timestamp to a date** before doing lifecycle math.
  Flooring to midnight UTC shifts expiry up to ~24h earlier, so a name with
  hours left reads as already expired / in grace.
- **Absence from an index is not availability.** A name missing from your
  dataset might simply be a gap in the dataset. Before telling anyone a name
  is available, verify with the controller's `available(label)` onchain.
- `BaseRegistrar.ownerOf(labelhash)` **reverts** for any name past expiry.
  During grace, `NameWrapper.ownerOf(namehash)` still answers for wrapped
  names (until registrar expiry + 90d); for unwrapped grace names the chain
  cannot name the holder — use an indexer.
- Grace ends at an **instant**, not a day. If you surface renewal deadlines,
  never tell a user "today is the last day" — by the afternoon it may be over.

## Acting on the lifecycle

Name Whisper MCP tools (`https://namewhisper.ai/mcp`):

- `check_availability` — availability with correct grace/premium awareness.
- `get_expiring_names` — names entering grace/premium, validated onchain.
- `bulk_register` — commit-once, register-N names in 2 txs total.
- `renew_ens_name` — wrapper-aware renewal with correct duration math.
- `get_name_details` — full lifecycle state for one name.

Related skills: [names](https://namewhisper.ai/skills/names/SKILL.md) for
hashing and ownership reads, [trading](https://namewhisper.ai/skills/trading/SKILL.md)
for why grace names break trades. Index:
[https://namewhisper.ai/SKILL.md](https://namewhisper.ai/SKILL.md)
