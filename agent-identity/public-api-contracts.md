# Public API Contracts

This page describes the public agent identity contracts exposed by Deside today.

It covers the public product surface, not the paid Directory API product.

## Public Endpoints

The public agent directory lives under:

```bash
GET https://api.deside.io/api/v1/public/agents
GET https://api.deside.io/api/v1/public/agents/:ref
GET https://api.deside.io/api/v1/public/agents/:ref/profile
```

Examples:

```bash
curl https://api.deside.io/api/v1/public/agents
curl https://api.deside.io/api/v1/public/agents/xona-agent-ssz7/profile
```

These endpoints are public, cacheable product reads.

They are rate-limited separately from private session APIs.

## Response Envelopes

The list endpoint returns:

```json
{
  "items": [],
  "total": 0,
  "limit": 20,
  "skip": 0,
  "hasMore": false
}
```

The single-agent endpoint returns one of:

```json
{ "item": {} }
```

```json
{ "disambiguation": true, "items": [] }
```

or redirects to the canonical slug when the reference is historical or
non-canonical.

The profile endpoint returns the public user/agent profile contract for the
resolved agent.

## Public Agent Reference

`:ref` is not only a database id.

The backend resolves public agent references in this order:

1. exact `catalogId`
2. current slug
3. historical slug, redirected to the current slug
4. source entry id, such as SATI mint or Metaplex Core Asset
5. owner wallet or catalog wallet

When a wallet resolves to exactly one public agent, the backend redirects to the
canonical slug when available.

When a wallet resolves to multiple public agents, the backend returns a
disambiguation response.

That disambiguation is a product truth:

- one owner wallet can control several agents
- Deside must not pick one silently
- shared owner wallet is not enough to merge source entries

## Public Directory Item Shapes

Since 2026-08-10 the public surface serves two different item shapes on
purpose:

- the public LIST (`GET /api/v1/public/agents`) returns a minimal card per
  agent
- the public agent DETAIL (`GET /api/v1/public/agents/:ref` and
  `/:ref/profile`) returns the full projection for that one agent

Bulk access to rich agent data is the API-key Directory API
(`GET /api/v1/directory/agents`, cursor pagination), not the public list.
Note that the Directory API exposes its own versioned contract
(`DirectoryAgentListItemV1` with `displayName`, registries, services, and
convergence metadata) rather than this raw projection shape; see the
Directory API section for that contract.

### List item (card)

```json
{
  "catalogId": "agent-catalog-id",
  "slug": "agent-slug",
  "canonicalPath": "/agents/agent-slug",
  "name": "Agent name",
  "avatarThumbUrl": "cached-card-avatar-or-null",
  "avatarOriginalUrl": "original-avatar-url-or-null",
  "avatar": "avatar-url-or-null",
  "category": "category-or-null",
  "isConnected": false,
  "services": [{ "kind": "mcp", "checked": true }],
  "curationPublic": {
    "verified": false,
    "humanUsable": null,
    "descCategory": "category-or-null",
    "state": "state-or-null"
  }
}
```

Notes:

- `wallet` is present only when `isConnected` is `true` (it powers the chat
  deep link)
- list `services` carry `kind` and `checked` only; resolved URLs are not
  part of the list
- `skip` beyond the pagination cap returns `400 invalid_request` instead of
  clamping silently

### Detail item (single agent)

The detail endpoints return the full backend-owned projection for one
agent, including `ownerWallet`, `agentWallet`, `sourceEntries`,
`description`, `website`, `serviceSignals`, `ownerScore`,
`registryPresence`, `mergeEvidence`, avatar cache fields, `capabilities`,
`channels`, `receipts` and timestamps.

Not every field is present for every agent.

The contract is that these fields are backend-owned directory projection fields.

The frontend should not reconstruct them from raw registry payloads.

## Profile Response

The profile endpoint returns the public user/agent profile contract for the
resolved agent.

The important top-level branches are:

- `visibleProfile`
- `userProfile`
- `agentProfile`
- `walletReputation`, when public wallet reputation is available
- `identity.slug`
- `identity.canonicalPath`
- `identity.mergeEvidence`

`agentProfile` is the public agent branch.

`walletReputation` is wallet-level reputation.

It is not the same thing as registry identity and must not be used as merge
evidence.

## Source Identifiers

Deside preserves source-native identifiers.

| Source | Public source key | Identity unit shown by Deside |
| --- | --- | --- |
| Metaplex Agent Registry | `mip14` | Core Asset |
| 8004-Solana | `8004solana` | source-native agent id |
| SATI | `sati` | SATI mint |
| SAID | `said` | SAID source entry |
| SAP | `sap` | SAP source entry |

The owner wallet is an ownership or authority relationship.

It is not a unique agent id.

## Scores, Tokens, And Holdings

Some profile sections are computed projections.

They do not participate in identity resolution.

### Owner Score

`ownerScore` is exposed only when the resolved public agent has presence in two
or more registries.

Today this is FairScale wallet reputation at the product layer.

It is not a registry merge rule.

### Agent Token

Agent token status is resolved by backend jobs from allowed source evidence.

Deside does not infer an agent token from payment assets, escrow assets, generic
mints, or wallet holdings.

### Wallet Holdings

Wallet holdings are backend snapshots.

They can apply to the owner wallet and, when canonical, the Metaplex
`agentWallet`.

They are not live frontend chain reads and they do not prove agent-token
identity.

## Directory API Boundary

The public endpoints above are open product reads.

The paid Directory API has a separate contract surface for API keys, billing,
rate limits, bulk export, and webhooks.

Do not treat this public contract as the complete commercial Directory API.
