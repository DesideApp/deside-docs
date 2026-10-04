# REST quickstart

This guide launches a token on devnet with `curl` and one Node.js script: read the terms, prepare the launch, sign it locally and send it. It uses only the REST routes; the [Deside MCP](../../mcp/README.md) offers the same operations as tools.

The responses below come from real devnet runs on 2026-10-01, trimmed. The creator wallet is shown as `YOUR_WALLET`.

## Prerequisites

* `curl`, `jq` and Node.js 18 or later.
* `@solana/web3.js` installed in the folder where you run the script (`npm install @solana/web3.js`).
* A Solana keypair file you control, as the JSON array of 64 numbers that `solana-keygen` writes. Its address is `YOUR_WALLET`.

{% hint style="info" %}
A wallet created with `solana-keygen new` derives its key from the seed phrase without a derivation path. Phantom derives `m/44'/501'/0'/0'` from the same phrase, so importing the phrase into Phantom shows a different address. To see this wallet in Phantom, import its private key instead.
{% endhint %}
* About 0.03 SOL on devnet in that wallet.
* A square logo file, PNG, JPG, WebP or GIF, of at most 1 MB (about 700 KB through the MCP).

## 0. Find the entry point

The root of the service lists where everything is:

```bash
curl -s https://launchpad.deside.io/
```

```json
{
  "name": "Deside Agent Launchpad",
  "mcp": "https://mcp.deside.io/mcp",
  "openapi": "https://launchpad.deside.io/openapi.json",
  "llms": "https://launchpad.deside.io/llms.txt",
  "terms": "https://launchpad.deside.io/terms",
  "start": "Connect an MCP client to `mcp` (sign in with your Solana wallet) and call get_launchpad_info, or GET /v1/info?network=mainnet."
}
```

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

