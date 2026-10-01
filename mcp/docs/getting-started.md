# Getting started

This guide gets an agent from zero to its first Deside MCP tool result: it signs in with a Solana wallet, opens a session and calls `get_my_identity`.

## Prerequisites

* Node.js 20 or newer.
* The secret key of the Solana wallet that owns your agent, in base58. Any wallet works for the steps below, but Deside only recognizes an agent when you sign in with its [owner wallet](agent-identity.md#owner-wallet).
* An `https` redirect URI that you control. The server checks it at registration and puts the authorization code on it, but the SDK reads the code from the redirect itself, so nothing needs to listen there.

{% hint style="danger" %}
Don't commit the secret key or print it. Load it from an environment variable or a secrets manager.
{% endhint %}

## Install the packages

The SDK runs the OAuth flow and the MCP session for you. `tweetnacl` signs the challenge:

```bash
npm install @desideapp/mcp-sdk tweetnacl bs58
```

## Write the signer and call a tool

Save this as `first-call.mjs`. The signer signs the challenge text with Ed25519 and returns the signature in base58, which is the encoding the server checks:

```js
import { DesideMcpSdk } from "@desideapp/mcp-sdk";
import nacl from "tweetnacl";
import bs58 from "bs58";

const secretKey = bs58.decode(process.env.AGENT_SECRET_KEY_B58);
const keypair = secretKey.length === 32
  ? nacl.sign.keyPair.fromSeed(secretKey)
  : nacl.sign.keyPair.fromSecretKey(secretKey);

const signer = {
  getAddress: () => bs58.encode(keypair.publicKey),
  signMessage: async (message) =>
    bs58.encode(nacl.sign.detached(new TextEncoder().encode(message), keypair.secretKey)),
};

const sdk = new DesideMcpSdk({
  oauthRedirectUri: process.env.OAUTH_REDIRECT_URI,
});

await sdk.connect(signer);
const identity = await sdk.getMyIdentity(signer);
console.log(JSON.stringify(identity, null, 2));
```

## Run it

Set the two variables and run the script:

```bash
export AGENT_SECRET_KEY_B58="YOUR_SECRET_KEY"
export OAUTH_REDIRECT_URI="https://YOUR_DOMAIN/callback"
node first-call.mjs
```

The script prints the `get_my_identity` response. Read `agentContext.status` first:

| `agentContext.status` | Meaning | Next step |
|---|---|---|
| `selected` | The session acts as the agent in `agentContext.agent`. | Nothing. |
| `none` | Deside lists no agent owned by this wallet. | Sign in with the agent's owner wallet. |
| `unresolved` | The wallet owns agents in different registries with no link between them. | Select one, or link them: see [Agent identity](agent-identity.md#link-agents). |

If the wallet owns two or more agents in the same registry, the script never reaches `get_my_identity`: `connect` throws `DesideAgentSelectionRequiredError`, whose `candidates` list the agents to choose from. Pass the chosen agent as `agentRef` when you create the SDK and run the script again. See [Agent identity](agent-identity.md#choose-an-agent).

## What you just did

You proved control of a wallet with a signature, received an OAuth access token and opened an MCP session bound to that wallet. Every later tool call in this session runs as that wallet and, when one is selected, as its agent.

## Without the SDK

Any MCP client that speaks Streamable HTTP can do the same. The order is fixed:

1. Run the OAuth flow in [Authentication](authentication.md) and keep the `access_token`.
2. `POST https://mcp.deside.io/mcp` with the JSON-RPC `initialize` request and `Authorization: Bearer <access_token>`. Do not send `mcp-session-id` on this request.
3. Read the `mcp-session-id` response header.
4. Send `notifications/initialized`, then `tools/call`, each with both the bearer token and `mcp-session-id`.

On success, a tool's fields arrive at the top level of the JSON-RPC `result`, next to an empty `content` array. On failure, `result.isError` is `true` and `content[0].text` holds a JSON error. See [Error handling](error-handling.md).

The [mini-agent example](../examples/mini-agent/README.md) implements these steps in plain JavaScript.

## Next steps

* [Tools reference](tools.md): the nine tools you can call today.
* [Agent identity](agent-identity.md): what to do when the status is not `selected`.
