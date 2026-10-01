# REST quickstart

This guide launches a token on devnet with `curl` and one Node.js script: read the terms, prepare the launch, sign it locally and send it. It uses only the REST routes; [Getting started](getting-started.md) does the same over MCP.

The responses below come from real devnet runs on 2026-10-01, trimmed. The creator wallet is shown as `YOUR_WALLET`.

## Prerequisites

* `curl`, `jq` and Node.js 18 or later.
* `@solana/web3.js` installed in the folder where you run the script (`npm install @solana/web3.js`).
* A Solana keypair file you control, as the JSON array of 64 numbers that `solana-keygen` writes. Its address is `YOUR_WALLET`.
* About 0.03 SOL on devnet in that wallet.
* A square logo file, PNG, JPG, WebP or GIF, of at most 1 MB.

## 1. Read the terms

Ask for the terms of the network you will use. It needs no wallet:

```bash
curl -s 'https://launchpad.deside.io/v1/info?network=devnet'
```

The response carries the fees, the costs, the 4-step flow and the on-chain configuration of the network:

```json
{
  "name": "Deside Agent Launchpad",
  "preset": {
    "id": "agent-standard",
    "curveFee": "1% per trade: creator 0.40%, Deside 0.40%, Meteora 0.20%",
    "antiSniper": "trading fee starts at 25% and falls linearly to 1% over the first 120 seconds"
  },
  "costs": {
    "launchSol": 0.0206,
    "agentIdentitySol": 0.0049,
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

## 2. Prepare the launch

Send the token with the logo in base64 and save the response to `launch.json`. `jq` builds the body so the base64 is quoted correctly:

```bash
jq -n --arg img "$(base64 -w0 logo.png)" '{
  network: "devnet",
  wallet: "YOUR_WALLET",
  token: { name: "My Agent Token", symbol: "MYAGT", description: "Token of my agent", imageBase64: $img }
}' | curl -s -X POST https://launchpad.deside.io/v1/launch \
  -H 'content-type: application/json' --data-binary @- > launch.json
```

On macOS, write `base64 -i logo.png` instead of `base64 -w0 logo.png`.

Nothing is sent yet. `launch.json` holds the unsigned transaction, the addresses it will create and the final Arweave URLs. A devnet launch without identity returned:

```json
{
  "network": "devnet",
  "mint": "2bV3HTG6ukie8J74yaprpKiPKjti4LZ9UeB2rsPXseRv",
  "pool": "A196AYexv116g7Jr7Vu1nUpCe4JjducYfmgHiGWDPjWk",
  "agentAsset": null,
  "creator": "YOUR_WALLET",
  "files": {
    "image": "https://deside.io/icon-512.png",
    "tokenMetadata": "https://devnet.irys.xyz/43Ry4eMwiADkPZJmGBsEE6MD6XhihYofASMh42HchuGu"
  },
  "cost": { "arweaveSol": 0.000047176, "estimatedTotalSol": 0.020647 },
  "transaction": "AgAAAAAAAAAA...",
  "bytes": 789,
  "simulation": { "ok": true },
  "expiresInSeconds": 60,
  "next": "Sign `transaction` with the wallet above (it is already signed by the new mint) and pass it to submit_transaction within about 60 seconds (blockhash lifetime). If it expires, call launch_token again."
}
```

That run passed the logo as a URL in `token.image`, so `files.image` is that URL. With `imageBase64`, `files.image` is an Arweave URL like `tokenMetadata`.

Check the simulation before signing:

```bash
jq '.simulation' launch.json
```

When `simulation.ok` is `false`, `simulation.hint` says why. An empty wallet returned `{ "ok": false, "error": "AccountNotFound", "hint": "The signing wallet has no SOL on this network.", "logs": [] }`.

## 3. Sign the transaction

Save this script as `sign.mjs`. It reads a prepared response, signs its `transaction` with your keypair and prints the body that `POST /v1/submit` expects:

```javascript
import fs from 'node:fs';
import { Keypair, Transaction } from '@solana/web3.js';

