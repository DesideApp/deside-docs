# Statistics reference

The Deside Launchpad publishes its statistics in four read-only routes on `https://launchpad.deside.io`: totals for the whole launchpad, the detail of one token, and a time series of each. Every figure is read from the chain or from the swaps of each pool. A figure that cannot be read exactly comes back `null`, never estimated.

{% hint style="info" %}
On this page:
- `GET /v1/stats`: totals, totals per version and one row per token;
- the `launchpad` block of `GET /v1/tokens/{mint}`: one token, with holders;
- `GET /v1/stats/history` and `GET /v1/tokens/{mint}/history`: hourly or daily points;
- the limits, the errors and what these figures do not include.
{% endhint %}

The routes need no key. Their schema is in `https://launchpad.deside.io/openapi.json`. They are not MCP tools, except the `launchpad` block, which `get_token` also returns.

## Core concepts

### Versions

**A version is one Meteora DBC config of the launchpad, and it never changes.** Each token stays bound to the version it was launched with, shown in its `version` and `dbcConfig`. A new fee schedule means a new version, not a change to an old one. Today there is one version, `v1`, with the config `6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn`. `totals` adds up every version and `byVersion` gives each one apart.

### Fees by recipient

Every fee is given per recipient as `{ generated, claimed, unclaimed }`, in SOL. `generated` is `claimed` plus `unclaimed`.

- **Curve fees** (`curveFeesSol`) are the fees of the bonding curve, for `creator`, `deside` and `meteora`.
- **Pool fees** (`poolFeesSol`) are the fees of the DAMM v2 pool after graduation, read position by position. `thirdPartyLps` is what liquidity providers outside the launch earned, given as `generated` only. **Pool fees are counted only in SOL.** The v1 pool collects its fees in SOL; a version that collected fees in the token too would return `null` here.

### Locked liquidity

**`lockedLiquidity` is the liquidity of the 2 positions locked at graduation: 90% for the creator and 10% for Deside.** Liquidity added later by anyone else is apart, in `thirdPartyLiquidity`. Amounts are the pool reserves in proportion to each position's share of the liquidity.

### Holders

**`holders` counts the token accounts with a balance above 0 whose owner is a wallet.** Accounts owned by a program, such as the pool vaults and other program addresses, are left out and counted in `programOwnedAccounts`. The rule travels with the figure in `holdersRule`. A wallet is any key that can sign, so a bot's wallet counts as a holder. If the accounts cannot be read, `holders`, `programOwnedAccounts` and `holdersRule` are missing from the response.

### Prices

`priceSol` is the price on the curve, or on the pool once graduated. `solUsd` is the SOL price from Jupiter Price v3, refreshed every 60 seconds. `marketCapUsd` uses the real supply of the mint, which can be slightly under 1,000,000,000 when tokens were burned.

## GET /v1/stats

Returns the totals of the launchpad, the totals per version, the 20 most recent launches and one row per token.

The response is cached for 60 seconds per network. While a new read runs, the previous one is served if it is less than 5 minutes old; `updatedAt` says when it was read.

### Parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. |

### Example request

```bash
curl "https://launchpad.deside.io/v1/stats?network=mainnet"
```

### Example response

Real, 2026-10-06, trimmed: `byVersion.v1` repeats the shape of `totals`, and one of the 3 token rows is shown.

