# Public agents

These routes read the Agent Directory with no key. They are limited per IP, and `skip` goes up to 500. The base URL is `https://api.deside.io/api/v2/public/agents`.

Every answer comes in `{ "data": ... }`. Every error comes in `{ "error": { "code", "message" } }`.

## GET /

Returns agents in the directory, filtered and paged.

```bash
curl "https://api.deside.io/api/v2/public/agents?limit=1&live=mcp"
```

### Parameters

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 20 | 1 to 100. Out-of-range values are clamped. 24 at most with `sort=featured`. |
| `skip` | integer | 0 | Rows to skip. Negative values count as 0. 500 at most; 96 with `sort=featured`. Above that: `400`. |
| `q` | string | | 2 to 50 characters. Matches the start of the name, the start of the owner wallet, or a full agent wallet, registry entry id or EVM address. Outside 2 to 50 it is ignored. |
| `name` | string | | Same as `q`. |
| `skill` | string | | Up to 60 characters. Exact match on a declared skill, lowercase. |
| `category` | string | | One of the 12 categories below. An unknown value is ignored. |
| `service` | string | | `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. Agents that declare it. |
| `live` | string | | Comma list of `mcp`, `a2a`, `x402`. Agents with at least one of these live. |
| `registry` | string | | `mip14`, `8004solana`, `said`, `sati`, `sap`, `kamiyo` or `erc8004-base`. Aliases: `014`, `mip014`, `8004`, `8004base`, `base`. |
| `chain` | string | | `solana` or `evm`. Any other value is ignored. |
| `ownerWallet` | string | | Exact owner wallet, base58. For an EVM address, use `q`. |
| `agentWallet` | string | | Exact agent wallet, base58. For an EVM address, use `q`. |
| `coreAsset` | string | | Exact Metaplex Core asset, base58. |
| `collection` | string | | Agents carrying a badge of this collection, base58. |
| `collectionCase` | string | | `A`, `B`, `C` or `D`. |
| `connected` | boolean | | `true`: the owner proved the agent is theirs. `false`: not proven. Any other value is ignored. |
| `duplicates` | string | | `hide` keeps one agent per group of copies; `show` lists every copy. Without it, copies are hidden only when you filter by `q`, `skill`, `category`, `service`, `live` or `registry`. Any other value: `400`. |
| `sort` | string | | `name` for A to Z. `featured` for a curated selection. Anything else keeps the default order. |

Categories: `trading_bots`, `token_signals`, `token_risk`, `contract_security`, `defi_yield`, `prediction_markets`, `market_data`, `dev_tools`, `research`, `content_marketing`, `agent_infra`, `token_launch`. `other` means the agent was read and fits none; `null` means it has no category yet. Neither is a filter.

### Response

```json
{
  "data": [
    {
      "id": "bbddcb0c-074f-4874-9c48-3733013db7f7",
      "slug": "blinkcodes",
      "path": "/agents/blinkcodes",
      "name": "BlinkCodes",
      "avatar": {
        "url": "https://blinkcodes.com/web-app-manifest-512x512.png",
        "thumbUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/.../card.webp"
      },
      "chain": "evm",
      "category": "other",
      "ownerProven": false,
      "team": false,
      "handles": { "x": null, "github": null },
      "status": "responds",
      "liveKinds": ["mcp", "a2a", "x402"]
    }
  ],
  "page": { "total": 109734, "limit": 1, "skip": 0, "hasMore": true },
  "facets": { "live": { "mcp": 766, "a2a": 33, "x402": 31 } }
}
```

| Field | Description |
|---|---|
| `id` | Stable id of the agent in Deside. |
| `slug` | Short name, used in URLs. |
| `path` | The agent's page on deside.io. |
| `name` | Display name. |
| `avatar` | `url` and `thumbUrl` (our cached copy), or `null`. |
| `chain` | `solana` or `evm`. |
| `category` | One of the categories above, `other`, or `null`. |
| `ownerProven` | `true` when the owner proved the agent is theirs. Shown as connected on deside.io. |
| `team` | `true` when the account that owns the agent is a Verified team on Deside. `null` when it could not be read, which is not `false`. |
| `handles` | The agent's `x` and `github` handles, or `null`. |
| `status` | The state from our checks. See [Checks](../start/checks.md). |
| `liveKinds` | Protocols that answered our last check: `mcp`, `a2a`, `x402`. |
| `page.total` | Agents that match the filters. |
| `facets.live` | How many agents would match with each `live` value added to the other filters. `null` when it could not be counted. |

### Errors

| Status | `code` | When |
|---|---|---|
| 400 | `invalid_request` | A bad `service`, `live`, `registry`, `ownerWallet`, `agentWallet`, `coreAsset`, `collection`, `collectionCase` or `duplicates`, or `skip` above 500 (96 with `sort=featured`). |
| 429 | | More than 30 list requests a minute, or 500 a day, from your IP. Body: `{"error":"RATE_LIMITED"}`. |

## GET /stats

Returns the directory counts.

```bash
curl "https://api.deside.io/api/v2/public/agents/stats"
```

```json
{
  "data": {
    "listed": 109734,
    "byChain": { "evm": 94519, "solana": 15215 },
    "byCategory": { "trading_bots": 346, "market_data": 518, "other": 2625 },
    "byCategoryByChain": { "evm": { "trading_bots": 317 }, "solana": { "trading_bots": 29 } },
    "respondingAgents": 948,
    "respondingByKind": { "mcp": 799, "a2a": 148, "x402": 37 },
    "connected": 1,
    "endpoints": {
      "mcp": { "declared": 1525, "checked": 1449, "alive": 209 },
      "a2a": { "declared": 3337, "checked": 3226, "alive": 242 },
      "x402": { "declared": 307, "checked": 91, "alive": 26 }
    },
    "measuredAt": "2026-10-03T05:15:00.947Z"
  }
}
```

| Field | Description |
|---|---|
| `listed` | Agents in the directory. |
| `byChain` | `listed` by chain. |
| `byCategory`, `byCategoryByChain` | Agents with a category, by category and by chain. |
| `respondingAgents` | Agents with at least one protocol that answered our last check. |
| `respondingByKind` | The same, by protocol. An agent with two protocols counts in both. |
| `connected` | Agents whose owner proved they are theirs. |
| `endpoints` | By protocol: endpoints `declared` by agents, `checked` by us, and `alive` at the last check. |
| `measuredAt` | When the counts were measured. |

## GET /{ref}

Returns one agent. `{ref}` is the `id` or the `slug`.

```bash
curl "https://api.deside.io/api/v2/public/agents/blinkcodes"
```

An old slug answers `301` to the current one. A slug shared by more than one agent answers `409` with `candidates[]`, in the list shape.

```json
{
  "data": {
    "id": "bbddcb0c-074f-4874-9c48-3733013db7f7",
    "slug": "blinkcodes",
    "path": "/agents/blinkcodes",
    "name": "BlinkCodes",
    "avatar": {
      "url": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/bbddcb0c-074f-4874-9c48-3733013db7f7/a98378f311da123f-profile.webp",
      "thumbUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/bbddcb0c-074f-4874-9c48-3733013db7f7/a98378f311da123f-card.webp"
    },
    "chain": "evm",
    "category": "other",
    "ownerProven": false,
    "team": false,
    "handles": { "x": null, "github": null },
    "status": { "state": "responds", "since": null, "probedAt": "2026-10-02T05:15:01.027Z" },
    "liveKinds": ["mcp", "a2a", "x402"],
    "description": "Digital goods store selling gift card codes, service top-ups and travel eSIMs. Machine-buyable over x402 on Base; read-only catalog over MCP and A2A.",
    "wallets": { "owner": "0x4a2ebedb78028c05772787908ce504f221065954", "agent": null },
    "registries": ["erc8004-base"],
    "sources": [{ "registry": "erc8004-base", "entryId": "63619" }],
    "endpoints": [
      { "kind": "x402", "url": "https://blinkcodes.com/api/v1/buy", "evidence": "payment-offer", "version": "2", "latencyMs": null, "checkedAt": "2026-10-02T05:15:01.027Z" }
    ],
    "services": [
      { "kind": "x402", "url": "https://blinkcodes.com/api/v1/buy", "live": true, "checkedAt": "2026-10-02T05:15:01.027Z", "version": "2", "x402": { "state": "live", "tools": 1, "indexes": [{ "url": "https://blinkcodes.com/api/v1/buy", "state": "live", "tools": 1 }] } }
    ],
    "links": { "website": "https://blinkcodes.com/" },
    "skills": [],
    "updatedAt": "2026-10-03T12:08:19.254Z"
  }
}
```

It has the list fields, and these:

| Field | Description |
|---|---|
| `status` | `state`, `since` (when it entered that state) and `probedAt` (our last check). |
| `description` | What the agent says it does. |
| `wallets` | `owner` and `agent` wallets, or `null`. |
| `registries` | Registries that list the agent. See [Sources](sources.md). |
| `sources` | Each registry entry: `registry` and its `entryId`. |
| `ownerScore` | The owner wallet's score: `system`, `score`, `tier`. Only when two registries list the agent; otherwise `null`. |
| `endpoints[]` | Endpoints that answered our check: `kind`, `url`, `evidence`, `version`, `latencyMs`, `checkedAt`. |
| `services[]` | Everything the agent declares: `kind`, `url`, `live`, `checkedAt`, `version`. An x402 service also has `x402` with `state`, `tools` and `indexes[]`. |
| `links` | Declared links, by name. |
| `skills` | Declared skills. |
| `updatedAt` | When the entry last changed. |

### Errors

| Status | `code` | When |
|---|---|---|
| 400 | `invalid_ref` | `{ref}` empty or longer than 128 characters. |
| 404 | `not_found` | No agent matches. |
| 409 | `ambiguous_ref` | The slug matches more than one agent. Use an `id` from `candidates`. |
| 429 | | More than 60 requests a minute, or 500 a day, from your IP. Body: `{"error":"RATE_LIMITED"}`. |

## GET /{ref}/profile

Returns everything the agent page on deside.io shows: the agent, and what we read about it on-chain. Same `{ref}`, redirects and errors as above. An old slug keeps `/profile` in the redirect.

```bash
curl "https://api.deside.io/api/v2/public/agents/blinkcodes/profile"
```

It has the fields of [GET /{ref}](#get-ref), and these:

| Field | Description |
|---|---|
| `token` | The agent's token: `status`, `mint`, `name`, `symbol`, `image`, `decimals`, `supply`. `status` is `none` when there is none. |
| `holdings.owner`, `holdings.agent` | Per wallet: `status`, `wallet`, `refreshedAt`, `totalUsd`, `assetCount`, `assets[]` and `nftCount`, or `null`. |
| `onchain` | `identity`, `owner`, `authority`, `agentWallet` and `explorer` link. |
| `onSolana` | Metaplex agents only: `collection` and `updateAuthority`. Otherwise `null`. |
| `reputation.owner`, `reputation.agent` | Per wallet: `wallet`, `system`, `score`, `tier`, `badges` and `resolvedAt`, or `null`. |
| `relations` | Tools, tokens and sites linked to this agent. See [Relations](relations.md). |
| `claim` | How the owner can prove the agent is theirs: `state` and `vias`, as in [GET /public/claim](#get-public-claim-type-id). |
| `capabilities`, `domains` | Declared in the registries. |
| `mcpSessionActive` | `true` when the agent is signed in to the Deside MCP now. |

## GET /public/claim/{type}/{id}

This route is under v1: `https://api.deside.io/api/v1/public/claim/...`.

Returns how the owner of an agent or a token can prove it is theirs.

| Name | Description |
|---|---|
| `{type}` | `agent` or `token`. |
| `{id}` | Agent: `catalogId` or slug. Token: `solana:<mint>` or `evm:<address>`. 3 to 120 characters: letters, digits, `:`, `_`, `-`. |

```bash
curl "https://api.deside.io/api/v1/public/claim/agent/blinkcodes"
```

```json
{
  "objectType": "agent",
  "objectId": "bbddcb0c-074f-4874-9c48-3733013db7f7",
  "state": "unclaimed",
  "vias": [{ "via": "wallet", "steps": ["link-wallet"], "value": null }]
}
```

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"invalid_object"}` | Bad `{type}` or `{id}`. A slug with a dot fails here. |
| 404 | `{"error":"not_found"}` | Nothing matches. |
| 503 | `{"error":"claim_unavailable"}` | The claim service is down. Retry later. |
