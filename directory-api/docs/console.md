# Owner Console

The owner console routes manage a Directory API project: its keys and its
usage. They belong to the project owner, not to an API key, and they
authenticate with a **console proof**: a short-lived bearer token obtained by
signing a message with the owner wallet. The console at
`https://deside.io/developer/api` calls these routes for you.

A project is created the first time its owner creates a key. The project is
always resolved from the wallet inside the proof, never from an id in the
request, so an owner can only act on their own project.

## Get a console proof

1. `GET /api/v1/directory/console/nonce` returns `{ "nonce": "..." }`. It needs
   no credential.
2. Sign a message that contains `Nonce: <nonce>` with the owner wallet.
3. `POST /api/v1/directory/console/auth` with `{ pubkey, signature, message }`.
4. The answer carries `consoleProof`, `audience` (`directory-console`) and
   `expiresInSeconds` (600). Send the proof as `Authorization: Bearer <proof>`.

This request gets a nonce:

```bash
curl -sS "https://api.deside.io/api/v1/directory/console/nonce"
```

The proof has no refresh: when it expires, sign again. It is accepted only on
the console routes, and a chat session cookie is not accepted on them.

## Routes

| Method and path | What it does |
| --- | --- |
| `GET /api/v1/directory/keys` | Lists the project's keys and the project. |
| `POST /api/v1/directory/keys` | Creates a key. Body `{ name?, allowedOrigins? }`. Answers `201` with the raw `key` (shown once), `apiKey` and `project`. |
| `PATCH /api/v1/directory/keys/{keyId}` | Renames a key or changes its allowed origins. Body `{ name?, allowedOrigins? }`. |
| `DELETE /api/v1/directory/keys/{keyId}` | Revokes a key. Answers `{ revoked: true, apiKey }`. |
| `GET /api/v1/directory/usage` | Returns the project's usage and quota, plus `tiers`: `monthlyRequests` and `requestsPerMinute` of every tier. |

The subscription routes use the same proof and are on
[Subscription and billing](subscription.md).

## Errors

| Status | Body | When |
| --- | --- | --- |
| `400` | `{ "error": "MISSING_FIELDS" }` | `pubkey`, `signature` or `message` missing on `/console/auth`. |
| `401` | `{ "error": "INVALID_SIGNATURE" }` | The signature does not match. |
| `401` | `{ "error": "MISSING_CONSOLE_PROOF" }` | No `Authorization` header on a console route. |
| `401` | `{ "error": "INVALID_CONSOLE_PROOF" }` | The proof is expired, malformed or for another audience. Sign again. |
| `400` | keyed error envelope, `invalid_request`, message `Directory API project not found.` | `GET /usage` before the wallet has a project. Create a key first. |
| `400` | keyed error envelope, `invalid_request`, message `API key not found.` | `PATCH` or `DELETE` of a key that is not in your project. |
| `403` | keyed error envelope, `project_blocked` | The project is blocked. |

`GET /keys` before the wallet has a project answers `200` with `keys: []` and
`project: null`.

The nonce and auth routes share the rate limits of the Deside sign-in.
