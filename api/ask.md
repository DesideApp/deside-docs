# Ask

`POST /ask` answers a question in plain words with what Deside holds: AI agents, x402 tools, and who is behind a token. It answers with a short text and the lists it is built from. Whether an agent or a tool is live comes from our checks, never from the AI: **use `agents`, `tools` and `tokens` to decide, not `answer`.**

Ask in any language. The answer comes in the language of the question.

Ask is free on deside.io and for users signed in with a wallet. Any other caller pays 0.01 USDC per answered question over x402, on Solana. See [Pay with x402](#pay-with-x402). A question that is not a search gets a fixed answer and is not charged. The agent and x402 tool reads are also sold over x402: see [Paid Reads](paid-reads.md).

```bash
curl -X POST "https://api.deside.io/api/v1/ask" \
  -H "content-type: application/json" \
  -d '{"question":"Which x402 tools give the current weather for a city?"}'
```

## Body

| Name | Type | Required | Description |
|---|---|---|---|
| `question` | string | Yes | 3 to 500 characters after trimming. |

## What the question is read as

Ask reads the question first and answers it one of these ways:

| The question asks for | The answer carries |
|---|---|
| Agents | `agents`, `claimed` and `sources`. |
| x402 tools only | `tools`, each with a check made during the answer. `agents` is empty. |
| Agents and x402 tools | `agents`, `tools`, `claimed` and `sources`. |
| Who is behind a token | `tokens`. |
| Nothing to search: a greeting, an opinion, advice, or how Deside works | A fixed `answer` with `noSearch: true`. Not charged. |

## Fields

| Field | Description |
|---|---|
| `answer` | The text answer. `null` when `degraded` is `true`, and when the tools come as a ranked list with no text: then `note` says so. |
| `agents[]` | Up to 5 agents that fit, with `id`, `name`, `avatarUrl`, `category`, `state`, `connected`, `lastCheckedAt` and `services[]`. |
| `agents[].connected` | The owner proved the agent is theirs. |
| `tools[]` | Up to 5 live x402 tools. See [Tools in the answer](#tools-in-the-answer). |
| `tokens[]` | The tokens named in the question. See [Tokens](#tokens). |
| `claimed[]` | Agents whose description matches the question but that are not live in our checks. Up to 100. |
| `claimedTotal` | How many of those there are. |
| `unmet` | `true` when nothing live fits the question. |
| `intent` | The category the question was read as. `other` for tools, tokens and fixed answers. |
| `measuredAt` | When the directory data was measured. `null` on tokens and fixed answers. |
| `sources[]` | On answers about agents: one row per public route the answer used, with `route` (such as `/api/v2/public/agents/stats`), `asOf` (when that data was measured) and `stale` (`true` after 48 hours). |
| `curated[]` | Only when the question names a wallet, an x402 tool URL or a Deside agent URL: what Deside has on it, with `kind` (`wallet`, `tool` or `agent`), `ref`, `found` (`false` when Deside has no record of it) and `asOf`. |
| `noSearch` | `true` on a fixed answer. Not charged. |
| `degraded` | `true` when the answer could not be built. `answer` is then `null` and the lists are empty. Not charged. |

### Agents and tools

Real on 2026-11-22, for "Which agents and x402 tools check token risk on Solana?". `agents` and `services` are cut to one item:

```json
{
  "answer": "Two agents check token risk on Solana: HostDeFi Token Risk API does it exactly, with a free token safety scanner that grades Solana contracts A+ to F using on-chain checks, wallet audits, and trend alerts. SolEnrich comes close: it offers Solana onchain intelligence including wallet risk scoring, token due-diligence, and rug detection, but via a pay-per-call model rather than a free scanner. No live x402 tools in the directory cover this question.",
  "agents": [
    {
      "id": "0d9eb666-12f1-4600-ae7f-459dd70500ed",
      "name": "HostDeFi Token Risk API",
      "avatarUrl": "https://pub-9ddd9cb4402f4d04acd1f55113bf4cea.r2.dev/agent-avatar-cache/deside-main/0d9eb666-12f1-4600-ae7f-459dd70500ed/c1138781e8ac639e-card.webp",
      "category": "token_risk",
      "state": "responds",
      "connected": false,
      "lastCheckedAt": "2026-11-16T14:54:29.565Z",
      "services": [
        {
          "kind": "mcp",
          "url": "https://hostdefi.com/api/v1/mcp",
          "declared": true,
          "checked": true,
          "checkedAt": "2026-10-24T05:15:00.747Z",
          "source": "registry",
          "version": "2025-06-18"
        }
      ]
    }
  ],
  "tools": [],
  "claimed": [],
  "claimedTotal": 0,
  "unmet": false,
  "intent": "token_risk",
  "measuredAt": "2026-11-21T22:47:33.833Z",
  "sources": [
    {
      "route": "/api/v2/public/agents",
      "asOf": "2026-11-21T22:47:33.833Z",
      "stale": false
    },
    {
      "route": "/api/v2/public/agents/stats",
      "asOf": "2026-11-22T01:45:00.646Z",
      "stale": false
    },
    {
      "route": "/api/v1/public/x402/census",
      "asOf": "2026-11-21T06:19:13.072Z",
      "stale": false
    }
  ]
}
```

`tools` is empty here: no live x402 tool fits.

## Tools in the answer

**A tool enters the answer only if it asked for payment on our nightly check**: `probe.verdict` is always `offer`. Each tool has the fields of [x402 Tools](x402-tools.md#get-public-x402-tools), plus:

| Field | Description |
|---|---|
| `key` | The tool's URL. |
| `hostShort`, `tags`, `method`, `retired` | The short host, its tags, its HTTP method when known, and whether it is retired. |
| `probe.daysAgo` | Whole days from our last nightly check to this answer. |
| `liveCheck` | When the question asks only for tools: `answeredNow` (`true` when the tool asked for payment just now, `null` when we did not call it), `latencyMs`, `checkedAt` (when we called it) and `probedAt` (the nightly check). |

How fresh a tool must be:

- **Up to 7 days** since the nightly check: the tool can enter.
- **More than 7 and up to 14 days**: only when the question asks only for tools, and only if the tool asks for payment again when Deside calls it during the answer. That call does not pay, and waits up to 2.5 seconds. A tool that answers without asking for payment is left out.
- **More than 14 days**: never.

When the question asks only for tools, Deside also calls the first tools during the answer, fresh or not, and says in `liveCheck` what each did.

Real on 2026-11-22, for "Which x402 tools give the current weather for a city?". `tools` is cut to one item:

```json
{
  "answer": "Several x402 tools provide current weather for a city. The best option is Weather Forecast, priced at 0.001 USDC on Base, checked just now in 299ms and 0 days ago. It offers current weather and a 3-day forecast for any city or latitude/longitude, including temperature, humidity, wind and conditions.",
  "tools": [
    {
      "slug": "x402-agentindex-weather",
      "key": "https://x402.agentindex.world/weather",
      "scheme": "https",
      "host": "x402.agentindex.world",
      "hostShort": "x402-agentindex",
      "path": "/weather",
      "title": "Weather Forecast",
      "titleSource": "name",
      "tags": [
        "weather forecast",
        "weather"
      ],
      "method": null,
      "prices": [
        {
          "bazaar": "cdp",
          "scheme": "exact",
          "network": "eip155:8453",
          "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
          "amount": "1000",
          "payTo": "0xb3F32bdfe8D07825BC0D7387295aB1D7559BA69d",
          "maxTimeoutSeconds": 300
        }
      ],
      "distinctAmounts": 1,
      "bazaars": [
        "cdp",
        "thirdweb"
      ],
      "bazaarCount": 2,
      "retired": false,
      "probe": {
        "verdict": "offer",
        "at": "2026-11-22T07:20:00.630Z",
        "httpStatus": 402,
        "quote": {
          "amount": "1000",
          "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
          "network": "eip155:8453"
        },
        "comparison": {
          "sameAmount": true,
          "sameWallet": true
        },
        "daysAgo": 0
      },
      "type": "http",
      "description": "Weather forecast API. Current weather and a 3-day forecast f…",
      "hasInputSchema": true,
      "hasOutputSchema": true,
      "identity": null,
      "liveCheck": {
        "answeredNow": true,
        "latencyMs": 299,
        "checkedAt": "2026-11-22T10:31:40.983Z",
        "probedAt": "2026-11-22T07:20:00.630Z"
      }
    }
  ],
  "agents": [],
  "claimed": [],
  "claimedTotal": 0,
  "unmet": false,
  "intent": "other",
  "measuredAt": "2026-11-22T06:19:51.912Z",
  "degraded": false
}
```

## Tokens

Name a Solana token by its mint, its `$TICKER` or its name. Ask says who is behind each token, from what Deside lists: the Deside account behind it, its agents and their x402 tools.

| Field | Description |
|---|---|
| `mint`, `chain`, `name`, `symbol` | The token. `chain` is `solana`. |
| `createdByWallet` | The wallet that created it. |
| `account` | The Deside account behind it: `Team`, `Connected` or `null`. |
| `agents[]` | Up to 5 agents behind it: `catalogId`, `name`, `step` (how far the relation is proven), `via` (how), `x402Tools` (how many of its x402 tools respond), `live`, `checkedAt`, and `checkedDaysAgo` when the check is older than 7 days. |
| `agentsNotShown` | How many more agents there are. |
| `tools[]` | Up to 5 x402 tools behind it: `slug`, `title`, `host`, `priceUsdc`, `probe` (our last check, in words), `probedAt`, `step` (`Matching` or `Verified owner`), `via`, and `checkedDaysAgo` when older than 7 days. |

**An agent is live only if one of its services responded on a check from the last 7 days.** With an older check, `live` is `false` and the answer says "responded when checked 20d ago", with the real number of days.

Real on 2026-11-22, for "Who is behind $DESIDE?":

```json
{
  "answer": "Behind DESIDE: Team account with agent Deside (Connected, Created by this agent's owner, responded when checked 20d ago, 18 of 18 x402_tools respond).",
  "agents": [],
  "claimed": [],
  "claimedTotal": 0,
  "unmet": false,
  "intent": "other",
  "measuredAt": null,
  "degraded": false,
  "tokens": [
    {
      "mint": "Ec9FVEahXUhQRkPneCmDYzXc3jFZWX4URcLfyPwHaRE1",
      "chain": "solana",
      "name": "Deside",
      "symbol": "DESIDE",
      "createdByWallet": "5DZsMz44aH4JotKKcp58Xo6V4WCUMg4bF7CMQ9ySVZJN",
      "account": "Team",
      "agents": [
        {
          "catalogId": "5f0998a5-2bf1-4093-85be-2ba7e114e96f",
          "name": "Deside",
          "step": "Connected",
          "via": "Created by this agent's owner",
          "x402Tools": "18 of 18 respond, checked 20d ago",
          "live": false,
          "checkedAt": "2026-11-02T01:45:00.490Z",
          "checkedDaysAgo": 20
        }
      ],
      "agentsNotShown": 0,
      "tools": []
    }
  ]
}
```

A token Deside does not list gets the fixed answer "That token is not among the tokens Deside lists." with `tokenNotFound: true`. When token search is not available, the answer is "Token search is not available right now." with `tokenSearchUnavailable: true`. Both carry `noSearch: true` and are not charged.

## Fixed answers

A question that is not a search gets this answer, with `noSearch: true`. Real on 2026-11-22, for "hola, ¿qué tal?", with `reading` trimmed:

```json
{
  "answer": "Ask answers with what Deside holds: AI agents, x402 tools and tokens, who is behind them, what they cost and whether they are live. Ask for one of those, for example \"who can summarize a PDF and at what price?\" or \"who is behind the token X?\". This answer is not charged.",
  "agents": [],
  "claimed": [],
  "claimedTotal": 0,
  "unmet": false,
  "intent": "other",
  "measuredAt": null,
  "degraded": false,
  "noSearch": true,
  "reading": { "family": "no_search" }
}
```

`reading` is how Ask read the question.

## Pay with x402

A caller without a Deside session pays for each question with [x402](https://www.x402.org). You only need USDC on Solana: the facilitator pays the network fee, so you need no SOL.

| | |
|---|---|
| Price | 0.01 USDC per answered question (`amount` `"10000"`, 6 decimals) |
| Network | Solana mainnet, `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` |
| Asset | USDC, `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| Pay to | `Bth9CibjQGiLJxm2A6CdN6URKDaefdcQLrnNdkUsd4K` |
| Scheme | `exact`, `x402Version` 2, `maxTimeoutSeconds` 60 |
| Facilitator | [PayAI](https://facilitator.payai.network) |
| Discovery | `GET https://api.deside.io/.well-known/x402`, and the [PayAI catalog](https://facilitator.payai.network/discovery/resources) |

### Flow

1. Send the question with no payment. Deside answers `402` with the offer in the body and, base64-encoded, in the `PAYMENT-REQUIRED` header.
2. Sign the payment with an x402 v2 client for Solana, such as `@x402/core` with `@x402/svm`, and send the same request again with the `PAYMENT-SIGNATURE` header. The v1 header `X-PAYMENT` is also accepted, with the same value.
3. Deside verifies the payment, answers, and settles it only if the answer was built. The `200` carries the answer and the `PAYMENT-RESPONSE` header: base64 JSON with `success`, `transaction`, `network` and `payer`.

You are not charged, and you still get the answer, when it comes back with `degraded: true` or `noSearch: true` (a fixed answer, a token Deside does not list, or token search not available), and **when it takes more than 20 seconds from the moment your payment arrives**. Then the payment is not settled and the `200` carries no `PAYMENT-RESPONSE` header.

One payment pays for one question. Sending the same question with the same payment again returns the saved answer; to ask something else, pay again.

This request gets the offer:

```bash
curl -i -X POST "https://api.deside.io/api/v1/ask" \
  -H "content-type: application/json" \
  -d '{"question":"Which x402 tools give the current weather for a city?"}'
```

Real on 2026-11-22, with `extensions` trimmed:

```json
{
  "x402Version": 2,
  "error": "payment required",
  "resource": {
    "url": "https://api.deside.io/api/v1/ask",
    "description": "Deside directory question: one answered question about AI agents, x402 tools with their live check and price, and the relations Deside curates between wallets, tools and agents",
    "mimeType": "application/json",
    "serviceName": "Deside Ask",
    "tags": ["ai-agents", "x402", "tools", "wallets", "search"],
    "iconUrl": "https://deside.io/icon-512.png"
  },
  "accepts": [{
    "scheme": "exact",
    "network": "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
    "amount": "10000",
    "asset": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "payTo": "Bth9CibjQGiLJxm2A6CdN6URKDaefdcQLrnNdkUsd4K",
    "maxTimeoutSeconds": 60,
    "extra": { "feePayer": "CjNFTjvBhbJJd2B5ePPMHRLx1ELZpa8dwQgGL727eKww" }
  }],
  "extensions": { "bazaar": { "info": { "input": { "type": "http", "method": "POST", "bodyType": "json", "body": { "question": "Which agents and x402 tools audit Solana tokens, and is https://api.tokenaudit.example/v1/scan alive?" } } } } }
}
```

Without a payment the `402` comes first, before the body is checked and before the question is read: `POST {}` and a greeting both get the `402`. `extensions.bazaar` describes the input and output for x402 catalogs, with a JSON Schema; trimmed here.

## Limits

| Caller | Price | Limit |
|---|---|---|
| People on deside.io/ask | Free | 10 questions a minute per IP, and 120 a minute for all visitors together. Over it, the answer is the `402` offer instead of a `429`. |
| Signed in to Deside with a wallet | Free | 10 questions a minute per account |
| Anyone else | 0.01 USDC per answered question | 3 requests a minute per IP, with or without a new payment. A retry of a payment Deside already received has its own limit of 10 a minute per payment. |

A `429` carries `Retry-After` in seconds.

## Errors

| Status | `error` | When | What to do |
|---|---|---|---|
| 400 | `INVALID_QUESTION` | Missing or bad `question`, on a request with a payment or a session. | Send 3 to 500 characters. `nextStep` says the rule. |
| 402 | `payment required` | No payment. | Pay and retry. |
| 402 | `invalid_payment`, `requirements_mismatch` or the facilitator's reason | The payment is not valid or does not match the offer. | Sign a new payment for the offer in this `402`. |
| 409 | `PAYMENT_ALREADY_USED` | That payment already paid for a different question. | Pay again. |
| 409 | `PAYMENT_IN_PROGRESS` | The same payment is being processed in another request. | Wait and retry with the same payment. |
| 429 | `rate_limited` | Over the limit. | Wait `retryAfterSec`. |
| 503 | `FACILITATOR_UNAVAILABLE` | The facilitator does not answer. You are not charged. | Retry later. |
| 503 | `SETTLEMENT_PENDING` | The answer was built but the payment did not settle. | Retry with the same payment. Deside settles it and returns the saved answer. Do not pay again. |
| 503 | `ASK_PAYMENTS_UNAVAILABLE` | Payments are down. You are not charged. | Retry later. |
