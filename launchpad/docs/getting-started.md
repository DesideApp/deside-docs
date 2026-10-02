# Getting started

This guide launches a token on devnet through the MCP server: read the terms, prepare the launch, sign it with your wallet and send it. The same steps work over REST; each step shows the route too, and the [REST quickstart](rest-quickstart.md) does them with `curl`.

The responses below come from a real devnet run on 2026-10-01, trimmed. The creator wallet is shown as `YOUR_WALLET`.

## Prerequisites

* A Solana keypair you control. It becomes the token creator and pays the launch.
* About 0.03 SOL on devnet in that wallet.
* Node.js with `@modelcontextprotocol/sdk` and `@solana/web3.js` installed.
* A square logo: an https URL, or a PNG, JPG, WebP or GIF file of at most 1 MB.

## Connect to the MCP server

The server needs no login and keeps no session. Connect with any MCP client:

```javascript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';

const client = new Client({ name: 'my-agent', version: '1.0.0' });
await client.connect(new StreamableHTTPClientTransport(new URL('https://launchpad.deside.io/mcp')));

async function tool(name, args) {
  const r = await client.callTool({ name, arguments: args });
  const body = JSON.parse(r.content[0].text);
  if (r.isError) throw new Error(body.error);
  return body;
}
```

`tools/list` returns 8 tools: `get_launchpad_info`, `launch_token`, `submit_transaction`, `get_token`, `list_my_launches`, `swap`, `claim_fees` and `migrate`.

## Read the terms

Call `get_launchpad_info` for the network you will use (REST: `GET /v1/info?network=devnet`):

```javascript
const info = await tool('get_launchpad_info', { network: 'devnet' });
```

The response carries the fees, the costs, the flow and the on-chain configuration. On devnet the run read:

```json
{
  "name": "Deside Agent Launchpad",
  "preset": {
    "id": "agent-standard",
    "curveFee": "1% per trade: creator 0.40%, Deside 0.40%, Meteora 0.20%"
  },
  "networks": {
    "devnet": {
      "dbcConfig": "F9hj6wtoa7rD8FyyzH88Zygks4nCTCnno1ytKAJb4Tzv",
      "graduationRaiseSol": 0.640674862
    }
  }
}
```

## Prepare the launch

Call `launch_token` with your wallet, the token and `acceptTerms: true`, which accepts the [creator terms](fees-and-rules.md#creator-terms) (REST: `POST /v1/launch`). This example also registers an agent identity and sends the logo as base64, so it is stored on Arweave:

```javascript
import fs from 'node:fs';

const prepared = await tool('launch_token', {
  network: 'devnet',
  wallet: 'YOUR_WALLET',
  acceptTerms: true,
  token: {
    name: 'Deside Test A',
    symbol: 'DTEST',
    description: 'A test token',
    imageBase64: fs.readFileSync('logo.png').toString('base64'),
  },
  registerAgentIdentity: true,
  agent: {
    services: [{ name: 'MCP', endpoint: 'https://example.com/mcp' }],
  },
});
```

Nothing is sent yet. The response holds the transaction, the addresses it will create and the final Arweave URLs:

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
    "agentRegistration": "https://devnet.irys.xyz/5cYUiKjcGWBxt3dv49yfFRxPtNzS3SSeBM4SBQ7a4gtL",
    "agentMetadata": "https://devnet.irys.xyz/<agent-metadata-id>"
  },
  "cost": { "arweaveSol": 0.000047176, "estimatedTotalSol": 0.025547 },
  "transaction": "<base64>",
  "bytes": 1183,
  "simulation": { "ok": true },
  "expiresInSeconds": 60
}
```

Check `simulation.ok` before signing. When it is `false`, `simulation.hint` says why. With an empty wallet the run returned:

```json
{ "ok": false, "error": "AccountNotFound", "hint": "The signing wallet has no SOL on this network.", "logs": [] }
```

## Sign and send

Sign the transaction with your wallet and pass it to `submit_transaction` (REST: `POST /v1/submit`). **The transaction expires about 60 seconds after `launch_token` returns.**

```javascript
import { Keypair, Transaction } from '@solana/web3.js';

const wallet = Keypair.fromSecretKey(Uint8Array.from(JSON.parse(fs.readFileSync('wallet.json', 'utf8'))));
const tx = Transaction.from(Buffer.from(prepared.transaction, 'base64'));
tx.partialSign(wallet);

const sent = await tool('submit_transaction', {
  network: 'devnet',
  transaction: tx.serialize().toString('base64'),
});
```

Deside simulates the signed transaction, uploads the Arweave files it paid for and only then sends it. Indexers read the token JSON when the mint is created and do not retry, so the files must exist before the mint does. If the simulation fails, you get a `422` with a `hint` and nothing is uploaded or sent. Each uploaded file has the ID that was already inside the transaction:

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
      { "id": "<agent-metadata-id>", "url": "https://devnet.irys.xyz/<agent-metadata-id>", "matchesPrepared": true, "role": "agent-metadata" },
      { "id": "5cYUiKjcGWBxt3dv49yfFRxPtNzS3SSeBM4SBQ7a4gtL", "url": "https://devnet.irys.xyz/5cYUiKjcGWBxt3dv49yfFRxPtNzS3SSeBM4SBQ7a4gtL", "matchesPrepared": true, "role": "agent-registration" }
    ]
  }
}
```

The wallet in this run paid 0.025420416 SOL in total for the launch with identity. That run predates the agent NFT metadata file, so `<agent-metadata-id>` stands for the fourth file a launch with identity now uploads, and its cost is slightly higher.

## Check the token

Call `get_token` with the mint (REST: `GET /v1/tokens/{mint}?network=devnet`):

```javascript
const token = await tool('get_token', { network: 'devnet', mint: prepared.mint });
```

Right after the launch the curve has raised nothing:

```json
{
  "network": "devnet",
  "mint": "298bT5xFzvWFe1LC2z1QTBi6ggR7vtkA3SY2ErLCYGBE",
  "name": "Deside Test A",
  "symbol": "DTEST",
  "uri": "https://devnet.irys.xyz/DRQFJN6Gtv8ePzELwiTWHJdfPz8eCNVH6htCp2UgJWpE",
  "creator": "YOUR_WALLET",
  "pool": "ChFjyzozWXzvfN9ysBtyW2yC9o8nj86d6eCqdTA1a1cg",
  "launchedHere": true,
  "graduated": false,
  "curve": { "raisedSol": 0, "thresholdSol": 0.640674862, "progress": 0 },
  "unclaimedCurveFeesSol": { "creator": 0, "partner": 0 }
}
```

## Send the transaction yourself

You may send the signed transaction through your own RPC instead. Then call `submit_transaction` with its signature so Deside checks the Arweave payment on chain and uploads the files:

```javascript
await tool('submit_transaction', { network: 'devnet', signature: 'YOUR_SIGNATURE' });
```

Do this within 15 minutes of `launch_token`: the prepared files are held in memory for that long.

{% hint style="warning" %}
**In this mode the files are uploaded after the mint exists, so explorers and wallets may never show the logo.** They read the token JSON once, when the mint is created. Pass the signed transaction to `submit_transaction` instead whenever you can.
{% endhint %}

## What you just did

You created a token with its bonding curve, an EIP-8004 agent identity and permanent Arweave files, all with one signature from your wallet. Deside built the transaction and never had your key.

## Next steps

* [Operations reference](operations.md): trade with `swap`, collect fees with `claim_fees`, list your tokens with `list_my_launches`.
* [Fees and rules](fees-and-rules.md): what each trade pays and when the token graduates.
