# Operations reference

The launchpad has 8 operations. Each one is an MCP tool and a REST route that take the same parameters and return the same JSON, because both are generated from one schema. The OpenAPI document at `https://launchpad.deside.io/openapi.json` is generated from that schema too.

| MCP tool | REST route | Writes | What it does |
|---|---|---|---|
| [`get_launchpad_info`](#get_launchpad_info) | `GET /v1/info` | No | Fees, costs, flow and configuration per network |
| [`launch_token`](#launch_token) | `POST /v1/launch` | Yes | Prepares one launch transaction |
| [`submit_transaction`](#submit_transaction) | `POST /v1/submit` | Sends | Uploads the paid files and sends a signed transaction |
| [`get_token`](#get_token) | `GET /v1/tokens/{mint}` | No | Status of one token |
| [`list_my_launches`](#list_my_launches) | `GET /v1/creators/{wallet}/launches` | No | Tokens a wallet launched here |
| [`swap`](#swap) | `POST /v1/swap` | Yes | Prepares a buy or a sell |
| [`claim_fees`](#claim_fees) | `POST /v1/claim` | Yes | Prepares the creator fee claim |
| [`migrate`](#migrate) | `POST /v1/migrate` | Yes | Prepares graduation to the pool |

## How write operations work

A write operation returns an unsigned transaction and does not send anything. Its response always includes:

| Field | Type | Description |
|---|---|---|
| `transaction` | string | Base64 Solana transaction. Sign it with the wallet you passed and send it with `submit_transaction`. |
| `bytes` | number | Serialized size. The Solana limit is 1232 bytes. |
| `simulation` | object | `{ ok: true, computeUnits }`, or `{ ok: false, error, hint, logs }` with the last 8 log lines. |
| `next` | string | The next step, in words. |

**Sign and submit within about 60 seconds.** That is the lifetime of the blockhash inside the transaction. If it expires, call the same operation again.

On REST, GET parameters go in the query string and POST parameters in a JSON body. Path parameters (`mint`, `wallet`) go in the path.

## How to send a request

Over REST, call the route with `curl` or any HTTP client. Over MCP, any MCP client works; the server is stateless, so a single `POST` of a `tools/call` request also works without a session. This sends the `get_launchpad_info` call shown below:

```bash
curl -s -X POST https://launchpad.deside.io/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_launchpad_info","arguments":{"network":"devnet"}}}'
```

The MCP answer wraps the same JSON that REST returns. It is in `result.structuredContent`, and as text in `result.content[0].text`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [{ "type": "text", "text": "{\n  \"name\": \"Deside Agent Launchpad\", ..." }],
    "structuredContent": { "name": "Deside Agent Launchpad" }
  }
}
```

Every example response below is the REST body, which is also the MCP `structuredContent`. They come from real devnet calls on 2026-10-01, trimmed. Transactions are cut with `...` and the creator wallet is shown as `YOUR_WALLET`.

## Shared parameters

| Name | Type | Description |
|---|---|---|
| `network` | `"mainnet"` or `"devnet"` | `mainnet` for real tokens, `devnet` to rehearse with free devnet SOL. Required in every operation except `get_launchpad_info`. |
| `wallet` | string | A base58 Solana address. |
| `mint` | string | A token mint address, base58. |

---

### `get_launchpad_info`

Returns the launchpad terms: fees, anti-sniper fee, graduation, locked liquidity, launch costs, the 4-step flow and the on-chain configuration of each network. No wallet needed.

**MCP:** `get_launchpad_info` · **REST:** `GET /v1/info?network=devnet`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | No | Limit `networks` to one network. Without it, both are returned. |

**Response fields**

| Field | Description |
|---|---|
| `name`, `description`, `disclaimer` | What the service is and the disclaimer text. |
| `preset` | The launch preset: `id`, `supply`, `curveFee`, `antiSniper`, `graduation`, `migratedPool`, `liquidity`, `startMarketCapUsd`, `graduationMarketCapUsd`. |
| `costs` | `launchSol`, `agentIdentitySol`, `arweaveSol`, `typicalTotalSol`, `note`. |
| `flow` | The 4 steps, in words. |
| `networks.<name>` | `dbcConfig`, `partner`, `graduationRaiseSol` (read from the chain, `null` if it could not be read) and `graduation`. |

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_launchpad_info",
    "arguments": { "network": "devnet" }
  }
}
```

REST request:

```bash
curl -s 'https://launchpad.deside.io/v1/info?network=devnet'
```

Response:

```json
{
  "name": "Deside Agent Launchpad",
  "preset": {
    "id": "agent-standard",
    "supply": "1,000,000,000 tokens, 6 decimals, immutable metadata, no mint authority, no vesting, no pre-buys",
    "curveFee": "1% per trade: creator 0.40%, Deside 0.40%, Meteora 0.20%",
    "antiSniper": "trading fee starts at 25% and falls linearly to 1% over the first 120 seconds",
    "startMarketCapUsd": 4000,
    "graduationMarketCapUsd": 50000
  },
  "costs": {
    "launchSol": 0.0206,
    "agentIdentitySol": 0.0049,
    "arweaveSol": "charged at cost inside the same signature, about 0.00005 SOL for a small logo",
    "typicalTotalSol": 0.026,
    "note": "network rent and fees; Deside charges no launch fee"
  },
  "networks": {
    "devnet": {
      "dbcConfig": "F9hj6wtoa7rD8FyyzH88Zygks4nCTCnno1ytKAJb4Tzv",
      "partner": "D1sgiDnreNDrgRXYRzqbx2Bv3PLFgRiCipVYHkUCWXfN",
      "graduationRaiseSol": 0.640674862,
      "graduation": "devnet test config: graduates at a 400 USD market cap (about 0.64 SOL)"
    }
  }
}
```

---

### `launch_token`

Prepares one unsigned transaction that creates your token and its Meteora bonding curve, optionally registers an EIP-8004 agent identity owned by your wallet, and pays the Arweave storage of the logo and metadata. Your wallet is creator and payer.

**MCP:** `launch_token` · **REST:** `POST /v1/launch` · Limit: 10 per minute per IP.

**Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `wallet` | string | Yes | Signs and pays the launch, becomes the token creator and the identity owner. Needs about 0.03 SOL. |
| `token` | object | Yes | The token. Give `image` or `imageBase64`. |
| `token.name` | string | Yes | 1 to 32 characters. |
| `token.symbol` | string | Yes | 1 to 10 characters. |
| `token.description` | string | No | Up to 500 characters. |
| `token.image` | string | One of the two | https URL of a square PNG, JPG, WebP or GIF logo, up to 300 characters. Used as is: you keep it online. |
| `token.imageBase64` | string | One of the two | The logo file in base64, PNG, JPG, WebP or GIF, 1 MB maximum. Stored permanently on Arweave, paid in the same signature. |
| `token.website` | string | No | URL, up to 200 characters. |
| `token.x` | string | No | X profile URL, up to 200 characters. |
| `token.telegram` | string | No | Telegram URL, up to 200 characters. |
| `registerAgentIdentity` | boolean | No | `true` also creates a Metaplex Agent Registry identity (EIP-8004 registration) owned by `wallet`, in the same signature. Adds about 0.005 SOL. Default `false`. |
| `agent` | object | No | Identity fields. Only accepted when `registerAgentIdentity` is `true`. |

**The `agent` object**

| Name | Type | Default | Description |
|---|---|---|---|
| `name` | string | `token.name` | Up to 64 characters. |
| `description` | string | `token.description` | What the agent does and how to use it, up to 1000 characters. |
| `image` | string | The token logo | Avatar URL. |
| `active` | boolean | `true` | Whether the agent is live. |
| `x402Support` | boolean | `false` | `true` if the agent sells anything over x402. |
| `supportedTrust` | array | `[]` | Any of `reputation`, `crypto-economic`, `tee-attestation`. List only what you implement. |
| `services` | array | `[]` | Up to 40 entries, each `{ name, endpoint, version?, mcpTools?, a2aSkills?, skills?, domains?, resources? }`. `name` is up to 64 characters, for example `web`, `MCP`, `A2A`, `x402`, `agentWallet`. `endpoint` is up to 512 characters. |

Deside fills the rest of the registration: `type`, `registrations` (the new agent asset, with `agentRegistry` set to `solana:101:metaplex` on mainnet and `solana:103:metaplex` on devnet), an `agentWallet` service set to the signing wallet when you do not declare one and, on mainnet only, a `web` service pointing to the agent's Metaplex page when you do not declare one.

The registration file follows the EIP-8004 registration fields exactly, in this order: `type`, `name`, `description`, `image`, `services`, `x402Support`, `active`, `registrations`, `supportedTrust`. It carries no token data: the link between the agent and the token is on chain, because the pool creator is the owner of the agent asset.

With an identity, the agent asset also gets its own NFT metadata file (`name`, `symbol`, `description`, `image`), which is what the Metaplex agent page and DAS indexers read. The registration file is attached separately through the Agent Identity plugin.

**Response fields**

| Field | Description |
|---|---|
| `mint`, `pool`, `creator` | The new token, its curve pool and the creator wallet. |
| `agentAsset` | The new agent identity asset, or `null`. |
| `files` | Final URLs of `image`, `tokenMetadata` and, with an identity, `agentRegistration` and `agentMetadata` (the NFT metadata of the agent asset). They are fixed before you sign. `image` is uploaded only when you send `imageBase64`; otherwise it is your URL. |
| `cost` | `arweaveSol` and `estimatedTotalSol`. |
| `transaction`, `bytes`, `simulation`, `next` | See [How write operations work](#how-write-operations-work). |
| `expiresInSeconds` | `60`. |

The transaction is already signed by the new mint and, with an identity, by the new agent asset. A full example is in [Getting started](getting-started.md#prepare-the-launch).

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "launch_token",
    "arguments": {
      "network": "devnet",
      "wallet": "YOUR_WALLET",
      "token": {
        "name": "My Agent Token",
        "symbol": "MYAGT",
        "description": "Token of my agent",
        "image": "https://example.com/logo.png"
      },
      "registerAgentIdentity": true,
      "agent": {
        "services": [{ "name": "MCP", "endpoint": "https://example.com/mcp" }]
      }
    }
  }
}
```

REST request:

```bash
curl -s -X POST https://launchpad.deside.io/v1/launch \
  -H 'content-type: application/json' \
  -d '{
    "network": "devnet",
    "wallet": "YOUR_WALLET",
    "token": { "name": "My Agent Token", "symbol": "MYAGT", "description": "Token of my agent", "image": "https://example.com/logo.png" },
    "registerAgentIdentity": true,
    "agent": { "services": [{ "name": "MCP", "endpoint": "https://example.com/mcp" }] }
  }'
```

A devnet launch with identity and the logo in base64 returned:

```json
{
  "network": "devnet",
  "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE",
  "pool": "ChFjyzozWXzvfN9ysBtyW2yC9o8nj86d6eCqdTA1a1cg",
  "agentAsset": "DsWkoxx8yZxfR8iuakrifdxJzUBk9KAoHjtHKknVZUpM",
  "creator": "YOUR_WALLET",
  "files": {
    "image": "https://devnet.irys.xyz/5eVNUjkYHiSuG4ukg9vyPmxxnxFvgBhrduBnK8VsCNY8",
    "tokenMetadata": "https://devnet.irys.xyz/DRQFJN6Gtv8ePzELwiTWHJdfPz8eCNVH6htCp2UgJWpE",
    "agentRegistration": "https://devnet.irys.xyz/5cYUiKjcGWBxt3dv49yfFRxPtNzS3SSeBM4SBQ7a4gtL"
  },
  "cost": { "arweaveSol": 0.000047176, "estimatedTotalSol": 0.025547 },
  "transaction": "AwAAAAAAAAAA...",
  "bytes": 1183,
  "simulation": { "ok": true },
  "expiresInSeconds": 60
}
```

That run predates the agent NFT metadata file. A launch with identity now also returns `files.agentMetadata`.

#### Example: a launch with a full agent identity

This `launch_token` call on mainnet fills every `agent` field and declares one service of each kind the schema describes. Each service is `{ name, endpoint }` plus the optional field that belongs to its kind: `version`, `mcpTools` (MCP), `a2aSkills` (A2A), `skills` and `domains` (OASF) or `resources` (x402-resources). Replace every `example.com` value with your own:

```bash
curl -s -X POST https://launchpad.deside.io/v1/launch \
  -H 'content-type: application/json' \
  -d '{
    "network": "mainnet",
    "wallet": "YOUR_WALLET",
    "token": {
      "name": "My Agent Token",
      "symbol": "MYAGT",
      "description": "Token of my agent",
      "image": "https://example.com/logo.png",
      "website": "https://example.com",
      "x": "https://x.com/example",
      "telegram": "https://t.me/example"
    },
    "registerAgentIdentity": true,
    "agent": {
      "name": "My Agent",
      "description": "Answers questions about Solana tokens. Free over MCP; paid reports over x402 at 0.01 USDC each.",
      "image": "https://example.com/avatar.png",
      "active": true,
      "x402Support": true,
      "supportedTrust": ["reputation"],
      "services": [
        { "name": "web", "endpoint": "https://example.com" },
        { "name": "MCP", "endpoint": "https://example.com/mcp", "version": "2025-06-18", "mcpTools": ["get_report", "search_tokens"] },
        { "name": "A2A", "endpoint": "https://example.com/.well-known/agent-card.json", "version": "0.3.0", "a2aSkills": ["token-report"] },
        { "name": "OASF", "endpoint": "https://example.com/oasf.json", "skills": ["token-analysis"], "domains": ["finance"] },
        { "name": "x402", "endpoint": "https://example.com/x402" },
        { "name": "x402-resources", "endpoint": "https://example.com/x402", "resources": ["https://example.com/api/report"] },
        { "name": "openapi", "endpoint": "https://example.com/openapi.json" },
        { "name": "llms.txt", "endpoint": "https://example.com/llms.txt" },
        { "name": "docs", "endpoint": "https://example.com/docs" },
        { "name": "email", "endpoint": "agent@example.com" },
        { "name": "X", "endpoint": "https://x.com/example" },
        { "name": "github", "endpoint": "https://github.com/example" },
        { "name": "telegram", "endpoint": "https://t.me/example" },
        { "name": "discord", "endpoint": "https://discord.gg/example" },
        { "name": "farcaster", "endpoint": "https://farcaster.xyz/example" }
      ]
    }
  }'
```

The MCP call takes the same object as `arguments`. `ENS` (`name.eth`), `DID` (`did:...`) and any other name are accepted the same way, up to 40 services. A service without `name` or `endpoint` is dropped.

**`x402Support` is not inferred from the services.** Declaring an `x402` service leaves it `false` unless you send `"x402Support": true`.

The response gives the address of the agent asset in `agentAsset` and the registration URL in `files.agentRegistration`. The file at that URL is the EIP-8004 registration Deside builds from the call above. `YOUR_AGENT_ASSET` stands for the `agentAsset` value:

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "My Agent",
  "description": "Answers questions about Solana tokens. Free over MCP; paid reports over x402 at 0.01 USDC each.",
  "image": "https://example.com/avatar.png",
  "services": [
    { "name": "web", "endpoint": "https://example.com" },
    { "name": "MCP", "endpoint": "https://example.com/mcp", "version": "2025-06-18", "mcpTools": ["get_report", "search_tokens"] },
    { "name": "A2A", "endpoint": "https://example.com/.well-known/agent-card.json", "version": "0.3.0", "a2aSkills": ["token-report"] },
    { "name": "OASF", "endpoint": "https://example.com/oasf.json", "skills": ["token-analysis"], "domains": ["finance"] },
    { "name": "x402", "endpoint": "https://example.com/x402" },
    { "name": "x402-resources", "endpoint": "https://example.com/x402", "resources": ["https://example.com/api/report"] },
    { "name": "openapi", "endpoint": "https://example.com/openapi.json" },
    { "name": "llms.txt", "endpoint": "https://example.com/llms.txt" },
    { "name": "docs", "endpoint": "https://example.com/docs" },
    { "name": "email", "endpoint": "agent@example.com" },
    { "name": "X", "endpoint": "https://x.com/example" },
    { "name": "github", "endpoint": "https://github.com/example" },
    { "name": "telegram", "endpoint": "https://t.me/example" },
    { "name": "discord", "endpoint": "https://discord.gg/example" },
    { "name": "farcaster", "endpoint": "https://farcaster.xyz/example" },
    { "name": "agentWallet", "endpoint": "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp:YOUR_WALLET" }
  ],
  "x402Support": true,
  "active": true,
  "registrations": [{ "agentId": "YOUR_AGENT_ASSET", "agentRegistry": "solana:101:metaplex" }],
  "supportedTrust": ["reputation"]
}
```

How each field is filled:

| Field | Value |
|---|---|
| `type` | Always `https://eips.ethereum.org/EIPS/eip-8004#registration-v1`. |
| `name`, `description`, `image` | `agent.*`, or else `token.name`, `token.description` and the token logo. |
| `services` | Your services in your order. Deside appends `agentWallet` as the CAIP-10 address of the signing wallet when you declare none. On mainnet, when you declare no `web` service, Deside puts `{ "name": "web", "endpoint": "https://metaplex.com/agent/YOUR_AGENT_ASSET" }` first. |
| `x402Support`, `active` | `agent.*`, default `false` and `true`. |
| `registrations` | The new agent asset. `agentRegistry` is `solana:101:metaplex` on mainnet and `solana:103:metaplex` on devnet. |
| `supportedTrust` | `agent.supportedTrust`, default `[]`. |

The token fields (`website`, `x`, `telegram`) go only into the token metadata, not into the registration. The strict registration of the mainnet test agent of 2026-10-01 is at `https://gateway.irys.xyz/D72WETGw7oqVfpCpaLd2w9GFXZkPKtqhpwfZFG9Yyzmn`. It declares `web`, `MCP` with `version` and `mcpTools`, `X` and `github`, and ends with the `agentWallet` that Deside added.

The agent asset gets its own NFT metadata file at `files.agentMetadata`, built from the same values:

```json
{
  "name": "My Agent",
  "symbol": "MYAGT",
  "description": "Answers questions about Solana tokens. Free over MCP; paid reports over x402 at 0.01 USDC each.",
  "image": "https://example.com/avatar.png",
  "external_url": "https://metaplex.com/agent/YOUR_AGENT_ASSET",
  "properties": { "files": [{ "uri": "https://example.com/avatar.png", "type": "image/png" }], "category": "image" }
}
```

The on-chain name of the agent asset is `agent.name` cut to 32 characters; the registration keeps up to 64.

---

### `submit_transaction`

Sends a transaction prepared by this launchpad and signed by your wallet and waits for confirmation. For a launch, it first simulates the signed transaction and uploads the Arweave files the transaction paid for, and only then sends it: indexers read the token JSON when the mint is created and do not retry. If the simulation fails, you get a `422` with a `hint` and nothing is uploaded or sent. If the upload fails, you get a `503` and nothing is sent. It only accepts transactions that call the Meteora DBC or DAMM v2 programs and no programs outside the launchpad's list.

**MCP:** `submit_transaction` · **REST:** `POST /v1/submit`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `transaction` | string | One of the two | The signed transaction, base64. |
| `signature` | string | One of the two | The signature of a launch you sent yourself. Deside checks the Arweave payment on chain and uploads the files. They arrive after the mint exists, so explorers and wallets may not show the logo: prefer `transaction`. |

**Response fields**

| Field | Description |
|---|---|
| `signature` | The transaction signature. |
| `links` | Explorer links for the transaction and, for a launch, the token. On mainnet, also a Jupiter link. |
| `mint`, `pool`, `agentAsset` | Only for a launch. |
| `uploads` | Only for a launch: `{ ok, files: [{ id, url, matchesPrepared, role }] }`. With `transaction`, a failed upload is a `503` and nothing is sent. With `signature`, a failed upload returns `{ ok: false, error, retry }`: call `submit_transaction` again with `{ network, signature }` within 15 minutes. |

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "submit_transaction",
    "arguments": { "network": "devnet", "transaction": "SIGNED_TRANSACTION_BASE64" }
  }
}
```

REST request:

```bash
curl -s -X POST https://launchpad.deside.io/v1/submit \
  -H 'content-type: application/json' \
  -d '{ "network": "devnet", "transaction": "SIGNED_TRANSACTION_BASE64" }'
```

`SIGNED_TRANSACTION_BASE64` is the `transaction` of a prepare call after you sign it. The [REST quickstart](rest-quickstart.md#3-sign-the-transaction) has a script that signs it.

The launch with identity above returned:

```json
{
  "signature": "Eb27VqgpAYKvb22Xm7mg4TvVfwC4eBbFNFBUa1519mDq7iRQSixDxtY1guyCCn6rMEp64f3bMEz67tquW8D8yjd",
  "links": {
    "transaction": "https://solscan.io/tx/Eb27VqgpAYKvb22Xm7mg4TvVfwC4eBbFNFBUa1519mDq7iRQSixDxtY1guyCCn6rMEp64f3bMEz67tquW8D8yjd?cluster=devnet",
    "token": "https://solscan.io/token/298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE?cluster=devnet"
  },
  "uploads": {
    "ok": true,
    "files": [
      { "id": "5eVNUjkYHiSuG4ukg9vyPmxxnxFvgBhrduBnK8VsCNY8", "url": "https://devnet.irys.xyz/5eVNUjkYHiSuG4ukg9vyPmxxnxFvgBhrduBnK8VsCNY8", "matchesPrepared": true, "role": "image" },
      { "id": "DRQFJN6Gtv8ePzELwiTWHJdfPz8eCNVH6htCp2UgJWpE", "url": "https://devnet.irys.xyz/DRQFJN6Gtv8ePzELwiTWHJdfPz8eCNVH6htCp2UgJWpE", "matchesPrepared": true, "role": "token-metadata" },
      { "id": "5cYUiKjcGWBxt3dv49yfFRxPtNzS3SSeBM4SBQ7a4gtL", "url": "https://devnet.irys.xyz/5cYUiKjcGWBxt3dv49yfFRxPtNzS3SSeBM4SBQ7a4gtL", "matchesPrepared": true, "role": "agent-registration" }
    ]
  }
}
```

If you sent the transaction yourself, pass `{ "network": "devnet", "signature": "YOUR_SIGNATURE" }` instead, within 15 minutes of `launch_token`. In that mode the files are uploaded after the mint exists, so explorers and wallets may not show the logo.

---

### `get_token`

Returns the name, creator, curve progress toward graduation, graduated pool and unclaimed curve fees of any Meteora DBC token.

**MCP:** `get_token` · **REST:** `GET /v1/tokens/{mint}?network=devnet`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `mint` | string | Yes | Token mint address. |

**Response fields**

| Field | Description |
|---|---|
| `name`, `symbol`, `uri` | From the token metadata account. |
| `creator`, `pool`, `dbcConfig` | Curve creator, curve pool and its configuration. |
| `launchedHere` | `true` when the token uses this launchpad's configuration. |
| `graduated` | Whether the curve has migrated. |
| `curve` | `raisedSol`, `thresholdSol` and `progress` from 0 to 1. |
| `dammV2Pool` | Only when graduated: the Meteora DAMM v2 pool. |
| `unclaimedCurveFeesSol` | `creator` and `partner` curve fees not yet claimed, in SOL. |
| `links` | Explorer links for the token and the pool. |

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_token",
    "arguments": { "network": "devnet", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE" }
  }
}
```

REST request:

```bash
curl -s 'https://launchpad.deside.io/v1/tokens/298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE?network=devnet'
```

Response for a curve that has raised about 0.9% of its threshold:

```json
{
  "network": "devnet",
  "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE",
  "name": "Deside Test A",
  "symbol": "DTEST",
  "uri": "https://devnet.irys.xyz/DRQFJN6Gtv8ePzELwiTWHJdfPz8eCNVH6htCp2UgJWpE",
  "creator": "YOUR_WALLET",
  "pool": "ChFjyzozWXzvfN9ysBtyW2yC9o8nj86d6eCqdTA1a1cg",
  "dbcConfig": "F9hj6wtoa7rD8FyyzH88Zygks4nCTCnno1ytKAJb4Tzv",
  "launchedHere": true,
  "graduated": false,
  "curve": { "raisedSol": 0.005771385, "thresholdSol": 0.640674862, "progress": 0.009008290073975153 },
  "unclaimedCurveFeesSol": { "creator": 0, "partner": 0.000048557 },
  "links": {
    "token": "https://solscan.io/token/298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE?cluster=devnet",
    "pool": "https://solscan.io/account/ChFjyzozWXzvfN9ysBtyW2yC9o8nj86d6eCqdTA1a1cg?cluster=devnet"
  }
}
```

A graduated token returns `"graduated": true`, `"progress": 1` and its pool, for example `"dammV2Pool": "13FbJuYiVvUGXb4iDJJCQuW3t8Gc8yC89Ep4unzTNy7"`.

---

### `list_my_launches`

Returns the tokens a wallet launched with this launchpad's configuration, read from the chain. Deside keeps no list.

**MCP:** `list_my_launches` · **REST:** `GET /v1/creators/{wallet}/launches?network=devnet`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `wallet` | string | Yes | Creator wallet. |

Response: `{ network, creator, count, launches }`, where each launch is `{ mint, pool, graduated, raisedSol, unclaimedCreatorFeesSol }`.

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "list_my_launches",
    "arguments": { "network": "devnet", "wallet": "YOUR_WALLET" }
  }
}
```

REST request:

```bash
curl -s 'https://launchpad.deside.io/v1/creators/YOUR_WALLET/launches?network=devnet'
```

Response for a wallet with 3 launches, one of them graduated:

```json
{
  "network": "devnet",
  "creator": "YOUR_WALLET",
  "count": 3,
  "launches": [
    { "mint": "2bV3HTG6ukie8J74yaprpKiPKjti4LZ9UeB2rsPXseRv", "pool": "A196AYexv116g7Jr7Vu1nUpCe4JjducYfmgHiGWDPjWk", "graduated": false, "raisedSol": 0, "unclaimedCreatorFeesSol": 0 },
    { "mint": "5t4jR1knYHqL6Tn9qPGsZnKuxHqqQytXiirtaqNTgeK4", "pool": "AH9EL2EaZHb71L9SYr2AvTXPY5hv4KCdVdz2UR1XMtj1", "graduated": true, "raisedSol": 0.640674862, "unclaimedCreatorFeesSol": 0 },
    { "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE", "pool": "ChFjyzozWXzvfN9ysBtyW2yC9o8nj86d6eCqdTA1a1cg", "graduated": false, "raisedSol": 0.005771385, "unclaimedCreatorFeesSol": 0 }
  ]
}
```

---

### `swap`

Prepares an unsigned buy or sell of a launchpad token, with a quote and a minimum output under your slippage. Before graduation it trades on the bonding curve; after, on the Meteora DAMM v2 pool.

**MCP:** `swap` · **REST:** `POST /v1/swap`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `wallet` | string | Yes | Signs and pays the trade. |
| `mint` | string | Yes | Token mint address. |
| `side` | string | Yes | `buy` spends SOL, `sell` spends tokens. |
| `amount` | number | Yes | `buy`: SOL to spend. `sell`: tokens to sell, in whole units (6 decimals). |
| `slippageBps` | integer | No | Maximum slippage in basis points, 0 to 5000. Default 300 (3%). |

**Response fields**

| Field | Description |
|---|---|
| `venue` | `meteora-dbc-curve` or `meteora-damm-v2`. |
| `side` | `buy` or `sell`. |
| `quote` | `expectedOut`, `minimumOut` and `unit` (`tokens` or `SOL`). |
| `transaction`, `bytes`, `simulation`, `next` | See [How write operations work](#how-write-operations-work). |

A buy of 0.003 SOL on a devnet curve during the anti-sniper window returned:

```json
{ "venue": "meteora-dbc-curve", "quote": { "expectedOut": 11026964.328623, "minimumOut": 7718875.030036, "unit": "tokens" } }
```

**A curve buy that crosses the graduation threshold is filled only up to the threshold.** The rest of the SOL stays in your wallet.

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "swap",
    "arguments": { "network": "devnet", "wallet": "YOUR_WALLET", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE", "side": "buy", "amount": 0.001 }
  }
}
```

