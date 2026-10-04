# Tools reference

The Deside MCP has 22 tools.
Read tools need `deside:read`; tools that prepare, send or change something need `deside:write`. Launchpad tools always act with the wallet you signed in with.

**A tool that changes something returns an unsigned transaction. Nothing moves until you sign it.**

On success, the tool's fields arrive in `structuredContent` and, as JSON text, in `content[0].text`. On failure, the result has `isError: true`; see [Errors and limits](errors.md).

| Tool | Scope | Changes anything |
|---|---|---|
| `search_agents` | read | No |
| `agent_trust_card` | read | No |
| `ask_directory` | read | No |
| `get_directory_stats` | read | No |
| `token_card` | read | No |
| `get_user_info` | read | No |
| `get_my_identity` | read | No |
| `list_my_agent_identities` | read | No |
| `select_agent_identity` | read | Only which agent this session acts as; it writes nothing outside the session |
| `prepare_agent_identity_link` | write | No, it returns the text to sign |
| `create_agent_identity_link` | write | Yes |
| `revoke_agent_identity_link` | write | Yes, it removes a link |
| `get_launchpad_info` | read | No |
| `launch_token` | write | Yes, you sign it |
| `register_agent_identity` | write | Yes, you sign it |
| `update_agent_identity` | write | Yes, you sign it |
| `submit_transaction` | write | Yes, it sends a signed transaction |
| `get_token` | read | No |
| `list_launches` | read | No |
| `swap` | write | Yes, you sign it |
| `claim_fees` | write | Yes, you sign it |
| `migrate` | write | Yes, you sign it |

## Directory

### search_agents

Finds agents in the Agent Directory.

| Name | Type | Required | Description |
|---|---|---|---|
| `name` | string | No | Part of the public name. |
| `wallet` | string | No | One wallet, base58. Not together with `name`. |
| `registry` | string | No | Such as `8004solana` or `erc8004-base`. See [Sources](../api/sources.md). |
| `chain` | string | No | `solana` or `evm`. |
| `service` | string | No | `web`, `mcp`, `a2a`, `x402`, `api` or `contact`. |
| `capability` | string | No | `trading`, `payments`, `analytics`, `defi`, `content`, `mcp_server`, `a2a_task_receiver`, `x402_acceptor` or `identity`. |
| `limit` | integer | No | 1 to 50. Default 10. |
| `offset` | integer | No | Default 0. |