Send the token with the logo in base64 and save the response to `launch.json`. `acceptTerms: true` accepts the [creator terms](fees-and-rules.md#creator-terms); without it the launch is rejected. `jq` builds the body so the base64 is quoted correctly:

```bash
jq -n --arg img "$(base64 -w0 logo.png)" '{
  network: "devnet",
  wallet: "YOUR_WALLET",
  acceptTerms: true,
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
  "next": "Sign `transaction` with the wallet above (it is already signed by the new mint) and pass it to submit_transaction within about 60 seconds (blockhash lifetime), or give the person signUrl to sign in their browser wallet. If it expires, call launch_token again."
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

Deside simulates it, uploads the Arweave files the transaction paid for and only then sends it and waits for confirmation. The files go first because indexers read the token JSON when the mint is created and do not retry. If the simulation fails, the response is a `422` with a `hint`, and nothing is uploaded or sent. If the upload fails, the response is a `503` and nothing is sent. The launch above returned:

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

## Launch on mainnet with one script

This script runs the whole launch in one command: it prepares the launch, stops if the simulation fails or the transaction is not paid by your keypair, shows the summary, asks you to type `yes`, signs locally, sends it and saves the result. Save it as `launch.mjs`:

```javascript
import fs from 'node:fs';
import readline from 'node:readline/promises';
import { Keypair, Transaction } from '@solana/web3.js';

const [requestFile, keypairFile, base = 'https://launchpad.deside.io'] = process.argv.slice(2);
const wallet = Keypair.fromSecretKey(Uint8Array.from(JSON.parse(fs.readFileSync(keypairFile, 'utf8'))));
const request = JSON.parse(fs.readFileSync(requestFile, 'utf8'));
request.wallet = wallet.publicKey.toBase58();
if (request.token.imageFrom) {
  const r = await fetch(request.token.imageFrom);
  if (!r.ok) throw new Error(`could not download the logo: ${r.status}`);
  request.token.imageBase64 = Buffer.from(await r.arrayBuffer()).toString('base64');
  delete request.token.imageFrom;
}

const post = async (path, body) => {
  const r = await fetch(`${base}${path}`, { method: 'POST', headers: { 'content-type': 'application/json' }, body: JSON.stringify(body) });
  const j = await r.json();
  if (!r.ok) throw new Error(`${path} ${r.status}: ${JSON.stringify(j)}`);
  return j;
};

const prepared = await post('/v1/launch', request);
if (!prepared.simulation?.ok) { console.error('Simulation fails, not signing:', JSON.stringify(prepared.simulation, null, 2)); process.exit(1); }
const tx = Transaction.from(Buffer.from(prepared.transaction, 'base64'));
if (!tx.feePayer?.equals(wallet.publicKey)) throw new Error('the transaction is not paid by your wallet; not signing');

console.log(JSON.stringify({ network: prepared.network, wallet: prepared.creator, mint: prepared.mint, pool: prepared.pool, files: prepared.files, cost: prepared.cost,
  programs: tx.instructions.map((i) => i.programId.toBase58()) }, null, 2));
const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
const ok = (await rl.question('Sign and launch? (type "yes"): ')).trim().toLowerCase() === 'yes';
rl.close();
if (!ok) { console.log('Cancelled, nothing was sent.'); process.exit(0); }

tx.partialSign(wallet);
const sent = await post('/v1/submit', { network: prepared.network, transaction: tx.serialize().toString('base64') });
const out = requestFile.replace(/\.json$/, '') + `-result-${new Date().toISOString().replace(/[:.]/g, '-')}.json`;
fs.writeFileSync(out, JSON.stringify({ prepared: { ...prepared, transaction: undefined }, sent }, null, 2));
console.log(JSON.stringify(sent, null, 2));
console.log(`Saved to ${out}`);
```

Write the body of `launch_token` to `request.json`, without `wallet` (the script takes it from the keypair) and with `acceptTerms: true`. A logo is required: give `token.imageFrom`, a logo URL the script downloads and sends as `imageBase64`, or `token.image` to use the URL as is:

```json
{
  "network": "mainnet",
  "acceptTerms": true,
  "token": {
    "name": "Deside",
    "symbol": "DESIDE",
    "description": "Let your agent launch a token and trade knowing who is behind it.",
    "website": "https://deside.io",
    "x": "https://x.com/deside_app",
    "imageFrom": "https://example.com/logo.png"
  },
  "registerAgentIdentity": false
}
```

Run it with your keypair file:

```bash
node launch.mjs request.json wallet.json
```

DESIDE, the token of Deside, was launched this way on mainnet on 2026-10-02. The prepare call returned, trimmed:

```json
{
  "network": "mainnet",
  "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
  "pool": "D4x5pLnvD1PuX8RwcHQWCzsvzJeiTdsHiMifYWkL4pZp",
  "agentAsset": null,
  "creator": "YOUR_WALLET",
  "files": {
    "image": "https://gateway.irys.xyz/G3ymNAGETZUrU18XadeFN5gtUs25K5TgeFZLqCUD3T1u",
    "tokenMetadata": "https://gateway.irys.xyz/2pJgy1jA1Yr6KKWGyWaZ4LUZTcruyaJQm8cayRhN22RX"
  },
  "cost": { "arweaveSol": 0.000007087, "estimatedTotalSol": 0.020607 },
  "bytes": 783,
  "simulation": { "ok": true, "computeUnits": 104244 },
  "expiresInSeconds": 60
}
```

After `yes`, the submit call returned:

```json
{
  "signature": "3DoZFYKeUCWt25cPE8dVjf9MAbUihTEuxv1siTxzk6mVZVCc97xHKFjJpaarP8VddRB8JnZFj7xoZR5QPRF9L6Tr",
  "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
  "pool": "D4x5pLnvD1PuX8RwcHQWCzsvzJeiTdsHiMifYWkL4pZp",
  "agentAsset": null,
  "links": {
    "transaction": "https://solscan.io/tx/3DoZFYKeUCWt25cPE8dVjf9MAbUihTEuxv1siTxzk6mVZVCc97xHKFjJpaarP8VddRB8JnZFj7xoZR5QPRF9L6Tr",
    "token": "https://solscan.io/token/Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
    "jupiter": "https://jup.ag/tokens/Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1"
  },
  "uploads": {
    "ok": true,
    "files": [
      { "id": "G3ymNAGETZUrU18XadeFN5gtUs25K5TgeFZLqCUD3T1u", "url": "https://gateway.irys.xyz/G3ymNAGETZUrU18XadeFN5gtUs25K5TgeFZLqCUD3T1u", "matchesPrepared": true, "role": "image" },
      { "id": "2pJgy1jA1Yr6KKWGyWaZ4LUZTcruyaJQm8cayRhN22RX", "url": "https://gateway.irys.xyz/2pJgy1jA1Yr6KKWGyWaZ4LUZTcruyaJQm8cayRhN22RX", "matchesPrepared": true, "role": "token-metadata" }
    ]
  }
}
```

The launch cost about 0.0206 SOL, with no agent identity.

## What you just did

You created a token with its bonding curve and its metadata on Arweave using 3 HTTP calls and one local signature. Your key never left your machine.

## Next steps

* [Operations reference](operations.md): every operation with its MCP and REST request and a real response.
* [Launch with an agent identity](operations.md#example-a-launch-with-a-full-agent-identity): the `agent` fields and the EIP-8004 registration they produce.
* [Fees and rules](fees-and-rules.md): what each trade pays and when the token graduates.
