# DESIDE reference

`GET /v1/deside` returns what the DESIDE token does in the Deside trading app: the fee table, the weekly buyback of DESIDE, what each buyback burned and sent to liquidity, and a daily point of the DESIDE supply and the liquidity balance.

{% hint style="info" %}
On this page:
- what the route is for;
- `GET /v1/deside`: the parameters, a real response and every field;
- `GET https://api.deside.io/api/v1/trading/fees`: the fee table on its own;
- the errors.
{% endhint %}

## What it is for

In the Deside trading app, holding DESIDE lowers the fee you pay on other tokens: the more DESIDE you hold, the lower the fee, in three tiers. Part of every fee paid in a tier is set aside to buy back DESIDE once a week. Half of what is bought back is burned, and the other half goes as USDC to the Deside USDC Liquidity wallet. This route publishes each of those steps with the amounts read on chain, so anyone can check them.

## GET /v1/deside

Returns the DESIDE data of the trading app. It exists only on mainnet.

The route needs no key and shares the limit of the [statistics routes](stats.md). It is refreshed once an hour.

### Parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `network` | string | yes | `mainnet`. Any other network answers `400`. |

### Example request

```bash
curl "https://launchpad.deside.io/v1/deside?network=mainnet"
```

### Example response

A real response from 2026-10-08, the first day of the series:

```json
{
  "schemaVersion": 1,
  "network": "mainnet",
  "updatedAt": "2026-10-08T00:17:28.121Z",
  "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
  "decimals": { "deside": 6, "usdc": 6 },
  "wallets": {
    "feeOwner": "HX9n42adqPmjTp99DgjPBrHmYyWfdYE76u7cLrJJtsq6",
    "liquidity": "BTajayfse6t9dtpa17aGAvzb4PSCWJXw3UVFybFuPbTX"
  },
  "fees": {
    "schemaVersion": 1,
    "enabled": true,
    "convertBps": 10,
    "launchpadBps": 30,
    "restBps": 75,
    "tiers": [
      { "tier": "t1", "minDeside": 100000, "bps": 73, "buybackBpsX100": 175 },
      { "tier": "t2", "minDeside": 200000, "bps": 70, "buybackBpsX100": 250 },
      { "tier": "t3", "minDeside": 400000, "bps": 64, "buybackBpsX100": 400 }
    ],
    "holdDays": 7,
    "desideMint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
    "feeOwner": "HX9n42adqPmjTp99DgjPBrHmYyWfdYE76u7cLrJJtsq6",
    "updatedAt": "2026-10-08"
  },
  "weeks": [
    { "start": "2026-10-05", "end": "2026-10-12", "closed": false, "byMint": [], "burn": null }
  ],
  "totals": { "desideBurnedRaw": "0", "usdcToLiquidityRaw": "0" },
  "days": [
    { "day": "2026-10-08", "supplyRaw": "999999962007006", "burnedCumulativeRaw": "0", "liquidityUsdcRaw": "0" }
  ],
  "note": "raw amounts are in the smallest unit of each mint (DESIDE and USDC have 6 decimals); ..."
}
```

### Raw amounts

**Every field ending in `Raw` is a string in base units of its mint.** DESIDE and USDC have 6 decimals: `"999999962007006"` DESIDE is 999,999,962.007006 DESIDE. A `byMint` row carries its own `decimals`, because a buyback can be owed in SOL (9 decimals). Read them as big integers, not as floating point numbers.

### Response fields

