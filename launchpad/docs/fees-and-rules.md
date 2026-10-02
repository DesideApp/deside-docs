# Fees and rules

Every token from the launchpad uses one launch preset, `agent-standard`. This page lists what that preset charges, who receives each share, what a launch costs and the limits of the service. `get_launchpad_info` returns the same terms at run time.

## The token

Each token has a supply of 1,000,000,000 tokens with 6 decimals. Its metadata is immutable and it has no mint authority, no vesting and no pre-buys.

## Trading fees on the curve

| Share | Of each trade |
|---|---|
| Creator | 0.40% |
| Deside | 0.40% |
| Meteora | 0.20% |
| Total | 1% |

The creator collects its share with [`claim_fees`](operations.md#claim_fees).

### Anti-sniper fee

**For the first 120 seconds after launch the trading fee starts at 25% and falls linearly to 1%.** Trades in that window pay more; set `slippageBps` on `swap` with that in mind.

## Graduation

When the curve has raised its threshold in SOL, it migrates to a Meteora DAMM v2 pool.

| Network | Threshold |
|---|---|
| `mainnet` | 50,000 USD market cap: about 93.5 SOL raised, fixed in SOL when the configuration was created |
| `devnet` | 400 USD market cap: about 0.64 SOL raised, for testing |

On mainnet the Meteora keeper migrates the curve automatically, normally within seconds. If it has not, anyone can run [`migrate`](operations.md#migrate).

## After graduation

| Item | Rule |
|---|---|
| Pool fee | Starts at 5% and falls to 0.5% as the price grows 10x, or after 24 hours, plus a volatility fee |
| Liquidity | 100% of the pool liquidity is locked permanently: 90% creator position, 10% Deside position |
| Locked positions | Both keep earning pool fees. The creator collects with `claim_fees`. |

## What a launch costs

Deside charges no launch fee. The wallet that launches pays the network rent and fees and the Arweave storage, all in the launch signature.

| Item | SOL |
|---|---|
| Token and curve | About 0.0206 |
| Agent identity, if requested | About 0.0049 |
| Arweave storage | At cost, about 0.00005 for a small logo |

A devnet launch with an agent identity and an 8.8 KB PNG logo cost the launching wallet 0.025420416 SOL in total, of which 0.000047176 SOL paid the Arweave storage.

## Limits

| Limit | Value |
|---|---|
| Requests per IP | 60 per minute |
| Launches per IP | 10 per minute |
| Request body | 2 MB |
| Logo file (`imageBase64`) | 1 MB, PNG, JPG, WebP or GIF |
| Token name, symbol, description | 32, 10 and 500 characters |
| Transaction size | 1232 bytes |
| Time to sign and submit | About 60 seconds |
| Time to report a self-sent launch | 15 minutes |

## Custody

Deside holds no user keys and sends nothing you have not signed. `submit_transaction` only forwards transactions that call the Meteora DBC or DAMM v2 programs and no program outside the launchpad's list, so it is not a general relay.

## Creator terms

**`launch_token` requires `acceptTerms: true`.** By sending it you accept the creator terms. In short:

* You are the only one responsible for your token and for everything you or your agent say or do about it.
* A token launched here raises no capital and gives its holders no rights: no ownership, profit share, dividend or vote.
* You must not promise or suggest returns, misrepresent the fees, or manipulate the market of any token.

This summary does not replace the terms. The full text, with its version, is at [launchpad.deside.io/terms](https://launchpad.deside.io/terms) and in the `creatorTerms` field of `get_launchpad_info`.

## Disclaimer

The service returns this text in `get_launchpad_info` and in every `launch_token` response:

> Deside does not custody funds or keys. This service only builds Meteora Dynamic Bonding Curve transactions that you sign with your own wallet. Tokens are created by their signer, not by Deside. Deside is the fee partner of the launch configuration and receives the partner share of trading fees stated below. Nothing here is investment advice; launching or trading a token can lose all the money involved.
