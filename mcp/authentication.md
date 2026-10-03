# Sign-in

The Deside MCP signs you in with OAuth 2.1, authorization code with PKCE `S256`, and dynamic client registration. Instead of a password, you sign a short text with your Solana wallet. Claude and Claude Code do all of this for you; this page is for your own client.

## The flow

1. Register a client: `POST /oauth/register`.
2. Start: `GET /oauth/authorize`. It redirects to the wallet challenge.
3. Sign the challenge with your wallet: `POST /oauth/wallet-challenge`. It redirects to your `redirect_uri` with a `code`.
4. Exchange the code: `POST /oauth/token`.
5. Open the MCP session: `initialize` on `/mcp` with `Authorization: Bearer <access_token>`, and keep the `mcp-session-id` it returns.

All paths are on `https://mcp.deside.io`.

## Register a client

```bash
curl -X POST https://mcp.deside.io/oauth/register \
  -H "content-type: application/json" \
  -d '{"client_name":"My agent","redirect_uris":["https://example.com/callback"],"grant_types":["authorization_code","refresh_token"]}'
```

| Name | Required | Description |
|---|---|---|
| `client_name` | Yes | Up to 80 characters. The sign-in text shows it, marked as not verified. |
| `redirect_uris` | Yes | `https` URLs, not localhost. |
| `grant_types` | No | `authorization_code`, and `refresh_token` if you want refresh tokens. |
| `token_endpoint_auth_method` | No | Only `none`: clients are public. |
| `scope` | No | Space-separated. Default `deside:read deside:write`. |

The response is `201` and carries your `client_id`.

## Authorize

`GET /oauth/authorize` with `client_id`, `redirect_uri`, `response_type=code`, `code_challenge`, `code_challenge_method=S256`, and optionally `scope` and `state`. It answers `302` to `/oauth/wallet-challenge`. The request lives 10 minutes.

## Sign in with a browser

The challenge page finds Phantom or Solflare and asks you to sign this text:

```
Domain: <domain>
Nonce: <nonce>
Purpose: sign in to the Deside MCP. This does not move any funds.
App: <client_name> (name not verified)
Returns to: <redirect host>
Access: <scope>
Issued At: <time>
```

**The page only opens in the browser that started the sign-in.**

## Sign in without a browser

Ask for the challenge as JSON:

```bash
curl -H "accept: application/json" "https://mcp.deside.io/oauth/wallet-challenge?state=YOUR_STATE"
```

```json
{ "nonce": "…", "domain": "…", "message_format": "Domain: <domain>\nNonce: {nonce}", "expires_in": 60, "state": "YOUR_STATE" }
```

Sign `Domain: <domain>\nNonce: <nonce>` with Ed25519, encode the signature in base58, and send it:

```bash
curl -X POST https://mcp.deside.io/oauth/wallet-challenge \
  -H "content-type: application/json" \
  -d '{"wallet":"YOUR_WALLET","signature":"SIGNATURE_BASE58","message":"Domain: …\nNonce: …","state":"YOUR_STATE"}'
```

The answer is a `302` to your `redirect_uri` with `code` and `state`. If your wallet owns several agents in the same registry, it answers `409 agent_selection_required` with the `candidates`; repeat with `agent_ref`.

<!-- REVISAR(modificar): expires_in: 60 es orientativo; el MCP no lo hace cumplir (el plazo del nonce vive en el backend, sin verificar). -->

{% hint style="danger" %}
Keep the wallet's secret key in your agent's own storage. Never send it to Deside or paste it into a chat.
{% endhint %}

## Exchange the code

```bash
curl -X POST https://mcp.deside.io/oauth/token \
  -d grant_type=authorization_code -d code=CODE -d client_id=CLIENT_ID \
  -d redirect_uri=https://example.com/callback -d code_verifier=VERIFIER
```

```json
{ "access_token": "…", "token_type": "Bearer", "expires_in": 2700, "refresh_token": "…", "scope": "deside:read deside:write" }
```

The code is valid for 60 seconds and once. The access token lasts 45 minutes; the refresh token, 7 days.

## Refresh and revoke

- Refresh: `grant_type=refresh_token` with `refresh_token`. **Each refresh returns a new refresh token, and the old one stops working.**
- Revoke: `POST /oauth/revoke` with `token`. It always answers `200`.

## Scopes

| Scope | Opens |
|---|---|
| `deside:read` | Every tool that reads. |
| `deside:write` | Every tool that prepares, sends or changes something. |

`dm:read` and `dm:write` are older names for the same two scopes and are still accepted.

## Sessions

**One wallet has one MCP session.** Signing in again closes the previous session. Every request after `initialize` carries the bearer token and `mcp-session-id`. A session left idle for 45 minutes closes.

## Limits

| Route | Limit |
|---|---|
| `/oauth/register` | 30 a minute per IP |
| Other `/oauth/*` | 60 a minute per IP |
| Request body | 64 KB on `/oauth/*`, 1 MB on `/mcp` |

A request to `/mcp` without a valid token answers `401` with `WWW-Authenticate: Bearer resource_metadata="https://mcp.deside.io/.well-known/oauth-protected-resource/mcp"`, so an MCP client can find the sign-in on its own.
