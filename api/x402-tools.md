# x402 tools

These routes read the x402 Tool Directory with no key. An **x402 tool** is an HTTP API you pay per call, with no account. The directory reads the 5 public catalogs (PayAI, Coinbase CDP, Dexter, thirdweb and OpenFacilitator) and the x402 indexes that agents publish. **One URL is one tool**, even when several catalogs list it.

## GET /public/x402/tools

Returns tools with their last check.

```bash
curl "https://api.deside.io/api/v1/public/x402/tools?limit=1&network=eip155:8453"
```

| Name | Type | Default | Description |
|---|---|---|---|
| `q` | string | | Text search, up to 512 characters. Returns one page; it cannot be combined with `cursor`. |
| `host` | string | | Exact host. |
| `bazaar` | string | | Exact catalog: `payai`, `cdp`, `dexter`, `thirdweb` or `openfac` (OpenFacilitator). |
| `payTo` | string | | Exact receiving wallet. |
| `network` | string | | Exact network, such as `eip155:8453` or `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`. |
| `agent` | string | | An agent's `catalogId` or slug: its tools. The response adds an `agent` key. |
| `index` | string | | An index URL, up to 2,048 characters: the tools it lists. Cannot be combined with `agent`. |
| `indice` | string | | Same as `index`. <!-- REVISAR(borrar): alias en español; la web aun lo envia (features/x402/toolsApi.js:25). --> |
| `live` | string | | Only `1`: tools whose last check got an answer. |
| `limit` | integer | 25 | 1 to 100. Outside that: `400`. |
| `cursor` | string | | `pagination.nextCursor`. It only works with the same filters. |

**Any other parameter returns `400 unknown filter`.** Without `q`, a page cannot go past row 500: `400 cursor exceeds the public depth limit`.


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
      "type": "http",
      "description": "Fetch a verified contract's ABI and metadata by address, keylessly, from Sourcify...",
      "bazaars": ["cdp"],
      "bazaarCount": 1,
      "hasInputSchema": true,
      "hasOutputSchema": true,
      "walletCount": 1,
      "firstWallet": { "address": "0x058D0Cc5CC97e61e8A9f38D6d6365bce525921B2", "family": "evm" },
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
        "quote": { "amount": "3000", "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913", "network": "eip155:8453" },
        "comparison": { "sameAmount": true, "sameWallet": true }
      },
      "declaredByAgent": false,
      "logo": { "card": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/x402-tool-icon/...-card.webp", "profile": "..." },
      "tier": 1,
      "completeness": 5
    }
  ],
  "pagination": { "nextCursor": "eyJvIjoyLCJmIjoiYmYyMWE5ZThmYmM1YTM4NCJ9", "hasMore": true, "limit": 1, "total": 35309 }
}
```

| Field | Description |
|---|---|
| `slug` | Stable id of the tool. |
| `title`, `description`, `type` | What the tool is. `type` is how it is called, such as `http`. |
| `titleSource` | Whether `title` came from the tool's name or its path. |
| `host`, `path`, `scheme` | Where it lives. |
| `bazaars[]` | The catalogs that list it. |
| `bazaarCount` | <!-- REVISAR(borrar): longitud de bazaars. --> |
| `hasInputSchema`, `hasOutputSchema` | Whether the tool publishes its input and output format. |
| `walletCount`, `firstWallet` | How many wallets it collects to, and the first one. |
| `prices[]` | The price each catalog publishes: `bazaar`, `scheme`, `network`, `asset`, `amount` in the asset's smallest unit, `payTo`, `maxTimeoutSeconds`. |
| `distinctAmounts` | <!-- REVISAR(borrar): se deriva de prices. --> |
| `probe` | Our last call. See below. |
| `declaredByAgent` | `true` when an agent in the Agent Directory lists the tool as its own. |
| `logo` | Our cached icon, two sizes. |
| `tier`, `completeness` | <!-- REVISAR(borrar): son las claves de orden internas (bloque, completitud). La web lee tier (ToolCard.jsx:75, X402ToolPage.jsx:152): que derive el estado de probe.verdict antes. --> |

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
| `retired`, `retiredAt` | `true` when no catalog lists it any more. |
| `agent` | The agent that lists it, or `null`. |
| `index` | The indexes that list it: `agents`, `indexes`, `served`, `wellKnown`. <!-- REVISAR(modificar): served sin significado publico. --> |
| `fieldSources` | <!-- REVISAR(borrar): que catalogo gano cada campo en NUESTRA fusion; ademas revela tags y serviceName, que no se sirven. --> |
| `payToSources` | <!-- REVISAR(borrar): duplica wallets[].bazaars. --> |
| `railMarkers[]` | <!-- REVISAR(modificar): fontaneria de fusion; definir o quitar. --> |
| `projectedAt` | <!-- REVISAR(borrar): marca de tiempo de nuestra canalizacion. --> |

`404 {"error":"not_found"}` when no tool has that slug.

## GET /public/x402/census

Returns the directory counts. Accepts `host`, `payTo`, `network` and `live=1`.

```json
{
  "tools": 35309,
  "hosts": 2990,
  "wallets": 2530,
  "bazaars": 5,
  "byBazaar": [{ "bazaar": "cdp", "n": 22012 }, { "bazaar": "payai", "n": 11224 }],
  "topHosts": [{ "host": "market.datapackvibe.com", "n": 1400 }],
  "byNetwork": [{ "network": "eip155:8453", "n": 26865 }],
  "byVerdict": [{ "verdict": "offer", "n": 22965 }, { "verdict": "no-endpoint", "n": 5819 }],
  "projectedAt": "2026-10-02T06:21:25.359Z"
}
```

`byBazaar` sums more than `tools` because one tool can be in several catalogs.

<!-- REVISAR(borrar): topHosts es un ranking de concentracion por host (un host con 1.400 tools es un operador); misma logica que D6. Decision del owner, recomendado quitar. -->

## GET /public/x402/indices

Returns the x402 indexes that agents publish, with what each lists.

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 25 | 1 to 100. |
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
  "pagination": { "nextCursor": "eyJvIjoyfQ", "hasMore": true, "limit": 2, "total": 92 }
}
```

| Field | Description |
|---|---|
| `id`, `host`, `indexes[]` | The index. |
| `agents` | How many agents declare it. |
| `content.state` | `live`, `down` or `no-tools`. |
| `content.tools` | Tools it lists, then how many quote a price, respond with no price, are down, or are not checked yet. |

<!-- REVISAR(modificar): existen tambien /public/x402/wallet-edges y /public/x402/wallet-revenue, publicas y sin documentar. -->

## Errors

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"invalid_request","nextStep":"unknown filter: foo"}` | A bad parameter. `nextStep` says which. |
| 404 | `{"error":"not_found"}` | No such tool. |
| 429 | `{"error":"RATE_LIMITED"}` | Over the limit for your IP. |
| 500 | `{"error":"internal_error"}` | Retry later. |
