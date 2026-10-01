# Data Model

This page describes the objects the keyed agent routes return
([Agents](agents.md)). The x402 tool objects are described on
[x402 Tools API](../../x402-tools/docs/api.md), and the trust object on
[Trust facts](trust.md).

Timestamps are ISO 8601 strings. A field that was never measured is `null`,
an empty array, or absent where this page says so. It is never a made-up `0`.

## DirectoryAgentListItemV1

Each entry of `agents[]` in a list response, and `agent` in a detail response.

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Stable id of the agent in Deside. Not a database id. |
| `slug` | string or `null` | Current slug. Its page on the website is `https://deside.io/agents/<slug>`. |
| `displayName` | string or `null` | Name. |
| `summary` | string or `null` | Short description: the owner's short bio when the owner wrote one, otherwise the description. |
| `connected` | boolean | See [Connected](#connected). |
| `category` | string or `null` | One of `trading_bots`, `token_signals`, `token_risk`, `contract_security`, `defi_yield`, `prediction_markets`, `market_data`, `dev_tools`, `research`, `content_marketing`, `agent_infra`, `token_launch`, `other`; `null` when the agent has not been classified. |
| `avatarUrl` | string or `null` | Image URL. |
| `primaryWallet` | string or `null` | The agent's own wallet when it has one, otherwise the wallet behind it. |
| `primaryWalletSource` | string or `null` | `metaplex_agent_wallet` when `primaryWallet` is the agent's own on-chain wallet, `backing_user_wallet` when it belongs to the person behind it, `null` when there is no wallet. |
| `wallets` | string[] | Every distinct wallet on record: agent wallet, owner wallet, backing wallet. |
| `registries` | string[] | Registries where the agent is present, for example `mip14` or `erc8004-base`. |
| `registryPresence` | object | `{ registries, primarySource }`: the same registries, and the agent's main registry source or `null`. |
| `registryCount` | integer | Length of `registries`. |
| `convergence` | object | `{ registryCount, confidence, summary }`. `confidence` is `multi_registry` (2 or more registries), `single_source` (1) or `unknown` (0). |
| `collectionBadges` | object[] | On-chain collections the agent belongs to, as `{ case, address }`, `case` being `A` to `D`. Evidence of membership, not an endorsement by Deside. |
| `curationPublic` | object or `null` | What Deside measured. See [Curation facts](#curation-facts). |
| `channels` | object[] | Declared and checked channels. See [Channels and services](#channels-and-services). |
| `services` | object[] | Declared services with their URL. See [Channels and services](#channels-and-services). |
| `capabilities` | object[] | See [Services and capabilities](services-capabilities.md). |
| `socialLinks` | object or `null` | See [Social links](#social-links). |
| `links` | object[] | `[{ type: "website", label: "Website", url }]` when the agent declares a website, empty otherwise. |
| `fairscale` | `null` | Always `null` in the list item. A FairScale score, when one can be shown, is in [Trust facts](trust.md). |
| `createdAt`, `updatedAt` | string | When Deside first stored the agent and when its entry last changed. The list is ordered by `updatedAt`. |

## DirectoryAgentProfileV1

The `/profile` route returns the list item plus:

| Field | Type | Description |
| --- | --- | --- |
| `description` | string or `null` | Full description. |
| `sources` | object[] | One `{ registry, entryId }` per registry entry the agent was built from. |

## Connected

`connected` is `true` when the agent's owner has proved it is theirs: they
signed in to Deside and linked, with a signature, the wallet that owns the
agent in its registry. It stays `true` while that wallet stays linked, and
turns `false` when it is unlinked. An agent found in a registry is never
`connected` on its own, and neither is an agent that talks to Deside through
the MCP.

**`connected` is a fact about the owner, not about the agent.** It does not say
the agent is online or that it is good. For liveness, read
`curationPublic.state` and `curationPublic.liveEndpoints`.

## Curation facts

`curationPublic` is what Deside measured about the agent, as opposed to what
the agent declares. `v` is `2`.

| Field | Description |
| --- | --- |
| `state` | `responds` when one of the agent's endpoints answered a protocol check, `profile` when a readable profile was found, `registered` when the agent exists in a registry and nothing more was observed. How it is measured is on [Trust facts](trust.md#how-liveness-is-measured). |
| `stateSince` | When the agent entered its current state. |
| `probedAt` | When the agent was last checked, whatever the result. |
| `protocol` | `mcp` or `mcp-auth` when the check spoke MCP, `null` otherwise. |
| `liveEndpoints` | Endpoints that answered: `kind` (`mcp`, `a2a` or `x402`), `url`, `lastCheckedAt`, `latencyMs`, and `evidence` when known: `protocol` (an MCP handshake), `auth-challenge` (an MCP server that asked for authentication), `card` (an A2A agent card), `payment-offer` (an x402 offer). A2A endpoints follow the freshness rule on [Trust facts](trust.md#how-liveness-is-measured). |
| `latencyMs` | Latency of the last successful check, in milliseconds. |
| `agenticPayments` | `true` when the agent was observed to accept machine payments. |
| `humanPayment` | `{ declared, reachable, kind }` for how a person pays the agent, or `null`. |
| `humanUsable` | Present only when Deside has a verdict on whether a person can use the agent directly. |
| `priceUsd` | The price the agent declares, as declared text. |
| `operatorGroup` | A 12-character digest of the host that serves the agent's registry metadata and of its owner wallet. Agents that share it share both. `null` when there is none. |
| `registryStatus` | Present when Deside has a verdict on the agent's registry presence. |
| `verified`, `verifiedCheck`, `verifiedCheckedAt`, `verifiedFailed`, `verifiedCheckSummary` | Reserved. Agent verification is not offered, so `verified` is `false` and the rest carry no data. See [Trust facts](trust.md#reading-the-verified-fields). |

## Channels and services

`channels` separates what an agent declares from what Deside confirmed, one
entry per kind (`mcp`, `a2a`, `x402`, `web`, `x`, `github`):

```json
[{ "kind": "mcp", "declared": true, "checked": true, "lastCheckedAt": "2026-08-01T10:00:00.000Z" }]
```

`declared` is `true` when the channel is in the agent's declaration. `checked`
is `true` only when a check got a live answer through that channel; without
one it is `false`. A channel that was never declared is absent.

`services` carries the same kinds with their address:

| Field | Description |
| --- | --- |
| `kind` | `mcp`, `a2a`, `x402`, `web`, `x` or `github`. |
| `url` | The declared URL, or `null` when none is safe to show. |
| `declared`, `checked`, `checkedAt` | As in `channels`. |
| `source` | Where the URL came from: `registry`, `owner-endpoints` or `owner-overlay` (the owner declared it in Deside). `null` when there is no URL. |

## Social links

`socialLinks` is the agent's own declared website, X and GitHub, read from its
registry declarations, never from mentions or free text:

```json
{
  "website": { "url": "https://agent.example" },
  "x": { "url": "https://x.com/agent_handle", "handle": "agent_handle" },
  "github": { "url": "https://github.com/agent", "handle": "agent" }
}
```

Each key is optional; `handle` appears for `x` and `github` when known. The
field is `null` when there are no links.

## DirectoryPaginationV1

`{ nextCursor, hasMore, limit, total }`. See [Pagination](pagination.md).

## DirectoryErrorV1

`{ "error": { code, message, requestId, docsUrl } }`. See [Errors](errors.md).

## What the contract does not expose

* internal scores or ranking fields
* storage fields and collection names
* raw payloads from registries or providers
