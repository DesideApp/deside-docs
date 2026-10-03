# Tools reference

The Deside MCP has 20 tools. Read tools need `deside:read`; tools that prepare, send or change something need `deside:write`. Launchpad tools always act with the wallet you signed in with.

**A tool that changes something returns an unsigned transaction. Nothing moves until you sign it.**

On success, the tool's fields arrive in `structuredContent` and, as JSON text, in `content[0].text`. On failure, the result has `isError: true`; see [Errors and limits](errors.md).

| Tool | Scope | Changes anything |
|---|---|---|
| `search_agents` | read | No |
| `agent_trust_card` | read | No |
| `ask_directory` | read | No |
| `get_directory_stats` | read | No |
| `get_user_info` | read | No |
| `get_my_identity` | read | No |
| `list_my_agent_identities` | read | No |
| `select_agent_identity` | read | The agent this session acts as |
| `prepare_agent_identity_link` | write | No, it returns the text to sign |
| `create_agent_identity_link` | write | Yes |
| `revoke_agent_identity_link` | write | Yes, it removes a link |
| `get_launchpad_info` | read | No |
| `launch_token` | write | Yes, you sign it |
| `register_agent_identity` | write | Yes, you sign it |
| `submit_transaction` | write | Yes, it sends a signed transaction |
| `get_token` | read | No |
| `list_launches` | read | No |
| `swap` | write | Yes, you sign it |
| `claim_fees` | write | Yes, you sign it |
| `migrate` | write | Yes, you sign it |

<!-- REVISAR(modificar): register_agent_identity esta en el codigo del MCP (fecc65236) y vivo en el launchpad; sin confirmar que el MCP de prod lo tenga. Token. -->
<!-- REVISAR(modificar): select_agent_identity se anuncia readOnly pero cambia la identidad de la sesion. Token. -->

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
| `limit` | number | No | 1 to 50. Default 10. |
| `offset` | number | No | Default 0. |

Returns `{agents, total, hasMore}`. Each agent has `wallet`, `name`, `description`, `avatar`, `category`, `website`, `createdAt`, `updatedAt` and, when known, `catalogId`, `slug`, `ownerWallet`, `agentWallet`, `primarySource`, `primarySourceEntryId`, `sourceEntries`, `registryPresence`.

<!-- REVISAR(borrar): search_agents tambien devuelve agentId, canonicalPath, mergeEvidence, backedByUser y backingUserWallet (agent-directory-mapper.js:23-91): duplicados, interno y mensajeria. -->
<!-- REVISAR(modificar): limit y offset aceptan cualquier numero; 2.5 llega como "2.5" al backend (search-agents.js:15-19). -->

### agent_trust_card

What we checked about one agent.

| Name | Type | Required | Description |
|---|---|---|---|
| `catalogId` | string | Yes | From `search_agents`. |

Returns `catalogId`, `name`, `description`, `image`, `registry` (`registered`, `source`, `coreAsset`, `readAt`), `ownerWallet`, `services[]` (`kind`, `url`, `declared`, `responds`, `checkedAt`, `version`) and `publicReceipts`. `responds` is `true`, `false`, or `null` when not checked.

<!-- REVISAR(borrar): ownerReputation es FairScale (agent-trust-mapper.js:96-119). -->

### ask_directory

Finds agents from a question in plain words. `question`: 3 to 500 characters. Returns the same body as [`POST /ask`](../api/ask.md).

### get_directory_stats

