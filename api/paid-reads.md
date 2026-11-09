# Paid reads

**Paid reads** are the public reads of agents and x402 tools, sold per call over [x402](https://www.x402.org). Each one returns the same data as its free route, in [the common contract](contract.md), for 0.005 USDC on Solana. You need no key and no account.

Pay when the free routes stop you: a paid read has no depth limit on the lists and does not count against the per-route limits of the free routes. It returns the same data, with the same [read code](contract.md#read-codes), as the free API and the MCP.

For Deside's own view on top of the declared data, such as whether to pay a tool or who is behind a wallet, see [Paid Lookups](paid-lookups.md), at 0.01 USDC.

## Routes

Every route is a `GET` under `https://api.deside.io/api/v2/paid`. Each URL is fixed: what the free route takes in its path goes in the query.

| Route | Free route with the same data | Query |
|---|---|---|
| `/agents` | [`GET /api/v2/public/agents`](public-agents.md#get) | The filters of the free route, `limit` and `cursor` |
| `/agent` | [`GET /api/v2/public/agents/{ref}/profile`](public-agents.md#get-ref-profile) | `ref` (required) |
| `/agent/holdings` | [`GET /api/v2/public/agents/{ref}/holdings`](public-agents.md#get-ref-holdings) | `ref` (required) |
| `/agent/discovery` | [`GET /api/v2/public/agents/{ref}/discovery`](public-agents.md#get-ref-discovery) | `ref` (required) |
| `/agents/stats` | [`GET /api/v2/public/agents/stats`](public-agents.md#get-stats) | None |
| `/domain/discovery` | [`GET /api/v2/public/domains/{host}/discovery`](domains.md#get-host-discovery) | `host` (required) |
| `/domain/agents` | [`GET /api/v2/public/domains/{host}/agents`](domains.md#get-host-agents) | `host` (required), `limit` and `cursor` |
| `/x402/tools` | [`GET /api/v1/public/x402/tools`](x402-tools.md#get-public-x402-tools) | The filters of the free route, `limit` and `cursor` |
| `/x402/tool` | [`GET /api/v1/public/x402/tools/{slug}`](x402-tools.md#get-public-x402-tools-slug) | `slug` (required) |
| `/x402/probes` | [`GET /api/v1/public/x402/probes`](x402-tools.md#get-public-x402-probes) | One of `tool`, `url` or `agent` |
| `/x402/census` | [`GET /api/v1/public/x402/census`](x402-tools.md#get-public-x402-census) | `host`, `payTo`, `network` and `live` |
| `/x402/indices` | [`GET /api/v1/public/x402/indices`](x402-tools.md#get-public-x402-indices) | `limit` and `cursor` |

The parameters, their values and the response are those of the free route in the common contract. So are its errors: a parameter the free route does not take answers `400 invalid_request`.

- **The response is always the common contract**, with `?minor=1.1` or without it.
- **A page holds up to 100 items** (50 on `/domain/agents`), and each page is one paid call. Follow `page.nextCursor` with the same filters.
- **The lists have no depth limit.** `/agents` and `/x402/tools` go past row 500, where the free routes stop.
- **A read that carries `about` on its free route carries it here too**, with the same `read` and `definitions`, plus `about.paidAccess`: one sentence on what the paid read changes. See [Read codes](contract.md#read-codes).

## Price and payment

| | |
|---|---|
| Price | 0.005 USDC per call (`amount` `"5000"`, 6 decimals) |
| Network | Solana mainnet, `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` |
| Asset | USDC, `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| Pay to | `Bth9CibjQGiLJxm2A6CdN6URKDaefdcQLrnNdkUsd4K` |
| Scheme | `exact`, `x402Version` 2, `maxTimeoutSeconds` 60 |
| Facilitator | [PayAI](https://facilitator.payai.network) |
| Discovery | `GET https://api.deside.io/.well-known/x402` lists every paid route with its price and its input |

You only need USDC on Solana: the facilitator pays the network fee, so you need no SOL.

### Flow

1. Call the route with its query and no payment. Deside answers `402` with the offer in the body and, base64-encoded, in the `PAYMENT-REQUIRED` header.
2. Sign the payment with an x402 v2 client for Solana, such as `@x402/core` with `@x402/svm`, and send the same request again with the `PAYMENT-SIGNATURE` header. The v1 header `X-PAYMENT` is also accepted, with the same value.
3. Deside verifies the payment and reads. **The payment is settled only if the read answers `200`.** The `200` carries the data and the `PAYMENT-RESPONSE` header: base64 JSON with `success`, `transaction`, `network` and `payer`.

The `402`, the paid `200` and every answer of the read carry `Cache-Control: private, no-store`: no shared cache keeps them.

This request gets the offer for one x402 tool:

```bash
curl -i "https://api.deside.io/api/v2/paid/x402/tool?slug=api-deside-v1-ask"
```

Real on 2026-11-06, with `extensions.bazaar.schema` cut:

```json
{
  "x402Version": 2,
  "error": "payment required",
  "resource": {
    "url": "https://api.deside.io/api/v2/paid/x402/tool",
    "description": "One tool from the Deside x402 tool directory by slug: what it charges, where it is served, its last probe and its declared usage. Same data as GET /api/v1/public/x402/tools/{slug}. 0.005 USDC per call, paid with x402 on Solana; charged only when the read succeeds.",
    "mimeType": "application/json",
    "serviceName": "Deside Directory",
    "tags": ["ai-agents", "x402", "directory", "solana"],
    "iconUrl": "https://deside.io/icon-512.png"
  },
  "accepts": [{
    "scheme": "exact",
    "network": "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
    "amount": "5000",
    "asset": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "payTo": "Bth9CibjQGiLJxm2A6CdN6URKDaefdcQLrnNdkUsd4K",
    "maxTimeoutSeconds": 60,
    "extra": { "feePayer": "CjNFTjvBhbJJd2B5ePPMHRLx1ELZpa8dwQgGL727eKww" }
  }],
  "extensions": { "bazaar": { "info": { "input": { "type": "http", "method": "GET", "queryParams": { "slug": "example-tool" } }, "output": { "type": "json", "example": { "data": {}, "asOf": "2026-11-06T00:00:00.000Z" } } } } }
}
```

Without a payment the `402` comes first, before the query is checked. `extensions.bazaar` describes the input and output for x402 catalogs; its `schema` lists every query parameter the route takes.

## What you pay for

**One payment pays for one request: the route with its query.** The order of the parameters does not matter, and `minor` and `cb` are not part of it.

- **Only a `200` is charged.** Any other answer of the read, such as `400`, `404` or `503`, comes back as it is and the payment is not settled. You can retry the same request with the same payment. Deside verifies it again, so it has to be still valid; if it is not, you get a new `402` and sign again.
- **A lost answer is read again for free.** Within 60 seconds of the payment settling, the same request with the same payment reads again and is not charged again. Deside does not keep the answer: you get the data as it is at that moment.
- **After those 60 seconds, the payment is spent.** The same payment answers `409 PAYMENT_ALREADY_USED`.
- **A payment does not move to another request.** The same payment on another route or with another query answers `409 PAYMENT_ALREADY_USED`. A payment for a paid read does not pay for [Ask](ask.md), which has its own price.

## Limits

| What | Limit |
|---|---|
| Answers with status 400 or above (the `402` offer, a bad payment, a `400`, a `409`, a `503`) | 60 a minute per IP. A paid `200` does not count. |
| Paid reads and [paid lookups](paid-lookups.md) at once | 1 at a time per payer, and 2 at a time per Deside server for all payers together. Over it: `503 PAID_READS_BUSY` with `Retry-After: 1`, and you are not charged. |
| All of `/api` | 500 requests per 15 minutes per IP. Paid reads count. See [Errors and limits](errors-and-limits.md). |

The per-route limits of the free routes, 30 or 60 a minute and 500 a day per IP, do not apply to paid reads.

## Errors

The errors of the read itself are those of the free route, in the common contract: `{ "error": { "code", "message" } }`. They are not charged. These come from the payment:

| Status | `error` | When | What to do |
|---|---|---|---|
| 400 | `{ "code": "invalid_request" }` | A payment came with a required parameter missing, or a parameter sent twice. | Fix the query. The payment was not used. |
| 402 | `payment required` | No payment. | Pay and retry. |
| 402 | `invalid_payment`, `requirements_mismatch` or the facilitator's reason | The payment is not valid or does not match the offer. | Sign a new payment for the offer in this `402`. |
| 409 | `PAYMENT_ALREADY_USED` | That payment already paid for another request, or for this one more than 60 seconds ago. | Pay again. |
| 409 | `PAYMENT_IN_PROGRESS` | The same payment is being processed in another request. | Wait and retry with the same payment. |
| 429 | `rate_limited` | More than 60 answers with status 400 or above in a minute from your IP. | Wait for the seconds in `RateLimit-Reset`. |
| 503 | `PAID_READS_BUSY` | Too many paid reads at once. You are not charged. | Retry after `Retry-After` with the same payment. |
| 503 | `FACILITATOR_UNAVAILABLE` | The facilitator does not answer. You are not charged. | Retry later. |
| 503 | `SETTLEMENT_PENDING` | The read answered but the payment did not settle. | Retry with the same payment. Do not pay again. |
| 503 | `PAID_READ_RETRY` | The payment settled but the read could not be delivered. | Retry with the same payment within 60 seconds of the payment. Do not pay again. |
| 503 | `PAID_READS_UNAVAILABLE` | Payments are down. You are not charged. | Retry later. |

## In the MCP

The read tools of the [MCP](../mcp/README.md) are not paid. They keep their own depth and limits.
