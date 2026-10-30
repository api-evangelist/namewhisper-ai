# NameWhisper

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Name Whisper is an ENS (Ethereum Name Service) intelligence layer built for AI agents: natural-language search across 3.5M indexed .eth names, three-track valuation, live market data, wallet portfolios, and non-custodial transaction building for registration, renewal, Seaport listings and offers, resolver records, NameWrapper operations and ERC-8004 / ENSIP-25 agent identity. Everything is exposed as a public, anonymous remote MCP server at https://namewhisper.ai/mcp (44 tools, streamable-http, on the MCP Registry as ai.namewhisper/ens-tools) and an A2A 0.3.0 agent at https://namewhisper.ai/a2a, described by an agent card, an MCP server card, RFC 8414 / 9728 auth metadata, an RFC 9727 api-catalog, an llms.txt and provider-published Agent Skills. Identity is opt-in ERC-8128 signed requests; x402 and MPP payments are wired but dormant. The service never holds keys - every mutating tool returns unsigned calldata for the caller's wallet. No OpenAPI is published for the REST twins.

**Areas:** ENS · Ethereum · Web3 · Domain Names · AI Agents · MCP · A2A · Valuation · NFT Marketplace · Agent Identity · Blockchain

## NameWhisper on APIs.io

Everything profiled for NameWhisper is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/namewhisper-ai/](https://apis.io/providers/namewhisper-ai/)** — or go straight to one of the APIs or properties below.

### APIs (3)

- [Name Whisper MCP Server](https://apis.io/apis/namewhisper-ai/namewhisper-mcp-server/) — Public, anonymous remote MCP server (streamable-http, session header mcp-session-id) exposing 44 tools over ENS names - 19 free read tools for search,…
- [Name Whisper A2A Agent](https://apis.io/apis/namewhisper-ai/namewhisper-a2a-agent/) — The agent-to-agent surface: an anonymous A2A 0.3.0 JSON-RPC endpoint at https://namewhisper.ai/a2a whose agent card (served at both…
- [Name Whisper REST API](https://apis.io/apis/namewhisper-ai/namewhisper-rest-api/) — The REST twins of the MCP tools. The docs state "All MCP tools have equivalent REST endpoints under /api/*. The JSON shapes are the same" and the guide says…

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [Name Whisper - ENS Intelligence for AI Agents](https://apis.io/servers/namewhisper-ai/nw-terminal/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Namewhisper Ai Agentic Access](https://apis.io/agentic-access/namewhisper-ai/namewhisper-ai-agentic-access/)

#### Security (3)

Authentication, trust centre, and vulnerability disclosure.

- [Namewhisper Ai Authentication](https://apis.io/security/namewhisper-ai/namewhisper-ai-authentication/)
- [Namewhisper Ai Domain Security](https://apis.io/security/namewhisper-ai/namewhisper-ai-domain-security/)
- [Namewhisper Ai Vulnerability Disclosure](https://apis.io/security/namewhisper-ai/namewhisper-ai-vulnerability-disclosure/)

#### OAuth scopes (1)

The permissions the API asks for.

- [Namewhisper Ai Scopes](https://apis.io/scopes/namewhisper-ai/namewhisper-ai-scopes/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Namewhisper Ai Plans Pricing](https://apis.io/plans/namewhisper-ai/namewhisper-ai-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Namewhisper Ai Rate Limits](https://apis.io/rate-limits/namewhisper-ai/namewhisper-ai-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://namewhisper.ai/)
- [Documentation](https://namewhisper.ai/docs)
- [Getting started](https://namewhisper.ai/guide)
- [API reference](https://namewhisper.ai/llms-full.txt)
- [Pricing](https://namewhisper.ai/guide#fees)
- [Support](https://x.com/namewhisper_ai)
- [Security](https://namewhisper.ai/.well-known/security.txt)
- [Terms of service](https://namewhisper.ai/terms)
- [Privacy policy](https://namewhisper.ai/privacy)
- [GitHub](https://github.com/eggybug42069)

## API Evangelist rating

**Kin Score 41.4** — developing · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 80.0 |
| Contract quality | 0.0 |
| Contract governance | 18.2 |
| Operational transparency | 50.0 |
| Developer ergonomics | 52.4 |
| Access clarity | 52.6 |

**Agent readiness 41.6** — agent-ready (gated down from agent-native)

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | — |
| Agentic Access | yes |
| Reversibility Documented | — |
| MCP Server | documented (partial credit) |
| Auth Clarity | served (partial credit) |
| Idempotency | documented (partial credit) |
| Error Semantics | documented (partial credit) |
| OpenAPI Examples | — |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | — |
| Agent Skills | yes |
| Well Known Catalog | yes |
| Consent Identity | — |
| Agent Card | conformant |
| Dry Run Mode | — |
| Delegated Identity | — |
| Protected Resource Metadata | verified |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | na |
| Operation Distinctiveness | na |

**Regulatory alignment 40.2** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/namewhisper-ai/](https://apis.io/providers/namewhisper-ai/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for NameWhisper: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/namewhisper-ai/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
