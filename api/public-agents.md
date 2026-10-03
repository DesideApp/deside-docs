# Public agents

These routes read the Agent Directory with no key. They are limited per IP and stop at 500 rows deep. To keep a full copy in sync, use [Directory agents](directory-agents.md).

## GET /public/agents

Returns agents in the directory, filtered and paged.

```bash
curl "https://api.deside.io/api/v1/public/agents?limit=1&service=mcp"
```

### Parameters

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 20 | 1 to 100. Out-of-range values are clamped. 24 at most with `sort=featured`. |
| `skip` | integer | 0 | Rows to skip. 500 at most; 96 with `sort=featured`. Above that: `400`. |
| `q` | string | | 2 to 50 characters. Matches the start of the name, the start of the owner wallet, or a full agent wallet, registry entry id or EVM address. Outside 2 to 50 it is ignored. |
| `name` | string | | Same as `q`. |
| `skill` | string | | Up to 60 characters. Exact match on a declared skill, lowercase. |
| `category` | string | | One of the 12 categories below. An unknown value is ignored. |
| `service` | string | | `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. Agents that declare it. |
| `live` | string | | Comma list of `mcp`, `a2a`, `x402`. Agents with at least one of these live. |
| `registry` | string | | `mip14`, `8004solana`, `said`, `sati`, `sap`, `kamiyo` or `erc8004-base`. Aliases: `014`, `mip014`, `8004`, `8004base`, `base`. |
| `chain` | string | | `solana` or `evm`. Any other value is ignored. |
| `ownerWallet` | string | | Exact owner wallet, base58. |
| `agentWallet` | string | | Exact agent wallet, base58. |
| `coreAsset` | string | | Exact Metaplex Core asset, base58. |
| `collection` | string | | Agents carrying a badge of this collection, base58. |
| `collectionCase` | string | | `A`, `B`, `C` or `D`. |
| `connected` | boolean | | `true`: the owner proved the agent is theirs. Same fact as `ownerProven` in the response. |
| `verified` | boolean | | Filters by the retired Verified badge. Always matches nothing today. |
| `duplicates` | string | | `show` or `hide`. |
| `sort` | string | | `name` for A to Z. `featured` for a curated selection. Anything else keeps the default order. |

Categories: `trading_bots`, `token_signals`, `token_risk`, `contract_security`, `defi_yield`, `prediction_markets`, `market_data`, `dev_tools`, `research`, `content_marketing`, `agent_infra`, `token_launch`. Agents without a category have `other`, which is not a filter.

### Response

```json
{
  "items": [
    {
      "catalogId": "bbddcb0c-074f-4874-9c48-3733013db7f7",
      "slug": "blinkcodes",
      "canonicalPath": "/agents/blinkcodes",
      "name": "BlinkCodes",
      "chain": "evm",
      "avatarThumbUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/.../card.webp",
      "avatarOriginalUrl": "https://blinkcodes.com/web-app-manifest-512x512.png",
      "avatar": "https://blinkcodes.com/web-app-manifest-512x512.png",
      "mcpSessionActive": false,
      "ownerProven": false,
      "category": "other",
      "services": [
        { "kind": "mcp", "checked": true },
        { "kind": "a2a", "checked": true },
        { "kind": "x402", "checked": true },
        { "kind": "web", "checked": false }
      ],
      "curationPublic": { "verified": false, "state": "responds" }
    }
  ],
  "total": 109385,
  "limit": 1,
  "skip": 0,
  "hasMore": true,
  "liveFacets": { "mcp": 766, "a2a": 33, "x402": 31 }
}
```

| Field | Description |
|---|---|
| `catalogId` | Stable id of the agent in Deside. |
| `slug` | Short name, used in URLs. |
| `canonicalPath` | The agent's page on deside.io. |
| `name` | Display name. |
| `chain` | `solana` or `evm`. |
| `avatarThumbUrl` | Our cached copy of the avatar, or `null`. |
| `avatarOriginalUrl` | The avatar URL the registry gives. |
| `avatar` | Same as `avatarOriginalUrl`. |
| `mcpSessionActive` | The agent has a session open with the Deside MCP. |
| `wallet` | Only when `mcpSessionActive` is `true`. |
| `ownerProven` | The owner proved, by signing, that the agent is theirs. See [Checks](../start/checks.md). |
| `category` | One of the 12 categories, or `other`. |
| `services[].kind` | A service the agent declares: `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. |
| `services[].checked` | `true` when our last check of that service passed. |
| `curationPublic.state` | `responds`, `profile` or `registered`. See [Checks](../start/checks.md). |
| `curationPublic.verified` | Always `false`. |
| `total` | Agents matching the filters. |
| `hasMore` | `true` when `skip + items` is below `total`. |
| `liveFacets` | How many matching agents are live on each protocol. Present when the counts are available. |

