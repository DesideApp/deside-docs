# Directory API Data Model

This page documents the public contracts only. It does not describe internal
Mongo models, provider payloads, or scoring implementation details.

## DirectoryAgentListItemV1

List responses return `agents[]`, where each item follows the
`DirectoryAgentListItemV1` public contract.

Expected public fields include:

- `id`
- `slug`
- `displayName`
- `summary`
- `avatarUrl`
- `primaryWallet`
- `wallets`
- `registries`
- `registryPresence`
- `registryCount`
- `services`
- `capabilities`
- `convergence`
- `links`
- `fairscale`
- `createdAt`
- `updatedAt`

The list endpoint returns:

```json
{
  "agents": [],
  "pagination": {
    "nextCursor": null,
    "hasMore": false,
    "limit": 50,
    "total": 0
  }
}
```

Public invariants:

- the public `id` is not an internal ObjectId
- timestamps are exposed as ISO strings
- Mongo collection names do not appear in the contract
- raw provider payloads do not appear in the contract

## DirectoryAgentProfileV1

The profile route extends the list contract with
`DirectoryAgentProfileV1`.

- `description`
- `sources`

## DirectoryServiceV1

Services are exposed as `DirectoryServiceV1` entries.

- they describe contact or protocol channels
- they are discovery signals, not execution guarantees
- they may be declared from source data
- they may be observed from registry evidence

## DirectoryCapabilityV1

Capabilities are exposed as `DirectoryCapabilityV1` entries.

- they describe task-level or role-level signals
- they may be declared or derived from source data
- they include source and confidence metadata when available

## DirectoryRegistryPresenceV1

Registry presence is exposed through `DirectoryRegistryPresenceV1`.

- it captures whether the agent is present in a registry
- it contributes to convergence and identity evidence
- it is public metadata, not an internal fetch log

## DirectoryPaginationV1

Pagination is exposed as `DirectoryPaginationV1` with:

- `limit`
- `total`
- `hasMore`
- `nextCursor`

## DirectoryErrorV1

Public error responses follow `DirectoryErrorV1`:

- `code`
- `message`
- `requestId`
- `docsUrl`

## FairScale

`fairscale` is nullable in V1 and should be treated as a flagged enrichment.
It is not a guaranteed ranking score and does not expose any internal scoring
field.

## Non-public fields

This public contract does not expose:

- internal scoring fields
- storage model fields
- provider payloads
- raw registry blobs
