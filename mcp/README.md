# MCP

The Deside MCP is a remote [Model Context Protocol](https://modelcontextprotocol.io/) server at `https://mcp.deside.io/mcp`. Claude, or your own agent, uses it to search the directories, ask in plain words, and launch, trade and claim with your own Solana wallet. You sign in once with that wallet; Deside never sees your key.

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
| Find agents, read what we checked about one, ask in plain words, read a token | `search_agents`, `agent_trust_card`, `ask_directory`, `get_directory_stats`, `token_card` |
| Launch a token, with or without the agent's identity, and change that identity | `get_launchpad_info`, `launch_token`, `register_agent_identity`, `update_agent_identity`, `submit_transaction` |
| Follow and trade your tokens, and claim fees | `get_token`, `list_launches`, `swap`, `claim_fees`, `migrate` |
| See who Deside recognizes you as, and choose your agent | `get_my_identity`, `get_user_info`, `list_my_agent_identities`, `select_agent_identity` |
| Declare that several of your agents are one | `prepare_agent_identity_link`, `create_agent_identity_link`, `revoke_agent_identity_link` |

Each one is in the [Tools reference](tools.md).

## How signing works

Tools that change something return an unsigned transaction. If it simulates well, they also return a **sign link**:

1. Open the link within 2 minutes.
2. Connect the same wallet and review the transaction.
3. Sign. The page sends that one transaction and shows its link on the explorer.

**The page sends only the operation that was prepared, once.** Phantom may add its own priority fee and protection; nothing else is accepted. An agent signs the transaction itself and calls `submit_transaction` within about 60 seconds.

## Quick reference

| What | Value |
|---|---|
| Endpoint | `https://mcp.deside.io/mcp` |
| Transport | Streamable HTTP, one session per wallet |
| OAuth metadata | `https://mcp.deside.io/.well-known/oauth-authorization-server` |
| Protected resource metadata | `https://mcp.deside.io/.well-known/oauth-protected-resource/mcp` |
| Scopes | `deside:read`, `deside:write` |
| Access token | 45 minutes |
| Refresh token | 7 days, a new one on every use |

## Next steps

- [Sign-in](authentication.md): the OAuth flow, step by step.
- [Tools reference](tools.md): every tool.
- [Errors and limits](errors.md): every code and what to do.
- [Skill](../skill.md): the whole guide in one file for an agent.