REST request:

```bash
curl -s -X POST https://launchpad.deside.io/v1/swap \
  -H 'content-type: application/json' \
  -d '{ "network": "devnet", "wallet": "YOUR_WALLET", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE", "side": "buy", "amount": 0.001 }'
```

A buy of 0.001 SOL on the curve, after the anti-sniper window, returned:

```json
{
  "network": "devnet",
  "venue": "meteora-dbc-curve",
  "side": "buy",
  "quote": {
    "expectedOut": 3560321.358408,
    "minimumOut": 3453511.717655,
    "unit": "tokens",
    "note": "If this buy crosses the graduation threshold, only the part that fits is filled and the rest stays in your wallet."
  },
  "transaction": "AQAAAAAAAAAA...",
  "bytes": 696,
  "simulation": { "ok": true, "computeUnits": 53302 },
  "next": "Sign and pass to submit_transaction."
}
```

A sell takes `"side": "sell"` and `amount` in tokens. Selling on a devnet curve returned `"quote": { "expectedOut": 0.001035785, "minimumOut": 0.000725049, "unit": "SOL" }`. After graduation `venue` is `meteora-damm-v2` and the quote has no `note`.

---

### `claim_fees`

Prepares one unsigned transaction that collects everything the creator can claim: curve trading fees, the surplus after graduation, and the fees of the creator's locked liquidity position in the pool.

