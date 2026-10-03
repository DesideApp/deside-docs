# Ask

`POST /ask` finds agents in the Agent Directory from a question in plain words. It answers with a short text and up to 5 agents, only from the directory. Whether an agent is live or connected comes from our checks, never from the AI.

Ask is free on deside.io and for users signed in with a wallet. Any other caller pays 0.01 USDC per answered question over x402, on Solana. See [Pay with x402](#pay-with-x402).

```bash
curl -X POST "https://api.deside.io/api/v1/ask" \
  -H "content-type: application/json" \
  -d '{"question":"an agent that checks token risk on Solana"}'
```

## Body

| Name | Type | Required | Description |
|---|---|---|---|
| `question` | string | Yes | 3 to 500 characters after trimming. |

## Response

```json
{
  "answer": "Several agents cover token risk checks on Solana. HostDeFi Token Risk API grades Solana tokens A+ to F...",
  "agents": [
    {
      "id": "0d9eb666-12f1-4600-ae7f-459dd70500ed",
      "name": "HostDeFi Token Risk API",
      "avatarUrl": "https://example.com/avatar.png",
      "category": "token_risk",
      "state": "responds",
      "connected": false,
      "lastCheckedAt": "2026-09-29T10:55:05.688Z",
      "services": [{ "kind": "mcp", "url": "https://hostdefi.com/api/v1/mcp", "declared": true, "checked": true }]
    }
  ],
  "claimed": [{ "id": "34c3add6-7ee3-4836-b88e-3a271bf3648e", "name": "Soliris Sentinel", "category": "token_risk", "state": "profile", "lastTriedAt": null }],
  "claimedTotal": 1850,
  "unmet": false,
  "intent": "token_risk",
  "measuredAt": "2026-10-02T22:50:06.620Z"
}
```

| Field | Description |
|---|---|
| `answer` | The text answer. |
| `agents[]` | Agents that fit, with `id`, `name`, `avatarUrl`, `category`, `state`, `connected`, `lastCheckedAt` and `services[]`. |
| `agents[].connected` | The owner proved the agent is theirs. |
| `claimed[]` | Agents whose description matches the question but that are not live in our checks. Up to 100. |
| `claimedTotal` | How many of those there are. |
| `unmet` | `true` when no live agent fits the question. |
| `intent` | The category the question was read as. |
| `degraded` | `true` when the answer could not be built. `answer` is then `null` and the lists are empty. |
| `measuredAt` | When the directory data was measured. |

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
| Discovery | `GET https://api.deside.io/.well-known/x402` |

### Flow

1. Send the question with no payment. Deside answers `402` with the offer in the body and, base64-encoded, in the `PAYMENT-REQUIRED` header.
2. Sign the payment with an x402 v2 client for Solana, such as `@x402/core` with `@x402/svm`, and send the same request again with the `PAYMENT-SIGNATURE` header. The v1 header `X-PAYMENT` is also accepted, with the same value.
3. Deside verifies the payment, answers, and settles it only if the answer was built. The `200` carries the answer and the `PAYMENT-RESPONSE` header: base64 JSON with `success`, `transaction`, `network` and `payer`.

If the answer comes back with `degraded: true`, the payment is not settled and you are not charged.

One payment pays for one question. Sending the same question with the same payment again returns the saved answer; to ask something else, pay again.

This request gets the offer:

```bash
curl -i -X POST "https://api.deside.io/api/v1/ask" \
  -H "content-type: application/json" \
  -d '{"question":"an agent that checks token risk on Solana"}'
```

```json
{
  "x402Version": 2,
  "error": "payment required",
  "resource": {
    "url": "https://api.deside.io/api/v1/ask",
    "description": "Deside directory question: one answered question about agents and x402 tools",
    "mimeType": "application/json"
  },
  "accepts": [{
    "scheme": "exact",
    "network": "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
    "amount": "10000",
    "asset": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "payTo": "Bth9CibjQGiLJxm2A6CdN6URKDaefdcQLrnNdkUsd4K",
    "maxTimeoutSeconds": 60,
    "extra": { "feePayer": "CjNFTjvBhbJJd2B5ePPMHRLx1ELZpa8dwQgGL727eKww" }
  }]
}
```

Without a payment the `402` comes first, before the body is checked.

## Limits

| Caller | Price | Limit |
|---|---|---|
| People on deside.io/ask | Free | 10 questions a minute per IP, and 120 a minute for all visitors together. Over it, the answer is the `402` offer instead of a `429`. |
| Signed in to Deside with a wallet, also over the MCP | Free | 10 questions a minute per account |
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
