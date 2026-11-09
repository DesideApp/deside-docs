# The common contract

Every public read of agents and x402 tools can answer in one shape: one list, one item, one error. You ask for it. Without asking, each route answers exactly as it did before, so nothing you already parse changes.

## Ask for it

Send one of these with the request:

| where | value |
| --- | --- |
| query | `?minor=1.1` |
| header | `X-API-Minor: 1.1` |
| header | `X-Deside-Version: 1.1` |

The response header `X-API-Minor` says which version you got. Without any of them, or with `1.0`, you get the old shape.

This request reads one agent in the common contract:

```bash
curl "https://api.deside.io/api/v2/public/agents?minor=1.1&limit=1"
```

These routes speak it:

- `GET /api/v1/public/x402/tools`, `/tools/{slug}`, `/census`, `/indices` and `/probes`.
- `GET /api/v2/public/agents`, `/stats`, `/{ref}` and `/{ref}/profile`.
- `GET /api/v2/public/agents/{ref}/holdings`, which has no old shape (see [Public agents](public-agents.md#get-ref-holdings)): it always answers in the common contract, with `minor` or without it.
- `GET /api/v2/public/agents/{ref}/discovery`, `GET /api/v2/public/domains/{host}/discovery` and `GET /api/v2/public/domains/{host}/agents`, which have no old shape either (see [Public agents](public-agents.md#get-ref-discovery) and [Domains](domains.md)).

## The shape

    list   { "data": [...], "page": { "limit", "nextCursor", "hasMore", "total" }, "asOf" }
    item   { "data": {...}, "asOf" }
    error  { "error": { "code", "message" } }

- `page.total` is there only when we know it.
- `nextCursor` is `null` whenever `hasMore` is `false`.
- `asOf` is when what you got was read or computed. A response served from a saved snapshot (the agent list, the agent counts, the x402 census) carries the time of that snapshot.
- A list can also carry `subject` (whose list it is, like the agent of `?agent=` on `/x402/tools`) and `about` (the code of the read and its definitions, see [Read codes](#read-codes)). Nothing else sits at the root.
- The census, the counts and a probe are items: `{ "data": {...}, "asOf" }`.

A real list on 2026-11-09, trimmed to one agent and one definition:

```json
{
  "data": [
    { "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f", "slug": "deside", "path": "/agents/deside", "name": "Deside", "chain": "solana", "category": "other", "handles": { "x": null, "github": null }, "status": "responds", "liveKinds": ["mcp", "x402"] }
  ],
  "page": { "limit": 1, "nextCursor": "eyJrIjoiYWdlbnRzLXYyIiwibyI6MSwiZiI6Ijk3ZDE3MGUxNTUwZWVlNGEifQ.btkSCGYNIakpzpBPeJEBwg", "hasMore": true, "total": 295283 },
  "asOf": "2026-11-09T16:38:32.000Z",
  "about": {
    "read": "agents.list",
    "definitions": { "data[].liveKinds": "Protocols (mcp, a2a, x402) with a declared endpoint that answered Deside's last check." }
  }
}
```

## Errors

| code | status | what to do |
| --- | --- | --- |
| `invalid_request` | 400, or 409 when a reference matches more than one thing | Fix the request. `message` says what to change. A 409 also carries `error.candidates`. |
| `not_found` | 404 | The thing you named does not exist. |
| `rate_limited` | 429 | Wait and retry. |
| `unavailable` | 503 | Our side failed. Retry later. |
| `payment_required` | 402 | Pay over x402 and retry. |

There are no other codes. A real error, for a filter the route does not know:

```json
{ "error": { "code": "invalid_request", "message": "unknown filter: foo" } }
```

Under the contract, a subject you name that does not exist is a 404: an item, and the `?agent=` or `?index=` of `/x402/tools`. Without the contract, that list still answers 200 with nothing in it.

## Parameters

A route that takes filters answers 400 `invalid_request` to a parameter it does not know: a misspelled filter never returns the whole list as if it were filtered. The x402 routes and `/{ref}/holdings` do it with the contract or without it. The agent list does it only under the contract, and `skip` is one of those parameters there.

`minor` and `cb` are accepted on every public route, with the contract or without it, and filter nothing. Use `cb` with a new value to skip the cache.

## Cursor

Lists page only with `cursor` and `limit`. The cursor is opaque: pass back `page.nextCursor` as it came, with the same filters. A cursor that was changed returns 400 `malformed cursor`. The same cursor with other filters returns 400 `cursor does not match the current filters`.

Through the public API, `/x402/tools` reaches row 500 and the agent list row 500 (96 with `sort=featured`). `hasMore` is `false` on the last page you can reach, so a client that follows `nextCursor` stops by itself. The [paid reads](paid-reads.md) have no depth limit.

## Freshness

Every object with a check date carries `checkedAt` (ISO) and `stale`. `stale` is `true` when the check is older than its family allows:

| family | where | stale after |
| --- | --- | --- |
| x402 probe | `probe` of a tool, each item of `/x402/probes`, `probe` of an agent's x402 service | 7 days |
| agent check | `status`, `endpoints[]` and `services[]` of an agent | 7 days |
| wallet read | each entry of `wallets[]` in `/{ref}/holdings` | 7 days |

With no date, both are `null`. Not knowing is not fresh.

In the contract `checkedAt` replaces the old date names: `probe.at` of a tool, `probedAt` of a probe, and `status.probedAt` of an agent.

## Ids

Every object carries `id`: a tool its `slug`, an agent its `catalogId`. The `?agent=` of `/x402/probes` takes the `id` or the agent `slug`. The `?agent=` of `/x402/tools` takes only the `id`: a slug there is an agent that does not exist.

## Read codes

**Each read has one code, and the code is the same on the free API, the MCP and the paid route.** The data and its meaning are the same through every door; only the access and the price change.

A response that carries the code puts it in `about`, next to the data:

| Key | What it is |
| --- | --- |
| `about.read` | The code of the read, such as `agents.list`. |
| `about.definitions` | One sentence for each figure or field that needs one, keyed by its path in the response (`data[]` is each item of a list). |
| `about.paidAccess` | Only on a paid route: what the paid read changes against the free one. |

A list carries `about` at the root, next to `data`. An item carries it inside `data`. In `get_directory_stats` it is at `data.agents.about`.

These reads exist on every door. The code column shows the reads whose responses carry `about` today; the others answer without it.

| Code | Free API | MCP tool | Paid route | Price |
| --- | --- | --- | --- | --- |
| `agents.list` | `GET /api/v2/public/agents` | `list_agents` | `GET /api/v2/paid/agents` | 0.005 USDC |
| `agent.profile` | `GET /api/v2/public/agents/{ref}/profile` | `get_agent` | `GET /api/v2/paid/agent?ref=` | 0.005 USDC |
| `agents.stats` | `GET /api/v2/public/agents/stats` | `get_directory_stats` (at `data.agents`) | `GET /api/v2/paid/agents/stats` | 0.005 USDC |
| `agent.summary` | `GET /api/v2/public/agents/{ref}` | None | None | Free only |
| None | `GET /api/v2/public/agents/{ref}/holdings` | `get_agent_holdings` | `GET /api/v2/paid/agent/holdings?ref=` | 0.005 USDC |
| None | `GET /api/v2/public/agents/{ref}/discovery` | `get_agent_discovery` | `GET /api/v2/paid/agent/discovery?ref=` | 0.005 USDC |
| None | `GET /api/v2/public/domains/{host}/discovery` | `get_host_discovery` | `GET /api/v2/paid/domain/discovery?host=` | 0.005 USDC |
| None | `GET /api/v2/public/domains/{host}/agents` | `list_host_agents` | `GET /api/v2/paid/domain/agents?host=` | 0.005 USDC |
| None | `GET /api/v1/public/x402/tools` | `list_x402_tools` | `GET /api/v2/paid/x402/tools` | 0.005 USDC |
| None | `GET /api/v1/public/x402/tools/{slug}` | `get_x402_tool` | `GET /api/v2/paid/x402/tool?slug=` | 0.005 USDC |
| None | `GET /api/v1/public/x402/probes` | `get_x402_probe` | `GET /api/v2/paid/x402/probes` | 0.005 USDC |
| None | `GET /api/v1/public/x402/census` | `get_x402_census` (also at `data.x402Tools` of `get_directory_stats`) | `GET /api/v2/paid/x402/census` | 0.005 USDC |
| None | `GET /api/v1/public/x402/indices` | `list_x402_indexes` | `GET /api/v2/paid/x402/indices` | 0.005 USDC |

Paid routes outside this table have no free route and no MCP tool: the two [paid lookups](paid-lookups.md), `GET /api/v2/paid/x402/tool-check` and `GET /api/v2/paid/wallet` at 0.01 USDC, and [Ask](ask.md), `POST /api/v1/ask` at 0.01 USDC.

## In the MCP

The 12 MCP tools in the table answer in the common contract, with the same `data`, `page` and `asOf` as their free route. `get_agent` returns the profile without the raw registry documents. `get_directory_stats` returns `/agents/stats` and `/x402/census` together: `{ "data": { "agents", "x402Tools" }, "asOf" }`.

You do not send `minor` to the MCP: these tools always answer in the contract. Each one is in the [Tools reference](../mcp/tools.md). A failed one returns `isError: true` with `{ "error": { "code", "message" } }` and the codes above. A missing session or scope still returns `AUTH_REQUIRED` or `insufficient_scope`, as every tool does.

The same 12 tools answer with no sign-in at `https://mcp.deside.io/mcp/public`, with the depth and the limits of the public API.

## The machine-readable reference

`https://api.deside.io/openapi.json` (OpenAPI 3.1) describes 58 public paths. Each operation carries `x-deside-contract`: `common` for the 13 routes on this page, `legacy` for the rest.

The MCP publishes its server card at `https://mcp.deside.io/.well-known/mcp/server-card.json`, with no session: the endpoint, the sign-in and the 23 tools with their input schemas.
