# Errors and limits

## Tool errors

A tool that fails returns `isError: true` and, in `content[0].text`, a JSON object:

```json
{ "error": "INVALID_INPUT", "status": 400, "message": "…", "data": { } }
```

| `error` | Status | When | What to do |
|---|---|---|---|
| `AUTH_REQUIRED` | 401 | No valid session. | Sign in again. |
| `insufficient_scope` | 403 | Your token lacks the scope; `requiredScope` says which. | Sign in again asking for that scope. |
| `INVALID_INPUT` | 400 | Bad parameters. | Read `message`; on launchpad tools, `data` holds the Launchpad's answer. |
| `PAYMENT_REQUIRED` | 402 | Launchpad: the transaction does not pay its Arweave storage. | Use the transaction the tool returned, unchanged. |
| `forbidden` | 403 | Launchpad: you are not the creator of the token. | Use the creator wallet. |
| `NOT_FOUND` | 404 | Nothing matches: no agent, no pool for that mint, or a transaction not found yet. | Check the id; retry a transaction a few seconds later. |
| `CONFLICT` | 409 | The state does not allow it: nothing to claim, already graduated, not ready to migrate. | |
| `transaction_failed` | 422 | Launchpad: the transaction is too big, its simulation failed, or it failed on chain. `data` carries `hint` and `logs`. | Prepare it again. |
| `RATE_LIMIT` | 429 | Directory and identity tools: over the limit. | Wait and retry. |
| `RATE_LIMITED` | 429 | Launchpad tools: over the limit. | Wait and retry. |
| `launchpad_unavailable` | 503 | The Launchpad is down or busy. | Retry later. |
| `UNKNOWN` | 500 or the original | Anything else. | Retry later. |

## Session errors

These come as an HTTP response on `/mcp`, before any tool runs:

| Status | `error` | When |
|---|---|---|
| 400 | `session_required` | A request after `initialize` without `mcp-session-id`. |
| 400 | `invalid_request` | `initialize` sent with `mcp-session-id`. |
| 401 | `AUTH_REQUIRED` | `initialize` without a valid bearer token. |
| 401 | `auth_required` | Any other request without a bearer token. |
| 401 | `invalid_token` | An expired or unknown bearer token. Refresh it. |
| 403 | `session_wallet_mismatch` | The bearer token belongs to another wallet than this session. |
| 404 | `session_not_found` | Unknown or closed session. Run `initialize` again. |
| 409 | `session_conflict` | Two `initialize` for the same wallet at the same moment. Retry one. |
| 413 | `payload_too_large` | Body over 1 MB. |

## OAuth errors

`/oauth/register` and `/oauth/token` answer `400` with `{"error": "…", "error_description": "…"}`: `invalid_client_metadata`, `invalid_redirect_uri`, `invalid_scope`, `invalid_request`, `invalid_grant`, `unsupported_grant_type`. Over the limit: `429 RATE_LIMITED`.

## Limits

| What | Limit |
|---|---|
| Launchpad, per wallet | 60 requests and 2 launches a minute |
| Launchpad, whole | 600 requests and 30 launches a minute |
| `/oauth/register` | 30 a minute per IP |
| Other `/oauth/*` | 60 a minute per IP |
| Sign link | 2 minutes, once |
| Prepared transaction | About 60 seconds |
| `ask_directory` | Shared by every MCP user: one Ask window for the whole server |

