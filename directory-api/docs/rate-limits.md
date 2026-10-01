# Rate Limits

Every keyed request counts against two limits: a per-minute rate and a monthly
quota. The public routes have per-IP limits instead, listed on
[Access model](access-model.md).

## Tiers

| Tier | Requests a month | Requests a minute |
| --- | --- | --- |
| Free | 5,000 | 30 |
| Developer | 50,000 | 90 |
| Pro | 500,000 | 180 |

These are the defaults of the server. The live values for every tier are in
the `tiers` object of `GET /api/v1/directory/usage` ([Owner console](console.md));
read them there if you display them.

A full walk of the agent directory costs one request per 100 agents. The
[Quickstart](quickstart.md#keep-a-copy-in-sync) shows what that cost is today
and how to keep a copy in sync for far less.

## How they are counted

* The minute window is 60 seconds. The rate applies per key and per project:
  several keys do not multiply it.
* The monthly quota applies per project and resets at the start of each UTC
  month. Creating or rotating keys does not reset it.
* A request with a missing, invalid, revoked or blocked key, or from an origin
  the key does not allow, is not counted.
* A request that answers `404` is refunded: looking up a missing agent or tool
  is not billed.
* Any other answer to an accepted key is counted, errors such as `400`
  included.
* A request rejected for rate or quota is not counted as a served request.

## Headers

Keyed responses carry:

| Header | Meaning |
| --- | --- |
| `X-RateLimit-Limit` | Requests allowed in the minute window. |
| `X-RateLimit-Remaining` | Requests left in it. |
| `X-RateLimit-Reset` | When it resets, as a Unix time in seconds. |
| `X-Deside-Quota-Limit` | Requests allowed this month. |
| `X-Deside-Quota-Remaining` | Requests left this month. |

## Errors

| Code | Status | When |
| --- | --- | --- |
| `rate_limit_exceeded` | 429 | The minute window is used up. |
| `quota_exceeded` | 429 | The monthly quota is used up. |
