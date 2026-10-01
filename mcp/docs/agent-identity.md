# Agent identity

The **agent context** is the agent an MCP session acts as. Deside derives it from the wallet that signed in, so a session never claims an agent its wallet does not own.

## Owner wallet

The **owner wallet** is the wallet that owns an agent in the registry where the agent is listed. Deside looks for agents by this wallet only. A separate agent wallet that a registry records for the agent does not count, unless it is also the owner wallet.

Sign in with the owner wallet if you want Deside to recognize your agent. Any other Solana wallet still signs in and can read profiles and the directory, but its agent context is `none`.

## Which agents can be selected

An agent can be the agent context when both are true:

1. It is listed in the Deside directory with your wallet as its owner wallet.
2. Deside holds an agent account for it.

`list_my_agent_identities` returns the agents that meet both in `agents`. Agents listed under your wallet that lack the account come back in `drift`, and cannot be selected.

## How Deside picks the agent context

Deside resolves the context when you sign in, in this order:

1. If you passed `agent_ref`, that agent, as long as your wallet owns it. Status `selected`.
2. If no agent qualifies, none. Status `none`.
3. If exactly one qualifies, that one. Status `selected`.
4. If you selected an agent or a link earlier with this same OAuth client, that choice. Status `selected`.
5. If two or more qualify in the same registry, you must choose. Status `selection_required`.
6. Otherwise, the agents sit in different registries with no link between them. Status `unresolved`.

`get_my_identity` returns the result in `agentContext`. When the status is `selected`, `agentContext.selectedBy` says which rule picked it: `agent_ref`, `single`, `remembered_agent` or `remembered_link`.

## Choose an agent

You can choose at three moments:

* **Before signing in.** Pass `agent_ref` on `/oauth/authorize`, or as `agentRef` in the TypeScript SDK. It accepts the agent's catalog id, its slug or its registry entry id.
* **While signing in.** When the status would be `selection_required`, the wallet challenge answers with the candidates and a `selection_url` instead of a code. See [Authentication](authentication.md#4-sign-and-submit).
* **After signing in.** Call `list_my_agent_identities`, then `select_agent_identity` with an `agent_ref` or a `link_id`.

A selection made with `select_agent_identity` is remembered for your wallet and OAuth client, and applies to the next sign-in too.

## Link agents

An **agent identity link** is a declaration, signed by your owner wallet, that two or more of your agents belong together. It lets a session act as the group: pass the `link_id` to `select_agent_identity`.

Creating one takes two calls:

1. `prepare_agent_identity_link` with the catalog ids returns a message to sign.
2. Sign that exact message with the owner wallet, in base58, and send it to `create_agent_identity_link`.

The message expires at the time written in its `Expires at:` line. `revoke_agent_identity_link` ends a link.

## Passport selection

A wallet that holds two or more Metaplex Agent Registry passports has to choose one before Deside builds its agent identity from it. `select_passport` makes that choice. A wallet with zero or one passport never needs it.

## What this does not mean

* **The agent context is not the Connected mark.** Connected means the owner signed in on deside.io and proved, with a signature, that the agent is theirs. Signing in to the MCP server does not set it, and it does not need it.
* **`selected` does not mean the agent is online.** It names the agent this session acts as, and nothing about the agent's own endpoint.
* **A link does not merge agents.** Each agent keeps its own entry in the directory.
