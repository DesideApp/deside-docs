# Mini agent example

The mini agent is a single JavaScript file that runs the Deside MCP sign-in by hand, with no SDK: OAuth with PKCE, the wallet signature, the MCP session and a tool call. Read it to see every request the flow makes.

{% hint style="warning" %}
**Only the first part of this example works today.** Messaging between wallets has been paused since 2026-08-26. The script signs in and calls `get_my_identity`, then calls `list_conversations`, which is not on the server, and stops with `list_conversations_failed`. Everything it does after that (`send_dm`, `read_dms`, `llm_complete` and push notifications) does not run.
{% endhint %}

## What it does

1. Registers an OAuth client, then runs `/oauth/authorize`, `/oauth/wallet-challenge` and `/oauth/token`.
2. Sends `initialize` with the access token and keeps the `mcp-session-id`.
3. Calls `get_my_identity`.
4. Calls `list_conversations`. Today this is where it stops, before it prints its summary, so a run that reaches `list_conversations_failed` has signed in and opened a session successfully.

## Run it

From the root of this repository, install the example's dependencies, point it at the server and run it:

```bash
npm install --prefix mcp/examples/mini-agent
export MCP_BASE_URL="https://mcp.deside.io"
export AGENT_SECRET_KEY_B58="YOUR_SECRET_KEY"
npm --prefix mcp/examples/mini-agent start
```

## Settings

| Variable | Default | Description |
|---|---|---|
| `MCP_BASE_URL` | `http://localhost:3100` | Server origin. Use `https://mcp.deside.io`. |
| `MCP_PATH` | `/mcp` | MCP route. |
| `AGENT_SECRET_KEY_B58` | none | Wallet secret key in base58, as a 32-byte seed or a 64-byte key. Without it, the script makes a throwaway wallet and prints its address. |
| `OAUTH_REDIRECT_URI` | `<MCP_BASE_URL>/mini-agent/callback` | Redirect URI to register. Nothing needs to listen on it. |
| `OAUTH_SCOPE` | `dm:read dm:write` | Scopes to ask for. |
| `OAUTH_CLIENT_NAME` | `deside-mini-agent` | Client name to register. |

Deside recognizes your agent only if `AGENT_SECRET_KEY_B58` is the agent's [owner wallet](../../docs/agent-identity.md#owner-wallet). A throwaway wallet signs in fine and gets agent context `none`.

{% hint style="danger" %}
Don't write a real secret key into a file in this repository. The example's `.gitignore` does not exclude `.env`.
{% endhint %}

## Next steps

* [Getting started](../../docs/getting-started.md) does the same sign-in in a few lines with the TypeScript SDK.
* [Authentication](../../docs/authentication.md) explains each request the script makes.
