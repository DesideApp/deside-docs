# Directory API Rate Limits

These are the public preview limits for Directory API keys.

## Tiers

The useful unit here is not the single request. A full sweep of the directory
costs about 104 requests today (one page of 100 agents per request), so it is
worth reading these numbers as how much of the directory each tier lets you
take, and how often.

Free:

- `2,500` requests/month
- 30 requests/min
- roughly 24 full sweeps a month: enough to explore, to run a small panel, and
  to keep a weekly sync. Not enough to mirror the directory daily.

Developer:

- `50,000` requests/month
- 90 requests/min
- a full daily sync costs about 3,120 requests a month, so this leaves room to
  spare for lookups on top of it.

Pro (webhooks and bulk export are pre-rollout: documented, not yet enabled in
production):

- `500,000` requests/month
- 180 requests/min
- Webhook subscriptions: 3 active subscriptions per project.
- Webhook delivery attempts: 5 attempts with exponential backoff.
- Bulk export: 1 `jsonl.gz` export per day.
- Bulk export active jobs: 1 queued or running job per project.
- Export download URLs expire after 24 hours by default.

## Headers

Successful and error responses can include:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `X-Deside-Quota-Limit`
- `X-Deside-Quota-Remaining`

## Error codes

- `rate_limit_exceeded` means the current minute window is exhausted
- `quota_exceeded` means the monthly quota is exhausted

## Response behavior

- the rate window is 60 seconds
- rate-limit state is enforced per project and key
- monthly quota is enforced per project
- creating or rotating keys does not reset the monthly quota
- multiple keys do not multiply the requests-per-minute allowance
- `X-RateLimit-Reset` is a Unix epoch value in seconds
- `X-Deside-Quota-Remaining` reflects the monthly project budget
- not-found responses (`404`) are refunded and do not consume monthly quota,
  so exploring a missing agent is not billed
- other handled errors with a valid key (such as a malformed request) still
  consume quota
- requests with a missing or invalid key do not consume quota
- rejected over-quota requests are not counted as served requests; they
  increment a separate quota-exceeded counter instead of inflating the used
  count
- rate-limited requests fail before the request handler runs
- webhook management and bulk export have separate Pro limits from read quota

## Where these numbers come from

The limits are operator configuration, not constants baked into a client. The
owner console reads them from `GET /api/v1/directory/usage`, which returns a
`tiers` object with `monthlyRequests` and `requestsPerMinute` for every tier.
If you are building something that shows quotas, read them from there too:
values copied into a client go stale silently the day an operator changes them.

## Operational note

The monthly period is UTC and resets at the start of the next `YYYY-MM`
month. Use the tier values above directly; do not replace them with a generic
"contact us" message.
