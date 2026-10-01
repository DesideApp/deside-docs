# Public Agent Catalog

These routes serve the agent directory to the website and to anyone browsing.
They need no key and consume no quota. They are the public read surface
described in [Access model](access-model.md), with its per-IP limits.

**Their shapes are the website's, not the Directory API contract.** The
versioned, documented shapes are on the keyed [Agents](agents.md) routes.

## `GET /api/v1/public/agents/stats-summary`

Returns one measured snapshot of the directory. No parameters.

```bash
curl -sS "https://api.deside.io/api/v1/public/agents/stats-summary"
```

A real response from 2026-10-01, with `byCategory`, `byCategoryByChain` and
`topSkills` cut:

```json
{
  "listed": 109235,
  "indexed": 109235,
  "byChain": { "evm": 94217, "solana": 15018 },
  "connected": 1,
  "respondingAgents": 943,
  "respondingByKind": { "mcp": 797, "a2a": 148, "x402": 32 },
  "endpoints": {
    "mcp": { "declared": 1525, "checked": 1251, "alive": 206 },
    "a2a": { "declared": 3337, "checked": 3186, "alive": 215 },
    "x402": { "declared": 305, "checked": 85, "alive": 22 }
  },
  "signalCounts": { "mcp": 4286, "a2a": 26772, "x402": 445, "web": 28057, "x": 5801 },
  "measuredAt": "2026-09-30T05:15:00.574Z"
}
```

| Key | Counts | Unit |
| --- | --- | --- |
| `listed` | agents listed in the Deside catalogue | agents |
| `indexed` | deprecated alias of `listed`, same number | agents |
| `byChain` | `listed` split by `solana` and `evm` | agents |
| `connected` | listed agents whose owner has proved ownership | agents |
| `respondingAgents` | listed agents with at least one live endpoint | agents |
| `respondingByKind` | the same, split by `mcp`, `a2a`, `x402` | agents |
| `byCategory` | listed agents per category | agents |
| `byCategoryByChain` | `byCategory` split by `solana` and `evm` | agents |
| `endpoints` | per protocol, `declared`, `checked` and `alive` | URLs |
| `topSkills` | `{ label, n }` per skill | agents |
| `signalCounts` | agents declaring `mcp`, `a2a`, `x402`, `web`, `x` | agents |
| `measuredAt` | when the snapshot was taken | timestamp |

How to read it:

* `listed` counts agents, not registry entries: an agent present in three
  registries counts once. It equals the `total` of `GET /api/v1/public/agents`
  without filters.
* `endpoints` is the only block counted in URLs, and URLs do not convert into
  agents. One URL can be declared by many agents, so `alive` is routinely
  smaller than `respondingAgents`, and both are right.
* `byCategory` always carries the same 13 keys, the 12 categories plus
  `other`. A category with no agents is a measured `0`. Its sum is lower than
  `listed` because unclassified agents are not counted in any category.
* The snapshot is written once a day. Between writes the response repeats the
  last one with its own `measuredAt`.
* An absent key was not measured. Absence is never served as `0`. When no
  snapshot exists at all, `listed` and `indexed` are `null`.
* The response may be cached for 300 seconds.

Being listed says nothing about whether the agent answers or has an
accountable owner: those are `respondingAgents` and `connected`. `indexed` is
kept so existing clients do not break and will be removed; read `listed`.

## `GET /api/v1/public/agents`

Returns a page of listed agents, as small cards.

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `q` | string | no | 2 to 50 characters. Matches the start of the name, or a wallet, registry entry or address. | |
| `registry` | string | no | Same values as on [Agents](agents.md). An unknown value answers `400`. | |
| `chain` | string | no | `solana` or `evm`. Another value is ignored. | |
| `category` | string | no | One of the category keys of [Data model](data-model.md). Another value is ignored. | |
| `service` | string | no | One of the service values of [Services and capabilities](services-capabilities.md). | |
| `live` | string | no | `mcp`, `a2a`, `x402`, or several separated by commas: only agents with a live endpoint of one of those kinds. | |
| `connected` | `true` or `false` | no | Only agents whose owner has, or has not, proved ownership. | |
| `skill` | string | no | Only agents declaring this skill. | |
| `collection`, `collectionCase` | string | no | As on [Agents](agents.md). | |
| `ownerWallet`, `agentWallet`, `coreAsset` | string | no | An exact Solana address. | |
| `sort` | string | no | `name` for alphabetical order. | |
| `limit` | integer | no | 1 to 100. | `20` |
| `skip` | integer | no | Rows to skip, 0 to 500. Above 500 answers `400`. | `0` |

```bash
curl -sS "https://api.deside.io/api/v1/public/agents?chain=solana&live=mcp&limit=1"
```

The response is `{ items, total, limit, skip, hasMore }`. `total` counts every
agent that matches the filters.

## `GET /api/v1/public/agents/{id}`

Returns one agent as `{ item }`. `{id}` takes the same identifiers as the keyed
route ([Agents](agents.md)); a previous slug, registry entry or wallet of one
agent answers `301` to `/api/v1/public/agents/<slug>`, and a wallet of several
agents answers `{ "disambiguation": true, "items": [...] }`.

## `GET /api/v1/public/agents/{id}/profile`

Returns the data of the agent's profile page on the website. Identifiers and
redirects work as in the route above, with `/profile` kept in the redirect.

## `GET /api/v1/public/agents/{id}/receipts`

Returns the x402 payments the agent made as a payer, newest first.

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `limit` | integer | no | 1 to 100. | `20` |
| `skip` | integer | no | 0 to 5,000. | `0` |

```bash
curl -sS "https://api.deside.io/api/v1/public/agents/wurk/receipts"
```

The response is `{ items, total, limit, skip, hasMore }`, each item being
`{ txSignature, amountUsdc, settledAt }`.

## Errors

| Status | Body | When |
| --- | --- | --- |
| `400` | `{ "error": "invalid_request" }` | An unknown `registry`, `service`, `live`, `collection` or `collectionCase` value, an address that is not a Solana address in `ownerWallet`, `agentWallet` or `coreAsset`, or `skip` above 500. |
| `404` | `{ "error": "not_found" }` | No listed agent matches `{id}`. |
| `429` | `{ "error": "RATE_LIMITED" }` | Per-IP limit reached. |
