# Directory API Authentication

## Authentication matrix

| Surface | Auth | Notes |
|---|---|---|
| `/api/v1/directory/agents` reads | `x-api-key: dapi_...` | Canonical Directory API read surface for Free and Developer. A key is required on every tier, including Free. |
| `/api/v1/ask` | none, session upgrades the lane | Outside the key surface. Does not read `x-api-key` and does not consume quota. |
| `/api/v1/directory/keys`, `/usage`, `/subscription`, `/webhooks` | console proof | Owner console. Signature to short-lived bearer, see below. |
| `/api/v1/directory/exports` | `x-api-key: dapi_...` | Pro bulk export uses the API key, not the console proof. |
| `/api/v1/public/agents` | public product surface | Open product reads (minimal list card, per-agent detail, and `stats-summary` counters); not the vendible developer contract and not deprecated. No key, no quota. |
| `/api/v1/mcp` | session auth | MCP uses session-based auth and does not consume Directory API quota. |
| SDK/widget legacy | out of scope | Not part of the Directory API auth model. |

## API-key family

Directory API uses `x-api-key` headers with keys that start with `dapi_...`.
The API key is the canonical auth input for the directory read surface.

## Header

```http
x-api-key: dapi_<public_prefix>_<secret>
```

## Auth states

The public contract exposes these states through the error envelope:

- `missing_api_key`
- `invalid_api_key`
- `api_key_revoked`
- `api_key_blocked`
- `project_blocked`

## Console proof

Managing a project, its keys and its subscription is a different credential
family. It is not the API key, and it is not the chat session cookie either.

1. `GET /api/v1/directory/console/nonce` returns a nonce
2. the owner wallet signs it
3. `POST /api/v1/directory/console/auth` returns a console proof
4. the proof travels as `Authorization: Bearer <proof>` and expires in minutes

The proof carries its own audience (`aud=directory-console`). A chat session
cookie is not accepted on these routes, and the proof is not accepted anywhere
else. It lives in memory in the console: it is never persisted to disk.

The project is always resolved from the wallet inside the proof, never from an
identifier in the request body, so an owner can only ever act on their own
project.

## Boundary

Directory API keys are not bearer tokens. They protect the developer read
surface only and do not apply to owner console routes or MCP session flows.

The three families do not mix: a `dapi_...` key never manages a project, a
console proof never reads the directory on a tier, and an MCP session never
consumes Directory API quota.

## Origin policy

Some keys can be restricted by allowed origin list. When a request fails that
policy, the error code is `origin_not_allowed`.

## Security note

Authentication failure is part of the public API contract. Clients should
expect explicit codes and preserve `requestId` for support.