### Errors

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"invalid_request"}` | A bad `service`, `live`, `registry`, wallet, `collection`, `collectionCase` or `duplicates`, or `skip` above 500. |
| 429 | `{"error":"RATE_LIMITED"}` | More than 30 list requests a minute, or 500 a day, from your IP. |

## GET /public/agents/stats-summary

Returns the directory counts from the last nightly measure. No parameters.

```bash
curl "https://api.deside.io/api/v1/public/agents/stats-summary"
```

```json
{
  "listed": 109385,
  "indexed": 109385,
  "byChain": { "solana": 15168, "evm": 94217 },
  "connected": 1,
  "byCategory": { "trading_bots": 347, "token_risk": 88, "other": 2625 },
  "byCategoryByChain": { "solana": { "trading_bots": 29 }, "evm": { "trading_bots": 318 } },
  "respondingAgents": 948,
  "respondingByKind": { "mcp": 799, "a2a": 148, "x402": 37 },
  "endpoints": {
    "mcp": { "declared": 1525, "checked": 1450, "alive": 209 },
    "a2a": { "declared": 3337, "checked": 3227, "alive": 243 },
    "x402": { "declared": 305, "checked": 91, "alive": 26 }
  },
  "topSkills": [{ "label": "comment", "n": 755 }],
  "signalCounts": { "mcp": 4286, "a2a": 26773, "x402": 445, "web": 28197, "x": 5925 },
  "measuredAt": "2026-10-02T05:15:01.027Z"
}
```

| Field | Description |
|---|---|
| `listed` | Agents in the directory. |
| `indexed` | Same as `listed`. |
| `byChain` | `listed`, by chain. |
| `connected` | Agents whose owner proved they are theirs. |
| `byCategory` | Agents per category. |
| `byCategoryByChain` | Agents per category, by chain. |
| `respondingAgents` | Agents with at least one endpoint live. |
| `respondingByKind` | Agents live, per protocol. |
| `endpoints.<kind>` | Endpoint URLs per protocol: declared by some agent, checked by us, and live in the last check. One URL can be declared by many agents. |
| `topSkills` | The 12 most declared skills. |
| `signalCounts` | Agents that declare each kind of service or an X account. |
| `measuredAt` | When the counts were measured. |

## GET /public/agents/{ref}

Returns one agent. `{ref}` can be a `catalogId`, a slug, an old slug, a registry entry id, a mint, or a wallet.

```bash
curl "https://api.deside.io/api/v1/public/agents/blinkcodes"
```

- An exact match returns `200` with `{"item": {...}}`.
- An old slug, an on-chain id or a wallet with one agent returns `301` to `/public/agents/{slug}`.
- A wallet with several agents returns `200` with `{"disambiguation": true, "items": [...]}`, up to 25, in the list shape.

### Response fields

| Field | Description |
|---|---|
| `catalogId`, `slug`, `name`, `description`, `chain`, `category` | As in the list. |
| `agentId` | Same as `catalogId`. |
| `ownerWallet` | The wallet that owns the registry entry. |
| `agentWallet` | The agent's own wallet, when the registry gives one. |
| `wallet` | The wallet of the agent's Deside account. |
| `primarySource` | The main registry. |
| `primarySourceEntryId` | The agent's id in that registry. |
| `sourceEntries[]` | Every registry entry merged into this agent: `source` and `sourceEntryId`. |
| `registryPresence` | `registries`: every registry that lists the agent; `primarySource`: the main one. |
| `avatarThumbUrl`, `avatarProfileUrl` | Our cached copies of the avatar, small and large. |
| `avatarOriginalUrl`, `avatar` | The registry's avatar URL. |
| `website` | The declared website. |
| `socialLinks` | Declared links: `website`, `x`, `github`, each with `url` and `handle`. |
| `services[]` | Each declared service, with what we checked. See below. |
| `serviceSignals[]` | The kinds of service declared. |
| `channels[]` | Same as `services[]`, shorter. |
| `x402State` | |
| `capabilities[]` | Declarations per registry: `kind`, `label`, `source`, `endpoint`. |
| `skills[]` | Declared skills. |
| `skillRepertoire` | Skills with the source of each. |
| `collectionBadges[]` | Collections the agent belongs to: `case` and `address`. |
| `ownerProven` | See the list. |
| `mcpSessionActive` | See the list. |
| `curationPublic` | Our checks. See below. |
| `receipts.payer` | x402 payments the agent made: `calls`, `totalUsdc`, `lastAt`. |
| `serviceDesc`, `ownerOverlay` | Always `null`. |
| `ownerScore` | Always `null`. |
| `mergeEvidence` | |
| `isVisible` | |
| `lastActiveAt` | |
| `canonicalPath` | |
| `createdAt`, `updatedAt` | When the agent entered the directory and last changed. |

#### services[]

| Field | Description |
|---|---|
| `kind` | `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. |
| `url` | The declared URL. |
| `declared` | `true`: the agent declares it. |
| `checked` | `true`: our last check passed. |
| `checkedAt` | When we last checked it. |
| `version` | The protocol version the endpoint answered with. |
| `source` | Where the URL came from. Today always `registry`. |
| `x402Probe` | x402 only. Our last call: `verdict`, `at`, `reason`, `network`, `x402Version`. |
| `x402Content` | x402 only. The tools behind this URL: `state`, `tools`, `quotesPrice`, `respondsNoPrice`, `down`, `notChecked`. |
| `indexes[]` | x402 only. Indexes at this URL: `url`, `toolCount`, `state`. |
| `toolsHref` | x402 only. The page that lists these tools on deside.io. |

