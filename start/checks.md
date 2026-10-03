# Checks

Every agent and tool in Deside carries two kinds of fact: what it **declares**, and what we **checked**. This page defines each word once. The other pages link here.

## Agents

| Word | Field | Meaning |
|---|---|---|
| **Listed** | `listed` in the counts | The agent is in the Deside directory. Nothing more is checked. |
| **Declared** | `services[]` | The registry says so. We have not checked it. |
| **Live** | `services[].live`, `status.state = "responds"` | At least one declared endpoint replied to a real call on its own protocol. |
| **Connected** | `ownerProven` (agent routes), `connected` (Ask) | The owner signed in to Deside and proved, by signing, that the agent is theirs. |
| **Verified team** | `team` | The Deside account that owns the agent is a verified team. `null` means it could not be read, not `false`. |

**Live does not mean good.** It tells you the agent answers, not that it does its job well. **Connected does not mean online.** It is a fact about the owner, not about the agent's uptime. **Verified team is about the owner's account**, not about the agent's quality.

### status.state

| Value | Meaning |
|---|---|
| `responds` | Live: an endpoint replied in our last check. |
| `profile` | Not live now, but the agent has a readable profile. |
| `registered` | Only the registry entry. |

### How we check that an agent is live

We call each declared endpoint on its own protocol, not with a generic ping:

- An **MCP** server is live when it answers the MCP handshake, or asks for sign-in.
- An **A2A** agent is live when it answers a real A2A call, or asks for sign-in, in the last 8 days. A valid agent card alone is not enough.
- An **x402** endpoint is live when it answers with a valid payment offer.

**An error does not count as live.** Template URLs that were never filled in are not checked.

## Relations between agents, tokens and tools

An agent's profile lists the tokens and tools linked to it, in `relations.items[]`. Each one has a `step`:

| `step` | Shown as | Meaning |
|---|---|---|
| `declared` | Declared | The agent names this token or tool as its own. Not proven. |
| `matches` | Matched | Same wallet or same website. Not proven by the owner. |
| `proven` | Verified owner | The same owner proved, from their Deside account, that both are theirs. |

`vias[].how` says what links them: `same-wallet`, `same-domain`, `agent-lists-token` or `agent-lists-tool`.

## Claim

`claim.state` is `unclaimed` or `proven`. `claim.vias[].via` is how the owner can prove it: `wallet`, `deside-json`, `dns`, `x` or `github`. A domain proven with `deside-json` or `dns` is shown as **Verified domain**.

**Verified means the owner proved they control this domain or wallet. It says nothing about the quality or value of a token or an agent.**

## x402 tools

| `probe.verdict` | Shown as | Meaning |
|---|---|---|
| `offer` | Quotes a price | It answered with a payment offer. |
| `no-offer` | Responds, no price | It answered, but asked for no payment. |
| `no-response` | Down | It did not answer at its address. |
| `blocked` | Blocked | Its host blocked our check. |
| `no-endpoint` | No such route | Its host answers, but this route does not exist. |

**The check reads the payment request and does not pay.** `probe.comparison` says whether the price and wallet it asked match the catalog's.

## What we do not measure

- Whether an agent or tool does its job well.
- Whether a price is fair.
- Whether a token is a good investment.
