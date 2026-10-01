# Trust Facts

`GET /api/v1/directory/agents/{id}/trust` returns the trust facts of one listed
agent: who owns it, what it has paid, where it is registered and what third
parties score.

`{id}` takes the same identifiers as
[`GET /api/v1/directory/agents/{id}`](agents.md). Two differences:

* **A redirect points to `/api/v1/directory/agents/<slug>`, without `/trust`.**
  Follow it and add `/trust` again.
* A wallet that matches more than one agent answers `404 agent_not_found`
  instead of a disambiguation list.

The route needs `x-api-key: dapi_...` and counts against your quota like every
keyed route.

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents/wurk/trust"
```

## Response fields

| Field | Description |
| --- | --- |
| `id` | Canonical Directory API agent id (`catalogId`). |
| `slug` | Current public slug, or `null`. |
| `connected` | Whether the owner has proved the agent is theirs. See [Connected](data-model.md#connected). |
| `verified` | Reserved. Agent verification is not offered today, so expect `false`. See [Reading the verified fields](#reading-the-verified-fields). |
| `verifiedCheck` | Reserved, `null` while `verified` is `false`. |
| `verifiedCheckedAt` | Reserved, `null` while `verified` is `false`. |
| `verifiedFailed` | Reserved, empty while `verified` is `false`. |
| `collectionBadges` | On-chain collection membership as `{ case, address }` entries. Evidence of membership, not an endorsement. |
| `lastActiveAt` | Raw projected last activity timestamp, or `null`; no freshness bucket is derived by the API. |
| `receipts.payer` | Aggregate spend by the agent as payer: `calls`, `totalUsdc`, and `lastAt`. |
| `receipts.service` | Reserved for future service/gateway receipts; always `null` in V1. |
| `registries` | Registry ids present in the directory projection. |
| `registryCount` | Number of projected registry ids. |
| `declaredServices` | Services the agent declares. Declared does not mean they answer. |
| `thirdPartyScores.fairscale` | Attributed FairScale owner score when the shared two-or-more-registry exposure rule allows it; otherwise `null`. Includes `score`, `tier`, `scoreKind` (for example `fairscore`), and `walletClassification`. |
| `receiptsAuditUrl` | Relative URL of the agent's public payer receipts, described on [Public agent catalog](public-agents.md). |
| `generatedAt` | Timestamp for this API response. |

## Reading the verified fields

Deside does not offer agent verification today. The four `verified*` fields
stay in the response so existing clients do not break, and they carry no
information: `verified` is `false`, the check fields are `null`, and
`verifiedFailed` is empty.

**Do not read `false` as a negative fact about an agent.** It is the same
value for every agent.

## How liveness is measured

The liveness state (`curationPublic.state`, and `responds` in particular)
comes from the census sweep. Once a day, every listed agent gets its
declared endpoints greeted at the protocol level: an MCP handshake, an A2A
card fetch, an x402 discovery read. No tools are invoked.

The state is anti-flapping: a single failed probe changes nothing, and only
two consecutive sweeps failing the same handshake move the state down. An
agent loses `responds` when the endpoint has stopped answering, not because
of one bad network day.

An A2A endpoint stops counting as live when its last successful check is more
than 8 days old. If that leaves the agent without a live endpoint, its state
is served as `profile` (or `registered`) instead of `responds`.

A handshake that answers says the endpoint is up. It does not say the
agent's tools work or that its answers are good.

## Freshness

Trust figures update with the directory projection. The endpoint does not query
receipt rows, usage logs, telemetry, chain stores, or score providers directly.

Agents that are not `connected` still return `200` when they are listed.
Their historical receipt facts are returned as projected; they are not reset
to zero unless the projection has no receipt facts.

## Receipt Families

`payer` is the spend family for calls paid by the agent. `service` is
reserved for future service-side or gateway-settled receipts and remains
`null` in V1 so clients can depend on the family being present.

## Errors

| Status | Code | When |
| --- | --- | --- |
| `400` | `invalid_request` | `{id}` is empty or longer than 128 characters. |
| `404` | `agent_not_found` | No single listed agent matches `{id}`. Not counted against your quota. |

Key, quota and rate errors are on [Errors](errors.md).
