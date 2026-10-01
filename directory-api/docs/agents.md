# Agents

These routes read the agent directory with a Directory API key. They return
the versioned contract described in [Data model](data-model.md), and every
request counts against your quota ([Rate limits](rate-limits.md)).

The trust facts of one agent have their own page: [Trust facts](trust.md).

## `GET /api/v1/directory/agents`

Returns a page of listed agents, newest `updatedAt` first.

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `limit` | integer | no | Page size. Values above 100 are read as 100, below 1 as 1. A value that is not a number answers `invalid_request`. | `50` |
| `cursor` | string | no | `pagination.nextCursor` of the previous page. See [Pagination](pagination.md). | |
| `registry` | string | no | Only agents present in this registry. Accepted values: `mip14` (also `mip014`, `014`), `said`, `sap`, `sati`, `kamiyo`, `8004solana` (also `8004`), `erc8004-base` (also `8004base`, `base`). | |
| `chain` | string | no | `solana` or `evm`. | |
| `service` | string | no | One of the service values on [Services and capabilities](services-capabilities.md). | |
| `capability` | string | no | One of the capability values on [Services and capabilities](services-capabilities.md). | |
| `collection` | string | no | A Solana collection address (32 to 44 base58 characters). Only agents with that collection badge. | |
| `collectionCase` | string | no | `A`, `B`, `C` or `D`: only agents with a collection badge of that case. | |
| `updatedSince` | string | no | ISO date. Only agents with `updatedAt` at or after it. | |

An unknown value of `registry`, `chain`, `service`, `capability`,
`collection`, `collectionCase` or `updatedSince` answers `invalid_request`.
There is no text search on this route: a `q` parameter is ignored.

### Example request

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?chain=solana&limit=100"
```

### Response

```json
{
  "agents": [],
  "pagination": { "nextCursor": null, "hasMore": false, "limit": 100, "total": 0 }
}
```

Each entry of `agents` is a `DirectoryAgentListItemV1`, documented field by
field on [Data model](data-model.md). `pagination.total` counts every agent
that matches the filters, not only this page.

## `GET /api/v1/directory/agents/{id}`

Returns one agent. `{id}` is, up to 128 characters, one of:

1. the agent's `id`;
2. its current `slug` (case-insensitive);
3. a previous slug of the agent;
4. a Solana address: the agent's registry entry, or a wallet that owns or
   backs it.

An exact `id` or current `slug` answers with the agent. Every other form
answers `301` to the current slug, except a wallet that matches more than one
agent, which answers a disambiguation list of up to 25 agents.

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents/wurk"
```

The response is one of:

| Case | Status | Body |
| --- | --- | --- |
| One agent matches | `200` | `{ "agent": DirectoryAgentListItemV1 }` |
| A previous slug, a registry entry or a wallet of one agent | `301` | Redirect to `/api/v1/directory/agents/<current slug>` |
| A wallet of more than one listed agent | `200` | `{ "disambiguation": true, "agents": [DirectoryAgentListItemV1, ...] }` |
| Nothing matches | `404` | `agent_not_found`. Not counted against your quota. |

## `GET /api/v1/directory/agents/{id}/profile`

Returns the same as the route above, with `agent` as a
`DirectoryAgentProfileV1`: the list item plus `description` and `sources`. See
[Data model](data-model.md#directoryagentprofilev1).

**A redirect from this route points to `/api/v1/directory/agents/<slug>`,
without `/profile`.** Follow it and add `/profile` again if you need the
profile shape.

## Errors

| Status | Code | When | What to do |
| --- | --- | --- | --- |
| `400` | `invalid_request` | An unknown filter value, a non-numeric `limit`, an empty or too long `{id}`. | Fix the parameter. |
| `400` | `invalid_cursor` | The cursor is malformed or was issued for other filters. | Restart without `cursor`. |
| `404` | `agent_not_found` | No listed agent matches `{id}`. | Check the id or slug. |

Key, quota and rate errors are on [Errors](errors.md).