```json
{
  "network": "mainnet",
  "dbcConfig": "6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn",
  "updatedAt": "2026-10-06T06:04:20.876Z",
  "solUsd": 119.4015056102662,
  "versions": [
    { "version": "v1", "dbcConfig": "6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn", "current": true }
  ],
  "totals": {
    "tokens": { "launched": 3, "graduated": 1, "inCurve": 2 },
    "volumeSol": { "curve": 285.806952885, "pool": 1432.72371901, "total": 1718.530671895 },
    "curveFeesSol": {
      "creators": { "generated": 1.19122787, "claimed": 0, "unclaimed": 1.19122787 },
      "deside": { "generated": 1.191228024, "claimed": 0, "unclaimed": 1.191228024 },
      "meteora": { "generated": 0.532412069, "claimed": 0.532412069, "unclaimed": 0 }
    },
    "poolFeesSol": {
      "creators": { "generated": 51.644568729, "claimed": 51.644568729, "unclaimed": 0 },
      "deside": { "generated": 5.738285413, "claimed": 0, "unclaimed": 5.738285413 },
      "meteora": { "generated": 12.896633776, "claimed": 12.556910592, "unclaimed": 0.339723184 },
      "thirdPartyLps": { "generated": 0.192737749 }
    },
    "lockedLiquidity": {
      "solSide": { "creators": 31.325346257, "deside": 3.480594028 },
      "valueSol": { "creators": 62.650693034, "deside": 6.961188114 }
    },
    "graduationThresholdSol": 93.493145759
  },
  "byVersion": { "v1": { "dbcConfig": "6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn", "current": true } },
  "recentLaunches": [
    {
      "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
      "symbol": "DESIDE",
      "version": "v1",
      "launchedAt": "2026-10-02T06:48:17.000Z",
      "hasAgentIdentity": true,
      "graduated": true,
      "progress": 1,
      "volumeSol": { "curve": 285.806952885, "pool": 1432.72371901, "total": 1718.530671895 }
    }
  ],
  "notIncluded": [
    "volume on a graduated DAMM v2 pool until the indexer has read all its swaps: until then volumeSol.pool and volumeSol.total are null",
    "pools of the same token created by third parties (not part of the launchpad)",
    "referral already claimed"
  ],
  "tokens": [
    {
      "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
      "name": "Deside",
      "symbol": "DESIDE",
      "version": "v1",
      "dbcConfig": "6d1TRnC8xvb43ErsUSVcHWPoTMta7zAq7s3pnihemtdn",
      "launchedAt": "2026-10-02T06:48:17.000Z",
      "graduatedAt": "2026-10-04T23:59:01.000Z",
      "migrationSignature": "5zCQkvvRSW6iZD9CR6joQGmcKujsAkVrQVcMhxAdvSMaE82yi3EfLoThCYyyvxiDs9EuUTj9EvzRGYYzBECLKAWR",
      "graduated": true,
      "curve": { "raisedSol": 93.493145823, "progress": 1, "volumeSol": 285.806952885 },
      "dammV2Pool": "ArDDTDJF5BT1ipUQr695FWhH6nLZwdVtcUnphjsHu1Pe",
      "lockedLiquidity": {
        "creator": { "tokens": 530886374.001553, "sol": 31.325346257 },
        "deside": { "tokens": 58987374.884003, "sol": 3.480594028 }
      },
      "thirdPartyLiquidity": { "tokens": 3220085.768982, "sol": 0.190003561 },
      "priceSol": 5.9005746448255643e-08,
      "priceUsd": 7.045374965579341e-06,
      "supply": 999999962.007006,
      "marketCapUsd": 7045.374697904453
    }
  ]
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `network`, `dbcConfig` | string | The network and its current config. |
| `updatedAt` | string | ISO time of the read. `asOf` carries the same value. |
| `solUsd` | number or null | SOL price in USD from Jupiter Price v3. |
| `versions[]` | array | Each version: `version`, `dbcConfig`, `current`. |
| `totals` | object | All versions together. See the table below. |
| `byVersion.{version}` | object | The same shape as `totals`, for one version, plus `dbcConfig` and `current`. |
| `recentLaunches[]` | array | Up to 20 tokens, newest first: `mint`, `name`, `symbol`, `logo`, `creator`, `version`, `launchedAt`, `hasAgentIdentity`, `graduated`, `progress`, `volumeSol`. |
| `notIncluded[]` | array of strings | What these figures leave out, in words. |
| `tokens[]` | array | One row per token, newest first. See [Token row](#token-row). |

Fields of `totals` and of each `byVersion` entry:

| Field | Type | Description |
|---|---|---|
| `tokens` | object | `launched`, `graduated` and `inCurve`. They count every token of the config, including internal test tokens that are not listed, so they can be higher than the rows in `tokens` and `recentLaunches`. |
| `volumeSol` | object | `curve`, `pool` and `total`. `pool` and `total` are `null` while any graduated pool has not been read swap by swap to the end. |
| `curveFeesSol` | object | `creators`, `deside`, `meteora`, each `{ generated, claimed, unclaimed }`. |
| `poolFeesSol` | object | `creators`, `deside`, `meteora`, each `{ generated, claimed, unclaimed }`, and `thirdPartyLps.generated`. |
| `lockedLiquidity` | object | `solSide`: the SOL in the locked positions. `valueSol`: their value in SOL, both sides. Token amounts are not added up, because they are of different mints. |
| `graduationThresholdSol` | number | SOL the curve must raise to graduate. |

`totals` and `byVersion` also carry the fields of an earlier shape (`launches`, `creators`, `traded`, `graduated`, `solInCurves`, `curveVolumeSol`, `deside`, `creatorFeesEarnedSol`, `meteoraFeesSol`). They are kept for existing readers. New clients read the fields above.

### Token row

| Field | Type | Description |
|---|---|---|
| `mint`, `name`, `symbol`, `logo`, `creator` | string or null | The token. `logo` is an Arweave or Irys URL, or `null`. |
| `pool` | string | The bonding curve pool. |
| `version`, `dbcConfig` | string | The version the token belongs to, and its config. |
| `launchedAt` | string | ISO time the curve opened. |
| `graduatedAt` | string or null | ISO time of the migration transaction to the DAMM v2 pool. |
| `migrationSignature` | string or null | That migration transaction. |
| `graduated`, `traded` | boolean | Whether it graduated, and whether it had at least one trade. |
| `agentAssets` | array or null | Agent identity assets of the creator. `null` when they could not be read. |
| `curve` | object | `raisedSol`, `progress` (0 to 1) and `volumeSol`. |
| `dammV2Pool` | string or null | The pool after graduation. |
| `volumeSol` | object | `curve`, `pool` and `total`, in SOL. |
| `curveFeesSol` | object or null | `creator`, `deside`, `meteora`, each `{ generated, claimed, unclaimed }`. `creator` and `deside` are `null` until every claim on the curve has been read. |
| `poolFeesSol` | object or null | `creator`, `deside`, `meteora` and `thirdPartyLps`. `null` before graduation. |
| `lockedLiquidity` | object or null | `creator` and `deside`, each `{ tokens, sol }`. `null` before graduation. |
| `thirdPartyLiquidity` | object or null | `{ tokens, sol }` of every other position in the pool. |
| `priceSol`, `priceUsd` | number or null | Price of one token. |
| `supply` | number or null | Real supply of the mint. |
| `marketCapUsd` | number or null | `priceUsd` times `supply`. |
| `feesSol` | object | Curve fees in an earlier shape, kept for existing readers. |

**Curve volume is `null` for a version with a dynamic fee, until every swap of that curve has been read.** The v1 curve has no dynamic fee.

## The launchpad block of GET /v1/tokens/{mint}

Returns, for a token launched here, the same [token row](#token-row) inside `launchpad`, plus its holders. For a token of another config the block is missing.

The block always has a `status`:

- `ready`: every field below is filled.
- `pending`: the token's statistics are still being read from the chain, for example right after a fee claim. The block then has only `status`, `retryAfterSeconds: 10` and a `note`. Ask again after those seconds.

The rest of the response is described in the [Operations reference](operations.md#get_token).

The block adds:

| Field | Type | Description |
|---|---|---|
| `holders` | number | Wallets that hold the token. See [Holders](#holders). |
| `programOwnedAccounts` | number | Token accounts with a balance whose owner is a program. |
| `holdersRule` | string | The rule, in words. |
| `solUsd` | number or null | SOL price in USD. |
| `updatedAt` | string | ISO time of the read. |

Holders are cached for 5 minutes. This route has the general limit of 60 requests a minute, not the statistics limit.

```bash
curl "https://launchpad.deside.io/v1/tokens/Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1?network=mainnet"
```

Real, 2026-10-06, trimmed to the fields the block adds:

```json
{
  "launchpad": {
    "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
    "version": "v1",
    "graduatedAt": "2026-10-04T23:59:01.000Z",
    "holders": 66,
    "programOwnedAccounts": 7,
    "holdersRule": "token accounts with balance > 0 whose owner is a wallet (on-curve); program-owned accounts (pool vaults, PDAs) excluded",
    "solUsd": 119.4015056102662,
    "updatedAt": "2026-10-06T06:05:02.220Z"
  }
}
```

## GET /v1/stats/history

Returns a point per hour or per day for the whole launchpad: launches, graduations, volume and fees in the period, and the running total.

**A point with no swaps is a real zero. Points are never filled in:** when a pool has not been read to the end, the response says so with `complete: false` and lists it in `pendingPools`.

### Parameters

| Name | Type | Required | Description | Default |
|---|---|---|---|---|
| `network` | string | Yes | `mainnet` or `devnet`. | |
| `interval` | string | No | `hour` or `day`. Points start at the hour or at 00:00 UTC. | `day` |
| `from` | string | No | ISO date. | 7 days before `to` for `hour`, 90 days for `day` |
| `to` | string | No | ISO date. | Now |
| `version` | string | No | Only the tokens of one version, such as `v1`. An unknown version returns zeros. | All versions |

**At most 2000 points per call.** Ask for a shorter range or a longer interval above that.

### Example request

```bash
curl "https://launchpad.deside.io/v1/stats/history?network=mainnet&interval=day&from=2026-10-05T00:00:00Z"
```

### Example response

Real, 2026-10-06, trimmed to the last point:

```json
{
  "network": "mainnet",
  "interval": "day",
  "version": null,
  "updatedAt": "2026-10-06T06:05:02.374Z",
  "since": "2026-10-01T20:38:59.000Z",
  "complete": true,
  "pendingPools": [],
  "feeNote": "curve fees are split per swap as the program does (creator = floor(trading fee x creator %), Deside the rest); pool lp fees are those of all positions, ours and third parties (the split by position is in /v1/stats); referral fees go to the swap referrer, not to Meteora",
  "points": [
    {
      "t": "2026-10-06T00:00:00.000Z",
      "launches": 0,
      "graduations": 0,
      "volumeSol": { "curve": 0, "pool": 3.70343459, "total": 3.70343459 },
      "feesSol": {
        "curve": { "creator": 0, "deside": 0, "meteora": 0, "referral": 0 },
        "pool": { "lp": 0.014945106, "meteora": 0.003736272, "referral": 0 }
      },
      "cumulative": {
        "launches": 3,
        "graduations": 1,
        "volumeSol": { "curve": 285.806952885, "pool": 1432.72371901, "total": 1718.530671895 },
        "feesSol": {
          "curve": { "creator": 1.19122787, "deside": 1.191228024, "meteora": 0.532412069, "referral": 0.063201737 },
          "pool": { "lp": 57.575591891, "meteora": 12.896633776, "referral": 1.497263046 }
        }
      }
    }
  ]
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `interval`, `version` | string or null | As asked. |
| `since` | string or null | ISO time of the first launch counted. |
| `complete` | boolean | `false` when a pool has not been read to the end. |
| `pendingPools[]` | array | Each pool not read yet: `mint`, `kind` (`curve` or `pool`) and `pool`. |
| `feeNote` | string | How fees are split, in words. |
| `points[].t` | string | ISO start of the period. |
| `points[].launches`, `points[].graduations` | number | In the period. |
| `points[].volumeSol` | object | `curve`, `pool` and `total` in the period. |
| `points[].feesSol.curve` | object | `creator`, `deside`, `meteora` and `referral`. |
| `points[].feesSol.pool` | object | `lp`, `meteora` and `referral`. |
| `points[].cumulative` | object | The same fields, from the first launch to the end of the period. |

**In the history, pool fees are split only into `lp`, `meteora` and `referral`.** `lp` is the fees of every position in the pool, the locked ones and those of third parties. The split by position (creator, Deside, third parties) is only in `GET /v1/stats`. `referral` is what the referrer of each swap received. On the curve, each swap's creator share is the trading fee times the creator percentage rounded down, and Deside gets the rest, as the program does.

## GET /v1/tokens/{mint}/history

Returns a point per hour or per day for one token launched here: volume, fees, curve progress, price and market cap.

It takes the same `network`, `interval`, `from` and `to` as [`GET /v1/stats/history`](#get-v1statshistory), with the same 2000-point cap. It has no `version`: the token's version comes in the response.

### Example request

```bash
curl "https://launchpad.deside.io/v1/tokens/Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1/history?network=mainnet&interval=hour&from=2026-10-06T06:00:00Z"
```

### Example response

Real, 2026-10-06, trimmed:

```json
{
  "network": "mainnet",
  "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
  "interval": "hour",
  "version": "v1",
  "launchedAt": "2026-10-02T06:48:17.000Z",
  "graduatedAt": "2026-10-04T23:59:01.000Z",
  "complete": true,
  "pendingPools": [],
  "usdNote": "marketCapUsd uses the SOL price the indexer stored for that hour; before the indexer existed it is null",
  "points": [
    {
      "t": "2026-10-06T06:00:00.000Z",
      "volumeSol": { "curve": 0, "pool": 0, "total": 0 },
      "feesSol": {
        "curve": { "creator": 0, "deside": 0, "meteora": 0, "referral": 0 },
        "pool": { "lp": 0, "meteora": 0, "referral": 0 }
      },
      "curveProgress": 1,
      "priceSol": 5.9005746448255643e-08,
      "marketCapSol": 59.00574420645068,
      "marketCapUsd": 7045.374697904453,
      "cumulative": {
        "volumeSol": { "curve": 285.806952885, "pool": 1432.72371901, "total": 1718.530671895 },
        "feesSol": {
          "curve": { "creator": 1.19122787, "deside": 1.191228024, "meteora": 0.532412069, "referral": 0.063201737 },
          "pool": { "lp": 57.575591891, "meteora": 12.896633776, "referral": 1.497263046 }
        }
      }
    }
  ]
}
```

### Response fields

Besides the fields shared with the global history (`complete`, `pendingPools`, `feeNote`, `volumeSol`, `feesSol`, `cumulative`):

| Field | Type | Description |
|---|---|---|
| `launchedAt`, `graduatedAt` | string or null | As in the [token row](#token-row). |
| `usdNote` | string | Where `marketCapUsd` comes from, in words. |
| `points[].curveProgress` | number or null | 0 to 1, from the last swap seen; 1 after graduation. `null` before the first swap. |
| `points[].priceSol` | number or null | Price after the last swap seen. A period with no swaps keeps the previous price. `null` before the first swap. |
| `points[].marketCapSol` | number or null | `priceSol` times today's supply. |
| `points[].marketCapUsd` | number or null | `marketCapSol` times the SOL price stored for the last hour of the period. |

**`marketCapUsd` has values only from 2026-10-05, when SOL prices started to be stored. Earlier points are `null`.** The first hourly point with a value is 2026-10-05T23:00Z. A daily point gets a value once its last hour has a stored price, so the point of the current day stays `null` until the day ends.

## Limits

| Limit | Value |
|---|---|
| Statistics routes (`/v1/stats`, `/v1/stats/history`, `/v1/tokens/{mint}/history`) | 10 requests a minute per IP, and 120 a minute for all callers together. |
| `GET /v1/tokens/{mint}` | 60 requests a minute per IP, the general limit. |
| `/v1/stats` cache | 60 seconds per network. |
| Holders cache | 5 minutes per token. |
| History | 2000 points per call. |

## Errors

| Code | When | What to do |
|---|---|---|
| `400` `invalid input` | `network` missing, or `interval` other than `hour` or `day`. `issues[]` names the field. | Fix the parameter. |
| `400` `from must be an ISO date` | `from` or `to` does not parse. | Send an ISO date, such as `2026-10-05T00:00:00Z`. |
| `400` `from must be before to` | The range is reversed. | Swap them. |
| `400` `too many points: at most 2000 per call` | The range holds more than 2000 periods. | Shorten the range or use `day`. |
| `404` `not a token of this launchpad` | Token history for a mint not launched here. | Check the mint and the network. |
| `429` `too many statistics requests, slow down` | Over the statistics limit. | Wait a minute. Cache what you read for 60 seconds. |
| `503` `history is not enabled on this server` | History is not available on this server. | Retry later. |

## What these figures do not include

- Volume of a graduated pool that has not been read swap by swap to the end: `volumeSol.pool` and `volumeSol.total` are `null` until then.
- Pools of the same token created by third parties, outside the launchpad.
- Referral fees already claimed.
