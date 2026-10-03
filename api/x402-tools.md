# x402 tools

These routes read the x402 Tool Directory with no key. An **x402 tool** is an HTTP API you pay per call, with no account. The directory reads the 5 public catalogs (PayAI, Coinbase CDP, Dexter, thirdweb and OpenFacilitator) and the x402 indexes that agents publish. **One URL is one tool**, even when several catalogs list it.

## GET /public/x402/tools

Returns tools that a catalog or index still lists, with their last check.

```bash
curl "https://api.deside.io/api/v1/public/x402/tools?live=1&limit=1"
```

| Name | Type | Default | Description |
|---|---|---|---|
| `q` | string | | Text search, up to 512 characters. Returns one page; it cannot be combined with `cursor`. |
| `host` | string | | Exact host. |
| `bazaar` | string | | Exact catalog: `payai`, `cdp`, `dexter`, `thirdweb` or `openfac` (OpenFacilitator). |
| `payTo` | string | | Exact receiving wallet. |
| `network` | string | | Exact network, such as `eip155:8453` or `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`. |
| `agent` | string | | An agent's `id` or slug: its tools. The response adds an `agent` key. |
| `index` | string | | An index URL, up to 2,048 characters: the tools it lists. Cannot be combined with `agent`. |
| `indice` | string | | Same as `index`. Cannot be combined with it. |
| `live` | string | | Only `1`: tools whose last check got an answer (`offer` or `no-offer`). |
| `limit` | integer | 25 | 1 to 100. Outside that: `400`. |
| `cursor` | string | | `pagination.nextCursor`. It only works with the same filters. |

Text filters accept up to 512 characters; `index` up to 2,048. **Any other parameter returns `400 unknown filter`.** Without `q`, a page cannot go past row 500: `400 cursor exceeds the public depth limit`.

```json
{
  "items": [
    {
      "slug": "abi-cyberwarex-abi",
      "title": "Contract ABI",
      "titleSource": "name",
      "host": "abi.cyberwarex.com",
      "path": "/abi",
      "scheme": "https",
      "bazaars": [
        "cdp"
      ],
      "bazaarCount": 1,
      "hasInputSchema": true,
      "hasOutputSchema": true,
      "walletCount": 1,
      "firstWallet": {
        "address": "0x058D0Cc5CC97e61e8A9f38D6d6365bce525921B2",
        "family": "evm"
      },
      "prices": [
        {
          "bazaar": "cdp",
          "scheme": "exact",
          "network": "eip155:8453",
          "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
          "amount": "3000",
          "payTo": "0x058D0Cc5CC97e61e8A9f38D6d6365bce525921B2",
          "maxTimeoutSeconds": 300
        }
      ],
      "distinctAmounts": 1,
      "probe": {
        "verdict": "offer",
        "at": "2026-10-01T07:20:00.467Z",
        "httpStatus": 402,
        "quote": {
          "amount": "3000",
          "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
          "network": "eip155:8453"
        },
        "comparison": {
          "sameAmount": true,
          "sameWallet": true
        }
      },
      "declaredByAgent": false,
      "description": "Fetch a verified contract's ABI and metadata by address, keylessly, from Sourcify. Returns the ABI plus a summarized list of function and event signatures, the…",
      "type": "http",
      "logo": {
        "card": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/x402-tool-icon/4531c746a008717a9d74e787bc2db1c0-card.webp",
        "profile": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/x402-tool-icon/4531c746a008717a9d74e787bc2db1c0-profile.webp"
      },
      "tier": 1,
      "completeness": 5
    }
  ],
  "pagination": {
    "nextCursor": "eyJvIjoxLCJmIjoiYWZlZTcxYjZiYmI4NDc4YyJ9",
    "hasMore": true,
    "limit": 1,
    "total": 25794
  }
}
```

| Field | Description |
|---|---|
| `slug` | Stable id of the tool. |
| `title`, `description`, `type` | What the tool is. `type` is how it is called, such as `http`. |
| `titleSource` | Whether `title` came from the tool's name or its path. |
| `host`, `path`, `scheme` | Where it lives. |
| `bazaars[]` | The catalogs that list it. |
| `bazaarCount` | How many catalogs list it. |
| `hasInputSchema`, `hasOutputSchema` | Whether the tool publishes its input and output format. |
| `walletCount`, `firstWallet` | How many wallets it collects to, and the first one. |
| `prices[]` | The price each catalog publishes: `bazaar`, `scheme`, `network`, `asset`, `amount` in the asset's smallest unit, `payTo`, `maxTimeoutSeconds`. |
| `distinctAmounts` | How many different amounts the catalogs publish for it. |
| `probe` | Our last call. See below. |
| `declaredByAgent` | `true` when an agent in the Agent Directory lists the tool as its own. |
| `logo` | Our cached icon, two sizes. |
| `tier` | Our last check, as a number from 1 to 5: 1 quoted a price, 2 answered without charging, 3 not checked yet, 4 and 5 failed. |
| `completeness` | How complete the tool's record is, 0 to 5 points. Tools on test networks always score 0. |

