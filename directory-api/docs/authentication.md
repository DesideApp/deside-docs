# Directory API Authentication

## Authentication matrix

| Surface | Auth | Notes |
|---|---|---|
| `/api/v1/directory` | `x-api-key: dapi_...` | Canonical Directory API read surface for Free and Developer. |
| `/api/v1/public/agents` | public legacy | Existing public catalog; not the vendible developer contract. |
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

## Boundary

Directory API keys are not bearer tokens. They protect the developer read
surface only and do not apply to owner/session routes or MCP session flows.

Owner/session credential management stays outside the developer read surface.

## Origin policy

Some keys can be restricted by allowed origin list. When a request fails that
policy, the error code is `origin_not_allowed`.

## Security note

Authentication failure is part of the public API contract. Clients should
expect explicit codes and preserve `requestId` for support.
