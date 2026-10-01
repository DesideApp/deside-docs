# Errors

The keyed routes answer errors in one envelope. Read `code`: it is the stable
contract. `message` is for people and can change.

```json
{
  "error": {
    "code": "missing_api_key",
    "message": "Missing API key.",
    "requestId": "req_23d0ad35-1ee3-422f-94d6-c9c9fe07c54d",
    "docsUrl": "https://docs.deside.io/directory-api/errors#missing_api_key"
  }
}
```

That is the real answer to a request without a key. Keep `requestId` when you
contact support.

Three groups of routes use other shapes, documented on their own pages: the
public routes and the x402 routes after the key is accepted
([x402 tool catalog](x402-tools.md#errors)), the console proof
([Owner console](console.md)) and the subscription routes
([Subscription and billing](subscription.md)).

## Codes

| Code | Status |
| --- | --- |
| `invalid_request` | 400 |
| `invalid_cursor` | 400 |
| `missing_api_key` | 401 |
| `invalid_api_key` | 401 |
| `api_key_revoked` | 403 |
| `api_key_blocked` | 403 |
| `project_blocked` | 403 |
| `origin_not_allowed` | 403 |
| `agent_not_found` | 404 |
| `rate_limit_exceeded` | 429 |
| `quota_exceeded` | 429 |
| `internal_error` | 500 |

Each code has its own section below, which is where `docsUrl` points.

### `invalid_request`

| Status | When | What to do |
| --- | --- | --- |
| 400 | A filter value, `limit` or identifier is not accepted. | Fix the parameter; `message` names it. |

### `invalid_cursor`

| Status | When | What to do |
| --- | --- | --- |
| 400 | The cursor is malformed or was issued for other filters. | Restart without `cursor`. |

### `missing_api_key`

| Status | When | What to do |
| --- | --- | --- |
| 401 | No `x-api-key` header. | Send the key. |

### `invalid_api_key`

| Status | When | What to do |
| --- | --- | --- |
| 401 | The key is unknown or malformed. | Check the key, or create a new one in the console. |

### `api_key_revoked`

| Status | When | What to do |
| --- | --- | --- |
| 403 | The key was revoked. | Use another key. |

### `api_key_blocked`

| Status | When | What to do |
| --- | --- | --- |
| 403 | The key was blocked. | Contact support with the `requestId`. |

### `project_blocked`

| Status | When | What to do |
| --- | --- | --- |
| 403 | The project is blocked. | Contact support with the `requestId`. |

### `origin_not_allowed`

| Status | When | What to do |
| --- | --- | --- |
| 403 | The request origin is not in the key's allowed list. | Add the origin in the console, or call from an allowed one. |

### `agent_not_found`

| Status | When | What to do |
| --- | --- | --- |
| 404 | No listed agent matches the identifier. Not counted against your quota. | Check the id or slug. |

### `rate_limit_exceeded`

| Status | When | What to do |
| --- | --- | --- |
| 429 | The per-minute rate of your tier is used up. | Wait until `X-RateLimit-Reset`. |

### `quota_exceeded`

| Status | When | What to do |
| --- | --- | --- |
| 429 | The monthly quota of your project is used up. | Wait for the next month (UTC) or move to a higher tier. |

### `internal_error`

| Status | When | What to do |
| --- | --- | --- |
| 500 | Server failure. | Retry later with the same request. |

## When every route answers 404

If every keyed route answers `404` with no envelope and no `code`, the
Directory API is switched off in that environment. A key will not fix it. The
same plain `404` comes from the Pro routes while they are not enabled
([Pro webhooks and exports](pro.md)).
