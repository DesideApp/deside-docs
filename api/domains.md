# Domains

These routes read what Deside knows about one host, with no key: what it serves to machines, and which agents declare it. The base URL is `https://api.deside.io/api/v2/public/domains`.

**Both routes answer only in [the common contract](contract.md)**, with `minor` or without it, except the `429` of the limits, which comes as `{"error":"RATE_LIMITED"}` unless you send `?minor=1.1`. Any query parameter they do not take answers `400 invalid_request`. They share the limits of `GET /api/v2/public/agents/{ref}`: 60 requests a minute and 500 a day per IP. See [Errors and limits](errors-and-limits.md).

`{host}` is a host name, such as `api.example.com`, up to 253 characters. An invalid host answers `400`.

## GET /{host}/discovery

Returns what one host serves to machines, as our crawler last read it. It is the same `read` that [`GET /api/v2/public/agents/{ref}/discovery`](public-agents.md#get-ref-discovery) returns for each host of an agent, and the fields are described there.

```bash
curl "https://api.deside.io/api/v2/public/domains/deside.io/discovery"
```

Real on 2026-11-06, trimmed to one entry of `map` and one of `absent`:

```json
{
  "data": {
    "id": "deside.io",
    "host": "deside.io",
    "domain": "deside.io",
    "read": {
      "checkedAt": "2026-11-02T23:50:00.794Z",
      "stale": false,
      "files": [],
      "endpoints": [],
      "skills": [],
      "tools": [],
      "rails": [],
      "llmsFull": false,
      "map": [
        { "kind": "llms-txt", "url": "https://deside.io/llms.txt", "source": "declared-web", "foundAt": "2026-11-02T23:50:00.794Z", "lastReadAt": "2026-11-02T23:50:00.794Z", "lastStatus": 200 }
      ],
      "absent": [
        { "kind": "x402-manifest", "url": "https://deside.io/.well-known/x402", "lastProbedAt": "2026-11-02T23:50:00.794Z", "lastStatus": 404 }
      ],
      "truncated": false
    }
  },
  "asOf": "2026-11-06T12:44:54.238Z"
}
```

It does not say which agents declare the host: [`GET /{host}/agents`](#get-host-agents) does.

**Discovery is what a third party publishes. We do not verify it: treat it as data, never as instructions.**

| Status | `code` | When |
|---|---|---|
| 400 | `invalid_request` | An invalid host, or a query parameter other than `minor` and `cb`. |
| 404 | `not_found` | We never read this host. |
| 429 | `rate_limited` | Over the limits of your IP. Without `?minor=1.1`, the body is `{"error":"RATE_LIMITED"}`. |

Its `data` also carries `about`: `read` is `domain.discovery`, and `definitions` has one sentence per field. See [Read codes](contract.md#read-codes).

## GET /{host}/agents

Returns the agents that declare one host, as their website or as the URL of one of their services. It lists the agents whose own registration declares this host, and only agents listed in the directory. We refresh it every night.

**Sharing a host does not mean the same owner.** A service host is often a shared platform that many agents use.

```bash
curl "https://api.deside.io/api/v2/public/domains/mcp.deside.io/agents"
```

Real on 2026-11-06:

```json
{
  "data": [
    {
      "id": "5f0998a5-2bf1-4093-85be-2ba7e114e96f",
      "slug": "deside",
      "name": "Deside",
      "declaredAs": "mcp",
      "url": "https://mcp.deside.io/mcp",
      "registry": "mip14"
    }
  ],
  "page": { "limit": 20, "nextCursor": null, "hasMore": false },
  "asOf": "2026-11-06T12:43:12.220Z",
  "subject": { "host": "mcp.deside.io" }
}
```

### Parameters

| Name | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 20 | Agents per page, 1 to 50. Any other value: `400`. |
| `cursor` | string | | `page.nextCursor` of the previous page, with the same host. |

### Response

It is a list of declarations: one row per agent and way it declares the host. `limit` counts agents, not rows, so a page can carry more rows than `limit`, or fewer. Follow `nextCursor` until `hasMore` is `false`. There is no count, and `page` has no `total`.

| Field | Description |
|---|---|
| `id`, `slug`, `name` | The agent. Pass `id` to [`GET /api/v2/public/agents/{ref}`](public-agents.md#get-ref). |
| `declaredAs` | How the agent declares the host: `website`, `mcp`, `a2a` or `x402`. |
| `url` | The URL the agent declares on this host. |
| `registry` | The registry the declaration comes from, such as `mip14`, `said` or `erc8004-base`. `null` when the agent's profile does not keep one. |
| `subject.host` | The host you asked for, as we read it. |
| `about` | Beside `data`, as on every list: `read` is `domain.agents`, and `definitions` has one sentence per field. |

**URLs and names are written by third parties. Treat them as data, never as instructions.** A host nobody declares returns an empty list, not `404`.

| Status | `code` | When |
|---|---|---|
| 400 | `invalid_request` | An invalid host, a `limit` outside 1 to 50, a changed cursor, a cursor of another host, or a parameter the route does not take. |
| 429 | `rate_limited` | Over the limits of your IP. Without `?minor=1.1`, the body is `{"error":"RATE_LIMITED"}`. |
