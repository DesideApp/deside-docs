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
- `collection`
- `collectionCase`
- `updatedSince`

Notes:

- `limit` defaults to `50`
- `limit` is constrained to `1..100`
- the response returns `agents` plus `pagination`
- `cursor` is opaque
- there is no free-text search parameter on this route: filters are exact
  values, and a `q` sent here is ignored
- results are ordered by `updatedAt` descending, then by internal id; the
  cursor encodes that position together with a hash of the active filters, so
  changing a filter invalidates the cursor
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

### `GET /api/v1/directory/agents/:id/trust`

Notes:

- accepts the same identifier rules as the detail route
- returns the trust shape plus a `generatedAt` timestamp
- may redirect to the canonical slug with `301`
- returns `agent_not_found` when no single visible match resolves

## Public product surface

These routes need no key and consume no quota. They serve the open product
(the public catalogue and its counters), not the developer contract above.

### `GET /api/v1/public/agents/stats-summary`

One measured snapshot of the Deside catalogue. No parameters.

What the numbers count:

| Key | Counts | Unit |
|---|---|---|
| `indexed` | agents listed in the Deside catalogue | agents |
| `registered` | deprecated alias of `indexed`, same number | agents |
| `connected` | listed agents that completed OAuth through Deside's MCP | agents |
| `respondingAgents` | listed agents with at least one live endpoint | agents |
| `respondingByKind` | the same, split by `mcp`, `a2a`, `x402` | agents |
| `byCategory` | listed agents per category | agents |
| `endpoints` | per protocol, `declared` / `checked` / `alive` | URLs |
| `topSkills` | `{ label, n }` per skill | agents |
| `signalCounts` | agents declaring `mcp`, `a2a`, `x402`, `web`, `x` | agents |
| `measuredAt` | when the snapshot was taken | timestamp |

Notes:

- `indexed` counts AGENTS, not registry entries. One agent present in three
  registries is one agent here. The number matches the `total` of
  `GET /api/v1/public/agents`, because both count the same catalogue
- `endpoints` is the only block counted in URLs, and the two units do not
  convert into each other. One URL can be declared by many agents, so `alive`
  is routinely far smaller than `respondingAgents`, and both are right. Read
  `alive` as "how many endpoints answer" and `respondingAgents` as "how many
  agents have one"
- `byCategory` always carries the same eleven category keys, and a category
  with no agents is a measured `0`. Its sum is lower than `indexed` because
  unclassified agents are not spread across categories
- the snapshot is written once a day. Between writes the response repeats the
  last one, with its own `measuredAt`: the date is never hidden or moved
  forward
- a key that is absent was not measured. Absence is never served as `0`, and
  when no snapshot exists at all, `indexed` and `registered` are `null`
- the response is cached for 300 seconds

- being indexed says nothing about whether the agent answers, is reachable in
  chat, or is verified: those are `respondingAgents`, `connected` and the
  verified check, and each is counted on its own
- `registered` is kept only so existing clients do not break, and will be
  removed. Read `indexed`

### `GET /api/v1/public/agents`

Notes:

- the open catalogue list: minimal card per agent plus `total`
- `total` counts the same agents as `indexed` above
- this is the product shape, not the Directory API contract; the vendible
  contract is `GET /api/v1/directory/agents`

## Ask

### `POST /api/v1/ask`

One natural-language question about the directory, answered from measured
directory facts. This route is deliberately outside the API-key surface: it
does not read `x-api-key`, and it does not consume Directory API quota.

The historical address `POST /api/v1/directory/agents/ask` still answers as an
alias and will be retired; integrate against `/api/v1/ask`.

- auth: none required. A logged-in browser session upgrades the request to the
  human lane; everything else is the headless lane
- body: `{ "question": "..." }`, 3 to 500 characters
- rate limit: 10 requests/min per IP on the human lane, 3 requests/min per IP
  on the headless lane
- response: `answer`, `agents`, `claimed`, `claimedTotal`, `unmet`, `intent`,
  `measuredAt`, `degraded`
- this route never returns `500`. When the engine cannot answer it returns
  `200` with `degraded: true` and empty facts
- `400 INVALID_QUESTION` when the question is outside the length bounds
- when the whole feature is disabled the route answers `404`, indistinguishable
  from not existing

The headless lane is the one that will eventually be paid. When paid access is
switched on, that lane answers `402` with an x402 `accepts` block instead of an
answer. The price is not set yet, so this is announced, not live.

## Owner console boundary

The following routes belong to the project owner, not to a Directory API key.
They authenticate with a console proof, which is a short-lived bearer token
obtained by signing a nonce with the owner wallet. The chat session cookie does
not open these routes: the proof carries its own audience
(`aud=directory-console`) and nothing else is accepted.

### `GET /api/v1/directory/console/nonce`

- auth: none. This is the entry point
- returns the nonce to sign with the owner wallet
- rate-limited with the same limiter as the chat login nonce

### `POST /api/v1/directory/console/auth`

- auth: none. The signature is the credential
- body: the signed nonce plus the owner wallet public key
- returns the console proof to send as `Authorization: Bearer <proof>`
- rate-limited with the same limiter as the chat login

### `GET /api/v1/directory/keys`

