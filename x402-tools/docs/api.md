# x402 Tools API

The x402 tools API serves the [x402 Tool Directory](../README.md) over HTTP without a key: a page of tools, one tool, counts, x402 indices and wallet matches. These routes are the public read surface described in [Access Model](../../directory-api/docs/access-model.md), with its per-IP limits and its 500-row depth limit. To walk the whole catalog, use [x402 data with an API key](../../directory-api/docs/x402-keyed.md).

What a tool is, where it comes from and what the live check means are in [How The Tool Directory Works](concepts.md). The values of `sonda.veredicto` are listed there.

## `GET /api/v1/public/x402/tools`

Returns a page of tools. Without `q`, tools that quote a price come first.

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `q` | string | no | Text search over the title, tags, description, host and path. Returns one page ordered by relevance and cannot be combined with `cursor`. | |
| `host` | string | no | Exact host, for example `abi.cyberwarex.com`. | |
| `bazaar` | string | no | Catalog id, for example `cdp`. | |
| `payTo` | string | no | Exact receiving wallet address. Case-sensitive. | |
| `network` | string | no | Exact network string of one of the tool's prices, for example `eip155:8453` or `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`. Not normalized: `base` and `eip155:8453` are different values. | |
| `indice` | string | no | URL of an x402 index, as returned by `/x402/indices` below. Returns the tools that index lists. | |
| `live` | `1` | no | Only tools whose last check got an answer (`offer` or `no-offer`). Any value other than `1` is rejected. | |
| `limit` | integer | no | 1 to 100. | `25` |
| `cursor` | string | no | `pagination.nextCursor` of the previous page. | |

Any other parameter answers `400` with `unknown filter: <name>`. Retired
tools are never listed.

### Example request

