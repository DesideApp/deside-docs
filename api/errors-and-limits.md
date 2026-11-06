# Errors and limits

## Limits

| Routes | Limit | Counted by |
|---|---|---|
| All of `/api` | 500 requests per 15 minutes | IP |
| Lists: `/v2/public/agents`, `/public/x402/tools`, `/public/x402/indices` | 30 a minute | IP |
| Public single reads and counts | 60 a minute | IP |
| Each public family, agents and x402 | 500 a day | IP |
| `/ask` | 10 a minute signed in with a wallet, by account. Others pay per question, 3 requests a minute. See [Ask](ask.md#limits). | Account or IP |
| `/v2/paid` | 60 answers with status 400 or above a minute. A paid `200` does not count. See [Paid Reads](paid-reads.md#limits). | IP |

Over the 15-minute limit, the `429` body is plain text, not JSON.

### Headers

Public routes send `ratelimit-limit`, `ratelimit-remaining`, `ratelimit-reset` (seconds) and `ratelimit-policy`.

## Errors

Each family of routes answers errors in its own shape. With `?minor=1.1`, the agent and x402 routes answer every error as `{ "error": { "code", "message" } }` with five codes instead, and `GET /api/v2/public/agents/{ref}/holdings` always does: see [The Common Contract](contract.md#errors).

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

### Paid reads

The paid reads answer the errors of their free route, and their payment errors are on the [Paid Reads](paid-reads.md#errors) page.
