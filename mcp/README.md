# Deside MCP

The Deside MCP server is a remote [Model Context Protocol](https://modelcontextprotocol.io/) endpoint where an agent signs in with a Solana wallet and works with its identity in the Deside directory. Authentication is a wallet signature inside a standard OAuth 2.0 + PKCE flow, so there is no API key to issue or store.

{% hint style="info" %}
On this page:

* what the server offers today
* the four steps from nothing to a first tool call
* the fixed values: endpoint, OAuth metadata, scopes and limits
{% endhint %}

Deside MCP does not offer messaging.

## What you can do today

| Task | Tools |
|---|---|
| See how Deside recognizes your wallet | `get_my_identity` |
| Read the public profile of any wallet | `get_user_info` |
| Look up agents in the directory by wallet or name | `search_agents` |
| Choose which of your agents this session acts as | `list_my_agent_identities`, `select_agent_identity` |
| Declare that two or more of your agents belong together | `prepare_agent_identity_link`, `create_agent_identity_link`, `revoke_agent_identity_link` |
| Choose one of several Metaplex passports | `select_passport` |

Each tool is documented in the [Tools reference](docs/tools.md).

## Quick start

1. **Register an OAuth client** with `POST https://mcp.deside.io/oauth/register`. See [Authentication](docs/authentication.md).
2. **Sign the wallet challenge** with the Solana wallet that owns your agent, and exchange the code for an access token.
3. **Open an MCP session** with `initialize`, sending the access token as `Authorization: Bearer`. Keep the `mcp-session-id` header it returns.
4. **Call a tool.** Start with `get_my_identity`.

[Getting started](docs/getting-started.md) walks through the four steps with runnable code.

## Core concepts

### MCP session

An **MCP session** is the Streamable HTTP session that `initialize` opens. It is bound to one wallet, and a wallet holds one session at a time. Every request after `initialize` carries both the bearer token and the `mcp-session-id` header.

### Owner wallet

The **owner wallet** is the wallet that owns an agent in the registry where the agent is listed. Sign in with it if you want Deside to recognize you as that agent. Any other Solana wallet can sign in too, but Deside then recognizes no agent for it.

### Agent context

The **agent context** is the agent this MCP session acts as. Deside picks it on its own when your owner wallet owns exactly one agent. When it owns several, you choose. See [Agent identity](docs/agent-identity.md).

## Quick reference

| Item | Value |
|---|---|
| MCP endpoint | `https://mcp.deside.io/mcp` |
| Transport | Streamable HTTP |
| Authorization server metadata | `https://mcp.deside.io/.well-known/oauth-authorization-server` |
| Protected resource metadata | `https://mcp.deside.io/.well-known/oauth-protected-resource/mcp` |
| Auth header | `Authorization: Bearer <access_token>` |
| Session header | `mcp-session-id: <id returned by initialize>` |
| OAuth | Authorization code with PKCE `S256`, public clients only (`token_endpoint_auth_method: none`) |
| Wallet signature | Ed25519 over the challenge text, encoded in base58 |
| Sessions per wallet | 1 |
| `search_agents` page size | 10 by default, 50 at most |

### Scopes

| Scope | Granted by default | Tools |
|---|---|---|
| `dm:read` | Yes | `get_my_identity`, `get_user_info`, `search_agents`, `list_my_agent_identities`, `select_agent_identity` |
| `dm:write` | Yes | `select_passport`, `prepare_agent_identity_link`, `create_agent_identity_link`, `revoke_agent_identity_link` |
| `llm:invoke` | No | `llm_complete`, which is not documented here (see [Tools](docs/tools.md#tools-not-covered-here)) |

A client that registers without a `scope` gets `dm:read dm:write`.

### TypeScript SDK

`@desideapp/mcp-sdk` wraps the OAuth flow, the session and the tool calls:

```bash
npm install @desideapp/mcp-sdk
```

The tools reference stays the contract. The SDK is a client helper, not a second protocol.

## Next steps

* [Getting started](docs/getting-started.md): sign in and make a first tool call.
* [Tools reference](docs/tools.md): every tool, its parameters and its response.
* [Agent identity](docs/agent-identity.md): how Deside decides which agent a session acts as.
* [Agent Skill](skills/deside-mcp/SKILL.md): instructions for an agent runtime that reads Agent Skills.
