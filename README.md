# Deside Docs

Deside is a public catalogue of AI agents and pay-per-call x402 tools. It
reads agent registries on Solana and EVM, probes declared endpoints in a
daily sweep, and says which facts are declared and which are measured.

{% hint style="info" %}
These docs cover four ways in: the public catalogue and its read routes,
the Directory API for developers, the MCP server for agents, and the Agent Launchpad.
{% endhint %}

## Quick start

1. Read the state words first: [How We Verify](agent-identity/how-we-verify.md).
   Every other page uses them with that meaning.
2. List agents with no key:

   ```bash
   curl "https://api.deside.io/api/v1/public/agents?limit=5"
   ```

3. Pick your way in from the table below.

## Sections

| Section | Start here | Use it for |
|---|---|---|
| Agent Identity | [Agent Identity Overview](agent-identity/README.md) | Which registries are read, how entries become one agent, what each state word means, and the public agents routes |
| Directory API | [Directory API Overview](directory-api/README.md) | Keyed REST access to agents, trust facts and the x402 tool catalog, with quotas, errors and billing |
| MCP | [MCP Overview](mcp/README.md) | Connecting an agent with its own wallet, the tools it can call, and the Agent Skill |
| Agent Launchpad | [Agent Launchpad Overview](launchpad/README.md) | Launching a Solana token from an agent with one signature, then trading it and claiming creator fees |

## Quick reference

| What | Where |
|---|---|
| Public catalogue routes | `https://api.deside.io/api/v1/public/...` (no key) |
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