Returns `{agents, total, hasMore}`. Each agent has the fields of an agent in [`GET /api/v2/public/agents`](../api/public-agents.md#response), with `catalogId` in place of `id`.

### agent_trust_card

What we checked about one agent.

| Name | Type | Required | Description |
|---|---|---|---|
| `catalogId` | string | Yes | From `search_agents`. |

Returns the fields of [`GET /api/v2/public/agents/{ref}`](../api/public-agents.md#get-ref), with `catalogId` in place of `id`. `catalogId` also accepts a slug.

### ask_directory

Finds agents from a question in plain words. `question`: 3 to 500 characters. Returns the same body as [`POST /ask`](../api/ask.md). Free for a signed-in account, 10 questions a minute per account. The MCP does not pay over x402: if the account is banned or its session revoked, the tool returns `UNKNOWN` with `status` 402.

### get_directory_stats

The directory counts. No parameters. Returns the `data` of [`GET /api/v2/public/agents/stats`](../api/public-agents.md#get-stats).

### token_card

Read one Solana token in Deside: its market data and who is behind it.

| Name | Type | Required | Description |
|---|---|---|---|
| `mint` | string | Yes | Solana mint address (base58, 32 to 44 characters). |

Returns `market` (phase, price, marketCap, change24h), `behindToken` (agents, tools and links related to the token, each with a step: Declared, Matched or Verified owner) and `claim` (the ways the owner can prove ownership: wallet, x, or domain via dns or deside-json, marked as Verified domain).

## Your identity

### get_my_identity

Who Deside recognizes you as. No parameters. Returns `principal` (your `wallet`), `agentContext`, `recognized`, `visibleProfile` and `agentProfile`.

| `agentContext.status` | Meaning |
|---|---|
| `selected` | This session acts as the agent in `agentContext.agent`. |
| `none` | Deside lists no agent owned by this wallet. |
| `unresolved` | The wallet owns agents in different registries with no link between them. Select one or link them. |

### get_user_info

The public profile of any wallet. `wallet`: base58. Returns `wallet`, `visibleProfile`, `agentProfile`.

### list_my_agent_identities

The agents your wallet owns, and the links between them. Returns `ownerWallet`, `agents[]`, `links[]` and `drift[]`: agents that no longer match.

### select_agent_identity

Chooses the agent this session acts as. Send exactly one: `agent_ref` (an agent's `catalogId`) or `link_id`.

### prepare_agent_identity_link, create_agent_identity_link, revoke_agent_identity_link

Declare that two or more of your agents are the same one.

1. `prepare_agent_identity_link` with `primary_agent_catalog_id`, `agent_catalog_ids` (2 or more) and an optional `label` (up to 120). It returns the text to sign.
2. Sign that text with your wallet.
3. `create_agent_identity_link` with the same fields plus `signed_message` and `signature`.

`revoke_agent_identity_link` with `link_id` removes a link.

These three tools return the backend's own error codes.

## Launchpad

Every Launchpad tool takes `network`: `mainnet` or `devnet`. It is required in all of them, `submit_transaction` and `get_token` included, and optional only in `get_launchpad_info`. Fees, limits and a real launch are in [Deside Launchpad](../launchpad/README.md).

### get_launchpad_info

Fees, costs, terms and the on-chain config. Same body as `GET https://launchpad.deside.io/v1/info`.

### launch_token

Prepares a token launch, with or without the agent's identity.

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `acceptTerms` | `true` | Yes | Read `https://launchpad.deside.io/terms` first. |
| `token.name` | string | Yes | 1 to 32 characters. |
| `token.symbol` | string | Yes | 1 to 10 characters. |
| `token.description` | string | No | Up to 500. |
| `token.image` | URL | One of the two | https URL of the logo, up to 300 characters. |
| `token.imageBase64` | string | One of the two | PNG, JPG, WebP or GIF, up to 950,000 characters of base64 (about 700 KB) through the MCP. Over REST, up to 1 MB. |
| `token.website`, `token.x`, `token.telegram` | URL | No | Up to 200 characters each. |
| `registerAgentIdentity` | boolean | No | `true` to create the identity in the same signature. |
| `agent` | object | No | `name` (up to 64), `description` (up to 1,000), `image`, `active`, `x402Support`, `supportedTrust` (`reputation`, `crypto-economic`, `tee-attestation`), and up to 40 `services` with `name`, `endpoint`, `version`. |

A logo is required: without `token.image` or `token.imageBase64` the call fails with `400`.

Returns `network`, `mint`, `pool`, `agentAsset` (or `null`), `creator`, `files`, `cost` (`arweaveSol`, `estimatedTotalSol`), `transaction` (base64, a legacy transaction already signed by the new mint and, with an identity, the agent asset: add your signature with `partialSign`), `bytes`, `simulation`, `expiresInSeconds: 60`, `next` and `disclaimer`. When `simulation.ok` is `true` it also returns `signUrl` and `signUrlExpiresAt`. When it is `false`, `error`, `hint` and the last `logs` say why.

### register_agent_identity

Adds the agent identity to a token you already launched here. Only its creator can. Takes `mint`, `acceptTerms: true` and an optional `agent`. Returns `mint`, `agentAsset`, `files`, `registration` (the EIP-8004 document), `cost` and the transaction, like `launch_token`.

### update_agent_identity

Changes the EIP-8004 registration of an agent identity your signed-in wallet owns in one unsigned transaction. Same agent, same address.

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `asset` | string | Yes | Agent asset address (agentAsset from launch_token or register_agent_identity). |
| `agent` | object | No | Fields to change: `name`, `description`, `image`, `active`, `x402Support`, `supportedTrust`, `services`. |

Omitted fields keep their value. `services` replaces the whole list: include every service to keep. Returns `transaction` (base64, unsigned), `mint`, `agentAsset`, `cost` and the same fields as `launch_token` and `register_agent_identity`.

### submit_transaction

Sends a transaction you signed, or confirms one you already sent. Takes `network` and either `transaction` (signed, base64) or `signature`. Returns `signature` and `links`; for a launch, also `mint`, `pool`, `agentAsset` and `uploads`.

**It sends any valid transaction prepared by this Launchpad, whoever signed it.**

### get_token

A token's progress. Takes `network` and `mint`. Returns `network`, `mint`, `name`, `symbol`, `uri`, `creator`, `pool`, `dbcConfig`, `launchedHere`, `graduated`, `curve` (`raisedSol`, `thresholdSol`, `progress`), `dammV2Pool` once graduated, `unclaimedCurveFeesSol` (`creator`, `partner`) and `links`.

### list_launches

Tokens a wallet launched here. `wallet` is optional and defaults to yours. Returns `creator`, `count` and `launches[]` with `mint`, `pool`, `graduated`, `raisedSol`, `unclaimedCreatorFeesSol`.

### swap

Prepares a buy or sell. Takes `mint`, `side` (`buy` or `sell`), `amount` and optional `slippageBps` (0 to 5,000, default 300). Returns `venue` (the curve, or the DAMM v2 pool after graduation), `quote` (`expectedOut`, `minimumOut`, `unit`) and the transaction.

### claim_fees

Prepares the claim of your creator fees. Takes `mint`. Returns `claims[]`, each from the curve, its surplus, or the locked pool position, and the transaction.

### migrate

Prepares the migration of a curve that reached its threshold. Takes `mint`. Meteora migrates automatically on mainnet; use this only if it has not. Returns `estimatedCostSol` (about 0.0165) and the transaction.

## Limits

`search_agents`, `agent_trust_card`, `get_directory_stats` and `token_card` share a limit of 20 calls a minute and 100 a day per OAuth client. The MCP enforces this limit and returns `RATE_LIMITED` (429) when you exceed it. See [Errors and limits](errors.md).
