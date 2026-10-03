# Errors and limits

## Limits

| Routes | Limit | Counted by |
|---|---|---|
| All of `/api` | 500 requests per 15 minutes | IP |
| Public lists: `/public/agents`, `/public/x402/tools`, `/public/x402/indices` | 30 a minute | IP |
| Public single reads and counts | 60 a minute | IP |
| Each public family, agents and x402 | 500 a day | IP |
| `/ask` | 10 a minute signed in on deside.io, 3 otherwise | IP |
| Directory routes, free plan | 5,000 a month and 30 a minute | Key and project |

### Headers

Public routes send `ratelimit-limit`, `ratelimit-remaining`, `ratelimit-reset` (seconds) and `ratelimit-policy`.

Directory routes, with a valid key, also send `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix seconds) and `X-Deside-Quota-Limit`, `X-Deside-Quota-Remaining`.

### Monthly quota

- The month is the UTC calendar month. It resets at 00:00 UTC on the 1st.
- A `404` is refunded. Other errors with a valid key count.
- A request with a missing or bad key does not count.
- A request over the per-minute limit does not count.
- More keys do not add more requests: the quota is per project.

## Errors

Each family of routes answers errors in its own shape.

### Public routes

```json
{ "error": "not_found" }
```

| Status | `error` | What to do |
|---|---|---|
| 400 | `invalid_request` | Fix the parameter. On x402 routes, `nextStep` says which. |
| 400 | `invalid_payload` | Shorten `{ref}` to 128 characters. |
| 400 | `invalid_object` | Fix `{type}` or `{id}` of the claim. |
| 404 | `not_found` | Nothing matches. |
| 429 | `RATE_LIMITED` | Wait and retry. |
| 500 | `internal_error` | Retry later. |
| 503 | `claim_unavailable` | Retry later. |

### Ask

| Status | `error` | What to do |
|---|---|---|
| 400 | `INVALID_QUESTION` | Send `question` with 3 to 500 characters. |
| 429 | `rate_limited` | Wait `retryAfterSec`. |

### Directory routes

```json
{
  "error": {
    "code": "missing_api_key",
    "message": "Missing API key.",
    "requestId": "req_03b560c0-...",
    "docsUrl": "https://docs.deside.io/directory-api/errors#missing_api_key"
  }
}
```

Keep `requestId` when you ask for help. You can send your own in `x-request-id`, up to 128 characters.

| Status | `code` | What to do |
|---|---|---|
| 400 | `invalid_request` | Fix the parameter named in `message`. |
| 400 | `invalid_cursor` | Start again without `cursor`, or with the same filters. |
| 401 | `missing_api_key` | Send `x-api-key`. |
| 401 | `invalid_api_key` | Check the key. |
| 403 | `api_key_revoked` | Create a new key. |
| 403 | `api_key_blocked` | Contact Deside. |
| 403 | `project_blocked` | Contact Deside. |
| 403 | `origin_not_allowed` | The key only accepts some origins. Call from one of them, or remove the restriction. |
| 404 | `agent_not_found` | Nothing matches. Not counted. |
| 404 | `project_not_found` | |
| 429 | `rate_limit_exceeded` | Wait until `X-RateLimit-Reset`. |
| 429 | `quota_exceeded` | Wait for next month. |
| 500 | `internal_error` | Retry later. |

