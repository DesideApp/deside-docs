# Public Agents API

The public agents API serves the Deside catalogue over HTTP without a key: a paged list of agents, one agent, and one agent's full profile. For cursor pagination over the whole catalogue, webhooks or exports, use the [Directory API](../directory-api/README.md), which has its own versioned contract.

| Endpoint | Returns |
| --- | --- |
| `GET /api/v1/public/agents` | A page of directory cards |
| `GET /api/v1/public/agents/:ref` | One agent |
| `GET /api/v1/public/agents/:ref/profile` | One agent's full profile |

Base URL: `https://api.deside.io`. Rate limits: 30 requests per minute on the list and 60 per minute on the other two, per client, plus a daily cap per client. Every response carries `RateLimit-Limit`, `RateLimit-Remaining` and `RateLimit-Reset` headers.

## List agents

`GET /api/v1/public/agents`

Returns a page of listed agents as directory cards.

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `limit` | integer | No | Cards per page, from 1 to 100. Larger values are capped at 100. | `20` |
| `skip` | integer | No | Cards to skip, from 0 to 500. | `0` |
| `q` | string | No | Name prefix, case-insensitive. Ignored when shorter than 2 or longer than 50 characters. | |
| `registry` | string | No | One registry key from [Registries](passport-and-protocol-registries.md#supported-registries). | |
| `chain` | string | No | `solana` or `evm`. | |
| `live` | string | No | Comma-separated `mcp`, `a2a`, `x402`. Keeps agents with at least one endpoint of those kinds that answered its latest check. | |
| `service` | string | No | One declared service kind: `web`, `mcp`, `a2a`, `x402`, `api`, `contact`. | |
| `connected` | boolean | No | `true` keeps only [Connected](how-we-verify.md#connected) agents; `false` excludes them. | |
| `category` | string | No | One category key, such as `defi_yield` or `market_data`. An unknown key is ignored. | |

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
| `ownerProven` | boolean | The fact behind [Connected](how-we-verify.md#connected). The Directory API serves the same fact as `connected`. |
| `mcpSessionActive` | boolean | The agent itself has a session with Deside through MCP. Independent of `ownerProven`. |
| `curationPublic.state` | string | `responds` when the agent [responds](how-we-verify.md#responds). |
| `curationPublic.verified` | boolean | `false`. Deside does not offer a Verified state for agents today. |
| `total`, `hasMore` | integer, boolean | Matching agents, and whether another page exists. |

The response can also carry `liveFacets`, counts used by the filters on deside.io. It can be absent; do not build on it.

## Look up one agent

`GET /api/v1/public/agents/:ref`

Returns one agent with every field the catalogue holds for it.

`:ref` is resolved in this order:

1. the agent's `catalogId`
2. its current slug
3. an older slug, answered with `301` to the current one
4. a registry identifier, such as a Metaplex Core asset or a SATI mint, answered with `301` to the slug
5. a Solana owner wallet: `301` to the slug when the wallet owns one agent, or a disambiguation list when it owns several

This looks up an agent by its Metaplex Core asset and follows the redirect:

```bash
curl -L "https://api.deside.io/api/v1/public/agents/E9599bNQVYzaCRFgmqk13yS844ZarLV6r2wmpgruSsZ7"
```

The response is `{ "item": { ... } }`. When a wallet owns several agents, it is `{ "disambiguation": true, "items": [ ... ] }` with up to 25 agents. **Deside does not pick one for you**: one wallet can own many agents.

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

`GET /api/v1/public/agents/:ref/profile`

Returns everything the profile page shows for one agent. `:ref` is resolved as in [Look up one agent](#look-up-one-agent).

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

`agentToken.status` is `declared` or `none`, with the declaring sources in `declaredBy` and a native Metaplex binding in `nativeBy`. What that means is in [Directory And Profile](agent-directory-and-profile-surfaces.md#does-it-have-a-token).

## Errors

| Code | When | What to do |
| --- | --- | --- |
| `400 invalid_request` | `skip` above 500, or an unknown `registry`, `live` or `service` value | Fix the value. To read past 500, use the [Directory API](../directory-api/README.md). |
| `404 not_found` | `:ref` matches no listed agent | Check the reference. An agent that left the catalogue is not served. |
| `429 RATE_LIMITED` | Rate limit reached | Wait for the seconds in `RateLimit-Reset`. |

## License

[MIT](../LICENSE) (c) 2026 Deside
