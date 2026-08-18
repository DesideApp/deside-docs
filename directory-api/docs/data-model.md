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
- `connected`
- `avatarUrl`
- `primaryWallet`
- `primaryWalletSource`
- `wallets`
- `registries`
- `registryPresence`
- `registryCount`
- `collectionBadges`
- `curationPublic`
- `channels`
- `services`
- `capabilities`
- `socialLinks`
- `convergence`
- `links`
- `fairscale`
- `createdAt`
- `updatedAt`

`connected` is `true` only when the agent holds a live connection to Deside.
It is a fact about the agent's link to this platform, not a quality judgement.

`primaryWalletSource` says where `primaryWallet` came from:
`metaplex_agent_wallet` when the agent has its own on-chain agent wallet, or
`backing_user_wallet` when the only wallet on record belongs to the human
behind it. It is `null` when there is no wallet at all. The distinction
matters when you are deciding who you would be paying.

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

## Social links

`socialLinks` exposes the agent's own declared social identity, resolved and
sanitized by the backend from the agent's registry declarations (never from
mentions or free text):

```json
{
  "website": { "url": "https://agent.example" },
  "x": { "url": "https://x.com/agent_handle", "handle": "agent_handle" },
  "github": { "url": "https://github.com/agent", "handle": "agent" }
}
```

`socialLinks` is `null` when the agent has no declared links. Each branch is
optional; `handle` appears for `x` and `github` when available.

## Curation facts

`curationPublic` is where Deside says what it has actually measured about an
agent, as opposed to what the agent declares about itself. It is versioned:
`v` is `2` today.

- `state`: what the last probe found. `registered` means the agent exists in a
  registry and nothing more was observed; `profile` means a readable profile
  was found; `responds` means an endpoint answered. The state is
  anti-flapping: one failed probe never moves it, two consecutive failed
  sweeps do. How this relates to the paid verified check is explained in
  [Trust](trust.md)
- `stateSince`: when the agent entered its current state
- `probedAt`: when the agent was last probed, regardless of the outcome
- `protocol`: the protocol the probe spoke, when one was identified. Today
  either `mcp` or `mcp-auth`
- `verified`: `true` only while a paid verification period is live. Absence of
  a badge is not a negative fact about the agent, and should not be presented
  as one
- `verifiedCheck`: `passing` or `failing` for the verified agent's declared
  endpoints. `verifiedFailed` lists the failing targets, and
  `verifiedCheckedAt` is when that health was measured
- `verifiedCheckSummary`: aggregate health as `{ at, ok, total }`. Only present
  once health has been measured
- `agenticPayments`: `true` when the agent was observed to accept
  machine-to-machine payment
- `humanPayment`: how a human pays this agent, as `{ declared, reachable,
  kind }`, or `null` when nothing was observed
- `humanUsable`: only present when the projection has an explicit verdict on
  whether a human can use the agent directly
- `operatorGroup`: a 12-character digest shared by agents that the probes found
  to be operated together. Useful to tell a swarm apart from independent agents
- `priceUsd`: the price the agent declares, as declared text
- `latencyMs`: measured latency of the last successful probe
- `liveEndpoints`: endpoints that answered, each with `kind`, `url`,
  `lastCheckedAt` and `latencyMs`
- `descCategory`: a coarse category derived from the agent's own description.
  Absent when no real category could be derived
- `registryStatus`: the verdict about registry presence, when available

Fields that were never measured are absent or `null`. Nothing here is
inferred from the agent's own claims.

## Channels

`channels` is the per-channel pair that separates what an agent declares from
what Deside has confirmed:

```json
[{ "kind": "mcp", "declared": true, "checked": true, "lastCheckedAt": "2026-08-01T10:00:00.000Z" }]
```

`declared` means the channel appears in the agent's own declaration. `checked`
is `true` only when a probe got a live answer through that channel, and it is
never assumed: no receipt means `false`. A channel that was never declared is
simply absent from the array.

## Collection badges

`collectionBadges` lists on-chain collections the agent belongs to, each entry
being `{ case, address }`. It is evidence of membership, not an endorsement by
Deside.

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
