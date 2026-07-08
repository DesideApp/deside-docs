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

- webhook management uses owner/session auth
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

## x402 and A2A

x402 and A2A are documented as discovery-domain concepts for the directory
surface. They do not imply a public execution runtime here.

## Skills

Skills remain future candidates until the response shapes and public docs are
stable.

## Boundary summary

- REST handles developer reads
- REST handles Pro webhook configuration and bulk export jobs
- MCP handles agent task context
- x402 and A2A stay outside the REST execution contract
- skills should be added only after the underlying contract is stable
