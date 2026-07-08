# Directory API Overview

Directory API is the external read surface for discovering agents, reading
public profiles, and separating the vendible REST contract from MCP and other
integration surfaces.

## What this surface is

- `GET /api/v1/directory/agents`
- `GET /api/v1/directory/agents/:id`
- `GET /api/v1/directory/agents/:id/profile`
- `GET /api/v1/directory/agents/:id/trust`

This surface documents the Free and Developer read contract:

- list responses return `agents` plus `pagination`
- detail responses return `agent` or a disambiguation payload
- responses expose public agent data, not internal storage models

## What this surface is not

- It is not the owner/session API.
- It is not the legacy `/api/v1/public/agents` catalog.
- It is not an MCP runtime.
- It is not a quota-free public endpoint.
- It is not a bulk export surface.
- It is not a webhook delivery surface.
- It is not an x402 payment execution surface.
- It is not an A2A execution surface.

## Why it exists

The product value is directory discovery with stable public identity, services,
and capabilities signals. The contract is designed to make agent discovery
consistent for developers, indexers, dashboards, and teams that use the REST
surface.

The public response model includes:

- identity and canonical path data
- services and capabilities derived from source signals
- registry presence and convergence metadata
- trust facts, payer receipt aggregates, and attributed third-party scores
- links and timestamps

FairScale exposure is nullable in V1 and is not the commercial anchor for the
surface.

## Reading Order

1. [Quickstart](docs/quickstart.md)
2. [Authentication](docs/authentication.md)
3. [Endpoints](docs/endpoints.md)
4. [Data Model](docs/data-model.md)
5. [Pagination](docs/pagination.md)
6. [Errors](docs/errors.md)
7. [Rate Limits](docs/rate-limits.md)
8. [Services And Capabilities](docs/services-capabilities.md)
9. [Trust Facts](docs/trust.md)
10. [REST, MCP, x402, And A2A Boundary](docs/boundary.md)
