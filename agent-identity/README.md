# Agent Directory

The Agent Directory is a public catalogue of AI agents registered onchain, with one profile per agent. Unlike a registry explorer, it reads several registries, joins the entries that belong to the same agent, and says which of the agent's endpoints answered when we called them.

{% hint style="info" %}
In this section you will find:

- the registries Deside reads, and how several entries become one agent
- what a profile shows: services, tokens, holdings and reputation
- the public routes that serve the directory, with real responses
- Ask, which answers a question about the directory in plain language
{% endhint %}

## Quick start

1. List agents. No key is needed:

   ```bash
   curl "https://api.deside.io/api/v1/public/agents?limit=5"
   ```

2. Narrow the list to one registry and to agents whose MCP endpoint answered:

   ```bash
   curl "https://api.deside.io/api/v1/public/agents?registry=mip14&live=mcp&limit=5"
   ```

3. Open one agent by its slug:

   ```bash
   curl "https://api.deside.io/api/v1/public/agents/xona-agent-ssz7"
   ```

4. See the same agent as a person sees it: [deside.io/agents/xona-agent-ssz7](https://deside.io/agents/xona-agent-ssz7).

## Core concepts

The state words Listed, Declared, Live and Connected are defined once in [State Words](../start/state-words.md).

### Registry

**A registry is an onchain program where agents are registered**, such as the Metaplex Agent Registry or ERC-8004 on Base. Deside treats every registry the same way: none is the preferred one. See [Registries](passport-and-protocol-registries.md).

### Agent

**An agent is one profile in the directory, backed by one or more registry entries.** Entries are joined only when the evidence is unambiguous. See [Identity Resolution](identity-resolution-and-auth-boundaries.md).

### Profile

**A profile is the full page for one agent**: where it is registered, how it can be reached, its token, its holdings and its reputation. See [Directory And Profile](agent-directory-and-profile-surfaces.md).

### Relation

**A relation is a link between an agent and a token or x402 tool, marked Declared, Matches or Proven.** See [How To Read A Relation](../start/relations.md).

## Quick reference

| Item | Value |
| --- | --- |
| Base URL | `https://api.deside.io/api/v1/public/agents` |
| Authentication | None |
| Rate limit, list | 30 requests per minute |
| Rate limit, one agent | 60 requests per minute |
| Page size | 20 by default, 100 at most |
| Deepest `skip` | 500 |
| Profile page on the web | `https://deside.io/agents/<slug>` |
| Bulk access with cursor pagination | [Directory API](../directory-api/README.md) |
| Agent access over MCP | [MCP](../mcp/README.md) |

## Next steps

- [Registries](passport-and-protocol-registries.md): which registries and chains are read, and the identifier each one uses.
- [Directory And Profile](agent-directory-and-profile-surfaces.md): what a profile shows, including tokens, holdings and reputation.
- [Public Agents API](public-api-contracts.md): every public route, parameter and error.
- [Prove It Is Yours](../start/prove-ownership.md): make your agent Connected.

## License

[MIT](../LICENSE) (c) 2026 Deside
