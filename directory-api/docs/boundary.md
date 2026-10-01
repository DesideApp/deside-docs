# What Is Not This API

Some routes share the `/api/v1/directory` prefix or the same data and are not
part of the Directory API contract. This page names them so that finding them
does not read as an undocumented part of this API.

## The agent owner's routes

Twelve routes under `/api/v1/directory/agents/{catalogId}/` belong to the owner
of one agent, signed in to Deside with a chat session and proven owner of that
agent: `declaration` (read and write), `declaration/status`, `overlay` (read
and write), `overlay/description-score`, `subscription` and its four
`accept-intent`, `accept-confirm`, `cancel-intent` and `cancel-confirm` steps,
and `check-report`.

They do not take an API key, do not count against your quota, and their shapes
are not part of this contract.

**Two subscriptions share a word and a prefix.**
`/api/v1/directory/subscription` is the paid plan of a Directory API project
([Subscription and billing](subscription.md)).
`/api/v1/directory/agents/{catalogId}/subscription` is a reserved route for
verifying one agent, which is not offered.

## The operator's routes

`/api/v1/admin/directory/...` is for Deside operators and needs an operator
session.

## MCP

The MCP server is a separate surface for agents with its own authentication,
documented in the [MCP docs](../../mcp/README.md). It does not take an API key
and does not count against your quota ([Access model](access-model.md)).

## Paying or calling an x402 tool

The x402 routes describe tools and the price they ask. They do not call a tool
or pay for it: to use a tool, call its address yourself with an x402 client.
