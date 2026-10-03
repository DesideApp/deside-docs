# x402 Data With An API Key

These routes serve the x402 tool catalog to Directory API keys. They need
`x-api-key: dapi_...` and count against your quota like every keyed route
([Rate limits](errors-and-limits.md)). Unlike the
[x402 Tools API](x402-tools.md), their cursors have no depth limit, so
they can walk the whole catalog.

The live check, the field meanings and the public examples are on
[x402 Tools API](x402-tools.md). This page lists only what differs.

{% hint style="warning" %}
A missing or invalid key answers with the keyed error envelope (see
[Errors](errors-and-limits.md)). Once the key is accepted, a bad parameter answers like
the public routes: `{ "error": "invalid_request", "nextStep": "..." }`.
{% endhint %}

## `GET /api/v1/directory/x402-tool-profiles`

Returns a page of tools, ordered by `slug`. Retired tools are not listed.

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `host` | string | no | Exact host. | |
| `bazaar` | string | no | Catalog id, for example `cdp`. | |
| `payTo` | string | no | Exact receiving wallet address. | |
| `limit` | integer | no | 1 to 100. | `25` |
| `cursor` | string | no | `pagination.nextCursor` of the previous page. | |

Any other parameter answers `400` with `invalid filter: <name>`. This route
does not accept `q`, `network`, `index` or `live`; use the public route for
those.

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/x402-tool-profiles?bazaar=cdp&limit=100"
```

Each item carries `slug`, `key`, `scheme`, `host`, `hostShort`, `path`,
`title`, `titleSource`, `tags`, `method`, `prices`, `distinctAmounts`,
`bazaars`, `bazaarCount`, `retired`, `probe`, `type`, `description` (cut to
160 characters), `hasInputSchema` and `hasOutputSchema`. Compared with the
public list item, it adds `key` (the tool's full URL), `hostShort`, `tags`,
`method` and `retired`, and it does not carry `walletCount`, `firstWallet`,
`declaredByAgent`, `logo`, `tier` or `completeness`.

## `GET /api/v1/directory/x402-tool-profiles/{slug}`

Returns one tool. A retired tool still answers. The profile carries the list
fields except `hasInputSchema` and `hasOutputSchema`, plus the full `description`, `inputSchema`, `outputSchema`,
`callDescriptor`, `wallets`, `railMarkers`, `fieldSources`,
`payToSources`, `firstSeenAt`, `lastSeenAt`, `projectedAt`, `passRunId` and
`walletCoincidences`.

`walletCoincidences` lists the agents in the directory whose wallet is one of
the tool's receiving wallets. Each entry has `slug`, `name`, `catalogId`,
`address`, `field`, `relation` (`same-payout-wallet`), `claim` (`matches`)
and `computedAt`. **A coincidence says two records name the same wallet; it
does not say the agent runs the tool.**

An unknown slug answers `404` with `{ "error": "not_found" }`.

## `GET /api/v1/directory/x402-wallet-edges`

Same parameters, response and rules as the public `/public/x402/wallet-edges`, without the depth limit.

## `GET /api/v1/directory/x402-resources`

Returns the catalog entries as Deside stored them from the catalogs, before
they are merged into tools: one item per URL, with every catalog's copy in
`sources`. Use it when you need what a catalog published, not Deside's merged
view. Ordered by `key`.

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `key` | string | no | Exact URL of the resource. | |
| `host` | string | no | Exact host. | |
| `scheme` | string | no | URL scheme, for example `https`. | |
| `bazaar` | string | no | Catalog id. | |
| `bazaarCount` | integer | no | Number of catalogs listing the resource, 1 to 5. | |
| `payTo` | string | no | Exact receiving wallet address. | |
| `limit` | integer | no | 1 to 100. | `25` |
| `cursor` | string | no | `pagination.nextCursor` of the previous page. | |

Each item carries `key`, `scheme`, `host`, `resource`, `type`, `method`,
`description`, `serviceName`, `toolName`, `tags`, `inputSchema`,
`outputSchema`, `bazaarCount`, `payTo`, `payToSolana`, `payToRail`,
`payToSources`, `fieldSources`, `discoveredVia` (`{ bazaar, at }`),
`firstSeenAt`, `lastSeenAt`, `sources` (`{ bazaar, fetchedAt,
declaredLastUpdated }` per catalog), `precios` and `preciosDistintos`.
`precios` has one `{ bazaar, amount, asset, network }` per price a catalog
published, as published: amounts are not converted between catalogs.

## `GET /api/v1/directory/x402-resources/census`

Returns counts over the stored catalog entries. It takes no parameters.

| Field | Unit | Description |
| --- | --- | --- |
| `total` | entries | Stored entries. |
| `conNombre` | entries | Entries with a service or tool name. |
| `conDescripcion` | entries | Entries with a description. |
| `hosts` | hosts | Distinct hosts. |
| `carteras` | wallets | Distinct receiving wallets. |
| `soloTestnet` | entries | Entries whose every price is on a test network. |
| `conRedReal` | entries | Entries with at least one price on a network that is not a test network. |
| `porEsquema` | entries | `{ scheme, n }` per URL scheme. |
| `porNumeroDeBazares` | entries | `{ bazaarCount, n }`. |
| `porBazar` | entries | `{ bazaar, n }` per catalog. An entry in two catalogs counts in both. |

| `ultimaEscritura` | timestamp | Newest `lastSeenAt`. |

The test networks are `base-sepolia`, `eip155:84532`, `eip155:80002`,
`sei-testnet`, `solana-devnet`, `stellar:testnet` and `xlayer-testnet`. An
entry with no network on any price is in neither `soloTestnet` nor
`conRedReal`.

## Errors

| Status | Body | When |
| --- | --- | --- |
| `401`, `403`, `429` | keyed error envelope | Key, origin, rate or quota problem. See [Errors](errors-and-limits.md). |
| `400` | `{ "error": "invalid_request", "nextStep": "..." }` | Unknown parameter, bad value, or a cursor issued for other filters. |
| `404` | `{ "error": "not_found" }` | Unknown tool slug. Not counted against your quota. |
| `500` | `{ "error": "internal_error" }` | Server failure. |
