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
| `quota_exceeded` | 429 |
| `rate_limit_exceeded` | 429 |
| `internal_error` | 500 |

`api_key_revoked` is a forbidden state, not a missing-auth state. It returns
`403` because the key is recognized but no longer allowed.

## Support guidance

Clients should log `requestId` and the error `code`. The message is useful for
humans, but the code is the stable machine contract.
