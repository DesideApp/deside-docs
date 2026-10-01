# Deside Agent Launchpad

The Deside Agent Launchpad is a service that prepares Solana token launches on Meteora Dynamic Bonding Curve for AI agents. You need no account and no approval: every write operation returns an unsigned transaction that you sign with your own wallet, so Deside never holds your key or your funds.

{% hint style="info" %}
On this page:

* what one launch signature contains
* the four steps from nothing to a launched token
* the fixed values: base URL, MCP endpoint, networks and fees
{% endhint %}

One launch signature creates the token and its bonding curve, optionally registers an EIP-8004 agent identity in the Metaplex Agent Registry, and pays the permanent Arweave storage of the logo and metadata.

{% hint style="warning" %}
The service is not public yet. `https://LAUNCHPAD_URL` stands for its base URL on every page of this section until the address is published.
{% endhint %}

## Quick start

1. **Read the terms** with `get_launchpad_info` (MCP) or `GET /v1/info`. It needs no wallet.
2. **Prepare the launch** with `launch_token` (MCP) or `POST /v1/launch`. You get one unsigned transaction.
3. **Sign it** with the wallet you passed as `wallet`.
4. **Send it** with `submit_transaction` (MCP) or `POST /v1/submit` within about 60 seconds.

[Getting started](docs/getting-started.md) walks through the four steps on devnet over MCP, and the [REST quickstart](docs/rest-quickstart.md) does it with `curl` and a Node.js signing script.

## Core concepts

### Unsigned transaction

An **unsigned transaction** is the base64 Solana transaction that every write operation returns. Deside has already added the signatures it owns (the new mint and, with an identity, the new agent asset). Your wallet adds the last one. Nothing reaches the chain until you sign and send it. See [Operations reference](docs/operations.md).

### Bonding curve and graduation

The **bonding curve** is the Meteora Dynamic Bonding Curve pool your token trades on after launch. **Graduation** is the moment the curve has raised its threshold in SOL and migrates to a Meteora DAMM v2 pool. On mainnet the Meteora keeper normally migrates within seconds; `migrate` exists for when it has not. See [Fees and rules](docs/fees-and-rules.md).

### Agent identity

An **agent identity** is an EIP-8004 registration in the Metaplex Agent Registry, owned by the launching wallet. You ask for it with `registerAgentIdentity: true` and describe the agent in the `agent` fields. Deside fills `type` and `registrations`, adds the `agentWallet` service unless you declare one and, on mainnet, a `web` service pointing to the agent's Metaplex page unless you declare one. The registration file has exactly the EIP-8004 fields and nothing specific to Deside. See [`launch_token`](docs/operations.md#launch_token).

### Network

Every operation except `get_launchpad_info` takes `network`: `"mainnet"` for real tokens or `"devnet"` to rehearse the full cycle with free devnet SOL. The devnet configuration graduates at about 0.64 SOL so you can reach graduation in a test.

## Quick reference

| Item | Value |
|---|---|
| Base URL | `https://LAUNCHPAD_URL` (not published yet) |
| MCP endpoint | `https://LAUNCHPAD_URL/mcp` (Streamable HTTP, stateless, `POST` only) |
| REST routes | `https://LAUNCHPAD_URL/v1/...` |
| OpenAPI schema | `https://LAUNCHPAD_URL/openapi.json` |
| LLM summary | `https://LAUNCHPAD_URL/llms.txt` |
| Authentication | None |
| Networks | `mainnet`, `devnet` |
| Rate limits | 60 requests per minute per IP, of which at most 10 launches |
| Request body | 2 MB maximum |

| Fee or cost | Value |
|---|---|
| Trading fee on the curve | 1% per trade: creator 0.40%, Deside 0.40%, Meteora 0.20% |
| Launch fee charged by Deside | None |
| Launch cost (network rent and fees) | About 0.0206 SOL, plus about 0.0049 SOL with an agent identity |
| Arweave storage | Charged at cost in the same signature |

The full fee schedule, the anti-sniper fee and the graduation rules are on [Fees and rules](docs/fees-and-rules.md).

## Next steps

* [Getting started](docs/getting-started.md): launch a token on devnet end to end over MCP.
* [REST quickstart](docs/rest-quickstart.md): the same launch with `curl` and a local signing script.
* [Operations reference](docs/operations.md): the 8 operations, their parameters, responses and errors.
* [Fees and rules](docs/fees-and-rules.md): who earns what, limits and the disclaimer.
