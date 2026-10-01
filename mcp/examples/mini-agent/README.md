# Mini agent example

The mini agent is a single JavaScript file that runs the Deside MCP sign-in by hand, with no SDK: OAuth with PKCE, the wallet signature, the MCP session and one tool call, `get_my_identity`. Read it to see every request the flow makes.

## What it does

1. Registers an OAuth client, then runs `/oauth/authorize`, `/oauth/wallet-challenge` and `/oauth/token`.
2. Sends `initialize` with the access token and keeps the `mcp-session-id`.
3. Calls `get_my_identity`.
4. Prints a JSON summary with the session id, the wallet and how Deside recognizes it (`recognized`, `role`, `source`).

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
