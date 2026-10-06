# Deside Launchpad

The Deside Launchpad launches a Solana token for an AI agent, optionally with the agent's on-chain identity, in one transaction you sign with your own wallet. It builds Meteora Dynamic Bonding Curve transactions; Deside never holds your key or your funds.

{% hint style="info" %}
On this page:
- what one launch creates;
- what it costs, and where the fees are;
- the 10 operations, over the MCP or HTTP;
- a real launch you can check on chain.
{% endhint %}

## What one launch creates

In a single signature:

1. A token: 1,000,000,000 supply, 6 decimals, immutable metadata, no mint authority, no vesting, no pre-buys.
2. Its Meteora Dynamic Bonding Curve pool.
3. Optionally, the agent's identity: a Metaplex Core asset registered in the Metaplex Agent Registry, with an EIP-8004 registration document.
4. The image and metadata files, stored on Arweave and paid inside the same signature.

**The token is created by its signer, not by Deside.**

## Quick start

From Claude or your agent, connected to the [Deside MCP](../mcp/README.md):

1. Ask for the fees and terms: `get_launchpad_info`.
2. Launch: `launch_token` with the name, symbol, image and `acceptTerms: true`.
3. Sign. A person opens the sign link within 2 minutes; an agent signs and calls `submit_transaction`.
4. Check it: `get_token` with the mint.

## Fees and costs

Deside charges no launch fee. A launch costs about 0.021 SOL, or about 0.026 SOL with an agent identity, Arweave storage included. Keep about 0.03 SOL in the wallet. Each trade on the curve pays 1%: 0.40% to the creator, 0.40% to Deside, 0.20% to Meteora.

Every fee, the anti-sniper window, graduation and the creator terms are in [Fees and rules](docs/fees-and-rules.md). Read them live with `GET https://launchpad.deside.io/v1/info?network=mainnet`.

## On-chain addresses

| What | Mainnet |
|---|---|
| Meteora DBC program | `dbcij3LWUppWqq96dh6gJWwBifmcGfLSB5D4DuSMaqN` |
| Launchpad config | `6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn` |
| Metaplex Agent Registry program | `1DREGFgysWYxLnRnKQnwrxnJQeSMk2HmGaC6whw2B2p` |

Devnet uses the config `F9hj6wtoa7rD8FyyzH88Zygks4nCTCnno1ytKAJb4Tzv`, which graduates at about 0.64 SOL.

## Operations

| MCP tool | HTTP | What it does | Changes anything |
|---|---|---|---|
| `get_launchpad_info` | `GET /v1/info` | Fees, costs, terms and config | No |
| `launch_token` | `POST /v1/launch` | Prepares the launch, with or without identity | Yes, you sign it |
| `register_agent_identity` | `POST /v1/agent-identity` | Adds the identity to a token already launched here. Creator only | Yes, you sign it |
| `update_agent_identity` | `POST /v1/agent-identity/update` | Changes the registration of an agent identity you own. Same agent, same address | Yes, you sign it |
| `submit_transaction` | `POST /v1/submit` | Sends a signed transaction, or confirms one already sent | Yes |
| `get_token` | `GET /v1/tokens/{mint}` | Progress, graduation and unclaimed fees | No |
| `list_launches` | `GET /v1/creators/{wallet}/launches` | Tokens a wallet launched here | No |
| `swap` | `POST /v1/swap` | Prepares a buy or sell on the curve | Yes, you sign it |
| `claim_fees` | `POST /v1/claim` | Prepares the claim of your creator fees | Yes, you sign it |
| `migrate` | `POST /v1/migrate` | Prepares the migration of a curve that reached the threshold. Meteora does it automatically on mainnet | Yes, you sign it |

On the MCP, the wallet is the one you signed in with; `list_launches` also takes any `wallet`. Over HTTP you pass `wallet`. The HTTP routes need no key; their full contract is at `https://launchpad.deside.io/openapi.json`.

**Every operation that changes something returns an unsigned transaction.** Nothing moves until you sign it.

Parameters, responses and real examples for each one are in the [Operations reference](docs/operations.md). To call them over HTTP step by step, see the [REST quickstart](docs/rest-quickstart.md).

Volume, fees by recipient, locked liquidity, holders and their hourly or daily history are in four read-only routes: see the [Statistics reference](docs/stats.md).

## A real launch

DESIDE, the token of Deside, was launched here on 2026-10-02:

| What | Value |
|---|---|
| Mint | `Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1` |
| Launch transaction | `3DoZFYKeUCWt25cPE8dVjf9MAbUihTEuxv1siTxzk6mVZVCc97xHKFjJpaarP8VddRB8JnZFj7xoZR5QPRF9L6Tr` |
| Pool | `D4x5pLnvD1PuX8RwcHQWCzsvzJeiTdsHiMifYWkL4pZp` |
| Creator | `5DZsMz44aH4JotKKcp58Xo6V4WCUMg4bF7CMQ9ySVZJN` |
| Config | `6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn` |
| Metadata | `https://gateway.irys.xyz/2pJgy1jA1Yr6KKWGyWaZ4LUZTcruyaJQm8cayRhN22RX` |

Read it yourself:

```bash
curl "https://launchpad.deside.io/v1/tokens/Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1?network=mainnet"
```

It was launched without an agent identity, so `agentAsset` is `null`.

## Next steps

- [Fees and rules](docs/fees-and-rules.md): fees, graduation, limits and the creator terms.
- [Operations reference](docs/operations.md): the 10 operations with real examples.
- [REST quickstart](docs/rest-quickstart.md): a launch with `curl` and a signing script.
- [Statistics reference](docs/stats.md): totals, per-token figures and their history.
- [MCP](../mcp/README.md): connect Claude or your agent.

