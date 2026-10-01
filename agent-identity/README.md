# Agent Identity

Deside is a public catalogue of AI agents registered onchain, with one profile per agent. Unlike a registry explorer, it reads several registries, joins the entries that belong to the same agent, and says which of the agent's endpoints answered when we called them.

{% hint style="info" %}
In this section you will find:

- the registries Deside reads, and how an agent enters and leaves the catalogue
- how several registry entries become one agent, and when they stay separate
- what each state word on a profile means and how it is measured
- the public endpoints that serve the catalogue, with real responses
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

### Listed

**An agent is listed when it appears in the Deside catalogue.** Deside reads the registries itself; nothing is self-submitted. See [How We Verify](how-we-verify.md#listed).

### Registry

**A registry is an onchain program where agents are registered**, such as the Metaplex Agent Registry or ERC-8004 on Base. Deside treats every registry the same way: none is the preferred one. See [Registries](passport-and-protocol-registries.md).

### Agent

**An agent is one profile in the catalogue, backed by one or more registry entries.** Entries are joined only when the evidence is unambiguous. See [Identity Resolution](identity-resolution-and-auth-boundaries.md).

### Declared, Responds, Connected

**A declaration is what a registry or owner says about the agent. Responds is what we measured. Connected is what the owner proved.** The three are independent. See [How We Verify](how-we-verify.md).

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

{% hint style="warning" %}
Wallet-to-wallet messaging between people and agents has been switched off since 2026-08-26. The catalogue, profiles and public endpoints are not affected.
{% endhint %}

## Next steps

- [How We Verify](how-we-verify.md): what Listed, Declared, Responds and Connected mean, so you know how much weight to give each.
- [Registries](passport-and-protocol-registries.md): which registries and chains are read, and the identifier each one uses.
- [Directory And Profile](agent-directory-and-profile-surfaces.md): what a profile shows, including tokens, holdings and reputation.
- [Public Agents API](public-api-contracts.md): every public endpoint, parameter and error.

## License

[MIT](../LICENSE) (c) 2026 Deside
