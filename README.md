# Deside Docs

Deside is a public directory of AI agents and pay-per-call x402 tools. It reads agent registries on Solana and EVM and public x402 catalogs, calls the endpoints they declare, and says which facts are declared, which are measured and which an owner proved.

{% hint style="info" %}
These docs are organized by product: the Agent Directory, the x402 Tool Directory and the Agent Token Launchpad. Developer Access covers the two ways in for code: the keyed Directory API and the MCP server for agents.
{% endhint %}

## Quick start

1. Read the state words first: [State Words](start/state-words.md).
   Every other page uses them with that meaning.
2. List agents with no key:

   ```bash
   curl "https://api.deside.io/api/v1/public/agents?limit=5"
   ```

3. Pick a product from the table below.

## Sections

| Section | Start here | Use it for |
|---|---|---|
| Start | [State Words](start/state-words.md) | What Listed, Declared, Live and Connected mean, how to read a relation, and how to prove an agent, domain, tool or token is yours |
| Agent Directory | [Agent Directory](agent-identity/README.md) | Which registries are read, how entries become one agent, what a profile shows, and the public agents routes |
| x402 Tool Directory | [x402 Tool Directory](x402-tools/README.md) | Which catalogs are read, what the live check measures, and the public tool routes |
| Agent Token Launchpad | [Agent Token Launchpad](launchpad/README.md) | Launching a Solana token from an agent with one signature, then trading it and claiming creator fees |
| Developer Access | [Directory API](directory-api/README.md), [MCP](mcp/README.md) | Keyed REST access with quotas, errors and billing, and connecting an agent with its own wallet |

## Quick reference

| What | Where |
|---|---|
| Public routes | `https://api.deside.io/api/v1/public/...` (no key) |
| Directory API | `https://api.deside.io/api/v1/directory/...` (API key) |
| MCP endpoint | `https://mcp.deside.io/mcp` |
| Agent Skill | `npx skills add https://github.com/DesideApp/deside-docs --skill deside-mcp` |
| TypeScript SDK | `@desideapp/mcp-sdk` |

Which surface is free and which is paid is on one page:
[Access Model](directory-api/docs/access-model.md).

## Repository

This repository is the GitBook source of docs.deside.io. Changes to a
contract are listed in the [Changelog](directory-api/docs/changelog.md).

## License

[MIT](LICENSE) (c) 2026 Deside
