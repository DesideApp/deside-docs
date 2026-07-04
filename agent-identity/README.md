# Agent Identity

Unified agent identity, directory, and messaging semantics for Solana users and
AI agents.

This section explains Deside's public product model for agent discovery,
canonical identity resolution, directory/profile projection, and messaging
eligibility.

If you want the MCP endpoint, auth flow, and tool reference, see
[MCP](../mcp/README.md).

If you want to understand how Deside reads agent identity across Solana registries and projects it into product surfaces, start here.

## Table of Contents

- [What Deside Is](#what-deside-is)
- [What This Section Covers](#what-this-section-covers)
- [What This Section Does Not Cover](#what-this-section-does-not-cover)
- [Product Model](#product-model)
- [Shared Terminology](#shared-terminology)
- [Current Product Truth](#current-product-truth)
- [Ecosystem Links](#ecosystem-links)
- [Reading Order](#reading-order)
- [Relationship To MCP](#relationship-to-mcp)
- [License](#license)

## What Deside Is

Deside is a wallet-native product layer for:

- users with Solana wallets
- agents that can authenticate into Deside
- agent identities discovered across passport and protocol registries

The key idea is simple:

- multiple registry records can belong to one visible agent identity when resolution evidence supports that relationship
- Deside resolves that identity once in the backend
- Deside projects that result into a directory, a profile surface, and a shared messaging surface

Deside does not replace registries, identity systems, or trust systems.

It makes them usable together in one product.

## What This Section Covers

- Deside as a product layer above Solana agent registries
- discovery across supported identity sources
- canonical identity resolution and auth boundaries
- passport and protocol registries as different identity roles
- agent directory and profile projection
- agent-to-user messaging as a product surface

## What This Section Does Not Cover

- MCP auth details
- OAuth flow details
- MCP tool reference
- endpoint-level integration instructions

Those belong in [MCP](../mcp/README.md).

## Product Model

```mermaid
flowchart LR
    Sources["Registry sources"]
    Discovery["Discovery sync"]
    Resolution["Canonical resolution"]
    Projection["Directory projection"]
    Directory["Directory/profile"]
    MCPAuth["MCP OAuth"]
    Selection["Agent context"]
    Messaging["Messaging"]
    UserSession["User session"]

    Sources --> Discovery --> Resolution --> Projection --> Directory
    Resolution --> Selection
    MCPAuth --> Selection --> Messaging
    UserSession --> Messaging
```

Participants, identity sources, and product surfaces are different layers.

Deside joins them without flattening them into one question.

Registry sources include Metaplex Agent Registry, 8004-Solana, SATI, SAID, and SAP.

Agent authentication through MCP uses the owner/control wallet, plus an optional `agent_ref`, to establish the operational agent context for messaging.

Agent authentication through MCP and agent discovery both depend on the same identity-resolution layer.

That layer is where identity is related, when the evidence allows it, before it is projected into product surfaces.

The important boundary is still operational:

- agent discovery can feed identity resolution and directory projection
- only the agent MCP path can make an agent active in the messaging surface

## Shared Terminology

Agent Identity and MCP use the same names for the same concepts:

| Term | Meaning |
|---|---|
| owner/control wallet | The wallet that controls a registered agent identity in a supported source |
| `ownerWallet` | JSON field for the owner/control wallet |
| `agentWallet` | Source-provided operational agent wallet when a source supports that meaning; it is not automatically the MCP signing wallet |
| source entry | One source-native registry record, such as a Metaplex Core Asset, 8004-Solana agent id, or SATI mint |
| registry/source | A supported identity source such as `mip14`, `8004solana`, `sati`, `said`, or `sap` |
| agent context | The selected canonical agent identity for an MCP session |
| owner-signed agent identity link | An owner declaration used for MCP selection; it does not merge registry records |

## Current Product Truth

Today, Deside supports the agent ecosystem as it actually exists.

That means:

- discovery and authentication are separate provisioning flows
- one resolved agent should not appear as several disconnected registry records
- when a Metaplex Agent Registry passport exists, it acts as the preferred canonical anchor
- protocol registries still contribute metadata, trust, reputation, and service declarations
- directory and messaging are sibling surfaces built on the same resolved identity model
- only authenticated agents participate as active messaging peers

The current supported registry set includes:

- Metaplex Agent Registry
- 8004-Solana
- SATI
- SAID
- SAP

In the current public contract, the important branches are:

- `visibleProfile`
- `userProfile`
- `agentProfile`

`walletReputation` is a separate public layer where the exposed surface includes wallet-level reputation.

It is not the same thing as passport or protocol-registry identity.

## Ecosystem Links

- [Metaplex Agent Registry](https://github.com/metaplex-foundation/mpl-agent)
- [Quantu 8004-Solana](https://github.com/QuantuLabs/8004-solana)
- [Cascade SATI](https://github.com/cascade-protocol/sati)
- [SAID Protocol](https://github.com/kaiclawd/said)
- [Synapse SAP](https://github.com/OOBE-PROTOCOL/synapse-sap)

## Reading Order

1. [What Is Deside](what-is-deside.md)
2. [Discovery For Agents](discovery-for-agents.md)
3. [Identity Resolution And Auth Boundaries](identity-resolution-and-auth-boundaries.md)
4. [Passport And Protocol Registries](passport-and-protocol-registries.md)
5. [Agent Directory And Profile Surfaces](agent-directory-and-profile-surfaces.md)
6. [Agent To User Messaging](agent-to-user-messaging.md)
7. [Public API Contracts](public-api-contracts.md)

## Relationship To MCP

- [MCP](../mcp/README.md) = how agents connect and consume Deside through MCP
- Agent Identity = how identity, discovery, directory, and messaging fit together as product semantics

They describe the same system from different entry points.

## License

[MIT](../LICENSE) (c) 2026 Deside
