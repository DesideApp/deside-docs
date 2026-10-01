# Public Agents API

The public agents API serves the Agent Directory over HTTP without a key: the directory counters, a paged list of agents, one agent, its full profile and its x402 payments. It consumes no quota. For cursor pagination over the whole directory, use the keyed [Agents](../directory-api/docs/agents.md) routes of the Directory API.

**These shapes are the website's, not the versioned Directory API contract.**

| Endpoint | Returns |
| --- | --- |
| `GET /api/v1/public/agents/stats-summary` | The directory counters |
| `GET /api/v1/public/agents` | A page of directory cards |
| `GET /api/v1/public/agents/{catalogId}` | One agent |
| `GET /api/v1/public/agents/{catalogId}/profile` | One agent's full profile |
| `GET /api/v1/public/agents/{catalogId}/receipts` | The x402 payments the agent made |

Base URL: `https://api.deside.io`. Rate limits, per IP: 30 requests per minute on the list and 60 per minute on the other routes, plus a daily cap shared by every `/public/agents` route ([Access Model](../directory-api/docs/access-model.md#public-read-surface)). Every response carries `RateLimit-Limit`, `RateLimit-Remaining` and `RateLimit-Reset` headers.

## The `{catalogId}` parameter

The parameter is named `catalogId` in the code, but it accepts any public identifier of one agent. It is resolved in this order:

1. the agent's `catalogId`
2. its current slug
3. an older slug, answered with `301` to the current one
4. a registry identifier, such as a Metaplex Core asset or a SATI mint, answered with `301` to the slug
5. a Solana owner wallet: `301` to the slug when the wallet owns one agent, or a disambiguation list when it owns several

The redirect keeps the `/profile` suffix. A disambiguation list is `{ "disambiguation": true, "items": [ ... ] }` with up to 25 agents. **Deside does not pick one for you**: one wallet can own many agents.

## Directory counters

`GET /api/v1/public/agents/stats-summary`

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

## List agents

`GET /api/v1/public/agents`

Returns a page of listed agents as directory cards.

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `q` | string | no | 2 to 50 characters. Matches the start of the name, or a wallet, registry entry or address. | |
| `registry` | string | no | One key from [Registries](passport-and-protocol-registries.md#supported-registries). An unknown value answers `400`. | |
| `chain` | string | no | `solana` or `evm`. Another value is ignored. | |
| `category` | string | no | One of the category keys of [Data model](../directory-api/docs/data-model.md). Another value is ignored. | |
| `service` | string | no | One of the service values of [Services and capabilities](../directory-api/docs/services-capabilities.md). | |
| `live` | string | no | `mcp`, `a2a`, `x402`, or several separated by commas: only agents with a live endpoint of one of those kinds. | |
| `connected` | `true` or `false` | no | Only agents whose owner has, or has not, proved ownership. | |
| `skill` | string | no | Only agents declaring this skill. | |
| `collection`, `collectionCase` | string | no | As on [Agents](../directory-api/docs/agents.md). | |
| `ownerWallet`, `agentWallet`, `coreAsset` | string | no | An exact Solana address. | |
| `sort` | string | no | `name` for alphabetical order. | |
| `limit` | integer | no | 1 to 100. | `20` |
| `skip` | integer | no | Rows to skip, 0 to 500. Above 500 answers `400`. | `0` |

### Example request

This returns one Metaplex agent whose MCP endpoint answered:

```bash
curl "https://api.deside.io/api/v1/public/agents?registry=mip14&live=mcp&limit=1"
```

### Example response

A real response from 2026-10-01:

```json
{
  "items": [
    {
      "catalogId": "a533b7ad-dd11-41ed-a121-ca339aad1f14",
      "slug": "wurk",
      "canonicalPath": "/agents/wurk",
      "name": "WURK",
      "chain": "solana",
      "avatarThumbUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/a533b7ad-dd11-41ed-a121-ca339aad1f14/20907e8647a4d166-card.webp",
      "avatarOriginalUrl": "https://wurkapi.fun/assets/twlogo.jpg",
      "avatar": "https://wurkapi.fun/assets/twlogo.jpg",
      "mcpSessionActive": false,
      "ownerProven": false,
      "category": "agent_infra",
      "services": [
        { "kind": "mcp", "checked": true },
        { "kind": "x402", "checked": true },
        { "kind": "web", "checked": false }
      ],
      "curationPublic": { "verified": false, "state": "responds" }
    }
  ],
  "total": 2,
  "limit": 1,
  "skip": 0,
  "hasMore": true
}
```

### Card fields

| Field | Type | Description |
| --- | --- | --- |
| `catalogId` | string | Stable id of the agent in Deside. |
| `slug`, `canonicalPath` | string | Current slug and the path of its profile on deside.io. |
| `name` | string | Display name. |
| `chain` | string | `solana` or `evm`. |
| `avatarThumbUrl`, `avatarOriginalUrl`, `avatar` | string or null | Cached card image, original image, and the image to show. |
| `category` | string | Category key, or `other`. |
| `services` | array | Declared service kinds. `checked` is `true` when that endpoint answered its latest check, `false` when it did not or is not checked. Card services carry no URL. |
| `ownerProven` | boolean | The fact behind [Connected](../start/state-words.md#connected). The Directory API serves the same fact as `connected`. |
| `mcpSessionActive` | boolean | The agent itself has a session with Deside through MCP. Independent of `ownerProven`. |
| `curationPublic.state` | string | `responds` when the agent is [Live](../start/state-words.md#live). |
| `curationPublic.verified` | boolean | Reserved. Always `false`. |
| `total`, `hasMore` | integer, boolean | Matching agents, and whether another page exists. |

The response can also carry `liveFacets`, counts used by the filters on deside.io. It can be absent; do not build on it.

## Look up one agent

`GET /api/v1/public/agents/{catalogId}`

Returns one agent with every field the catalogue holds for it, as `{ "item": { ... } }`.

This looks up an agent by its Metaplex Core asset and follows the redirect:

```bash
curl -L "https://api.deside.io/api/v1/public/agents/E9599bNQVYzaCRFgmqk13yS844ZarLV6r2wmpgruSsZ7"
```

### Selected fields

Real values for the agent above, on 2026-10-01:

```json
{
  "item": {
    "catalogId": "997f5adf-c8ee-4f56-a71a-925094d8438f",
    "slug": "xona-agent-ssz7",
    "ownerWallet": "9VaDVp1Wb78G4Wm6VuTiMrpESjrUymXefQTHcJGRSTEA",
    "agentWallet": "3WUXMBJHiSyxAUujWiMo3g5VD9JXog6RdwLsQi4rr1d6",
    "registryPresence": {
      "registries": ["mip14", "said", "8004solana", "sati", "sap"],
      "primarySource": "mip14"
    },
    "mergeEvidence": "owner-wallet",
    "ownerScore": { "system": "fairscale", "scoreKind": "fairscore", "score": 29.86, "tier": "silver" },
    "services": [
      {
        "kind": "mcp",
        "url": "https://api.xona-agent.com/mcp",
        "declared": true,
        "checked": true,
        "checkedAt": "2026-09-23T05:15:00.669Z"
      }
    ]
  }
}
```

| Field | Description |
| --- | --- |
| `ownerWallet` | Owner wallet. See [Owner and agent wallets](identity-resolution-and-auth-boundaries.md#owner-and-agent-wallets). |
| `agentWallet` | Agent wallet, set only from the Metaplex Agent Registry. Otherwise `null`. |
| `registryPresence` | Registries with an entry for this agent. |
| `sourceEntries` | Each entry by registry key and identifier. |
| `ownerScore` | FairScale score of the owner wallet. Present only when the agent is in two or more registries. |
| `services` | Each declared service with its URL and its latest check. `checkedAt` is `null` when it was never checked. |

### Merge evidence

`mergeEvidence` says how the agent's entries were joined. See [Identity Resolution](identity-resolution-and-auth-boundaries.md).

| Value | Meaning |
| --- | --- |
| `null` | The agent has entries in one registry only. |
| `owner-wallet` | Entries in several registries, all with the same owner wallet. |
| `editorial` | Entries in several registries that do not all share one owner wallet. |

## Read a profile

`GET /api/v1/public/agents/{catalogId}/profile`

Returns everything the profile page shows for one agent.

```bash
curl "https://api.deside.io/api/v1/public/agents/xona-agent-ssz7/profile"
```

The main branches are:

| Branch | What it holds |
| --- | --- |
| `visibleProfile` | The name, image and description shown for the agent. |
| `agentProfile.resolved.overview` | The profile sections: registry presence, onchain details, `agentToken`, `holdings`, declared services, skills, and registry reputation in `reputationBySource`. |
| `agentProfile.walletReputation` | FairScale reputation of the owner wallet and of the agent wallet. |
| `services` | Declared services with their latest check, as in the single-agent response. |
| `ownerProven`, `mcpSessionActive` | As in the card. |
| `identity` | `slug`, `canonicalPath` and `mergeEvidence`. |
| `relations` | Tokens and x402 tools related to the agent. See [Relation Fields](relations-api.md#agent-profile-relations). |

`agentToken.status` is `declared` or `none`, with the declaring sources in `declaredBy` and a native Metaplex binding in `nativeBy`. What that means is in [Directory And Profile](agent-directory-and-profile-surfaces.md#does-it-have-a-token).

## Read an agent's x402 payments

`GET /api/v1/public/agents/{catalogId}/receipts`

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

| Code | When | What to do |
| --- | --- | --- |
| `400 invalid_request` | An unknown `registry`, `service`, `live`, `collection` or `collectionCase` value, an address that is not a Solana address in `ownerWallet`, `agentWallet` or `coreAsset`, or `skip` above 500 | Fix the value. To read past 500, use the keyed [Agents](../directory-api/docs/agents.md) routes. |
| `404 not_found` | `{catalogId}` matches no listed agent | Check the reference. An agent that left the catalogue is not served. |
| `429 RATE_LIMITED` | Rate limit reached | Wait for the seconds in `RateLimit-Reset`. |

## License

[MIT](../LICENSE) (c) 2026 Deside
