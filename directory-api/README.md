# Directory API Overview

The Directory API is the HTTP interface to the Deside directory: the agents
listed in Deside and the x402 tools it has found in public x402 catalogs. It
serves the same data three ways: a public read surface with no key, a keyed
read surface with monthly quotas, and the Deside MCP server for agents.

{% hint style="info" %}
On these pages:

* which surface to use and what each one costs ([Access model](docs/access-model.md))
* one reference page per group of endpoints, with parameters, real example responses and errors
* the shapes the endpoints return ([Data model](docs/data-model.md)) and how to page through them ([Pagination](docs/pagination.md))
* every contract change, dated ([Changelog](docs/changelog.md))
{% endhint %}

## Quick start

1. Read one x402 tool from the public catalog. No key is needed:

   ```bash
   curl -sS "https://api.deside.io/api/v1/public/x402/tools?limit=1"
   ```

2. Read the directory's headline counters:

   ```bash
   curl -sS "https://api.deside.io/api/v1/public/agents/stats-summary"
   ```

3. To page through every agent with a stable cursor, create a key in the API
   console at `https://deside.io/developer/api` and follow the
   [Quickstart](docs/quickstart.md).

## Core concepts

### Listed

**An agent is listed when it is in the Deside catalogue.** Being listed says
nothing about whether the agent answers or who runs it. See
[Data model](docs/data-model.md).

### Connected

**`connected` is `true` when the agent's owner has proved the agent is theirs**
by linking, with a signature, the wallet that owns it. It is a fact about the
owner, not a measure of liveness. See [Data model](docs/data-model.md#connected).

### Responds

**`responds` means one of the agent's declared endpoints answered a protocol
check.** It does not say the agent is good at its job. See
[Trust facts](docs/trust.md#how-liveness-is-measured).

### x402 tool

**An x402 tool is one paid HTTP endpoint that a public x402 catalog lists.**
Deside reads the catalogs, merges duplicates into one tool, and calls each
tool's address without paying to record what it answers. See
[x402 tool catalog](docs/x402-tools.md).

## Quick reference

| Item | Value |
| --- | --- |
| Base URL | `https://api.deside.io` |
| Public read surface | `/api/v1/public/...`, no key |
| Keyed read surface | `/api/v1/directory/...`, header `x-api-key: dapi_...` |
| MCP endpoint | `https://mcp.deside.io/mcp` ([MCP docs](../mcp/README.md)) |
| Free tier | 5,000 requests a month, 30 a minute |
| Public list limits | 30 requests a minute per IP, 500 rows deep |
| Error format (keyed) | `{ "error": { "code", "message", "requestId", "docsUrl" } }` |

The full limits are on [Rate limits](docs/rate-limits.md).

## Next steps

* [Access model](docs/access-model.md): pick the surface that fits what you are building.
* [x402 tool catalog](docs/x402-tools.md): search tools by host, network, wallet or text.
* [Quickstart](docs/quickstart.md): get a key and walk the agent directory.
* [Errors](docs/errors.md): every code, when it happens and what to do.
