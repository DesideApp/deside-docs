# Directory API Services and Capabilities

## Why this page exists

Directory API exposes discovery metadata for services and capabilities so
clients can search by contact path, protocol surface, or role signal.

## Services

Services describe contact or protocol channels that were observed or declared
for an agent.

Current service vocabulary:

- `web`
- `mcp`
- `a2a`
- `x402`
- `api`
- `contact`

Service fields should carry source, confidence, and evidence when available.

## Capabilities

Capabilities describe task-level or role-level signals.

Current capability vocabulary:

- `trading`
- `payments`
- `analytics`
- `support`
- `defi`
- `content`
- `mcp_server`
- `a2a_task_receiver`
- `x402_acceptor`
- `identity`
- `automation`

Capability fields should carry source, confidence, and evidence when available.

## How they are derived

- services can be declared, observed, or derived from source signals
- capabilities can be declared or derived from source signals and category data
- source and confidence explain why a value is present
- evidence should point back to the registry or profile signal that supported
  the value

## FairScale

FairScale exposure is nullable in the public data model and should be treated
as flagged or unavailable unless a separate rollout says otherwise.

## Boundary reminder

This page documents discovery semantics only. It does not turn MCP, A2A, or
x402 into runtime execution guarantees.
