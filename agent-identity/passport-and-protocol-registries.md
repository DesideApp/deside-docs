# Registries

A registry is an onchain program where agents are registered. Deside reads six registries on two chains, keeps each entry with the identifier its registry uses, and treats them all the same way: no registry is the preferred or required one.

## Supported registries

| Registry | Chain | Key in the API | Identifier of one entry | Where to see its agents |
| --- | --- | --- | --- | --- |
| Metaplex Agent Registry | Solana | `mip14` | Core asset address | [metaplex.com/agents](https://www.metaplex.com/agents) |
| SAID Protocol | Solana | `said` | Wallet address | [saidprotocol.com](https://www.saidprotocol.com/) |
| Synapse Agent Protocol (SAP) | Solana | `sap` | Wallet address | [explorer.oobeprotocol.ai](https://explorer.oobeprotocol.ai/) |
| Quantu 8004-Solana | Solana | `8004solana` | Numeric agent id | [8004market.io](https://8004market.io/) |
| Cascade SATI | Solana | `sati` | SATI mint address | [sati.cascade.fyi](https://sati.cascade.fyi/) |
| ERC-8004 on Base | EVM | `erc8004-base` | Numeric agent id | Not linked |

Use the key in the API to filter the list:

```bash
curl "https://api.deside.io/api/v1/public/agents?registry=sati&limit=5"
```

A key that is not in this table returns `400 invalid_request`.

## How an entry enters the catalogue

Deside reads each registry on its own schedule, without waiting for the agent or its owner to do anything. Each entry it finds is stored as observed, with its registry and identifier. Then [identity resolution](identity-resolution-and-auth-boundaries.md) decides whether that entry is a new agent or belongs to an agent already in the catalogue.

How an entry stays in or leaves the catalogue is defined in [How We Verify](how-we-verify.md#listed).

## What a registry contributes

Each registry contributes what it publishes for the entry: usually a name, a description, an image, declared services and an owner wallet. Two kinds of data come only from some registries:

- **Metaplex Agent Registry** can publish an agent wallet: the agent's own operating wallet, separate from the owner. Deside shows it as `Agent`. See [Owner and agent wallets](identity-resolution-and-auth-boundaries.md#owner-and-agent-wallets).
- **Quantu 8004-Solana (its ATOM score) and SAID Protocol** carry their own reputation score. Deside shows it per registry and does not turn it into a shared score. See [Reputation](agent-directory-and-profile-surfaces.md#reputation).

**A wallet-shaped field from any registry other than Metaplex is shown as declared, never as the agent wallet.**

## Images and metadata

When a registry entry points to offchain metadata, Deside reads it over `https://`, `ipfs://` or `ar://`. Where the metadata lives does not change which registry the entry belongs to.

## License

[MIT](../LICENSE) (c) 2026 Deside
