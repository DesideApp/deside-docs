# Directory API Endpoints

## Developer read surface

These routes use `x-api-key: dapi_...`.

### `GET /api/v1/directory/agents`

Query parameters:

- `limit`
- `cursor`
- `registry`
- `service`
- `capability`
- `updatedSince`

Notes:

- `limit` defaults to `50`
- `limit` is constrained to `1..100`
- the response returns `agents` plus `pagination`
- `cursor` is opaque
- the envelope is the Directory API contract, not the legacy
  `/api/v1/public/agents` shape

### `GET /api/v1/directory/agents/:id`

Notes:

- accepts a catalog id or slug
- may redirect to the canonical slug
- returns `agent` when a single visible match is resolved
- returns a disambiguation payload when more than one visible match is found

### `GET /api/v1/directory/agents/:id/profile`

Notes:

- accepts the same identifier rules as the detail route
- returns the richer profile shape
- may redirect to the canonical slug

## Owner and dashboard boundary

The following routes are part of the owner/session boundary and are not part of
the API-key public read scope.

### `GET /api/v1/directory/keys`

- auth: owner/session via `protectRoute`
- response envelope: owner-managed key list
- main errors: unauthorized, forbidden, internal_error
- example:

```bash
curl -sS "$DESIDE_API_BASE_URL/api/v1/directory/keys"
```

### `POST /api/v1/directory/keys`

- auth: owner/session via `protectRoute`
- response envelope: created key metadata
- main errors: unauthorized, forbidden, invalid_request, internal_error

### `PATCH /api/v1/directory/keys/:keyId`

- auth: owner/session via `protectRoute`
- response envelope: updated key metadata
- main errors: unauthorized, forbidden, invalid_request, internal_error

### `DELETE /api/v1/directory/keys/:keyId`

- auth: owner/session via `protectRoute`
- response envelope: deleted key acknowledgement
- main errors: unauthorized, forbidden, key_not_found, internal_error

### `GET /api/v1/directory/usage`

- auth: owner/session via `protectRoute`
- response envelope: usage and quota summary
- main errors: unauthorized, forbidden, internal_error

These routes belong to the owner dashboard boundary. They do not appear in the API-key public OpenAPI scope.

## Pro webhooks

These routes are Pro owner/session configuration routes. They are not API-key
read routes.

### `GET /api/v1/directory/webhooks`

- auth: owner/session via `protectRoute`
- returns `{ "webhooks": [] }`
- secret material is never returned

### `POST /api/v1/directory/webhooks`

- auth: owner/session via `protectRoute`
- body: `{ "url": "https://example.com/webhook", "events": ["agent.indexed"] }`
- returns `{ "webhook": {}, "secret": "whsec_..." }`
- the secret is returned once on create

### `DELETE /api/v1/directory/webhooks/:id`

- auth: owner/session via `protectRoute`
- soft-deletes the subscription

### `POST /api/v1/directory/webhooks/:id/rotate-secret`

- auth: owner/session via `protectRoute`
- returns a new `whsec_...` secret once

### `POST /api/v1/directory/webhooks/:id/test`

- auth: owner/session via `protectRoute`
- queues a `test.ping` webhook delivery

Webhook deliveries include these signing headers:

- `X-Deside-Event-Id`: stable delivery event id, such as `evt_...`
- `X-Deside-Timestamp`: Unix timestamp in seconds
- `X-Deside-Signature-256`: `sha256=<hex digest>`

Receivers verify the signature by computing HMAC-SHA256 with the webhook secret
over this exact base string:

```txt
timestamp + "." + raw_body
```

Use the unmodified raw JSON request body bytes after the dot. Reject timestamps
outside a 5 minute tolerance window, and store processed `X-Deside-Event-Id`
values for idempotency because retries may deliver the same event more than
once. Delivery order is not strict; consumers should fetch the current agent
profile when they need the latest state.

## Pro bulk export

These routes use `x-api-key: dapi_...` and require a Pro project.

### `POST /api/v1/directory/exports`

- auth: Directory API key
- queues an async `jsonl.gz` export job
- returns `202` with the export status envelope

### `GET /api/v1/directory/exports/:id`

- auth: Directory API key
- returns export status
- when ready, returns a short-lived signed download URL

## Response conventions

- successful list responses use `200`
- missing or invalid identifiers use public error codes
- canonical slug redirects use `301`

## Filter behavior

- `registry` accepts the known registry aliases used by the read model
- `service` accepts a narrow service filter
- `capability` accepts a narrow capability filter
- `updatedSince` must be a valid ISO date string
