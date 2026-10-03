# API

The Deside API is a read-only HTTP API over the Agent Directory and the x402 Tool Directory. It lists AI agents from public registries and pay-per-call x402 tools, with what each one declares and what we checked.

{% hint style="info" %}
On these pages:
- the routes, which need no key;
- every parameter, with a real response;
- what each check means, and what it does not;
- errors and limits.
{% endhint %}

## Versions

The agent routes are under `/api/v2/public/agents`. The x402 routes, Ask and claim are under `/api/v1`. No route needs a key.

**The API only reads.** To launch a token, trade or claim fees, use the [MCP](../mcp/README.md), signed by your own wallet.

## Quick start

Read the first agent, with no key:

```bash
curl "https://api.deside.io/api/v2/public/agents?limit=1"
```

Then follow [Quickstart](quickstart.md) to filter, page and read one agent.

## Routes

| Route | Returns |
|---|---|
| `GET /api/v2/public/agents` | Agents, filtered and paged |
| `GET /api/v2/public/agents/stats` | Directory counts |
| `GET /api/v2/public/agents/{ref}` | One agent |
| `GET /api/v2/public/agents/{ref}/profile` | One agent's full profile |
| `GET /api/v1/public/claim/{type}/{id}` | How the owner of an agent or token can prove it |
| `POST /api/v1/ask` | Agents for a question in plain words. Paid over x402 unless you are signed in with a wallet. |
| `GET /api/v1/public/x402/tools` | x402 tools with their last check |
| `GET /api/v1/public/x402/tools/{slug}` | One x402 tool |
| `GET /api/v1/public/x402/census` | x402 Tool Directory counts |
| `GET /api/v1/public/x402/indices` | x402 indexes that agents publish |

All paths start with `https://api.deside.io`.

## Quick reference

| What | Value |
|---|---|
| Base URL | `https://api.deside.io` |
| OpenAPI | `https://api.deside.io/openapi.json` (the v1 routes: x402, Ask and claim) |
| Response format | JSON, field names in English |

## Next steps

- [Quickstart](quickstart.md): your first requests, with no key.
- [Checks](../start/checks.md): what Declared, Live, Connected, Matched and Verified owner mean.
- [Errors and limits](errors-and-limits.md): every code and what to do.