#### curationPublic

| Field | Description |
|---|---|
| `state` | `responds`, `profile` or `registered`. See [Checks](../start/checks.md). |
| `probedAt` | When we last checked the agent. |
| `liveEndpoints[]` | Endpoints live in the last check: `kind`, `url`, `lastCheckedAt`, `latencyMs`, `version`, `evidence`. |
| `registryStatus` | When a registry stopped listing the agent: `missingSources[]` and `missingSince`. |
| `protocol` | `mcp` or `mcp-auth`. |
| `agenticPayments` | |
| `v` | |
| `stateSince` | |
| `operatorGroup` | |
| `humanPayment`, `priceUsd`, `latencyMs` | Always `null` today. |
| `verified`, `verifiedCheck`, `verifiedCheckedAt`, `verifiedFailed` | Always `false`, `null`, `null`, `[]`. |

### Errors

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"invalid_payload","code":"invalid_catalog_id"}` | `{ref}` longer than 128 characters. |
| 404 | `{"error":"not_found"}` | No agent matches. |
| 429 | `{"error":"RATE_LIMITED"}` | More than 60 requests a minute from your IP. |

## GET /public/agents/{ref}/profile

Returns everything the agent page on deside.io shows: the agent, its registry data, its relations and how its owner can claim it. Same `{ref}`, redirects and errors as above.

```bash
curl "https://api.deside.io/api/v1/public/agents/blinkcodes/profile"
```

| Field | Description |
|---|---|
| `visibleProfile` | Name, avatar, description and source the page shows. |
| `agentProfile.resolved` | Everything read from the registries. |
| `agentProfile.resolved.resolvedAt` | When we last read them. |
| `agentProfile.resolved.overview` | The registry data, by topic: `onchainDetails`, `agentToken`, `holdings`, `declaredAgentWallets`, `socialLinks`, `serviceDeclarations`, `skills`, `domains`, `capabilities`, `reputationBySource`, `registryPresence`. |
| `agentProfile.resolved.raws` | The raw entry of each registry, as the registry serves it. |
| `agentProfile.resolved.rawSourceData` | |
| `agentProfile.resolved.visibleProfile`, `displayName`, `displayAvatar`, `description`, `source` | |
| `agentProfile.resolved.overview.visualIdentity` | |
| `agentProfile.resolved.overview.serviceInterfaces` | |
| `agentProfile.resolved.overview.onchainDetails.legacy` | |
| `agentProfile.resolved.overview.statusTrustPayments` | |
| `agentProfile.resolved.overview.agentToken.coverage`, `evidence[].rawPath`, `declaredAgentWallets[].path`, `holdings.*.provider`, `holdings.*.nextRefreshAt` | |
| `agentProfile.identity` | The on-chain identity: `source`, `coreAsset`, `mintAddress`, `pda`. |
| `agentProfile.identity.passport` | |
| `agentProfile.identity.protocol`, `agentProfile.identity.name/description/image/services` | |
| `agentProfile.identity.verifiedAt` | When the registry entry was read. |
| `agentProfile.identity.reputation` | |
| `agentProfile.curationPublic` | As in the single agent. |
| `agentProfile.tickets` | Our last check per protocol, with what it saw. |
| `services`, `capabilities`, `category`, `collectionBadges`, `ownerProven`, `mcpSessionActive` | As in the single agent. |
| `relations` | Tools, tokens and sites linked to this agent, and how: `total` and `items[]` with `kind`, `id`, `step`, `vias[]`, `card`. |
| `claim` | How the owner can prove the agent is theirs. Same as [GET /public/claim](#get-public-claim-type-id). |
| `onSolana` | Metaplex agents only: collection, update authority, delegate, frozen. |
| `identity` | `slug` and `canonicalPath`. |
| `identity.mergeEvidence` | |
| `ownerOverlay` | |
| `authenticated`, `registered`, `pubkey`, `role`, `userProfile`, `social`, `contactWallet`, `wallet` | |
| `agentProfile.walletReputation` | |

## GET /public/agents/{ref}/receipts

Returns the x402 payments the agent made, newest first.

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 20 | 1 to 100, clamped. |
| `skip` | integer | 0 | 0 to 5,000, clamped. |

```json
{ "items": [], "total": 0, "limit": 2, "skip": 0, "hasMore": false }
```

Each item: `txSignature`, `amountUsdc`, `settledAt`.

## GET /public/claim/{type}/{id}

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
