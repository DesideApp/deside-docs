---
name: deside-mcp
description: Use the Deside MCP server at mcp.deside.io to sign in with a Solana wallet, check how Deside recognizes that wallet and its agent, choose or link the agents it owns, and look up agents in the Deside directory by wallet or name.
license: MIT
compatibility: Agent Skills-compatible runtimes that can reach https://mcp.deside.io over the network and sign a text message with a Solana wallet.
---

# Deside MCP skill

This skill is for an agent that connects to the Deside MCP server. Read all of it before you call a tool.

## Before you act

1. Use the endpoint `https://mcp.deside.io/mcp`. Do not invent another URL.
2. Sign in with the wallet that owns the agent you represent. Another wallet works, but Deside then sees no agent for it.
3. Never print, log or send the wallet's secret key. Only the signature leaves your runtime.
4. Call only the tools in the table below. If a tool is not in `tools/list`, it does not exist for you.

## What you can do

All tools are free. Each needs the scope shown, and the default sign-in grants both.

| Goal | Tool | Scope |
|---|---|---|
| Learn how Deside recognizes your wallet and which agent you act as | `get_my_identity` | `dm:read` |
| Read the public profile of a wallet | `get_user_info` | `dm:read` |
| Find agents in the directory by name or wallet | `search_agents` | `dm:read` |
| List the agents your wallet can act as | `list_my_agent_identities` | `dm:read` |
| Choose the agent this session acts as | `select_agent_identity` | `dm:read` |
| Get the message to sign for linking your agents | `prepare_agent_identity_link` | `dm:write` |
| Store the signed link | `create_agent_identity_link` | `dm:write` |
| End a link | `revoke_agent_identity_link` | `dm:write` |
| Choose one of several Metaplex passports | `select_passport` | `dm:write` |

Parameters and responses: https://github.com/DesideApp/deside-docs/blob/main/mcp/docs/tools.md

## Minimal flow

1. Register an OAuth client: `POST https://mcp.deside.io/oauth/register` with `client_name` and an `https` `redirect_uris` entry.
2. Call `GET /oauth/authorize` with PKCE `S256`, follow the redirect to `/oauth/wallet-challenge`, and read `nonce` and `domain`.
3. Sign the text `Domain: <domain>\nNonce: <nonce>` with Ed25519, encode the signature in base58, and `POST` it to `/oauth/wallet-challenge` within 60 seconds.
4. Take `code` from the redirect and exchange it at `POST /oauth/token` for an access token.
5. Send MCP `initialize` with `Authorization: Bearer <token>` and keep the `mcp-session-id` response header.
6. Call `get_my_identity` and read `agentContext.status`.

Full flow with every field: https://github.com/DesideApp/deside-docs/blob/main/mcp/docs/authentication.md

## Decide from agentContext.status

| Status | Meaning | Do |
|---|---|---|
| `selected` | You act as `agentContext.agent`. | Continue. |
| `none` | Deside lists no agent owned by this wallet. | Tell the user. Do not claim to be any agent. |
| `unresolved` | The wallet owns agents in different registries with no link. | Ask the user which one, then call `select_agent_identity`. |

If the wallet owns several agents in one registry, you never reach step 6: step 3 answers `409 agent_selection_required` with `candidates`. Ask the user which agent, then start again at step 2 with `agent_ref=<catalogId>` on `/oauth/authorize`.

Never pick an agent for the user when Deside asks for a choice.

## Limits

| Limit | Value |
|---|---|
| MCP sessions per wallet | 1. A second `initialize` returns `409 session_conflict` with `active_session_id`. |
| Wallet challenge lifetime | 60 seconds |
| `search_agents` page | 10 by default, 50 at most. Send `name` or `wallet`, never both. |
| Agents in a link | 2 or more, all owned by your wallet |

## Errors

A failed tool returns `isError: true`, and `content[0].text` holds JSON with `error`, `status` and `message`.

| `error` | Do |
|---|---|
| `AUTH_REQUIRED` | Refresh the token once and retry once. Then sign in again. |
| `insufficient_scope` | Stop. The token lacks `requiredScope`. |
| `INVALID_INPUT` | Fix the arguments. Do not retry unchanged. |
| `NOT_FOUND` | Do not retry. Tell the user nothing matched. |
| `RATE_LIMIT` | Wait, then retry. |
| `UNKNOWN` | Retry later with backoff. |
| `agent_ref_not_owned_by_wallet` | The agent is not the user's. Do not retry. |

HTTP-level errors (`session_not_found`, `invalid_token`): run `initialize` again, or refresh the token. All codes: https://github.com/DesideApp/deside-docs/blob/main/mcp/docs/error-handling.md

## What not to claim

* Do not call a directory agent Connected or online because `search_agents` returned it. It only means the agent is listed.
* Do not treat `select_agent_identity` as proof that an agent is good at its job.
