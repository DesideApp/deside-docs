# Pro Webhooks And Exports

Pro webhooks and Pro bulk exports are two features of the `pro` tier that are
built and not enabled in production. **Until they are enabled, their routes
answer `404` without the Directory API error envelope**, as if they did not
exist.

This page describes them so that a `404` on these paths is not read as a bug.
Do not build on them yet.

## Webhooks

Managed with the console proof ([Owner console](console.md)):

| Method and path | What it does |
| --- | --- |
| `GET /api/v1/directory/webhooks` | Lists subscriptions. Secrets are never returned. |
| `POST /api/v1/directory/webhooks` | Creates one. Body `{ url, events }`. Returns the `whsec_...` secret once. |
| `DELETE /api/v1/directory/webhooks/{id}` | Deletes one. |
| `POST /api/v1/directory/webhooks/{id}/rotate-secret` | Returns a new secret once. |
| `POST /api/v1/directory/webhooks/{id}/test` | Queues a `test.ping` delivery. |

Events: `agent.indexed`, `agent.profile_changed`, `agent.registry_changed`,
`agent.services_changed`, `agent.capabilities_changed`,
`agent.convergence_changed`, `agent.removed`.

Limits: 3 active subscriptions per project, 5 delivery attempts per event.

Each delivery carries `X-Deside-Event-Id`, `X-Deside-Timestamp` (Unix seconds)
and `X-Deside-Signature-256` (`sha256=<hex>`), an HMAC-SHA256 with the
webhook secret over `timestamp + "." + raw_body`.

## Bulk exports

Requested with an API key:

| Method and path | What it does |
| --- | --- |
| `POST /api/v1/directory/exports` | Queues a `jsonl.gz` export. Answers `202`. |
| `GET /api/v1/directory/exports/{id}` | Returns its status and, when ready, a signed download URL. |

Limits: 1 export a day and 1 queued or running export per project. Download
URLs expire after 24 hours by default.
