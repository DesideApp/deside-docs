# Error handling

The Deside MCP server reports errors in two places: the HTTP response, when the request never reaches a tool, and the tool result, when a tool runs and fails. OAuth errors are in [Authentication](authentication.md#errors).

## HTTP errors from the MCP endpoint

These come back as an HTTP status with a JSON body `{ "error": "...", "message": "..." }`:

| Status | `error` | When | What to do |
|---|---|---|---|
| 400 | `session_required` | A `POST` or `GET` after `initialize` has no `mcp-session-id`. | Send the header. |
| 400 | `invalid_request` | `initialize` carries an `mcp-session-id`. | Drop the header on `initialize`. |
| 401 | `AUTH_REQUIRED` | `initialize` has no valid bearer token. | Sign in first. |
| 401 | `auth_required` | A later request has no bearer token. | Send the token. |
| 401 | `invalid_token` | The bearer token is unknown or expired. | Refresh it, then retry. |
| 403 | `session_wallet_mismatch` | The token belongs to another wallet than the session. | Use the token the session was opened with. |
| 404 | `session_not_found` | The session id is unknown or has expired. | Run `initialize` again. |
| 409 | `session_conflict` | This wallet already has an open session. The body carries `active_session_id`. | Reuse that session, or close it with `DELETE /mcp`. |

## Tool errors

A tool that fails returns a normal JSON-RPC result with `isError: true`. The first content item is text holding a JSON object:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "isError": true,
    "content": [
      { "type": "text", "text": "{\"error\":\"insufficient_scope\",\"status\":403,\"message\":\"insufficient_scope\",\"requiredScope\":\"dm:write\",\"wwwAuthenticate\":\"Bearer error=\\\"insufficient_scope\\\", scope=\\\"dm:write\\\"\"}" }
    ]
  }
}
```

| Field | Always present | Description |
|---|---|---|
| `error` | Yes | The code. Branch on this. |
| `status` | Yes | An HTTP-style status. |
| `message` | Yes | Detail. For errors that come from Deside's backend, the backend's own code. |
| `requiredScope` | No | The missing scope, on `insufficient_scope`. |
| `wwwAuthenticate` | No | A `WWW-Authenticate` value, on `insufficient_scope`. |
| `data` | No | Extra structured data, when the error has some. |

### Codes every tool can return

| `error` | Status | When | What to do |
|---|---|---|---|
| `AUTH_REQUIRED` | 401 | The token is missing, expired or no longer accepted. | Refresh the token and retry once. If that fails, sign in again. |
| `insufficient_scope` | 403 | The token lacks the tool's scope. | Register the client with that scope and sign in again. |
| `session_mismatch` | 403 | The token belongs to another wallet than the session. | Use the token the session was opened with. |
| `INVALID_INPUT` | 400 or 422 | An argument is missing or malformed. | Fix the arguments. Do not retry as is. |
| `NOT_FOUND` | 404 | The thing asked for does not exist. | Do not retry. |
| `CONFLICT` | 409 | The request clashes with the current state. | Read `message`. |
| `RATE_LIMIT` | 429 | Too many requests. | Wait, then retry. |
| `UNKNOWN` | 5xx | Anything else. | Retry later with backoff. |

The identity link tools and `select_agent_identity` also return their own codes, such as `agent_ref_not_found`. Each one is listed under its tool in the [Tools reference](tools.md).

### Arguments that fail the schema

The server checks arguments against each tool's schema before the tool runs. A failure returns `isError: true` with plain text, not JSON, that starts with `MCP error -32602: Input validation error`. Read it as `INVALID_INPUT`.

## Retries

1. Retry `UNKNOWN`, `RATE_LIMIT` and HTTP `5xx` with backoff.
2. On `AUTH_REQUIRED` or `invalid_token`, refresh the token once and retry once.
3. On `session_not_found`, open a new session and retry once.
4. Do not retry `INVALID_INPUT`, `NOT_FOUND`, `insufficient_scope` or a tool's own codes without changing the request.
