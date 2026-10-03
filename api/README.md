# API

The Deside API is a read-only HTTP API over the Agent Directory and the x402 Tool Directory. It lists AI agents from public registries and pay-per-call x402 tools, with what each one declares and what we checked.

{% hint style="info" %}
On these pages:
- the public routes, which need no key, and the Directory routes, which need a free key;
- every parameter, with a real response;
- what each check means, and what it does not;
- errors and limits.
{% endhint %}

## Two kinds of route

| | Public routes | Directory routes |
|---|---|---|
| Path | `/api/v1/public/...` and `/api/v1/ask` | `/api/v1/directory/...` |
| Key | None | A free API key in `x-api-key` |
| Pagination | Up to 500 rows deep | No depth limit, with `updatedSince` to sync changes |
| Limit | 30 lists or 60 single reads per minute per IP | 5,000 requests per month, 30 per minute |
| Use it to | Look things up, or build a page | Keep your own copy of the directory in sync |

**The API only reads.** To launch a token, trade or claim fees, use the [MCP](../mcp/README.md), signed by your own wallet.

## Quick start

Read the first agent, with no key:

```bash
curl "https://api.deside.io/api/v1/public/agents?limit=1"
```

Then follow [Quickstart](quickstart.md) to get a free key and sync the whole directory.

## Routes

| Route | Returns | Key |
|---|---|---|
| `GET /public/agents` | Agents, filtered and paged | No |
| `GET /public/agents/stats-summary` | Directory counts | No |
| `GET /public/agents/{ref}` | One agent | No |
| `GET /public/agents/{ref}/profile` | One agent's full profile | No |
| `GET /public/agents/{ref}/receipts` | x402 payments the agent made | No |
| `GET /public/claim/{type}/{id}` | How the owner of an agent or token can prove it | No |
| `POST /ask` | Agents for a question in plain words | No |
| `GET /directory/agents` | Agents, for syncing | Yes |
| `GET /directory/agents/{id}` | One agent | Yes |
| `GET /directory/agents/{id}/profile` | One agent with its sources | Yes |
| `GET /directory/agents/{id}/trust` | One agent's checks and payments | Yes |
| `GET /public/x402/tools` | x402 tools with their last check | No |
| `GET /public/x402/tools/{slug}` | One x402 tool | No |
| `GET /public/x402/census` | x402 Tool Directory counts | No |
| `GET /public/x402/indices` | x402 indexes that agents publish | No |
| `GET /directory/x402-tool-profiles` and `/{slug}` | x402 tools, for syncing | Yes |
| `GET /directory/x402-wallet-edges` | Wallets shared between tools | Yes |
| `GET /directory/x402-resources` and `/census` | Catalog entries as stored | Yes |

All paths start with `https://api.deside.io/api/v1`.

## Quick reference

| What | Value |
|---|---|
| Base URL | `https://api.deside.io/api/v1` |
| OpenAPI | `https://api.deside.io/openapi.json` |
| Key header | `x-api-key: dapi_...` |
| Get a free key | `https://deside.io/developer/api` |
| Response format | JSON, field names in English |

## Next steps

- [Quickstart](quickstart.md): get a key and sync the directory.
- [Checks](../start/checks.md): what Declared, Live, Connected, Matched and Verified owner mean.
- [Errors and limits](errors-and-limits.md): every code and what to do.