const [preparedFile, keypairFile] = process.argv.slice(2);
const prepared = JSON.parse(fs.readFileSync(preparedFile, 'utf8'));
const wallet = Keypair.fromSecretKey(Uint8Array.from(JSON.parse(fs.readFileSync(keypairFile, 'utf8'))));

const tx = Transaction.from(Buffer.from(prepared.transaction, 'base64'));
if (!tx.feePayer.equals(wallet.publicKey)) throw new Error(`prepared for ${tx.feePayer.toBase58()}, not for this keypair`);
tx.partialSign(wallet);

process.stdout.write(JSON.stringify({ network: prepared.network, transaction: tx.serialize().toString('base64') }));
```

The transaction is a legacy Solana transaction already signed by the new mint (and, with an identity, by the new agent asset). `partialSign` adds your signature and keeps theirs. The same script signs the response of `swap`, `claim_fees` and `migrate`.

Run it with the response and your keypair file:

```bash
node sign.mjs launch.json wallet.json > signed.json
```

{% hint style="danger" %}
Keep `wallet.json` on your machine. Don't send it, paste it into a request or commit it: Deside only ever needs the signed transaction.
{% endhint %}

## 4. Send it

Send the signed transaction. **It expires about 60 seconds after step 2 returned**; if it does, run step 2 again:

```bash
curl -s -X POST https://launchpad.deside.io/v1/submit \
  -H 'content-type: application/json' --data-binary @signed.json
```

Deside sends it, waits for confirmation and then uploads the Arweave files the transaction paid for. The launch above returned:

```json
{
  "signature": "64Z2zQPXvc3fqTu1yxBDs41entqQyDcV3VmczydEuGVnzru2p2d4bUa1Cs5D3vzSNZ3YVYh1RoWoisYhJ3rs2RSL",
  "mint": "2bV3HTG6ukie8J74yaprpKiPKjti4LZ9UeB2rsPXseRv",
  "pool": "A196AYexv116g7Jr7Vu1nUpCe4JjducYfmgHiGWDPjWk",
  "agentAsset": null,
  "links": {
    "transaction": "https://solscan.io/tx/64Z2zQPXvc3fqTu1yxBDs41entqQyDcV3VmczydEuGVnzru2p2d4bUa1Cs5D3vzSNZ3YVYh1RoWoisYhJ3rs2RSL?cluster=devnet",
    "token": "https://solscan.io/token/2bV3HTG6ukie8J74yaprpKiPKjti4LZ9UeB2rsPXseRv?cluster=devnet"
  },
  "uploads": {
    "ok": true,
    "files": [
      { "id": "43Ry4eMwiADkPZJmGBsEE6MD6XhihYofASMh42HchuGu", "url": "https://devnet.irys.xyz/43Ry4eMwiADkPZJmGBsEE6MD6XhihYofASMh42HchuGu", "matchesPrepared": true, "role": "token-metadata" }
    ]
  }
}
```

`matchesPrepared: true` means the uploaded file has the ID that was inside the transaction you signed.

## 5. Check the token

Read the token by its mint:

```bash
curl -s "https://launchpad.deside.io/v1/tokens/$(jq -r .mint launch.json)?network=devnet"
```

The response shows the creator, the curve and the unclaimed fees. See [`get_token`](operations.md#get_token) for a full example.

## What you just did

You created a token with its bonding curve and its metadata on Arweave using 3 HTTP calls and one local signature. Your key never left your machine.

## Next steps

* [Operations reference](operations.md): every operation with its MCP and REST request and a real response.
* [Launch with an agent identity](operations.md#example-a-launch-with-a-full-agent-identity): the `agent` fields and the EIP-8004 registration they produce.
* [Fees and rules](fees-and-rules.md): what each trade pays and when the token graduates.
