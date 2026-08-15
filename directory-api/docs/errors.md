# Directory API Errors

## Error envelope

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Invalid request.",
    "requestId": "req_...",
    "docsUrl": "https://docs.deside.io/directory-api/errors#invalid_request"
  }
}
```

## Public codes

- `invalid_request`
- `invalid_cursor`
- `agent_not_found`
- `missing_api_key`
- `invalid_api_key`
- `api_key_revoked`
- `api_key_blocked`
- `project_blocked`
- `project_not_found`
- `origin_not_allowed`
- `quota_exceeded`
- `rate_limit_exceeded`
- `internal_error`

## Status mapping

| Code | HTTP status |
|---|---:|
| `invalid_request` | 400 |
| `invalid_cursor` | 400 |
| `missing_api_key` | 401 |
| `invalid_api_key` | 401 |
| `api_key_revoked` | 403 |
| `api_key_blocked` | 403 |
| `project_blocked` | 403 |
| `origin_not_allowed` | 403 |
| `agent_not_found` | 404 |
| `project_not_found` | 404 |
| `quota_exceeded` | 429 |
| `rate_limit_exceeded` | 429 |
| `internal_error` | 500 |

`api_key_revoked` is a forbidden state, not a missing-auth state. It returns
`403` because the key is recognized but no longer allowed.

`project_not_found` belongs to the owner console routes: the wallet that signed
the console proof has no project yet. It never appears on the API-key read
surface.

## When the whole surface answers 404

The Directory API block is mounted behind `DIRECTORY_API_ENABLED`. When an
environment has it off, these routes do not exist at all: the answer is a plain
`404` with no Directory API error envelope, which is how you tell "not enabled
here" from "enabled and rejecting you". The same is true of the two Pro
surfaces, which need their own switch on top of that one: bulk export and
webhooks are only mounted when their rollout is enabled.

If you are testing against a staging environment and every route answers `404`
without a `code`, the surface is off; a key will not fix it.

## Support guidance

Clients should log `requestId` and the error `code`. The message is useful for
humans, but the code is the stable machine contract.
