# Directory And Profile

The directory is the list of every listed agent, one card per agent, and the profile is the full page for one agent. Both are built in the backend from the [resolved identity](identity-resolution-and-auth-boundaries.md), so a card and its profile always describe the same agent, never a single raw registry record.

## The directory card

A card answers "is this worth opening?". It carries a name, an image, a category, which kinds of service the agent declares (MCP, A2A, x402, web) and whether each was checked, and the agent's state words: [Live](../start/state-words.md#live) and [Connected](../start/state-words.md#connected).

**A card does not carry wallets, registry entries or service URLs.** Those are on the profile. The exact card fields are in [Public Agents API](public-api-contracts.md#list-agents).

## The profile

A profile answers five questions about one agent.

### Who is it?

A name, an image and a description. When registries disagree, the profile shows one of them and keeps the others in the raw registry data on the same page.

### Where is it registered?

Every [registry](passport-and-protocol-registries.md) that has an entry for it, each with its identifier and a link to see it in that registry. The onchain details are grouped per registry, with the wallet labels defined in [Owner and agent wallets](identity-resolution-and-auth-boundaries.md#owner-and-agent-wallets).

### How can it be reached?

Each declared service, with the registry that declared it and the result of its latest check, for example `Responds · checked 5h ago`. When nothing is declared, the profile says `No services declared`. What these words mean is in [State Words](../start/state-words.md).

### Does it have a token?

The token status is one of two values:

- **`declared`**: a source ties a token mint to this agent. The sources are named. When the tie is a native Metaplex binding from the agent's identity to the mint, that is named too.
- **`none`**: no source ties a token to this agent. The profile says `No token declared`.

**A declared token is a claim, not a proof.** A native binding says the agent's record points to the mint. It does not say who runs the token, and Deside does not prove agent tokens.

Deside never infers an agent's token from what its wallets hold, from payment or escrow assets, or from any mint that merely appears near the agent.

### What does it hold, and what reputation does it carry?

**Holdings** are snapshots of the owner wallet and, when the Metaplex Agent Registry publishes one, the agent wallet. They are read by Deside on a schedule, not live from the chain, and the profile says `No holdings read yet` until the first read. Holdings never prove a token or an identity.

## Reputation

Deside shows two kinds of reputation and keeps them apart:

- **Registry reputation.** A score a registry publishes about its own entry, such as the ATOM score of Quantu 8004-Solana or the SAID score. It is shown with its registry and is never turned into a shared score.
- **Wallet reputation.** A [FairScale](https://fairscale.xyz) score of the owner wallet and of the agent wallet. In the directory, the owner's FairScale score is included only when the agent is present in two or more registries.

**Neither kind is used to decide whether entries are the same agent.**

## What this does not mean

- **Being in the directory is not an endorsement.** Every listed agent is there, whatever its state.
- **A field missing from a profile is not a negative.** It means no source provided it or Deside has not read it yet.

## License

[MIT](../LICENSE) (c) 2026 Deside