This request asks for one live tool paid on Solana mainnet:

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/tools?network=solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp&live=1&limit=1"
```

### Example response

A real response from 2026-10-01, with the description cut:

```json
{
  "items": [
    {
      "slug": "aeml-x402-zeabur-stable-graveyard",
      "title": "Truth Bear",
      "titleSource": "name",
      "host": "aeml-x402.zeabur.app",
      "path": "/stable/graveyard",
      "scheme": "https",
      "bazaars": ["cdp"],
      "bazaarCount": 1,
      "hasInputSchema": true,
      "hasOutputSchema": true,
      "walletCount": 2,
      "firstWallet": { "address": "0x2d16a243ba9facc6ac85519d5efad2149dc4c6c3", "family": "evm" },
      "prices": [
        {
          "bazaar": "cdp",
          "scheme": "exact",
          "network": "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
          "asset": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
          "amount": "90000",
          "payTo": "8qEn3Y3GPXdQ28LHshPa4UYeD8V38Lq9b4jCRirQAY3n",
          "maxTimeoutSeconds": 300
        }
      ],
      "distinctAmounts": 1,
      "sonda": {
        "veredicto": "offer",
        "en": "2026-09-29T07:20:00.666Z",
        "estado": 402,
        "oferta": { "importe": "90000", "activo": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913", "red": "base" },
        "cotejo": { "mismoImporte": null, "mismaCartera": null }
      },
      "declaredByAgent": false,
      "description": "[BATCH+PROOF] Stablecoin death register...",
      "tipo": "http",
      "logo": {
        "card": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/x402-tool-icon/e330e2d47f821f504432118aaa95c85a-card.webp",
        "profile": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/x402-tool-icon/e330e2d47f821f504432118aaa95c85a-profile.webp"
      },
      "bloque": 1,
      "completitud": 5
    }
  ],
  "pagination": {
    "nextCursor": "eyJvIjoxLCJmIjoiNGEwYzFhZDQyZjdkYzk2ZCJ9",
    "hasMore": true,
    "limit": 1,
    "total": 6700
  }
}
```

The tool has two prices; the second was cut from this example.

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `slug` | string | Stable id of the tool. Its page on the website is `https://deside.io/x402/t/<slug>`. |
| `title` | string | Name of the tool. |
| `titleSource` | string | `name` when a catalog declared the name, `path` when Deside built it from the URL path. |
| `host`, `path`, `scheme` | string | Where the tool lives. |
| `bazaars` | string[] | Catalogs that list the tool. `bazaarCount` is their number. |
| `hasInputSchema`, `hasOutputSchema` | boolean | Whether a catalog published an input or output schema. |
| `walletCount` | integer | Number of distinct receiving wallets. `firstWallet` is the first of them, with its `family` (`evm`, `solana` or another chain family). |
| `prices` | object[] | One entry per price a catalog published: `bazaar`, `scheme`, `network`, `asset`, `amount` in the asset's base units, `payTo` and `maxTimeoutSeconds`. |
| `distinctAmounts` | integer | Number of different `amount` values across `prices`. |
| `sonda` | object or `null` | The last [live check](concepts.md#the-live-check): `veredicto`, `en`, the HTTP status `estado`, the offer read in `oferta` (`importe`, `activo`, `red`) and the comparison in `cotejo`: `mismoImporte` is `true` when the price matches the catalog, `mismaCartera` when the receiving wallet does, and either is `null` when there was nothing to compare. `sonda` is `null` when the tool has not been checked. |
| `declaredByAgent` | boolean | `true` when an agent in the directory declares this tool's address. The profile names that agent in `agente`. |
| `description` | string or `null` | Description from a catalog, cut to 160 characters in lists. |
| `tipo` | string or `null` | Resource type the catalog declared, for example `http`. |
| `logo` | object or `null` | Two image URLs, `card` and `profile`. |
| `bloque` | integer or `null` | Group the live check puts the tool in: `1` quotes a price, `2` responds without a price (`2xx` or `402`), `3` not checked or not classified, `4` answered with a `4xx` other than `402`, `5` answered with a `5xx`, is down, was blocked or has no such route. |
| `completitud` | integer | 0 to 5, one point for each of: a price offer read by the live check, a description, an input schema or call descriptor, a declared name, a logo. A tool priced only on test networks scores `0`. |

## `GET /api/v1/public/x402/tools/{slug}`

Returns one tool with every field of the list item plus the fields below. A
retired tool still answers, with `retired: true`.

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/tools/abi-cyberwarex-abi"
```

| Field | Type | Description |
| --- | --- | --- |
| `description` | string or `null` | Full description. |
| `method` | string or `null` | HTTP method a catalog declared. `null` when none was declared; do not assume one. |
| `inputSchema`, `outputSchema` | object or `null` | Schemas as the catalog published them. |
| `descriptorDeLlamada` | object or `null` | A call descriptor a catalog published outside the input schema, when there is one. |
| `wallets` | object[] | Every receiving wallet: `address`, `family`, `networks` and `bazaars`. |
| `railMarkers` | string[] | Payment rail markers a catalog published. |
| `fieldSources` | object | Which catalog supplied each field. |
| `payToSources` | object | Which catalog published each receiving wallet. |
| `firstSeenAt`, `lastSeenAt`, `projectedAt` | string | When a catalog first listed the tool, when it last did, and when Deside last rebuilt the entry. |
| `retired`, `retiredAt` | boolean, string or `null` | Whether the tool stopped being seen in its sources, and since when. |
| `agente` | object or `null` | `{ id }` of the agent that declared the tool, when there is one. |
| `indice` | object or `null` | The x402 index that lists the tool: the agents that declare it (`agentes`), the index URLs (`indices`), whether it was served (`servida`) and whether it sits at `/.well-known/x402` (`wellKnown`). |

In the profile, `sonda.oferta` also carries the receiving wallet (`cartera`)
and `sonda.cotejo` carries `redNoDeclarada`, `true` when the offer named a
network the catalog did not publish.

An unknown slug answers `404` with `{ "error": "not_found" }`.

## `GET /api/v1/public/x402/census`

Returns counts over the tool catalog. Retired tools are not counted.

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `host` | string | no | Count only this host. |
| `payTo` | string | no | Count only tools paying this wallet. |
| `network` | string | no | Count only tools with a price on this network. |
| `live` | `1` | no | Count only tools whose last check got an answer. |

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/census?live=1"
```

| Field | Unit | Description |
| --- | --- | --- |
| `tools` | tools | Tools that match. |
| `hosts` | hosts | Distinct hosts among them. |
| `wallets` | wallets | Distinct receiving wallets among them. |
| `bazaars` | catalogs | Distinct catalogs among them. |
| `porBazar` | tools | `{ bazaar, n }` per catalog. A tool in two catalogs counts in both. |
| `porRed` | tools | `{ network, n }` per network string, up to 100 networks. A tool priced on two networks counts in both. |
| `porSonda` | tools | `{ veredicto, n }` per live-check result. Unchecked tools are not counted here. |
| `topHosts` | tools | The 10 hosts with the most tools, `{ host, n }`. |
| `projectedAt` | timestamp | Newest rebuild among the counted tools. |

## `GET /api/v1/public/x402/indices`

Returns x402 indices that list at least one tool. An x402 index is a document
in which an agent lists its own tools, such as `https://host/.well-known/x402`.
Indices on the same host declared by the same agents are merged into one item.
Items are ordered by how many of their tools answer.

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `limit` | integer | no | 1 to 100. | `25` |
| `cursor` | string | no | `pagination.nextCursor` of the previous page. | |

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/indices?limit=1"
```

A real response from 2026-10-01:

```json
{
  "items": [
    {
      "id": "https://agent402.tools/.well-known/x402",
      "host": "agent402.tools",
      "indices": ["https://agent402.tools/.well-known/x402"],
      "agentes": 1,
      "contenido": { "estado": "vivo", "tools": 604, "cobran": 601, "sinCobrar": 1, "fallan": 2, "sinSondar": 0 }
    }
  ],
  "pagination": { "nextCursor": "eyJvIjoxfQ", "hasMore": true, "limit": 1, "total": 92 }
}
```

| Field | Description |
| --- | --- |
| `id` | The first index URL of the item. Pass it as `indice` to `/x402/tools` to list its tools. |
| `indices` | Every index URL merged into the item. |
| `agentes` | Number of agents that declare these indices. |
| `contenido.tools` | Tools listed, counting only tools that are not retired. |
| `contenido.cobran` | Of those, tools that quote a price. |
| `contenido.sinCobrar` | Tools that respond without a price. |
| `contenido.fallan` | Tools that refused the request or are down. |
| `contenido.sinSondar` | Tools not checked yet. |
| `contenido.estado` | `vivo` when at least one tool quotes a price or responds, `muerto` otherwise. |

## `GET /api/v1/public/x402/wallet-edges`

Returns the agents in the directory whose wallet is the receiving wallet of an
x402 tool. Send exactly one of `address` or `catalogId`.

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `address` | string | one of the two | A wallet address. | |
| `catalogId` | string | one of the two | An agent id from the directory. | |
| `limit` | integer | no | 1 to 100. | `25` |
| `cursor` | string | no | `pagination.nextCursor` of the previous page. | |

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/wallet-edges?address=4aet1MhW5gbf46dqzrQB1qxGjM3Q3hN7ndKPRrntW5vg"
```

A real response from 2026-10-01:

```json
{
  "items": [
    {
      "address": "4aet1MhW5gbf46dqzrQB1qxGjM3Q3hN7ndKPRrntW5vg",
      "family": "solana",
      "catalogId": "7a9b0bb1-f09f-4e05-8965-bb3a5e8e27ea",
      "field": "ownerWallet",
      "relation": "same-payout-wallet",
      "claim": "coincide",
      "computedAt": "2026-09-03T04:50:04.253Z"
    }
  ],
  "pagination": { "nextCursor": null, "hasMore": false, "limit": 25, "total": 1 }
}
```

`field` names which wallet of the agent matched: `agentWallet`, `ownerWallet`
or `wallet`. **A match says two records name the same wallet; it does not say
the agent runs the tool.** That is why `claim` is `coincide`.

## Pagination

The list routes return `pagination` with `nextCursor`, `hasMore`, `limit` and
`total`. Pass `nextCursor` back as `cursor` with the same filters. The rules
shared by every route are on [Pagination](../../directory-api/docs/pagination.md).

## Errors

These routes answer errors as `{ "error": "<code>" }`, plus `nextStep` on a
`400`. They do not use the keyed error envelope.

| Status | Body | When | What to do |
| --- | --- | --- | --- |
| `400` | `invalid_request`, `nextStep: unknown filter: <name>` | A parameter this route does not accept. | Remove it. |
| `400` | `invalid_request`, `nextStep: invalid filter: <name>` | A value out of range or of the wrong shape. | Fix the value. |
| `400` | `invalid_request`, `nextStep: q cannot be combined with cursor: search returns a single page` | `q` and `cursor` together. | Drop `cursor`; refine `q` instead. |
| `400` | `invalid_request`, `nextStep: cursor does not match the current filters` | The cursor was issued for other filters. | Restart without `cursor`. |
| `400` | `invalid_request`, `nextStep: cursor exceeds the public depth limit` | The page would go past row 500. | Narrow the filters, or use the [keyed routes](../../directory-api/docs/x402-keyed.md). |
| `404` | `not_found` | Unknown tool slug. | Check the slug. |
| `429` | `RATE_LIMITED` | Per-IP limit reached. | Wait for the minute window, or the day. |
| `500` | `internal_error` | Server failure. | Retry later. |