The directory counts. No parameters. Returns the same body as [`GET /public/agents/stats-summary`](../api/public-agents.md#get-public-agents-stats-summary).

## Your identity

### get_my_identity

Who Deside recognizes you as. No parameters. Returns `principal` (your `wallet`), `agentContext`, `recognized`, `visibleProfile` and `agentProfile`.

| `agentContext.status` | Meaning |
|---|---|
| `selected` | This session acts as the agent in `agentContext.agent`. |
| `none` | Deside lists no agent owned by this wallet. |
| `unresolved` | The wallet owns agents in different registries with no link between them. Select one or link them. |

<!-- REVISAR(borrar): tambien devuelve authenticated, role, userProfile y reputation (get-my-identity.js:47-71): contrato de mensajeria y FairScale. -->

### get_user_info

The public profile of any wallet. `wallet`: base58. Returns `wallet`, `visibleProfile`, `agentProfile`.

<!-- REVISAR(borrar): devuelve authenticated, registered, role, userProfile y social, del contrato de mensajeria (user-identity-mapper.js:73-99). -->

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

<!-- REVISAR(modificar): estas tres tools pasan los errores del backend sin traducir (sin try/catch). Token. -->

## Launchpad

All take `network`: `mainnet` or `devnet`. Fees, limits and a real launch are in [Agent Token Launchpad](../launchpad/README.md).

### get_launchpad_info

Fees, costs, terms and the on-chain config. Same body as `GET https://launchpad.deside.io/v1/info`.

### launch_token

Prepares a token launch, with or without the agent's identity.

| Name | Type | Required | Description |
|---|---|---|---|
| `acceptTerms` | `true` | Yes | Read `https://launchpad.deside.io/terms` first. |
| `token.name` | string | Yes | 1 to 32 characters. |
| `token.symbol` | string | Yes | 1 to 10 characters. |
| `token.description` | string | No | Up to 500. |
| `token.image` | URL | No | Up to 300 characters. |
| `token.imageBase64` | string | No | PNG, JPG, WebP or GIF. |
| `token.website`, `token.x`, `token.telegram` | URL | No | Up to 200 characters each. |
| `registerAgentIdentity` | boolean | No | `true` to create the identity in the same signature. |
| `agent` | object | No | `name` (up to 64), `description` (up to 1,000), `image`, `active`, `x402Support`, `supportedTrust` (`reputation`, `crypto-economic`, `tee-attestation`), and up to 40 `services` with `name`, `endpoint`, `version`. |

<!-- REVISAR(modificar): imageBase64 admite 1.400.000 caracteres ("max 1 MB"), pero /mcp corta el cuerpo en 1 MiB: por el MCP el limite real de imagen ronda 760 KB. Token. -->

Returns `mint`, `pool`, `agentAsset` (or `null`), `creator`, `files`, `cost` (`arweaveSol`, `estimatedTotalSol`), `transaction` (base64, unsigned by you), `bytes`, `simulation`, `expiresInSeconds: 60`, `next` and `disclaimer`. When `simulation.ok` is `true` it also returns `signUrl` and `signUrlExpiresAt`. When it is `false`, `error`, `hint` and the last `logs` say why.

### register_agent_identity

Adds the agent identity to a token you already launched here. Only its creator can. Takes `mint`, `acceptTerms: true` and an optional `agent`. Returns `mint`, `agentAsset`, `files`, `registration` (the EIP-8004 document), `cost` and the transaction, like `launch_token`.

### submit_transaction

Sends a transaction you signed, or confirms one you already sent. Takes `transaction` (signed, base64) or `signature`. Returns `signature` and `links`; for a launch, also `mint`, `pool`, `agentAsset` and `uploads`.

**It sends any valid transaction prepared by this Launchpad, whoever signed it.**

### get_token

A token's progress. Takes `mint`. Returns `name`, `symbol`, `uri`, `creator`, `pool`, `dbcConfig`, `launchedHere`, `graduated`, `curve` (`raisedSol`, `thresholdSol`, `progress`), `dammV2Pool` once graduated, `unclaimedCurveFeesSol` (`creator`, `partner`) and `links`.

### list_launches

Tokens a wallet launched here. `wallet` is optional and defaults to yours. Returns `creator`, `count` and `launches[]` with `mint`, `pool`, `graduated`, `raisedSol`, `unclaimedCreatorFeesSol`.

### swap

Prepares a buy or sell. Takes `mint`, `side` (`buy` or `sell`), `amount` and optional `slippageBps` (0 to 5,000, default 300). Returns `venue` (the curve, or the DAMM v2 pool after graduation), `quote` (`expectedOut`, `minimumOut`, `unit`) and the transaction.

### claim_fees

Prepares the claim of your creator fees. Takes `mint`. Returns `claims[]`, each from the curve, its surplus, or the locked pool position, and the transaction.

### migrate

Prepares the migration of a curve that reached its threshold. Takes `mint`. Meteora migrates automatically on mainnet; use this only if it has not. Returns `estimatedCostSol` (about 0.0165) and the transaction.
