# Errors and limits

## Limits

| Routes | Limit | Counted by |
|---|---|---|
| All of `/api` | 500 requests per 15 minutes | IP |
| Lists: `/v2/public/agents`, `/public/x402/tools`, `/public/x402/indices` | 30 a minute | IP |
| Public single reads and counts | 60 a minute | IP |
| Each public family, agents and x402 | 500 a day | IP |
| `/ask` | 10 a minute signed in with a wallet, by account. Others pay per question, 3 requests a minute. See [Ask](ask.md#limits). | Account or IP |

Over the 15-minute limit, the `429` body is plain text, not JSON.

### Headers

Public routes send `ratelimit-limit`, `ratelimit-remaining`, `ratelimit-reset` (seconds) and `ratelimit-policy`.

## Errors

Each family of routes answers errors in its own shape.

### Agent routes (v2)

```json
{ "error": { "code": "not_found", "message": "Agent not found." } }
```

| Status | `code` | What to do |
|---|---|---|
| 400 | `invalid_request` | Fix the query parameter. |
| 400 | `invalid_ref` | Send an `id` or slug up to 128 characters. |
| 404 | `not_found` | Nothing matches. |
| 409 | `ambiguous_ref` | Use an `id` from `candidates`. |
| 429 | | Body `{"error":"RATE_LIMITED"}`. Wait and retry. |
| 500 | `internal_error` | Retry later. |

### x402 and claim routes (v1)

```json
{ "error": "not_found" }
```

| Status | `error` | What to do |
|---|---|---|
| 400 | `invalid_request` | Fix the parameter. On x402 routes, `nextStep` says which. |
| 400 | `invalid_object` | Fix `{type}` or `{id}` of the claim. |
| 404 | `not_found` | Nothing matches. |
| 429 | `RATE_LIMITED` | Wait and retry. |
| 500 | `internal_error` | Retry later. |
| 503 | `claim_unavailable` | Retry later. |

### Ask

| Status | `error` | What to do |
|---|---|---|
| 400 | `INVALID_QUESTION` | Send `question` with 3 to 500 characters. `nextStep` says the rule. |
| 429 | `rate_limited` | Wait `retryAfterSec`. |

The payment errors (`402`, `409`, `503`) are on the [Ask](ask.md#errors) page.
