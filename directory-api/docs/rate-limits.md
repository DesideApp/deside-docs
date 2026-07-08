# Directory API Rate Limits

These are the public preview limits for Directory API keys.

## Tiers

Free:

- `10,000` requests/month
- 30 requests/min

Developer:

- `500,000` requests/month
- 120 requests/min

Pro:

- Directory API read limits are inherited from the configured Pro project
  policy.
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
- requests with a valid key can still consume quota if they fail later during
  request validation
- requests with a missing or invalid key do not consume quota
- rate-limited requests fail before the request handler runs
- webhook management and bulk export have separate Pro limits from read quota

## Operational note

The monthly period is UTC and resets at the start of the next `YYYY-MM`
month. Use the tier values above directly; do not replace them with a generic
"contact us" message.
