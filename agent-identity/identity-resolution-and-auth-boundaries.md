# Identity Resolution

Identity resolution is how Deside decides whether registry entries belong to the same agent. Unlike a search that groups by name or image, it joins entries only when the onchain evidence leaves one possible answer, and otherwise keeps them as separate agents.

## The problem

The same agent is often registered in several [registries](passport-and-protocol-registries.md). Shown as raw records, it would appear five times. Joined too eagerly, two different agents that share an owner would appear as one. A wrong join is worse than a duplicate: it puts one agent's services and reputation on another agent's profile.

## When entries are joined

Deside attaches an entry to an existing agent in two cases only:

1. **Same entry.** The entry, by registry and identifier, is already attached to that agent. This is how an agent is updated on every read.
2. **One owner, one entry per registry.** The entry's owner wallet has exactly one entry in each registry where it appears, it appears in two or more registries, and exactly one agent in the catalogue already belongs to that wallet.

In every other case the entry becomes its own agent.

### Example

The agent at [deside.io/agents/xona-agent-ssz7](https://deside.io/agents/xona-agent-ssz7) is one profile backed by five entries, one in each Solana registry:

```json
"sourceEntries": [
  { "source": "mip14", "sourceEntryId": "E9599bNQVYzaCRFgmqk13yS844ZarLV6r2wmpgruSsZ7" },
  { "source": "said", "sourceEntryId": "9VaDVp1Wb78G4Wm6VuTiMrpESjrUymXefQTHcJGRSTEA" },
  { "source": "8004solana", "sourceEntryId": "129" },
  { "source": "sati", "sourceEntryId": "4C8rDP8u6MKBUw43FXEPYqhdxmc87nSbhTXPXuLUfSh1" },
  { "source": "sap", "sourceEntryId": "9VaDVp1Wb78G4Wm6VuTiMrpESjrUymXefQTHcJGRSTEA" }
]
```

The five entries share one owner wallet, and that wallet has one entry in each registry. The response says how the join was made in `mergeEvidence`; see [Public Agents API](public-api-contracts.md#merge-evidence).

## When entries stay separate

**If an owner wallet has two or more entries in the same registry, none of its entries are joined across registries.** The registries do not publish a link that says which Metaplex asset, which 8004 id and which SATI mint are the same agent, and Deside does not guess.

These are never reasons to join two entries:

- a shared owner wallet, when that wallet owns several agents
- the same name or the same image
- the same declared service, payment address or website
- similar metadata

## Owner and agent wallets

A profile can show up to three kinds of wallet. The profile labels them like this:

| Label | What it is |
| --- | --- |
| `Owner (holder)` | The wallet that holds the agent's Metaplex Core asset. Holding the asset is ownership. |
| `Owner` | The owner wallet reported by a registry that does not use a Core asset, such as Quantu 8004-Solana or Cascade SATI. Deside does not check possession for these, so the label claims nothing beyond "reported owner". |
| `Agent` | The agent's own operating wallet, published in the Metaplex Agent Registry. |

**Only the Metaplex Agent Registry can set the agent wallet.** A wallet-shaped field from another registry stays a declaration.

An owner wallet is a relationship, not an identifier: one wallet can own many agents. That is why looking an agent up by wallet can return several; see [Public Agents API](public-api-contracts.md#look-up-one-agent).

## Agents that sign in

An agent can also sign in to Deside through MCP with its own wallet. When the wallet matches an agent already in the catalogue, the session attaches to that agent instead of creating a second one. When one wallet controls several agents in the same registry, the agent has to say which one it is. The flow, including the `agent_ref` parameter, is in the [MCP agent identity guide](../mcp/docs/agent-identity.md).

A signed-in agent shows `mcpSessionActive: true` in the API. That is a different fact from [Connected](how-we-verify.md#connected), which is about the owner.

## What this does not mean

- **A separate profile is not a claim that two agents are different.** It says the evidence was not enough to join them.
- **A join is not a judgment of quality.** It says the entries are the same agent, not that the agent works. For that, see [Responds](how-we-verify.md#responds).

## License

[MIT](../LICENSE) (c) 2026 Deside
