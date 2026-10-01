# Ask

`POST /api/v1/ask` answers one question about the directory in plain language,
from the facts Deside has measured. It needs no key and consumes no Directory
API quota.

## Request

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `question` | string | yes | 3 to 500 characters. |

This request asks one question without a session:

```bash
curl -sS -X POST "https://api.deside.io/api/v1/ask" \
  -H "content-type: application/json" \
  -d '{"question":"which x402 tools return weather data?"}'
```

The previous address, `POST /api/v1/directory/agents/ask`, still answers the
same way and will be retired. Use `/api/v1/ask`.

## Lanes and limits

A request with a signed-in Deside browser session uses the human lane. Every
other request uses the headless lane.

| Lane | Limit per IP |
| --- | --- |
| human | 10 requests a minute |
| headless | 3 requests a minute |

## Response

A real response from 2026-10-01 to the request above, cut to one entry per
list. The wording of `answer` changes from one call to the next.

```json
{
  "answer": "Location Intel API returns weather data directly, it's listed among its skills alongside geocoding and map tools. Flight Ops API comes close: ...",
  "agents": [
    {
      "id": "7de5b573-b9d6-4e68-9132-96156777bc6c",
      "name": "Location Intel API",
      "avatarUrl": "https://geo-mcp.thespawn.io/.well-known/avatar.png",
      "category": null,
      "state": "responds",
      "connected": false,
      "lastCheckedAt": "2026-09-25T05:15:00.474Z",
      "services": [
        { "kind": "mcp", "url": "https://geo-mcp.thespawn.io/mcp", "declared": true, "checked": true, "checkedAt": "2026-09-25T05:15:00.474Z", "source": "registry", "version": "2025-06-18" }
      ],
      "verified": false,
      "verifiedCheck": null,
      "verifiedFailed": []
    }
  ],
  "claimed": [
    { "id": "73539352-ea6b-4765-a74a-610baf8da819", "name": "AgentWeather", "category": null, "state": "registered", "lastTriedAt": null }
  ],
  "claimedTotal": 18,
  "unmet": false,
  "intent": "market_data",
  "measuredAt": "2026-10-01T05:25:05.920Z"
}
```

| Field | Description |
| --- | --- |
| `answer` | The answer in plain language, or `null`. |
| `agents` | Agents the answer is based on. |
| `claimed` | Agents that say they match, without a measured live endpoint. An agent in `agents` is not repeated here. |
| `claimedTotal` | How many such agents exist, which can be more than `claimed` lists. |
| `unmet` | `true` when no agent was found for the question. |
| `intent` | The category the question was filed under. |
| `measuredAt` | When the facts behind the answer were measured. |

`verified`, `verifiedCheck` and `verifiedFailed` are reserved, as on
[Trust facts](trust.md#reading-the-verified-fields).

**This route never answers `500`.** When the engine cannot answer, it returns
`200` with `answer: null`, empty lists and `degraded: true`.

## Paid headless lane

The headless lane is the one meant to be paid. When paid access is switched on,
that lane answers `402` with an x402 `accepts` block instead of an answer. It
is not switched on today.

## Errors

| Status | Body | When |
| --- | --- | --- |
| `400` | `{ "error": "INVALID_QUESTION", "nextStep": "question must be a string of 3..500 chars" }` | `question` missing or outside 3 to 500 characters. |
| `404` | `{ "error": "NOT_FOUND" }` | The feature is switched off. The route then looks as if it did not exist. |
| `429` | `{ "error": "rate_limited", "scope", "retryAfterSec" }`, header `Retry-After` | The lane's per-IP limit was reached. |
