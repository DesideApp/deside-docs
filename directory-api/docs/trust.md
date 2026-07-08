# Trust Facts

`GET /api/v1/directory/agents/:id/trust` returns machine-readable trust facts
for one listed agent. `:id` accepts the agent `catalogId` or current `slug`.

## Authentication

Use the same Directory API key contract as the rest of the V1 Directory API:

```http
GET /api/v1/directory/agents/agent-catalog-1/trust
x-api-key: dapi_...
```

Missing, invalid, revoked, blocked, rate-limited, and quota-exceeded keys use
the standard Directory API error envelope.

## Response

```json
{
  "id": "agent-catalog-1",
  "slug": "agent-slug",
  "connected": true,
  "lastActiveAt": "2026-07-06T22:33:11.000Z",
  "receipts": {
    "payer": {
      "calls": 12,
      "totalUsdc": 3.4,
      "lastAt": "2026-07-06T22:33:11.000Z"
    },
    "service": null
  },
  "registries": ["said", "sati"],
  "registryCount": 2,
  "declaredServices": ["web", "mcp", "x402"],
  "thirdPartyScores": {
    "fairscale": {
      "score": 82,
      "tier": "gold",
      "scoreKind": "owner_wallet"
    }
  },
  "receiptsAuditUrl": "/api/v1/public/agents/agent-catalog-1/receipts",
  "generatedAt": "2026-07-07T08:00:00.000Z"
}
```

Fields:

| Field | Description |
| --- | --- |
| `id` | Canonical Directory API agent id (`catalogId`). |
| `slug` | Current public slug, or `null`. |
| `connected` | `true` only when the directory projection marks the agent connected. |
| `lastActiveAt` | Raw projected last activity timestamp, or `null`; no freshness bucket is derived by the API. |
| `receipts.payer` | Verified aggregate spend by the agent as payer: `calls`, `totalUsdc`, and `lastAt`. |
| `receipts.service` | Reserved for future service/gateway receipts; always `null` in V1. |
| `registries` | Registry ids present in the directory projection. |
| `registryCount` | Number of projected registry ids. |
| `declaredServices` | Service signals declared by the projection. Declared does not mean verified. |
| `thirdPartyScores.fairscale` | Attributed FairScale owner score when the shared two-or-more-registry exposure rule allows it; otherwise `null`. |
| `receiptsAuditUrl` | Relative URL for the public payer-receipt audit trail. |
| `generatedAt` | Timestamp for this API response. |

## Freshness

Trust figures update with the directory projection. The endpoint does not query
receipt rows, usage logs, telemetry, chain stores, or score providers directly.

Disconnected agents still return `200` when they are listed. Their historical
receipt facts are returned as projected; they are not reset to zero unless the
projection has no receipt facts.

## Receipt Families

`payer` is the verified spend family for calls paid by the agent. `service` is
reserved for future service-side or gateway-settled receipts and remains
`null` in V1 so clients can depend on the family being present.

## Errors

| Status | Code | Meaning |
| --- | --- | --- |
| `400` | `invalid_request` | The agent id parameter is empty or invalid. |
| `401` | `missing_api_key` | No `x-api-key` header was sent. |
| `401` | `invalid_api_key` | The key is unknown or malformed. |
| `403` | `api_key_revoked` | The key was revoked. |
| `403` | `api_key_blocked` | The key was blocked. |
| `403` | `project_blocked` | The owning API project is blocked. |
| `403` | `origin_not_allowed` | The request origin is not allowed for the key. |
| `404` | `agent_not_found` | No listed agent matched the catalog id or slug. |
| `429` | `rate_limit_exceeded` | The per-minute rate limit was exceeded. |
| `429` | `quota_exceeded` | The monthly quota was exceeded. |
