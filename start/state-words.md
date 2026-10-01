# State Words

A state word is the one-word answer Deside gives about an agent: Listed, Declared, Live or Connected. Each is one of three things: a declaration with its source named, a measurement with its check date, or a proof made by the owner. This page defines each word once. Other pages link here.

When Deside has not measured something, it shows nothing rather than a guess. A missing value means "not measured", never "zero".

| Word | Kind | Says | API value |
| --- | --- | --- | --- |
| Listed | fact | The agent is in the Deside catalogue. | `listed` |
| Declared | declaration | A registry or owner says it, with the source named. | `declared` |
| Live | measurement | One of its endpoints answered its latest check. | `responds` |
| Connected | proof | Its owner proved it is theirs. | `connected` |

Links between an agent and a token or x402 tool use three other words, Declared, Matches and Proven, defined in [How To Read A Relation](relations.md).

## Listed

An agent is **listed** when it appears in the Deside catalogue. Deside reads these registries itself, so nothing in the catalogue is self-submitted:

| Chain | Registries |
| --- | --- |
| Solana | Metaplex Agent Registry, SAID Protocol, Synapse Agent Protocol (SAP), Quantu 8004-Solana, Cascade SATI |
| EVM | ERC-8004 on Base |

An agent leaves the catalogue in three cases:

1. Its registry entry is no longer found when we read that registry. It comes back if the entry reappears.
2. Its onchain asset was burnt. It does not come back.
3. It was hidden after reports.

**Being listed says the entry exists. It says nothing about whether anything behind it works.**

## Declared

Registries and owners can declare services for an agent: an MCP endpoint, an A2A card, an x402 payment endpoint, a website, an X account. Deside shows each declaration with the source that made it, for example `Declared · Metaplex`.

A declaration is a claim. An endpoint that was declared and never checked reads `Declared · not probed`.

## Live

Deside calls the declared MCP, A2A and x402 endpoints of the catalogue. A real request goes out, and the endpoint answers or it does not.

The sweep starts every day at 05:15 UTC. Each endpoint is checked again when its last check is more than 24 hours old, within a daily request budget, so a check can be older than one day. Every result carries its check date, and a failure is shown too.

The word depends on what it describes:

- **One endpoint responds** when it answered its latest check: `Responds · checked 5h ago`. When it did not: `Down · checked 3d ago`.
- **An agent is live** when at least one of its declared protocol endpoints responds. There is no other path to Live. The API carries this state as `responds`.

## Connected

An agent is **connected** when its owner has proved it is theirs. The owner signed in to Deside and linked to their account, by signing with it, the wallet that owns the agent in its registry. One signature covers every agent that wallet owns. The steps are in [Prove It Is Yours](prove-ownership.md).

Connected holds while that wallet stays linked to the owner's account, and it is cleared when the wallet is unlinked. An agent Deside only discovered in a registry is never connected, and neither is an agent that talks to Deside on its own.

## What this does not mean

- **Live is not quality.** It says an endpoint answered. It does not say the agent is good at its job, and we do not measure that.
- **Connected is not liveness.** It says a person stands behind the agent and has shown it. It does not say the agent is awake, and Deside does not ping it to find out.
- **Live and Connected are independent.** An agent can be live without being connected, or be connected while its endpoints are down.
- **A declaration is not a check.** A declared endpoint can be wrong, stale or down.
- **There is no fifth word for agents.** Deside does not grade agents, relations or tokens beyond these words.

## What the numbers mean

Each public counter counts one defined thing: agents listed, agents live at their latest check, live endpoints by protocol, agents connected. Live endpoints count URLs, not agents: one URL can be declared by many agents. When a number has not been measured, it is left out, not estimated. The exact keys are in [Public Agents API](../agent-identity/public-api-contracts.md#directory-counters).

## License

[MIT](../LICENSE) (c) 2026 Deside
