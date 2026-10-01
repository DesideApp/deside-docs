# Access Model

Deside serves its directory through three surfaces: the public read surface,
the keyed Directory API and the MCP server. They read the same agents and the
same x402 tools. They differ in who they are for, what they cost and how deep
you can read.

## The three surfaces

| Surface | For | Credential | Cost | Depth |
| --- | --- | --- | --- | --- |
| Public read surface, `/api/v1/public/...` | the website and anyone browsing | none | free | lists stop at 500 rows |
| Directory API, `/api/v1/directory/...` | developers syncing or analysing the directory | `x-api-key: dapi_...` | Free tier, paid tiers by subscription | no depth limit on cursor routes |
| MCP server, `https://mcp.deside.io/mcp` | agents | MCP session | free | see the [MCP docs](../../mcp/README.md) |

## Public read surface

The public read surface needs no key and consumes no quota. It is limited per
IP address:

| Limit | Value |
| --- | --- |
| List routes (`/public/agents`, `/public/x402/tools`, `/public/x402/indices`) | 30 requests a minute |
| Other routes | 60 requests a minute |
| All `/public/agents` routes together, and separately all `/public/x402` routes together | 500 requests a day |

**A public list cannot be walked past its first 500 rows.** On
`/public/x402/tools` and `/public/x402/wallet-edges`, a page whose offset plus
`limit` goes beyond 500 answers `400`. On `/public/agents`, a `skip` above 500
answers `400`. Everything stays reachable through
search and filters (`q`, `host`, `network`, `payTo` and the rest); what the
public surface does not allow is downloading a whole list.

A full download is what the Directory API is for: its cursor routes have no
depth limit.

## Directory API

Every route under `/api/v1/directory/` that reads data needs an API key, on
every tier including Free. Keys are self-serve: you sign in to the API console
with your wallet and create one. Each request counts against the monthly quota
of your project, and each tier has a per-minute rate. The figures are on
[Rate limits](rate-limits.md).

Paid tiers are bought with a recurring USDC authorization signed by your
wallet. See [Subscription and billing](subscription.md).

## MCP server

The MCP server is for agents. It does not read `x-api-key` and does not consume
Directory API quota. Its directory tools are `search_agents` and
`agent_trust_card`; their inputs and outputs are documented in
[MCP tools](../../mcp/docs/tools.md).

An agent that needs x402 tools reads the [x402 Tools API](../../x402-tools/docs/api.md)
or the [keyed x402 routes](x402-keyed.md).

## Which one to use

1. You are building a page or a one-off lookup: use the public read surface.
2. You need the whole directory, or to keep a copy in sync: use the Directory
   API with a key.
3. You are an agent looking for other agents: use the MCP server.

## What this does not mean

* Free does not mean unlimited. The public surface has per-IP limits and a
  depth limit.
* The keyed surface does not return more fields for the same object on every
  route. The x402 tool profile is the exception: the keyed version adds the
  fields listed in [x402 data with an API key](x402-keyed.md).
