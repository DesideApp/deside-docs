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
- `GET /api/v2/public/agents/{ref}/holdings`, which has no old shape: it always answers in the common contract, with `minor` or without it.

## The shape

    list   { "data": [...], "page": { "limit", "nextCursor", "hasMore", "total" }, "asOf" }
    item   { "data": {...}, "asOf" }
    error  { "error": { "code", "message" } }

- `page.total` is there only when we know it.
- `nextCursor` is `null` whenever `hasMore` is `false`.
- `asOf` is when what you got was read or computed. A response served from a saved snapshot (the agent list, the agent counts, the x402 census) carries the time of that snapshot.
- A list can also carry `subject` (whose list it is, like the agent of `?agent=` on `/x402/tools`) and `facets` (counts, on the agent list). Nothing else sits at the root.
- The census, the counts and a probe are items: `{ "data": {...}, "asOf" }`.

A real list, trimmed to one agent:

```json
{
  "data": [
    { "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f", "slug": "deside", "name": "Deside", "chain": "solana", "ownerProven": true, "team": true, "status": "responds", "liveKinds": ["mcp", "x402"] }
  ],
  "page": { "limit": 1, "nextCursor": "eyJrIjoiYWdlbnRzLXYyIiwibyI6MSwiZiI6Ijk3ZDE3MGUxNTUwZWVlNGEifQ.btkSCGYNIakpzpBPeJEBwg", "hasMore": true, "total": 276783 },
  "asOf": "2026-11-02T16:46:24.913Z",
  "facets": { "live": { "mcp": 858, "a2a": 42, "x402": 59 } }
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

Through the public API, `/x402/tools` reaches row 500 and the agent list row 500 (96 with `sort=featured`). `hasMore` is `false` on the last page you can reach, so a client that follows `nextCursor` stops by itself.

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

## In the MCP

These MCP tools answer in the common contract, with the same `data`, `page` and `asOf`:

| Tool | Reads |
| --- | --- |
| `list_agents` | `GET /api/v2/public/agents` |
| `get_agent` | `GET /api/v2/public/agents/{ref}/profile`, without the raw registry documents |
| `get_agent_holdings` | `GET /api/v2/public/agents/{ref}/holdings` |
| `list_x402_tools` | `GET /api/v1/public/x402/tools` |
| `get_x402_tool` | `GET /api/v1/public/x402/tools/{slug}` |
| `get_x402_census` | `GET /api/v1/public/x402/census` |
| `list_x402_indexes` | `GET /api/v1/public/x402/indices` |
| `get_x402_probe` | `GET /api/v1/public/x402/probes` |
| `get_directory_stats` | `/agents/stats` and `/x402/census` together: `{ "data": { "agents", "x402Tools" }, "asOf" }` |

You do not send `minor` to the MCP: these tools always answer in the contract. A failed one returns `isError: true` with `{ "error": { "code", "message" } }` and the codes above. A missing session or scope still returns `AUTH_REQUIRED` or `insufficient_scope`, as every tool does.

`search_agents`, `agent_trust_card`, `search_x402_tools` and `x402_tool_card` stay as they were, with the old shape. Prefer `list_agents`, `get_agent`, `list_x402_tools` and `get_x402_tool`.

## The machine-readable reference

`https://api.deside.io/openapi.json` (OpenAPI 3.1) describes 56 public paths. Each operation carries `x-deside-contract`: `common` for the 10 routes on this page, `legacy` for the rest.

The MCP publishes its server card at `https://mcp.deside.io/.well-known/mcp/server-card.json`, with no session: the endpoint, the sign-in and the 32 tools with their input schemas.