| Field | Type | Description |
|---|---|---|
| `schemaVersion` | number | `1`. |
| `network` | string | `mainnet`. |
| `updatedAt` | string or null | When the last daily point was taken, ISO 8601. `null` while there is no point. |
| `mint` | string | The DESIDE mint. |
| `decimals.deside` | number | `6`. |
| `decimals.usdc` | number | `6`. |
| `wallets.feeOwner` | string | The wallet that receives the trading fees and makes the weekly buyback and burn. |
| `wallets.liquidity` | string | The Deside USDC Liquidity wallet, which receives the USDC half of each buyback. |
| `fees` | object or null | The fee table of the trading app, exactly as `GET https://api.deside.io/api/v1/trading/fees` returns it (see [below](#get-api-v1-trading-fees)). Cached for 10 minutes. `null` when that route does not answer; it is never filled in by hand. |
| `weeks[]` | array | One week per Monday from `2026-10-05` to the current week, oldest first. |
| `weeks[].start` | string | The Monday the week starts, `YYYY-MM-DD`, at 00:00 UTC. |
| `weeks[].end` | string | The next Monday, at 00:00 UTC. The week does not include it. |
| `weeks[].closed` | boolean | `true` once `end` has passed. The current week is `false` and its figures still grow. |
| `weeks[].byMint[]` | array | The buyback owed that week, one row per currency the fees came in. Empty when no trade in a tier was confirmed that week. |
| `weeks[].byMint[].mint` | string | The mint the fees came in: USDC (`EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`) or wrapped SOL (`So11111111111111111111111111111111111111112`). |
| `weeks[].byMint[].decimals` | number | Decimals of that mint: `6` for USDC, `9` for SOL. |
| `weeks[].byMint[].trades` | number | Confirmed trades in a tier that week that paid fees in that mint. |
| `weeks[].byMint[].buybackRaw` | string | Buyback owed, in base units of that mint. Each trade adds `floor(volume × buybackBpsX100 / 1,000,000)`. |
| `weeks[].burn` | object or null | The buyback and burn of that week. `null` until it is made. |
| `weeks[].burn.signatures` | array of strings | The transactions of the buyback and burn. |
| `weeks[].burn.at` | string | When the burn was recorded, ISO 8601. |
| `weeks[].burn.desideBurnedRaw` | string | DESIDE burned by those transactions, read on chain from their burn instructions. |
| `weeks[].burn.usdcToLiquidityRaw` | string | USDC that entered the Liquidity wallet in those transactions, read on chain from its balance before and after. |
| `totals.desideBurnedRaw` | string | The sum of `desideBurnedRaw` over every week. |
| `totals.usdcToLiquidityRaw` | string | The sum of `usdcToLiquidityRaw` over every week. |
| `days[]` | array | One point per UTC day, oldest first, from 2026-10-08, the first day this service took one. |
| `days[].day` | string | The UTC day, `YYYY-MM-DD`. |
| `days[].supplyRaw` | string | The DESIDE supply read on chain that day. It drops when DESIDE is burned. |
| `days[].burnedCumulativeRaw` | string | DESIDE burned by the weekly burns up to and including that day. |
| `days[].liquidityUsdcRaw` | string | The USDC balance of the Liquidity wallet that day. `"0"` while it holds no USDC account. |
| `note` | string | A reminder of the units and rules on this page, in one sentence. |

**No gap is filled.** A day with no point is missing from `days`, and there are no days before the first point. A week whose buyback has not been made has `burn: null`.

## GET /api/v1/trading/fees

Returns the fee table of the Deside trading app, on `https://api.deside.io`. It is the same object as `fees` above, read from the same constants the app charges with.

The route needs no key and allows 60 requests a minute per IP. It takes no parameters.

```bash
curl "https://api.deside.io/api/v1/trading/fees"
```

A fee in basis points (`bps`) is a share of the trade: 1 bps is 0.01%.

| Field | Type | Description |
|---|---|---|
| `schemaVersion` | number | `1`. |
| `enabled` | boolean | `true` when this table is the one the app charges today. |
| `convertBps` | number | Fee on a conversion between SOL and USDC. Holding DESIDE does not change it. |
| `launchpadBps` | number | Fee on tokens launched on the [Deside Launchpad](../README.md). Holding DESIDE does not change it. |
| `restBps` | number | Fee on any other token for a holder with no tier. |
| `tiers[]` | array | The tiers, from the lowest to the highest. |
| `tiers[].tier` | string | `t1`, `t2` or `t3`. |
| `tiers[].minDeside` | number | DESIDE needed for the tier, in whole DESIDE, not base units. |
| `tiers[].bps` | number | Fee on other tokens in that tier. |
| `tiers[].buybackBpsX100` | number | Share of the trade volume set aside to buy back DESIDE, in hundredths of a basis point: `175` is 1.75 bps, 0.0175% of the volume. |
| `holdDays` | number | Days in a row the DESIDE must be held. The tier comes from the lowest of one daily balance per day over that many consecutive days; with fewer days there is no tier. The balance adds the trading wallet and the Solana wallets linked to the account. |
| `desideMint` | string | The DESIDE mint. |
| `feeOwner` | string | The wallet that receives the fees. |
| `updatedAt` | string | The date this table last changed, `YYYY-MM-DD`. |

## Errors

| Code | When | What to do |
|---|---|---|
| `400` `invalid input` | `network` missing on `/v1/deside`. `issues[]` names the field. | Send `network=mainnet`. |
| `400` `DESIDE trading data exists only on mainnet` | `network` is not `mainnet`. | Send `network=mainnet`. |
| `400` `INVALID_REQUEST` | Any query parameter on `/api/v1/trading/fees`. | Call it without parameters. |
| `429` | Over the limit of either route. | Wait a minute. Cache what you read: `/v1/deside` changes once an hour. |
| `503` `DESIDE data is not enabled on this server` | The data is not available on this server. | Retry later. |
