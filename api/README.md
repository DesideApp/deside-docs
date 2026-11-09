# API

The Deside API is a read-only HTTP API over the Agent Directory and the x402 Tool Directory. It lists AI agents from public registries and pay-per-call x402 tools, with what each one declares and what we checked.

{% hint style="info" %}
On these pages:
- the routes, which need no key;
- every parameter, with a real response;
- what each check means, and what it does not;
- errors and limits.
{% endhint %}

## Ways to read

The same reads reach you through several doors. The data and what it means are the same through each one; only the access and the price change.

| Door | What you get | Price | Limits |
|---|---|---|---|
| Free API, the routes below | What agents and tools declare, and what Deside checked: whether an endpoint answers, and whether a tool asks for payment | Free, no key | Per IP; lists stop at row 500. See [Errors and limits](errors-and-limits.md). |
| Free MCP, `https://mcp.deside.io/mcp/public` | The same reads, as 12 MCP tools | Free, no sign-in | 60 requests a minute per IP, plus the API limits of your IP |
| [Paid reads](paid-reads.md), `/api/v2/paid/...` | The same 12 reads, with no depth limit and none of the per-route limits of the free API | 0.005 USDC per call, over x402 | 1 paid read at a time per payer |
| [Paid lookups](paid-lookups.md), `/api/v2/paid/x402/tool-check` and `/api/v2/paid/wallet` | Deside's view on top of the declared data: whether to pay a tool, and who is behind a wallet | 0.01 USDC per call, over x402 | As the paid reads |
| [Ask](ask.md), `POST /api/v1/ask` | An answer in plain words about the directory | 0.01 USDC per answered question, over x402, or free signed in with a wallet | See [Ask](ask.md#limits) |

**Only a successful answer is charged.** A paid read or lookup is charged only when it answers `200`. Ask is charged only when it answers within 20 seconds of your payment; a later answer is delivered free.

Each read has one code, the same through every door. Its response says so in `about`, with one sentence for each figure. See [Read codes](contract.md#read-codes).

## Versions

The agent routes are under `/api/v2/public/agents`, and the domain routes under `/api/v2/public/domains`. The x402 routes, Ask and claim are under `/api/v1`. No route needs a key.

The agent and x402 routes also answer in one common shape for lists, items and errors when you send `?minor=1.1`. Without it, each route answers as it always did. See [The Common Contract](contract.md).

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
| `GET /api/v2/public/agents/{ref}` | One agent, in short form |
| `GET /api/v2/public/agents/{ref}/profile` | One agent's profile, with its token and holdings |
| `GET /api/v2/public/agents/{ref}/holdings` | What the wallets of one agent hold, and its token. Only in the common contract. |
| `GET /api/v2/public/agents/{ref}/discovery` | What the hosts of one agent serve to machines. Only in the common contract. |
| `GET /api/v2/public/domains/{host}/discovery` | What one host serves to machines. Only in the common contract. |
| `GET /api/v2/public/domains/{host}/agents` | The agents that declare one host. Only in the common contract. |
| `GET /api/v1/public/claim/{type}/{id}` | How the owner of an agent or token can prove it |
| `POST /api/v1/ask` | Agents for a question in plain words. Paid over x402 unless you are signed in with a wallet. |
| `GET /api/v1/public/x402/tools` | x402 tools with their last check |
| `GET /api/v1/public/x402/tools/{slug}` | One x402 tool |
| `GET /api/v1/public/x402/census` | x402 Tool Directory counts |
| `GET /api/v1/public/x402/indices` | x402 indexes that agents publish |
| `GET /api/v1/public/x402/probes` | The whole last probe of an x402 tool or agent |
| `GET /api/v2/paid/...` | The reads above, 0.005 USDC per call over x402. See [Paid Reads](paid-reads.md). |
| `GET /api/v2/paid/x402/tool-check` | Should I pay this x402 tool? 0.01 USDC per call. See [Paid Lookups](paid-lookups.md). |
| `GET /api/v2/paid/wallet` | Who is this wallet? 0.01 USDC per call. See [Paid Lookups](paid-lookups.md). |

All paths start with `https://api.deside.io`.

## Quick reference

| What | Value |
|---|---|
| Base URL | `https://api.deside.io` |
| OpenAPI | `https://api.deside.io/openapi.json`: 58 public paths, the routes on this page among them. Each operation carries `x-deside-contract`, `common` or `legacy`. The paid reads and lookups are not in it: they are listed at `https://api.deside.io/.well-known/x402`. |
| Response format | JSON, field names in English |

## Next steps

- [Quickstart](quickstart.md): your first requests, with no key.
- [Checks](../start/checks.md): what Declared, Live, Connected, Matching and Verified owner mean.
- [Errors and limits](errors-and-limits.md): every code and what to do.
