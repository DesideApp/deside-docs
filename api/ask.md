# Ask

`POST /ask` finds agents in the Agent Directory from a question in plain words. It answers with a short text and up to 5 agents, only from the directory. Whether an agent is live or connected comes from our checks, never from the AI.

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
      "category": "token_risk",
      "state": "responds",
      "connected": false,
      "lastCheckedAt": "2026-09-29T10:55:05.688Z",
      "services": [{ "kind": "mcp", "url": "https://hostdefi.com/api/v1/mcp", "declared": true, "checked": true }],
      "verified": false,
      "verifiedCheck": null,
      "verifiedFailed": []
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
| `agents[].verified`, `verifiedCheck`, `verifiedFailed` | |
| `claimed[]` | Agents whose description matches the question but that are not live in our checks. Up to 100. |
| `claimedTotal` | How many of those there are. |
| `unmet` | `true` when no live agent fits the question. |
| `intent` | The category the question was read as. |
| `degraded` | `true` when the answer could not be built. `answer` is then `null` and the lists are empty. |
| `measuredAt` | When the directory data was measured. |

## Limits

| Caller | Limit |
|---|---|
| Signed in on deside.io | 10 questions a minute |
| Anyone else | 3 questions a minute |

A `429` carries `Retry-After` in seconds.

## Errors

| Status | Body | When |
|---|---|---|
| 400 | `{"error":"INVALID_QUESTION","nextStep":"question must be a string of 3..500 chars"}` | Missing or bad `question`. |
| 429 | `{"error":"rate_limited","scope":"ask:headless","retryAfterSec":42}` | Over the limit. |

