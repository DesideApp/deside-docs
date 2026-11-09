# MCP

The Deside MCP is a remote [Model Context Protocol](https://modelcontextprotocol.io/) server at `https://mcp.deside.io/mcp`. Claude, or your own agent, uses it to read the Agent Directory and the x402 Tool Directory, and to launch, trade and claim with your own Solana wallet. You sign in once with that wallet; Deside never sees your key. The read tools also answer with no sign-in at `https://mcp.deside.io/mcp/public`.

{% hint style="info" %}
On these pages:
- how to connect from Claude, Claude Code or your own code;
- sign-in and scopes;
- every tool, with its parameters and response;
- errors and limits.
{% endhint %}

## Connect from Claude

Custom connectors are available on Claude plans that support them.

1. In Claude, open **Settings → Connectors → Add custom connector**.
2. Paste `https://mcp.deside.io/mcp`. Leave the OAuth fields on their defaults and add no headers.
3. Press **Connect**. The Deside sign-in page opens.
4. Connect Phantom or Solflare and sign the text. It says what you sign in to, which app asked, and that it moves no funds.
5. You are back in Claude, signed in with that wallet.

## Connect from Claude Code

```bash
claude mcp add --transport http deside https://mcp.deside.io/mcp
```

Then type `/mcp`, choose **deside** and sign in. The same page opens in your browser.

## Connect from your own agent

An agent without a browser signs in with the same OAuth flow, asking for the challenge as JSON and signing it with its own key. See [Sign-in](authentication.md#sign-in-without-a-browser). The agent creates and keeps its wallet itself; **no Deside tool ever creates or holds a key.**

## What you can do

| Task | Tools |
|---|---|
| Find agents, read what we checked about one and what its wallets hold, and read the counts | `list_agents`, `get_agent`, `get_agent_holdings`, `get_directory_stats` |
| Read what a host serves to machines, and which agents declare it | `get_agent_discovery`, `get_host_discovery`, `list_host_agents` |
| Find x402 tools, read one, count them, list the indexes and read a whole probe | `list_x402_tools`, `get_x402_tool`, `get_x402_census`, `list_x402_indexes`, `get_x402_probe` |
| Read a token and who is behind it | `token_card` |
| Launch a token, with or without the agent's identity, and change that identity | `get_launchpad_info`, `launch_token`, `register_agent_identity`, `update_agent_identity`, `submit_transaction` |
| Follow and trade your tokens, and claim fees | `get_token`, `list_launches`, `swap`, `claim_fees`, `migrate` |

Each one is in the [Tools reference](tools.md). The read tools of the directories answer in [the common contract](../api/contract.md#in-the-mcp) of the API. To ask in plain words, use [Ask](../api/ask.md) over the API.

## Use it without signing in

`https://mcp.deside.io/mcp/public` serves the 12 read tools of the directories with no sign-in: `list_agents`, `get_agent`, `get_agent_holdings`, `get_agent_discovery`, `get_host_discovery`, `list_host_agents`, `get_directory_stats`, `list_x402_tools`, `get_x402_tool`, `get_x402_census`, `list_x402_indexes` and `get_x402_probe`. They take the same arguments and return the same shape as at `/mcp`, and the same data as the free API: what agents and tools declare, and what Deside checked. Each read has the same [read code](../api/contract.md#read-codes) as its API route.

This request lists them:

```bash
curl -X POST https://mcp.deside.io/mcp/public \
  -H "content-type: application/json" \
  -H "accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

The answer is one server-sent event whose `data:` line holds the JSON-RPC result.

**It keeps no session: every request is a POST that stands on its own.** There is no `mcp-session-id`, no OAuth and no streaming over GET. Any other tool answers as a tool that does not exist.

| What | Limit |
|---|---|
| Requests | 60 a minute per IP. Opening a client session is 3 requests (`initialize`, `notifications/initialized`, `tools/list`); each tool call is 1. |
| Public API limits | Each call also counts against the [API limits](../api/errors-and-limits.md) of your IP. |
| Depth | The same as the public API: the agent list stops at `skip` 500, and the x402 tool list at row 500. |
| Request body | 64 KB, one JSON-RPC message. Batches are refused. |

To read deeper or to use the Launchpad and `token_card`, sign in at `/mcp`. To read with no depth limit and none of the free limits, without signing in, use the [paid reads](../api/paid-reads.md).

## How signing works

Tools that change something return an unsigned transaction. If it simulates well, they also return a **sign link**:

1. Open the link within 2 minutes.
2. Connect the same wallet and review the transaction.
3. Sign. The page sends that one transaction and shows its link on the explorer.

**The page sends only the operation that was prepared, once.** Phantom may add its own priority fee and protection; nothing else is accepted. An agent signs the transaction itself and calls `submit_transaction` within about 60 seconds.

## Quick reference

| What | Value |
|---|---|
| Endpoint | `https://mcp.deside.io/mcp`, signed in with your wallet |
| Endpoint without sign-in | `https://mcp.deside.io/mcp/public`, 12 read tools |
| Transport | Streamable HTTP, one session per wallet |
| OAuth metadata | `https://mcp.deside.io/.well-known/oauth-authorization-server` |
| Protected resource metadata | `https://mcp.deside.io/.well-known/oauth-protected-resource/mcp` |
| Server card | `https://mcp.deside.io/.well-known/mcp/server-card.json`, with no session: the endpoint, the sign-in and the 23 tools with their input schemas |
| Scopes | `deside:read`, `deside:write` |
| Access token | 45 minutes |
| Refresh token | 7 days, a new one on every use |

## Next steps

- [Sign-in](authentication.md): the OAuth flow, step by step.
- [Tools reference](tools.md): every tool.
- [Errors and limits](errors.md): every code and what to do.
- [Skill](../skill.md): the whole guide in one file for an agent.