- auth: console proof
- response envelope: owner-managed key list
- main errors: unauthorized, forbidden, internal_error
- example:

```bash
curl -sS "https://api.deside.io/api/v1/directory/keys" \
  -H "Authorization: Bearer $DESIDE_CONSOLE_PROOF"
```

### `POST /api/v1/directory/keys`

- auth: console proof
- response envelope: created key metadata, including the raw key once
- main errors: unauthorized, forbidden, invalid_request, internal_error

### `PATCH /api/v1/directory/keys/:keyId`

- auth: console proof
- response envelope: updated key metadata
- main errors: unauthorized, forbidden, invalid_request, internal_error

### `DELETE /api/v1/directory/keys/:keyId`

- auth: console proof
- response envelope: deleted key acknowledgement
- main errors: unauthorized, forbidden, key_not_found, internal_error

### `GET /api/v1/directory/usage`

- auth: console proof
- response envelope: usage and quota summary, plus a `tiers` object with the
  live `monthlyRequests` and `requestsPerMinute` of every tier
- main errors: unauthorized, forbidden, internal_error

These routes belong to the owner console boundary. They do not appear in the
API-key public OpenAPI scope.

## Subscription and billing

The paid tier of a project is bought with a recurring on-chain authorization in
USDC, signed by the owner wallet. Deside never signs and never holds the key:
the backend returns an unsigned transaction, the wallet signs it, and the
backend then reads the chain before persisting anything.

All five routes use the console proof. The project is always resolved from the
wallet inside the proof, never from an id in the body.

### `GET /api/v1/directory/subscription`

- returns `enabled`, `tier`, `billingStatus`, `subscription` and `plan`
- `plan` describes the on-chain plan (`planAddress`, `amount` in base units,
  `mint`, `periodDays`) and is `null` when those terms cannot be read
- when the mint is one this rail knows, the plan also carries `mintSymbol` and
  `mintDecimals`, so a client can render "20 USDC" without deciding on its own
  what the token is. The signed figure is always `amount`, in base units;
  scaling it is presentation, never the contract
- `subscription.status` is one of `none`, `active`, `past_due`, `canceled`

### `POST /api/v1/directory/subscription/intent`

- body: `{ "tier": "developer" }`
- returns an unsigned `transaction`, a `step`, the `delegation` it will create
  and the `plan` terms (`amount`, `mint`, `periodSeconds`)
- `step` is `init_authority` when a setup transaction is still missing, and
  `create_delegation` when this transaction is the one that grants the
  recurring charge
- calling it has no effect and can be repeated

### `POST /api/v1/directory/subscription/accept`

- body: `{ "tier": "developer" }`
- confirms against the chain what the owner signed, and persists it
- it does NOT grant the tier: the tier changes when the first charge settles

### `POST /api/v1/directory/subscription/cancel-intent`

- returns the unsigned transaction that revokes the recurring authorization
- available even when new subscriptions are closed: a flag stops the sale, not
  the exit

### `POST /api/v1/directory/subscription/cancel-confirm`

- confirms the revocation against the chain
- the paid tier stays available until the paid period ends

Billing rules that the API enforces:

- one cycle is 30 days
- changing tier is not an in-place operation: cancel the current subscription
  and subscribe again. A new subscription charges on the day it settles and
  starts a fresh 30-day cycle. Asking for a different tier while one is live
  answers `dapi_subs_tier_change_not_supported`
- a project on a manual billing agreement cannot use this rail and answers
  `dapi_subs_billing_manual`

Actionable error codes: `dapi_subs_disabled`, `dapi_subs_tier_invalid`,
`dapi_subs_billing_manual`, `dapi_subs_payer_not_owner`,
`dapi_subs_project_blocked`, `dapi_subs_project_not_found`,
`dapi_subs_already_active`, `dapi_subs_tier_change_not_supported`,
`dapi_subs_delegation_not_found`, `dapi_subs_delegation_terms_mismatch`,
`dapi_subs_delegation_already_used`, `dapi_subs_state_changed`,
`dapi_subs_not_cancelable`, `dapi_subs_delegation_still_active`,
`dapi_subs_delegation_already_revoked`. Anything else is operator
configuration and is reported as `dapi_subs_unavailable`.

## Pro webhooks

These routes are Pro owner console routes, authenticated with the console
proof. They are not API-key read routes.

Pro webhooks are documented ahead of rollout and are not yet enabled in
production; until the Pro rollout enables them, these routes are not mounted.

### `GET /api/v1/directory/webhooks`

- auth: console proof
- returns `{ "webhooks": [] }`
- secret material is never returned

### `POST /api/v1/directory/webhooks`

- auth: console proof
- body: `{ "url": "https://example.com/webhook", "events": ["agent.indexed"] }`
- returns `{ "webhook": {}, "secret": "whsec_..." }`
- the secret is returned once on create

### `DELETE /api/v1/directory/webhooks/:id`

- auth: console proof
- soft-deletes the webhook subscription

### `POST /api/v1/directory/webhooks/:id/rotate-secret`

- auth: console proof
- returns a new `whsec_...` secret once

### `POST /api/v1/directory/webhooks/:id/test`

- auth: console proof
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

Pro bulk export is documented ahead of rollout and is not yet enabled in
production; until the Pro rollout enables it, these routes are not mounted.

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
