---
name: ens-agent-identity
description: ENS names as the identity layer for AI agents — ENSIP-25 agent-registration records, the canonical ERC-8004 IdentityRegistry, ENSIP-26 agent-endpoint records (MCP / A2A / web), agent verification, x402 payments, and launching hosted agents on .eth names. Use when giving an agent an onchain identity, verifying an agent's claims, or wiring agent-to-agent discovery and payments.
---

# ENS Agent Identity — names for machines

An ENS name is the natural identity primitive for an AI agent: human-readable,
owner-controlled, resolvable to endpoints and keys, and tradable. The standards
are live — ERC-8004 identity registries deployed January 2026 at the same
address across dozens of networks — but the binding between a *name* and an
*agent* has sharp edges.

## The three-layer identity stack

1. **ERC-8004 IdentityRegistry** — the agent NFT itself. The canonical
   mainnet registry is `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`. An agent
   is an ID in this registry.
2. **ENSIP-25 claim** — a text record on the ENS name binding name → agent:
   key `agent-registration[<ERC-7930 chain+address of the registry>][<agentId>]`,
   value `"1"`. Worked example for an agent with id 35459 on the canonical
   mainnet registry (the ERC-7930 blob is version + chain type + chain ref +
   address length + the 20 address bytes):
   `agent-registration[0x000100000101148004a169fb4a3325136eb29fa0ceb6d2e539a432][35459]` → `"1"`
   (the spec accepts any non-empty value; `"1"` is the conventional one).
   This record is what explorers (8004scan, ens8004) check to mark an agent
   **verified**.
3. **ENSIP-26 endpoints** — text records advertising how to reach the agent:
   `agent-endpoint[mcp]`, `agent-endpoint[a2a]`, `agent-endpoint[web]`, plus
   `agent-context`. (The `ai.agent = "true"` marker you'll see in the wild is
   an ecosystem convention, not part of the ENSIP.)

## The mistakes that make agents read as fake

- **Bind to the canonical registry, not an adapter.** The ENSIP-25 key encodes
  a registry address. If a platform minted your agent through a custodial
  adapter contract, the claim must still encode the *canonical* IdentityRegistry
  (where the agent NFT actually lives) — a claim encoding the adapter reads as
  `CLAIM MISSING` / unverified on every explorer, even though the binding
  "works" on the platform that wrote it.
- **The claim is usually the second signature, and users abandon it.** Typical
  launch flows are two transactions: (1) mint + bind, (2) write the text
  records. The wild is full of agents that were minted and bound but never
  claimed — functional, yet permanently "unverified" until someone sends the
  one missing record transaction. If you operate a launch flow, surface
  unfinished verification; if you're verifying an agent, distinguish "no
  agent" from "agent with an incomplete claim".
- **`agentWallet` being empty is normal for custodial agents.** Don't treat a
  zero agent-wallet as a defect — custodial/adapter-routed agents deliberately
  leave it unset.
- **Emoji and unicode names are valid agent names.** Any ENSIP-15-valid label
  can carry an agent. Reject-listing anything outside `[a-z0-9-]` breaks real
  names (see [names](https://namewhisper.ai/skills/names/SKILL.md)).
- **An agent's name can expire.** Agent identity inherits the full name
  lifecycle — an agent on a name in grace is an agent about to lose its
  identity. Watch expiry like you would a TLS cert
  ([lifecycle](https://namewhisper.ai/skills/lifecycle/SKILL.md)).

## Discovery and payments

- **A2A**: agents publish an Agent Card at
  `<endpoint>/.well-known/agent-card.json`; the ENS `agent-endpoint[a2a]`
  record is the pointer into that flow.
- **MCP**: `agent-endpoint[mcp]` advertises a Model Context Protocol endpoint —
  the agent's tool surface.
- **x402**: HTTP 402 pay-per-call, live in production, works agent-to-agent.
  Services advertise pricing at `/.well-known/x402`. Name Whisper's
  transaction-class tools are x402-gated; read tools are free.

One prerequisite that trips people: all of this requires a **name you already
own** — registered and resolvable — before any binding can be written.

## Acting on agent identity

Name Whisper runs a hosted-agent platform on this exact stack (mint,
bind, claim, endpoints — verifiable by any third-party explorer). MCP tools
(`https://namewhisper.ai/mcp`):

- `register_agent` / `provision_agent_identity` — mint + bind + claim,
  against the canonical registry.
- `get_agent_reputation` — verification state, including incomplete-claim
  detection.
- `search_agent_directory` — discover agents by capability.
- `launch_hosted_agent` — full hosted launch on a name you own.
- Human entry point: `https://namewhisper.ai/launch`.

Related: [names](https://namewhisper.ai/skills/names/SKILL.md) for text-record
mechanics. For the general agent-infra picture (ERC-8004 across chains, x402
SDKs), see also
[ethskills standards](https://ethskills.com/standards/SKILL.md). Index:
[https://namewhisper.ai/SKILL.md](https://namewhisper.ai/SKILL.md)
