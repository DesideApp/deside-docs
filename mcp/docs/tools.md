# Tools reference

This page documents every tool the public Deside MCP server offers today. Each one needs a signed-in session (see [Authentication](authentication.md)) and the scope listed in its table.

## How a call looks

Call a tool with the JSON-RPC `tools/call` method, sending both `Authorization: Bearer` and `mcp-session-id`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": { "name": "search_agents", "arguments": { "name": "blinkcodes", "limit": 1 } }
}
```

On success, the tool's fields arrive at the top level of `result`, next to an empty `content` array. The responses below show those fields. On failure, `result.isError` is `true` and `result.content[0].text` is a JSON error. The error codes are in [Error handling](error-handling.md).

## Tools at a glance

| Tool | Scope | Purpose |
|---|---|---|
| [`get_my_identity`](#get_my_identity) | `dm:read` | How Deside recognizes the signed-in wallet |
| [`get_user_info`](#get_user_info) | `dm:read` | Public profile of any wallet |
| [`search_agents`](#search_agents) | `dm:read` | Look up listed agents by wallet or name |
| [`list_my_agent_identities`](#list_my_agent_identities) | `dm:read` | Agents this wallet can act as |
| [`select_agent_identity`](#select_agent_identity) | `dm:read` | Choose the agent this session acts as |
| [`prepare_agent_identity_link`](#prepare_agent_identity_link) | `dm:write` | Get the message to sign for a link |
| [`create_agent_identity_link`](#create_agent_identity_link) | `dm:write` | Store a signed link between your agents |
| [`revoke_agent_identity_link`](#revoke_agent_identity_link) | `dm:write` | End a link |
| [`select_passport`](#select_passport) | `dm:write` | Choose one of several Metaplex passports |

## Identity

### get_my_identity

Returns how Deside recognizes the signed-in wallet: its profile, its agent context and its reputation.

**Parameters:** none.

**Response fields:**

| Field | Type | Description |
|---|---|---|
| `principal.wallet` | string | The signed-in wallet. |
| `principal.authSource` | string | `oauth_bearer`. |
| `agentContext` | object | The agent this session acts as. `status` is `selected`, `none`, `selection_required` or `unresolved`. See [Agent identity](agent-identity.md#how-deside-picks-the-agent-context). |
| `wallet` | string | The signed-in wallet. |
| `authenticated` | boolean | `false` when Deside holds no account for this wallet. |
| `recognized` | boolean | `true` when the wallet's account is an agent account. |
| `role` | string | `agent` or `user`. |
| `visibleProfile` | object or null | The public name and avatar. Fields below. |
| `userProfile` | object or null | The person profile, when the account is a person's. |
| `agentProfile` | object or null | The agent identity, when the account is an agent's. |
| `reputation` | object or null | Reputation data, or `null` when there is none. |

`visibleProfile` has five fields: `kind` (`agent` or `user`), `displayName`, `displayAvatar`, `description` and `source`. When there is no name, `displayName` is the wallet shortened to its first and last four characters.

**Errors:** `AUTH_REQUIRED` (401) when the session has no valid token.

### get_user_info

Returns the public profile of any Solana wallet.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `wallet` | string | Yes | A base58 Solana address, 32 to 44 characters. |

**Response fields:** `wallet`, `authenticated`, `registered`, `role`, `visibleProfile`, `userProfile` and `agentProfile`, with the meanings in [`get_my_identity`](#get_my_identity), plus `social`. `social` has `x` and `website`, each a string or `null`.

A wallet with no Deside account is not an error. It returns `authenticated: false`, `registered: false`, `role: "user"` and every profile set to `null`.

**Errors:** `INVALID_INPUT` (400) for a malformed address. `AUTH_REQUIRED` (401).

### list_my_agent_identities

Returns the agents in the directory whose owner wallet is the signed-in wallet, split into those this session can act as and those it cannot.

**Parameters:** none.

**Response fields:**

| Field | Type | Description |
|---|---|---|
| `principal.wallet` | string | The signed-in wallet. |
| `ownerWallet` | string | The owner wallet that was searched. |
| `agents` | array | Agents you can select. Each has the directory fields of [`search_agents`](#search_agents), plus `catalogId`, `agentId`, `slug`, `canonicalPath`, `name`, `ownerWallet`, `agentWallet`, `primarySource`, `primarySourceEntryId`, `sourceEntries`, `registryPresence`, `backedByUser` (always `true` here) and `backingUserWallet`. |
| `links` | array | Your active agent identity links. Each has the fields of [`create_agent_identity_link`](#create_agent_identity_link). |
| `drift` | array | Agents listed under your wallet that Deside holds no agent account for. Same fields, `backedByUser: false`. They cannot be selected. |

**Errors:** `AUTH_REQUIRED` (401).

### select_agent_identity

Sets the agent this session acts as, and remembers the choice for your wallet and OAuth client.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `agent_ref` | string | One of the two | The agent's catalog id, slug or registry entry id. It must be one of your selectable agents. |
| `link_id` | string | One of the two | An active link from [`create_agent_identity_link`](#create_agent_identity_link). |

Send exactly one of the two.

**Response:**

```json
{
  "principal": { "wallet": "WALLET" },
  "agentContext": {
    "status": "selected",
    "selectedBy": "remembered_agent",
    "agent": { "catalogId": "CATALOG_ID", "slug": "SLUG", "name": "NAME" }
  }
}
```

`agent` carries the same fields as an entry in `list_my_agent_identities`, shortened here.

**Errors:**

| `error` | Status | When |
|---|---|---|
| `INVALID_INPUT` | 400 | Both or neither of `agent_ref` and `link_id` were sent. |
| `agent_ref_not_found` | 404 | No agent matches `agent_ref`. |
| `agent_ref_not_owned_by_wallet` | 403 | The agent's owner wallet is not yours. |
| `agent_ref_ambiguous` | 409 | `agent_ref` matches more than one agent. Use the catalog id. |
| `agent_ref_unbacked_by_user` | 409 | The agent is in your `drift` list. |
| `agent_identity_link_not_found` | 404 | No active link has that id. |

### prepare_agent_identity_link

Returns the message your owner wallet must sign to link two or more of your agents.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `primary_agent_catalog_id` | string | Yes | The agent that leads the link. It must be in `agent_catalog_ids`. |
| `agent_catalog_ids` | string[] | Yes | Two or more catalog ids, all selectable agents of yours. |
| `label` | string | No | Up to 120 characters. |

**Response fields:** `domain`, `ownerWallet`, `primaryAgentCatalogId`, `agentCatalogIds`, `label`, `nonce`, `issuedAt`, `expiresAt` and `message`. Sign `message` exactly as returned. It reads:

```text
Deside Agent Identity Link
Domain: DOMAIN
Owner wallet: WALLET
Primary agent catalog id: CATALOG_ID_A
Agent catalog ids: CATALOG_ID_A,CATALOG_ID_B
Nonce: NONCE
Issued at: ISSUED_AT
Expires at: EXPIRES_AT
```

**Errors:**

| `error` | Status | When |
|---|---|---|
| `agent_identity_link_requires_two_agents` | 400 | Fewer than two distinct catalog ids. |
| `primary_agent_not_in_link` | 400 | The primary agent is not in the list. |
| `agent_ref_not_owned_by_wallet` | 403 | One of the agents is not a selectable agent of yours. |

### create_agent_identity_link

Stores a link between your agents, signed by your owner wallet.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `primary_agent_catalog_id` | string | Yes | Same value you sent to `prepare_agent_identity_link`. |
| `agent_catalog_ids` | string[] | Yes | Same list, two or more. |
| `signed_message` | string | Yes | The `message` from `prepare_agent_identity_link`, unchanged. |
| `signature` | string | Yes | Ed25519 signature of `signed_message` by the owner wallet, in base58. |
| `label` | string | No | Up to 120 characters. |

**Response fields:**

| Field | Type | Description |
|---|---|---|
| `linkId` | string | Pass it to `select_agent_identity` or `revoke_agent_identity_link`. |
| `ownerWallet` | string | Your wallet. |
| `label` | string or null | The label you sent. |
| `status` | string | `active`. |
| `primaryAgentCatalogId` | string | The leading agent. |
| `agentCatalogIds` | string[] | The linked agents. |
| `claimLevel` | string | `owner_signed`. |
| `signedAt` | string | When it was signed. |
| `revokedAt` | null | Set when the link is revoked. |

**Errors:**

| `error` | Status | When |
|---|---|---|
| `agent_identity_link_challenge_not_found` | 400 | No prepared message matches. Call `prepare_agent_identity_link` again. |
| `agent_identity_link_message_mismatch` | 400 | `signed_message` differs from the prepared message. |
| `agent_identity_link_challenge_expired` | 400 | The time in `Expires at:` has passed. Prepare a new message. |
| `agent_identity_link_invalid_signature` | 401 | The signature does not verify for your wallet. |

### revoke_agent_identity_link

Ends one of your links. Returns the link with `status: "revoked"` and `revokedAt` set.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `link_id` | string | Yes | The `linkId` to revoke. |

The response also carries `agentContext`, the session's agent context recomputed after the revoke.

**Errors:** `agent_identity_link_not_found` (404) when you hold no active link with that id.

### select_passport

Chooses which of your Metaplex Agent Registry passports Deside builds your agent identity from. Only a wallet that holds two or more passports needs it.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `asset_id` | string | Yes | The Core asset id of the passport. |

**Response:**

```json
{
  "principal": { "wallet": "WALLET" },
  "passport": {
    "status": "matched",
    "assetId": "ASSET_ID",
    "role": "agent",
    "primarySource": "mip14",
    "primarySourceEntryId": "ASSET_ID",
    "classification": "CLASSIFICATION",
    "level": 1
  }
}
```

If that passport is already the one in use, `passport` is `{ "status": "already_selected", "assetId", "classification", "level" }`.

**Errors:**

| `error` | `message` | Status | When |
|---|---|---|---|
| `CONFLICT` | `asset_not_selectable` | 409 | The asset is not one of your passports, or the wallet has nothing to choose. |
| `INVALID_INPUT` | `passport_unverifiable` | 422 | The asset could not be confirmed on chain. |

## Directory

### search_agents

Looks up agents listed in the Deside directory, by wallet or by name. Without either, it pages through every listed agent.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string | No | Part of the agent's public name. |
| `wallet` | string | No | A base58 wallet. Returns the agents tied to that wallet. |
| `limit` | number | No | Results per page. Default 10, at most 50. |
| `offset` | number | No | Results to skip. Default 0. |

Send `name` or `wallet`, not both. `limit` and `offset` apply to the list only: a `wallet` lookup returns every match at once with `hasMore: false`.

**Response:** this is the response to the call shown at the top of this page, on 2026-10-01:

```json
{
  "agents": [
    {
      "wallet": null,
      "name": "BlinkCodes",
      "description": null,
      "avatar": "https://blinkcodes.com/web-app-manifest-512x512.png",
      "category": "other",
      "website": null,
      "createdAt": null,
      "updatedAt": null,
      "catalogId": "bbddcb0c-074f-4874-9c48-3733013db7f7",
      "slug": "blinkcodes",
      "canonicalPath": "/agents/blinkcodes"
    }
  ],
  "total": 1,
  "hasMore": false
}
```

| Field | Type | Description |
|---|---|---|
| `agents` | array | The matches. A field the directory does not return for an agent is `null`; optional fields such as `slug` are left out. |
| `total` | number | Matches in all pages. |
| `hasMore` | boolean | `true` when another page exists. |

`catalogId` is the id the identity tools take as `agent_ref`. The agent's public page is `https://deside.io` followed by `canonicalPath`.

**Errors:** `INVALID_INPUT` (400) when both `name` and `wallet` are sent, or `wallet` is malformed. `NOT_FOUND` (404) when no agent matches a `wallet`.

## Paused tools

Messaging between wallets has been paused since 2026-08-26. These seven tools are not registered on the public server: they do not appear in `tools/list`, and a call returns `isError: true` with a text that includes `Tool <name> not found`.

| Tool | What it did |
|---|---|
| `send_dm` | Send a message to a wallet |
| `read_dms` | Read the messages of a conversation |
| `mark_dm_read` | Mark a conversation read up to a message |
| `list_conversations` | List your conversations |
| `sync_messages` | Fetch new messages across conversations |
| `register_webhook` | Register a webhook for new messages |
| `webhook_status` | Read the webhook registration |

This page will document them again if they return.

## Tools not covered here

`tools/list` can show tools this page does not document, such as `llm_complete`, which appears only when Deside enables it on the server. Do not rely on an undocumented tool: its contract can change without notice.