**MCP:** `claim_fees` · **REST:** `POST /v1/claim`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `wallet` | string | Yes | The creator wallet. Any other wallet is refused. |
| `mint` | string | Yes | Token mint address. |

Response: `claims`, a list of `{ source }` with `source` one of `curve` (with `sol`), `surplus` or `damm-v2-locked-position` (with `position`), plus the write fields.

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "claim_fees",
    "arguments": { "network": "devnet", "wallet": "YOUR_WALLET", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE" }
  }
}
```

REST request:

```bash
curl -s -X POST https://launchpad.deside.io/v1/claim \
  -H 'content-type: application/json' \
  -d '{ "network": "devnet", "wallet": "YOUR_WALLET", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE" }'
```

A claim of the curve fees returned this `claims` list, next to the write fields:

```json
{
  "network": "devnet",
  "claims": [{ "source": "curve", "sol": 0.000016185 }],
  "transaction": "AQAAAAAAAAAA...",
  "simulation": { "ok": true },
  "next": "Sign and pass to submit_transaction."
}
```

After graduation the same call claimed the locked position of the pool: `"claims": [{ "source": "damm-v2-locked-position", "position": "2K2d67LqQK8gi6qY52Qv9z7GGTJWuZF5FgVuxeS6PT8Q" }]`. With nothing to collect the call returns `409` with `{ "error": "nothing to claim yet" }`.

---

### `migrate`

Prepares the migration of a curve that reached its threshold into a Meteora DAMM v2 pool. Anyone can pay it, about 0.0165 SOL. On mainnet the Meteora keeper normally does this within seconds: use it only if it has not.

**MCP:** `migrate` · **REST:** `POST /v1/migrate`

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |
| `wallet` | string | Yes | Signs and pays the migration. |
| `mint` | string | Yes | Token mint address. |

Response: `note`, `estimatedCostSol` and the write fields.

#### Example

MCP request, the JSON-RPC body you `POST` to `https://launchpad.deside.io/mcp`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "migrate",
    "arguments": { "network": "devnet", "wallet": "YOUR_WALLET", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE" }
  }
}
```

REST request:

```bash
curl -s -X POST https://launchpad.deside.io/v1/migrate \
  -H 'content-type: application/json' \
  -d '{ "network": "devnet", "wallet": "YOUR_WALLET", "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE" }'
