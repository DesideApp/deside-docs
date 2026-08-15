# Directory API Boundary

## REST surface

REST is the API-key protected read surface for the directory contract.
It is the surface that consumes quota and rate limits.

## MCP surface

MCP is a separate task-oriented surface for agent context.
It uses session auth, not `x-api-key`, and it does not consume Directory API
quota.

Current MCP contract:

- session auth
- no `x-api-key`
- no Directory API quota
- limit/offset pagination
- not a REST envelope
- not bulk

Current MCP capability:

- `search_agents`

Pro webhooks and Pro bulk exports are not MCP capabilities. They are REST/backend
capabilities:

- webhook management uses the owner console proof, not a session cookie
- bulk export uses Directory API key auth
- export files are `jsonl.gz`
- signed webhook deliveries use `X-Deside-Signature-256`

Future candidate MCP capability names:

- `get_agent_profile`
- `verify_agent_identity`
- `find_agent_services`
- `find_agents_by_capability`

`search_agents` is the existing MCP discovery tool, and its output is MCP-native
rather than a REST envelope. The future candidate names are discovery labels
only; they are not a promise that the public REST docs expose a 1:1 wrapper.

## Owner rails under the same prefix

Not everything under `/api/v1/directory` is this API. Two other rails live under
the same prefix, each with its own person and its own credential, and neither is
part of the developer read contract.

The one that will confuse you first is a name collision, so it is worth stating
plainly:

- `POST /api/v1/directory/subscription/...` is the API plan of a Directory API
  project. Its person is the project owner and its credential is the console
  proof. This is the one documented in [Endpoints](endpoints.md).
- `POST /api/v1/directory/agents/:catalogId/subscription/...` is something else
  entirely: the Verified subscription of one agent. Its person is the agent
  owner and its credential is a chat session plus proof of ownership of that
  agent. Same word, same prefix, different rail.

Eleven routes belong to that agent-owner rail and are deliberately absent from
this documentation: the agent's declaration and its derived status, the owner
overlay, the check report, and the four accept/cancel routes of the Verified
subscription, plus the two read routes above them. They are not reachable with a
Directory API key, they do not consume Directory API quota, and their shapes are
not part of this contract.

If you are integrating against the directory as a developer, none of them are
yours. They are listed here only so that finding them does not read as an
undocumented part of this API.

## x402 and A2A

x402 and A2A are documented as discovery-domain concepts for the directory
surface. They do not imply a public execution runtime here.

## Skills

Skills remain future candidates until the response shapes and public docs are
stable.

## Boundary summary

- REST handles developer reads
- REST handles Pro webhook configuration and bulk export jobs
- the owner console rail (keys, usage, subscription, webhooks) is a separate
  credential family, documented in [Authentication](authentication.md)
- the agent-owner rail under `/directory/agents/:catalogId/` is a third rail and
  is outside this contract
- MCP handles agent task context
- x402 and A2A stay outside the REST execution contract
- skills should be added only after the underlying contract is stable
