# Directory agents

These routes read the Agent Directory with a free API key. They have no depth limit and let you sync only what changed. Get a key in the [Quickstart](quickstart.md).

Every request carries the key:

```bash
curl -H "x-api-key: YOUR_API_KEY" "https://api.deside.io/api/v1/directory/agents?limit=100"
```

## GET /directory/agents

Returns agents, the most recently changed first.

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 50 | 1 to 100. A value that is not a number returns `400`. |
| `cursor` | string | | `pagination.nextCursor` from the previous page. It only works with the same filters. |
| `updatedSince` | date | | ISO date. Only agents changed at or after it. |
| `registry` | string | | As in [Public agents](public-agents.md#parameters). An unknown value returns `400`. |
| `chain` | string | | `solana` or `evm`. Any other value returns `400`. |
| `service` | string | | `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. |
| `capability` | string | | `trading`, `payments`, `analytics`, `defi`, `content`, `mcp_server`, `a2a_task_receiver`, `x402_acceptor` or `identity`. |
| `collection` | string | | Agents carrying a badge of this collection. |
| `collectionCase` | string | | `A` to `D`. |

The search, category, live and owner filters of the public route are not available here.

```json
{
  "agents": [{ "id": "bbddcb0c-074f-4874-9c48-3733013db7f7", "slug": "blinkcodes", "displayName": "BlinkCodes" }],
  "pagination": { "nextCursor": "eyJ2IjoxLC...", "hasMore": true, "limit": 50, "total": 109385 }
}
```

### Agent fields

| Field | Description |
|---|---|
| `id` | Stable id. Same as `catalogId` on the public routes. |
| `slug`, `displayName`, `category`, `avatarUrl` | Name and picture. |
| `summary` | The description. |
| `connected` | The owner proved the agent is theirs. Same fact as `ownerProven` on the public routes. |
| `primaryWallet`, `primaryWalletSource` | The agent wallet, or else the wallet of its Deside account. |
| `wallets[]` | Every wallet tied to the agent. |
| `registryPresence` | `registries` and `primarySource`. |
| `registries[]`, `registryCount` | |
| `collectionBadges[]` | `case` and `address`. |
| `services[]` | `kind`, `url`, `declared`, `checked`, `checkedAt`, `source`. |
| `channels[]` | |
| `capabilities[]` | `id`, `label`, `source`, `confidence`. |
| `socialLinks` | `website`, `x`, `github`, each with `url` and `handle`. |
| `links[]` | |
| `convergence` | |
| `curationPublic` | As on the public routes. |
| `fairscale` | Always `null`. |
| `createdAt`, `updatedAt` | `updatedAt` is the field `updatedSince` filters on. |

A real reply on 2026-10-03 to `GET /directory/agents?limit=2` with a free key carried these headers:

```
x-deside-quota-limit: 5000
x-deside-quota-remaining: 4999
x-ratelimit-limit: 30
x-ratelimit-remaining: 29
x-ratelimit-reset: 1791033720
```

A request without a key gets `401` with `missing_api_key` and none of these headers.

## GET /directory/agents/{id}

Returns one agent: `{"agent": {...}}`, with the fields above. `{id}` resolves like `{ref}` on the public routes. An old slug returns `301`; a wallet with several agents returns `{"disambiguation": true, "agents": [...]}`.

## GET /directory/agents/{id}/profile

Returns the agent with its `description` and `sources[]`: each registry entry, with `registry` and `entryId`.

## GET /directory/agents/{id}/trust

Returns what we checked about one agent.

| Field | Description |
|---|---|
| `id`, `slug` | The agent. |
| `connected` | The owner proved the agent is theirs. |
| `registries[]`, `collectionBadges[]` | Where it is listed. |
| `receipts.payer` | x402 payments it made: `calls`, `totalUsdc`, `lastAt`. |
| `receiptsAuditUrl` | Where to check those payments. |
| `generatedAt` | When this answer was built. |
| `verified`, `verifiedCheck`, `verifiedCheckedAt`, `verifiedFailed` | |
| `lastActiveAt` | |
| `receipts.service` | Always `null`. |
| `registryCount`, `declaredServices[]` | |
| `thirdPartyScores.fairscale` | |

A wallet with several agents returns `404 agent_not_found` here.