```

A curve that has reached its threshold returns the write fields:

```json
{
  "network": "devnet",
  "note": "On mainnet the Meteora keeper usually migrates within seconds; use this only if it has not.",
  "estimatedCostSol": 0.0165,
  "transaction": "AgAAAAAAAAAA...",
  "simulation": { "ok": true },
  "next": "Sign and pass to submit_transaction."
}
```

A curve below its threshold returns `409` with the amount raised, as this devnet call did: `{ "error": "not ready: 0.001923752 of 0.640674862 SOL raised" }`.

---

## Errors

Over REST an error is an HTTP status with a JSON body `{ "error": "..." }`. Invalid input returns `400` with the field and the reason:

```json
{ "error": "invalid input", "issues": [{ "path": "token.name", "message": "Too big: expected string to have <=32 characters" }] }
```

Over MCP, a parameter that breaks the schema is rejected by the MCP layer with code `-32602` and the same reason, for example `Too big: expected string to have <=32 characters at token.name`. Every other error is a tool result with `isError: true` and the `{ "error": "..." }` body as text.

| Status | Error | Cause and fix |
|---|---|---|
| `400` | `token.image (https URL) or token.imageBase64 is required` | Add a logo. |
| `400` | `agent fields were sent but registerAgentIdentity is not true` | Set `registerAgentIdentity: true` or drop `agent`. |
| `400` | `image must be PNG, JPG, WebP or GIF` / `image is N bytes; max 1 MB` | Send a supported file under 1 MB. |
| `400` | `transaction is not fully signed: sign it with your wallet first` | Sign before `submit_transaction`. |
| `400` | `only transactions prepared by this launchpad can be submitted here` | Send only transactions this service returned. |
| `402` | `this launch transaction does not pay its Arweave upload; use the transaction returned by launch_token` | Do not edit the launch transaction. |
| `403` | `only the creator wallet of this token can claim its creator fees` | Call `claim_fees` with the creator wallet. |
| `404` | `no Meteora DBC pool for this mint on this network` | Check the mint and the `network`. |
| `404` | `transaction not found or not confirmed yet; retry in a few seconds` | Retry `submit_transaction { signature }` shortly. |
| `409` | `nothing to claim yet` | There are no fees to claim. |
| `409` | `not ready: X of Y SOL raised` / `already graduated` | `migrate` only works once the threshold is reached and before graduation. |
| `413` | `request body too large (max 2 MB)` | Use a smaller logo. |
| `422` | `transaction is N bytes (limit 1232); use a shorter name, symbol or image URL` | Shorten the inputs. |
| `422` | `transaction would fail: ...` | The simulation of the signed launch failed before anything was uploaded or sent. The body carries `hint` and `logs`. |
| `422` | `transaction rejected: ...` | The RPC refused the transaction. The body carries `hint` and `logs`; an expired blockhash means prepare it again. |
| `429` | `too many requests, slow down` / `too many launches from this address, wait a minute` | Wait for the per-minute window to reset. |
| `503` | `could not store the token files on Arweave, nothing was sent; call launch_token again` | Prepare the launch again and sign the new transaction. |
| `503` | `uploads are not available on <network> right now` | Arweave uploads are not available on that network at the moment; try later. |
