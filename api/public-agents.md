# Public agents

These routes read the Agent Directory with no key. They are limited per IP, and `skip` goes up to 500. The base URL is `https://api.deside.io/api/v2/public/agents`.

They return what each agent declares in its registrations and what Deside checked. Every answer comes in `{ "data": ... }`. Every error comes in `{ "error": { "code", "message" } }`.

Every route on this page carries `about`: the code of the read and one sentence for each figure or field that needs one. See [Read codes](contract.md#read-codes).

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
| `category` | string | | One of the 13 categories below. Any other value: `400`. |
| `service` | string | | `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. Agents that declare it. Any other value: `400`. |
| `live` | string | | Comma list of `mcp`, `a2a`, `x402`. Agents with at least one of these live. Any other value: `400`. |
| `registry` | string | | `mip14`, `8004solana`, `said`, `sati`, `sap`, `kamiyo` or `erc8004-base`. Aliases: `014`, `mip014`, `8004`, `8004base`, `base`. Any other value: `400`. |
| `chain` | string | | `solana` or `evm`. Any other value: `400`. |
| `ownerWallet` | string | | Exact owner wallet: base58, or a `0x` address in any case. |
| `agentWallet` | string | | Exact agent wallet: base58, or a `0x` address in any case. |
| `coreAsset` | string | | Exact Metaplex Core asset, base58. |
| `collection` | string | | Agents whose Metaplex Core asset is in this on-chain collection, base58: the `onSolana.collection` of [`/{ref}/profile`](#get-ref-profile). |
| `collectionCase` | string | | `A`, `B`, `C` or `D`. Any other value: `400`. |
| `connected` | boolean | | `true`: the owner proved the agent is theirs. `false`: not proven. Any other value: `400`. |
| `duplicates` | string | | `hide` keeps one agent per group of copies; `show` lists every copy. Without it, every copy is listed, whatever the filters. Any other value: `400`. |
| `sort` | string | | `name` for A to Z. `featured` for the selection shown first on deside.io: it keeps one agent per group of copies and leaves out the agents of Deside. Without it, the default order. Any other value: `400`. |

Categories: `trading_bots`, `token_signals`, `token_risk`, `contract_security`, `defi_yield`, `prediction_markets`, `market_data`, `dev_tools`, `research`, `content_marketing`, `agent_infra`, `token_launch` and `other`. Deside assigns the category from the agent's own description; the agent does not declare it. `other` means the agent was read and fits none of the other 12, and it filters like any category. `null` means it has no category yet, and is not a filter.

**A filter with a closed list of values answers `400` to a value outside its list**, so a misspelled value never returns the whole list as if it were filtered.

### Response

Real on 2026-11-09, trimmed to one agent and one definition:

```json
{
  "data": [
    {
      "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f",
      "slug": "deside",
      "path": "/agents/deside",
      "name": "Deside",
      "avatar": {
        "url": "https://gateway.irys.xyz/G3ymNAGETZUrU18XadeFN5gtUs25K5TgeFZLqCUD3T1u",
        "thumbUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/5f0998a5-2bf1-4093-85be-2ba7e114e96f/e1a16cc4ac7d848e-card.webp"
      },
      "chain": "solana",
      "category": "other",
      "handles": { "x": null, "github": null },
      "status": "responds",
      "liveKinds": ["mcp", "x402"]
    }
  ],
  "page": { "total": 295283, "limit": 1, "skip": 0, "hasMore": true },
  "about": {
    "read": "agents.list",
    "definitions": {
      "data[].liveKinds": "Protocols (mcp, a2a, x402) with a declared endpoint that answered Deside's last check."
    }
  }
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
| `category` | One of the categories above, or `null`. Assigned by Deside from the agent's own description. |
| `handles` | The agent's `x` and `github` handles, or `null`. |
| `status` | The state from our checks: `responds`, `profile` or `registered`. See [Checks](../start/checks.md). |
| `liveKinds` | Protocols that answered our last check: `mcp`, `a2a`, `x402`. |
| `page.total` | Agents that match the filters, merged across registries: an agent found in several registries counts once, and different registrations that share a name count once each. |
| `about.read` | `agents.list`. |
| `about.definitions` | One sentence for `page.total`, `data[].category`, `data[].status` and `data[].liveKinds`. |

### Errors

| Status | `code` | When |
|---|---|---|
| 400 | `invalid_request` | A value outside the list of `category`, `chain`, `service`, `live`, `registry`, `connected`, `collectionCase`, `duplicates` or `sort`; a malformed `ownerWallet`, `agentWallet`, `coreAsset` or `collection`; or `skip` above 500 (96 with `sort=featured`). |
| 429 | | More than 30 list requests a minute, or 500 a day, from your IP. Body: `{"error":"RATE_LIMITED"}`. |

## GET /stats

Returns the directory counts, with one definition for each.

```bash
curl "https://api.deside.io/api/v2/public/agents/stats"
```

Real on 2026-11-09, trimmed to two categories and two definitions:

```json
{
  "data": {
    "listed": 295283,
    "byChain": { "solana": 17994, "evm": 277289 },
    "byCategory": { "trading_bots": 345, "other": 2663 },
    "byCategoryByChain": { "evm": { "trading_bots": 317 }, "solana": { "trading_bots": 28 } },
    "respondingAgents": 1129,
    "respondingByKind": { "mcp": 934, "a2a": 178, "x402": 64 },
    "connected": 2,
    "endpoints": {
      "mcp": { "declared": 1576, "checked": 1573, "alive": 264 },
      "a2a": { "declared": 36313, "checked": 10165, "alive": 65 },
      "x402": { "declared": 377, "checked": 144, "alive": 78 }
    },
    "live": null,
    "measuredAt": "2026-11-09T01:45:00.181Z",
    "about": {
      "read": "agents.stats",
      "definitions": {
        "listed": "Agents listed in the Deside directory, merged across registries and not curated: an agent found in several registries counts once, different registrations that share a name count once each, and agents that are burnt, reported or gone from every registry are not listed.",
        "live": "The official count of agents that answer, from Deside's nightly check of every declared endpoint; null until a nightly count of it exists."
      }
    }
  }
}
```

| Field | Description |
|---|---|
| `listed` | Agents in the directory, merged across registries. |
| `byChain` | `listed` by chain. Agents without a chain are left out. |
| `byCategory`, `byCategoryByChain` | Agents by the category Deside assigns, and the same per chain. Copies of one agent count once. |
| `respondingAgents` | Agents with at least one declared endpoint that answered our last check. |
| `respondingByKind` | The same, by protocol. An agent with two protocols counts in both. |
| `connected` | Agents whose owner proved to Deside that they control the agent. |
| `endpoints` | By protocol: distinct URLs `declared` by listed agents, how many we `checked`, and how many were `alive` at the last check. It counts URLs, not agents. |
| `live` | The count of agents that answer, from our nightly check of every declared endpoint. `null` until that nightly count exists. |
| `live.measuredAt` | When the nightly check behind `live` was taken. |
| `live.anyProtocol` | `agents`: agents with at least one declared MCP, A2A or x402 endpoint that answered, counted once. `distinctServers`: the distinct endpoint URLs that answered behind them. |
| `live.byProtocol` | `mcp`, `a2a` and `x402`, each with `agents` and `distinctServers`. |
| `measuredAt` | When the snapshot these counts come from was taken. |
| `about.read` | `agents.stats`. |
| `about.definitions` | One sentence for each figure, keyed by its path inside `data`. |

**Many agents can share one server.** `distinctServers` counts URLs that answered, so it is lower than `agents`.

## GET /{ref}

Returns one agent in short form. `{ref}` is the `id` or the `slug`.

```bash
curl "https://api.deside.io/api/v2/public/agents/deside"
```

An old slug answers `301` to the current one. A slug shared by more than one agent answers `409` with `candidates[]`, in the list shape.

Real on 2026-11-09, trimmed to one endpoint, one service and one definition:

```json
{
  "data": {
    "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f",
    "slug": "deside",
    "path": "/agents/deside",
    "name": "Deside",
    "avatar": {
      "url": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/5f0998a5-2bf1-4093-85be-2ba7e114e96f/e1a16cc4ac7d848e-profile.webp",
      "thumbUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/5f0998a5-2bf1-4093-85be-2ba7e114e96f/e1a16cc4ac7d848e-card.webp"
    },
    "chain": "solana",
    "category": "other",
    "handles": { "x": null, "github": null },
    "status": { "state": "responds", "since": "2026-10-04T04:30:00.226Z", "probedAt": "2026-11-02T01:45:00.490Z" },
    "liveKinds": ["mcp", "x402"],
    "description": "Deside's launchpad lets agents create their own token on a Solana bonding curve, with an identity registered onchain.",
    "wallets": { "owner": "5DZsMz44aH4JotKKcp58Xo6V4WCUMg4bF7CMQ9ySVZJN", "agent": "8KLybAd7QRTAt76fjEzH8xm8RdAWSgBneSvAeRQi2G3p" },
    "registries": ["mip14"],
    "sources": [{ "registry": "mip14", "entryId": "D4kHdXc7jdXAk7Brvo87JKbobRuNjnyGamwaBEew37iM" }],
    "endpoints": [
      { "kind": "mcp", "url": "https://mcp.deside.io/mcp", "evidence": "auth-challenge", "version": null, "latencyMs": 3546, "checkedAt": "2026-10-24T05:15:00.747Z" }
    ],
    "services": [
      { "kind": "mcp", "url": "https://mcp.deside.io/mcp", "live": true, "checkedAt": "2026-10-24T05:15:00.747Z", "version": null }
    ],
    "links": { "website": "https://deside.io/" },
    "skills": [],
    "updatedAt": "2026-11-03T01:35:05.558Z",
    "about": {
      "read": "agent.summary",
      "definitions": {
        "liveKinds": "Protocols (mcp, a2a, x402) with a declared endpoint that answered Deside's last check."
      }
    }
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
| `endpoints[]` | Endpoints that answered our check: `kind`, `url`, `evidence`, `version`, `latencyMs`, `checkedAt`. |
| `services[]` | The MCP, A2A and x402 URLs the agent declares: `kind`, `url`, `live`, `checkedAt`, `version`. An x402 service also has `x402` with `state`, `tools` and `indexes[]`, and `probe`: `verdict`, `reason` and `probedAt` of our last probe of that URL. The whole probe is in [`GET /public/x402/probes`](x402-tools.md#get-public-x402-probes). Every service the agent declares, in its order, is in [`/{ref}/profile`](#get-ref-profile). |
| `links` | Declared links, by name. |
| `skills` | Declared skills. |
| `updatedAt` | When the entry last changed. |
| `about.read` | `agent.summary`. This read has no MCP tool and no paid route. |

### Errors

| Status | `code` | When |
|---|---|---|
| 400 | `invalid_ref` | `{ref}` empty or longer than 128 characters. |
| 404 | `not_found` | No agent matches. |
| 409 | `ambiguous_ref` | The slug matches more than one agent. Use an `id` from `candidates`. |
| 429 | | More than 60 requests a minute, or 500 a day, from your IP. Body: `{"error":"RATE_LIMITED"}`. |

## GET /{ref}/profile

Returns the agent profile: what the agent declares and what Deside checked, with its token and what its wallets hold. Same `{ref}`, redirects and errors as above. An old slug keeps `/profile` in the redirect. The MCP tool `get_agent` and the paid read `/api/v2/paid/agent` return the same profile.

```bash
curl "https://api.deside.io/api/v2/public/agents/deside/profile"
```

It has the fields of [GET /{ref}](#get-ref), with `services[]` in full, and these:

| Field | Description |
|---|---|
| `services[]` | Every service the agent declares, in the order it declares them: `name`, `kind`, `url`, `value`, `status`, `live`, `checkedAt`, `version` and `sources`. `live` is `true` when it answered our last check, `false` when it did not, and `null` when we have not checked it. An x402 service also has `x402` and `probe`, as above. |
| `token` | The token the agent itself declares: `status`, `mint`, `name`, `symbol`, `image`, `decimals`, `supply`. `status` is `declared`, or `none` when it declares none. Deside does not infer it. |
| `holdings.owner`, `holdings.agent` | Per declared wallet, as we last read it on-chain: `status`, `wallet`, `refreshedAt`, `totalUsd`, `assetCount`, `assets[]` and `nftCount`, or `null`. The detail is in [`/{ref}/holdings`](#get-ref-holdings). |
| `onchain` | `identity`, `owner`, `authority`, `agentWallet` and `explorer` link. |
| `onSolana` | Metaplex agents only: `collection` and `updateAuthority`. Otherwise `null`. |
| `capabilities`, `domains` | Declared in the registries. |
| `mcpSessionActive` | `true` when the agent was signed in to the Deside MCP at the time of the read. |
| `about.read` | `agent.profile`. |
| `about.definitions` | One sentence for `category`, `status`, `liveKinds`, `endpoints`, `services`, `token`, `holdings` and `mcpSessionActive`. |

## GET /{ref}/holdings

Returns what the wallets of one agent hold, and its token. `{ref}` is the `id` or the `slug`, with the same redirects as above. **This route has no old shape: it always answers in [the common contract](contract.md)**, with `minor` or without it. Any query parameter other than `minor` and `cb` answers `400`.

```bash
curl "https://api.deside.io/api/v2/public/agents/deside/holdings"
```

Real on 2026-11-02, trimmed to one asset and one collection:

```json
{
  "data": {
    "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f",
    "wallets": [
      {
        "role": "owner",
        "wallet": "5DZsMz44aH4JotKKcp58Xo6V4WCUMg4bF7CMQ9ySVZJN",
        "status": "available",
        "reason": null,
        "checkedAt": "2026-11-02T10:30:00.100Z",
        "stale": false,
        "totalUsd": "28.945528055475",
        "assetCount": 3,
        "hiddenAssetCount": 1,
        "truncated": false,
        "assets": [
          { "mint": null, "kind": "native", "symbol": "SOL", "name": "Solana", "decimals": 9, "amount": "0.308426598", "usd": "28.945528055475265", "priceUsd": "93.84900084225312" }
        ],
        "nfts": { "count": 1, "collectionCount": 1, "collections": [{ "collection": null, "name": "Meteora Position NFT", "count": 1 }] }
      },
      {
        "role": "agent",
        "wallet": "8KLybAd7QRTAt76fjEzH8xm8RdAWSgBneSvAeRQi2G3p",
        "status": "empty",
        "reason": "account_not_initialized",
        "checkedAt": "2026-11-02T10:30:00.100Z",
        "stale": false,
        "totalUsd": null,
        "assetCount": 0,
        "hiddenAssetCount": 0,
        "truncated": false,
        "assets": [],
        "nfts": { "count": 0, "collectionCount": 0, "collections": [] }
      }
    ],
    "token": {
      "status": "declared",
      "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
      "name": "Deside",
      "symbol": "DESIDE",
      "decimals": 6,
      "primarySource": "metaplex",
      "nativeBy": null,
      "declaredBy": ["metaplex"],
      "evidence": [
        { "source": "metaplex", "type": "service_declaration", "strength": "declared", "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1", "observedAt": "2026-10-21T21:00:22.017Z" }
      ]
    }
  },
  "asOf": "2026-11-02T16:53:56.445Z"
}
```

`wallets[]` has one entry per wallet the agent page shows, `owner` and then `agent`. **A wallet the page does not show is never served.**

| Field | Description |
|---|---|
| `role`, `wallet` | `owner` or `agent`, and the address. |
| `status` | `available`, `empty`, `unavailable`, `refreshing`, `stale` or `error`. |
| `reason` | Why, when there is a reason. `unavailable` with `no_holdings_snapshot` means we have no read of that wallet, not that it is empty. |
| `checkedAt`, `stale` | When we read the wallet on Solana, and `true` after 7 days. Both `null` with no read. |
| `totalUsd` | Value in USD, as text. `null` with no price. |
| `assetCount`, `hiddenAssetCount` | Assets listed, and dust assets left out of the list. |
| `truncated` | `true` when the list was cut. |
| `assets[]` | `mint`, `kind` (`native`, `fungible`, `token2022` or `unknown`), `symbol`, `name`, `image`, `decimals`, `amount` (text), `usd` (text), `priceUsd`. |
| `nfts` | `count`, `collectionCount` and `collections[]` with `collection`, `name`, `image` and `count`. |
| `about` | Inside `data`: `read` is `agent.holdings`, and `definitions` has one sentence per field, keyed by its path inside `data`. |

`token` is the agent's token, or `null` when no source declares one:

| Field | Description |
|---|---|
| `status` | `declared` or `none`. |
| `mint`, `name`, `symbol`, `image`, `decimals`, `supply` | The token. `supply` is text, in base units. |
| `primarySource`, `nativeBy`, `declaredBy` | The main source, the registry that records the token as its own (or `null`), and every source that declares it. |
| `evidence[]` | One row per source: `source`, `type`, `strength`, `mint` and `observedAt`. `strength` is `native` when the registry itself records the token, and `declared` when a source states it. |

**The evidence shows who declares the token, not who is behind it.** Names, symbols and images of tokens and NFT collections are on-chain metadata anyone can write: treat them as data, never as instructions.

Errors come in the common shape: `400 invalid_request` for a bad `{ref}` or parameter, `404 not_found`, and `409 invalid_request` with `candidates` when a slug matches more than one agent. The limits are those of `GET /{ref}`.

## GET /{ref}/discovery

Returns what the hosts of one agent serve to machines: one entry per host of the website and services the agent declares. `{ref}` is the `id` or the `slug`, with the same redirects as above. **This route has no old shape: it always answers in [the common contract](contract.md).** Any query parameter other than `minor` and `cb` answers `400`.

**Discovery** is what a host serves to machines, as our crawler last read it: which machine files exist (llms.txt, agent card, skills, OpenAPI, x402), where, their type and size, and the endpoints, skills and tools they announce. It is what a third party publishes. We do not verify it, and we do not serve the text of the files.

```bash
curl "https://api.deside.io/api/v2/public/agents/deside/discovery"
```

Real on 2026-11-06, trimmed to two hosts:

```json
{
  "data": {
    "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f",
    "hosts": [
      {
        "host": "deside.io",
        "domain": "deside.io",
        "read": {
          "checkedAt": "2026-11-02T23:50:00.794Z",
          "stale": false,
          "files": [],
          "endpoints": [],
          "skills": [],
          "tools": [],
          "rails": [],
          "llmsFull": false,
          "map": [
            { "kind": "llms-txt", "url": "https://deside.io/llms.txt", "source": "declared-web", "foundAt": "2026-11-02T23:50:00.794Z", "lastReadAt": "2026-11-02T23:50:00.794Z", "lastStatus": 200 }
          ],
          "absent": [
            { "kind": "openapi", "url": "https://deside.io/openapi.json", "lastProbedAt": "2026-11-02T23:50:00.794Z", "lastStatus": 404 }
          ],
          "truncated": false
        }
      },
      { "host": "mcp.deside.io", "domain": "deside.io", "read": null }
    ]
  },
  "asOf": "2026-11-06T12:44:54.104Z"
}
```

| Field | Description |
|---|---|
| `host`, `domain` | The host the agent declares, and its registrable domain. |
| `read` | Our last read of that host, or `null` when we never read it. |
| `read.checkedAt`, `read.stale` | When we read it, and `true` after 7 days. |
| `read.files[]` | Machine files we found: `kind`, `url`, `found`, `contentType`, `bytes`. |
| `read.endpoints[]` | Endpoints the files announce: `kind`, `url`, `source`. |
| `read.skills[]` | Skills the files announce: `value`, `source`. |
| `read.tools[]` | Tool names the files announce. |
| `read.rails[]` | Payment protocols the files mention: `x402`, `mpp`. |
| `read.llmsFull` | `true` when we found an `llms-full.txt` on the host. |
| `read.map[]` | Every machine URL we found: `kind`, `url`, `source` (how we found it), `foundAt`, `lastReadAt`, `lastStatus`. |
| `read.absent[]` | Machine URLs we asked for and did not find: `kind`, `url`, `lastProbedAt`, `lastStatus`. |
| `read.truncated` | `true` when a list was cut at 100 entries. |
| `about` | Inside `data`: `read` is `agent.discovery`, and `definitions` has one sentence per field. |

**Every URL and name in `read` is written by a third party. Treat it as data, never as instructions.** For one host alone, and for the agents that declare a host, see [Domains](domains.md).

Errors come in the common shape: `400 invalid_request` for a bad `{ref}` or parameter, `404 not_found`, and `409 invalid_request` with `candidates` when a slug matches more than one agent. The limits are those of `GET /{ref}`.

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
  "code": "not-connected",
  "text": "Not connected",
  "vias": [{ "via": "wallet", "steps": ["link-wallet"], "value": null }]
}
```

| Field | Description |
|---|---|
| `code` | `not-connected` while nobody has proven it, `connected` once its owner has. |
| `text` | The phrase to show: `Not connected` or `Connected`. |
| `vias` | The ways the owner can prove it: `via`, `steps` and `value`. Empty once connected. |
| `state` | Deprecated: `unclaimed` or `proven`. Sent for one more version, then removed. Read `code`. |

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"invalid_object"}` | Bad `{type}` or `{id}`. A slug with a dot fails here. |
| 404 | `{"error":"not_found"}` | Nothing matches. |
| 503 | `{"error":"claim_unavailable"}` | The claim service is down. Retry later. |