### probe

**The check reads the payment request and does not pay.**

| Field | Description |
|---|---|
| `verdict` | `offer`: answered with a price. `no-offer`: answered, asked for no payment. `no-response`: did not answer. `blocked`: its host blocked our check. `no-endpoint`: the host answers, but this route does not exist. |
| `at` | When we checked. |
| `httpStatus` | The status the tool answered with. |
| `quote` | The price the tool asked: `amount`, `asset`, `network`, and on the single tool `payTo`. |
| `comparison` | `sameAmount` and `sameWallet`: whether the asked price and wallet match the catalog's. The single tool adds `undeclaredNetwork`. |

## GET /public/x402/tools/{slug}

Returns one tool, including retired ones. Adds to the list fields:

| Field | Description |
|---|---|
| `method` | The HTTP method, when known. |
| `inputSchema`, `outputSchema` | The formats the tool publishes, as published. |
| `callDescriptor` | How to call it, when known. |
| `wallets[]` | Every receiving wallet: `address`, `family`, `networks`, `bazaars`. |
| `firstSeenAt`, `lastSeenAt` | When a catalog first and last listed it. |
| `retired`, `retiredAt` | `true` when no catalog and no agent lists it any more, and when that happened. |
| `agent` | The agent that lists it, or `null`. |
| `index` | The indexes that list it: `agents`, `indexes`, `served`, `wellKnown`. |
| `fieldSources` | Which catalog gave each field. |
| `payToSources` | Which catalog gave each receiving wallet. |
| `projectedAt` | When we last rebuilt this record. |

`404 {"error":"not_found"}` when no tool has that slug.

## GET /public/x402/census

Returns the directory counts, without retired tools. This reads the counts:

```bash
curl "https://api.deside.io/api/v1/public/x402/census"
```

Accepts `host`, `payTo`, `network` and `live=1`.

Real on 2026-10-03, trimmed:

```json
{
  "tools": 38715,
  "hosts": 3005,
  "wallets": 2551,
  "bazaars": 5,
  "byBazaar": [{ "n": 24404, "bazaar": "cdp" }, { "n": 12333, "bazaar": "payai" }],
  "byNetwork": [{ "n": 29250, "network": "eip155:8453" }],
  "byVerdict": [{ "verdict": "offer", "n": 24337 }, { "verdict": "no-endpoint", "n": 6676 }],
  "projectedAt": "2026-10-03T06:22:06.370Z"
}
```

`byBazaar` sums more than `tools` because one tool can be in several catalogs. `byVerdict` sums less than `tools`: tools not checked yet have no verdict.

## GET /public/x402/indices

Returns the x402 indexes that agents publish, with what each lists. Groups with 0 tools are filtered out.

```bash
curl "https://api.deside.io/api/v1/public/x402/indices?limit=2"
```

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 25 | 1 to 100. Outside that: `400`. Any other parameter returns `400 unknown filter`. |
| `cursor` | string | | `pagination.nextCursor`. |

```json
{
  "items": [
    {
      "id": "https://agent402.tools/.well-known/x402",
      "host": "agent402.tools",
      "indexes": ["https://agent402.tools/.well-known/x402"],
      "agents": 1,
      "content": { "state": "live", "tools": 604, "quotesPrice": 601, "respondsNoPrice": 1, "down": 2, "notChecked": 0 }
    }
  ],
  "pagination": { "nextCursor": "eyJvIjoyfQ", "hasMore": true, "limit": 2, "total": 93 }
}
```

| Field | Description |
|---|---|
| `id`, `host`, `indexes[]` | The index. |
| `agents` | How many agents declare it. |
| `content.state` | `live` (at least one tool answers) or `down`. |
| `content.tools` | Tools it lists, then how many quote a price, respond with no price, are down, or are not checked yet. |

## Errors

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"invalid_request","nextStep":"unknown filter: foo"}` | A bad parameter. `nextStep` says which. Both `/tools` and `/indices` reject unknown filters. `indice` and `index` cannot be combined. |
| 404 | `{"error":"not_found"}` | No such tool. |
| 429 | `{"error":"RATE_LIMITED"}` | Over the limit for your IP: 30 requests per minute on `/tools` and `/indices`, 60 on the others, 500 per day in total. Wait for the seconds in `RateLimit-Reset`. |
| 500 | `{"error":"internal_error"}` | Retry later. |
