# Passport And Protocol Registries

Deside does not treat all agent identity inputs as if they answered the same question.

Some inputs are stronger as canonical identity anchors.

Others are better understood as protocol registries that add metadata, trust signals, reputation, or service declarations around that identity.

That distinction matters.

## Why Deside Separates Passport From Protocol Registries

The product should not flatten every source into one undifferentiated list.

If every source is treated as semantically identical, Deside loses an important part of its model:

- some sources are better starting points for canonical onchain identity
- some sources are better at enrichment around that identity

Today, Deside models that distinction as:

- one passport anchor per resolved agent when available
- multiple protocol registries and enrichment sources around it

## Passport

In Deside, a passport is the strongest available starting point for canonical onchain agent identity.

When a passport exists, Deside should prefer it as the base anchor for the resolved identity.

This does not mean that the passport is the only source that matters.

It means that the passport is the best place to start when deciding what
onchain identity should anchor the agent: the product prefers strong
onchain anchors over weaker local heuristics whenever they exist.

### Metaplex Agent Registry

Official reference: [metaplex-foundation/mpl-agent](https://github.com/metaplex-foundation/mpl-agent)

In the current model, Metaplex Agent Registry is treated as the passport anchor.

Its role is:

- canonical passport anchor
- minimal onchain identity base
- strong product starting point for canonical agent identity
- source of the canonical operational `agentWallet` when the registry exposes one

Its limit is also important:

- it is not, by itself, a full reputation system
- it is not the complete product profile for the agent

So this should not be read as passport-only.

## Protocol Registries

Protocol registries are not meaningless secondary copies.

They are important identity and enrichment inputs.

They can add:

- protocol-native metadata
- protocol-native trust signals
- reputation models
- service declarations
- source-specific onchain references

Deside should preserve those signals without letting them fragment the visible product identity.

In product terms, the ideal outcome is not to force an agent to choose exactly one registry forever.

The better outcome is:

- a resolved agent can register across multiple protocol registries
- those records can preserve source-specific structure
- Deside can project them through one visible identity when identity resolution establishes that relationship

The rules for that relationship belong to identity resolution.

## Quantu 8004-Solana

Official reference: [Quantu 8004-Solana](https://github.com/QuantuLabs/8004-solana)

Role in Deside:

- protocol registry
- identity input
- protocol-native reputation and metadata source

Strength:

- ATOM reputation
- Core-based identity model
- richer registry-native metadata than a minimal passport anchor

Limit:

- not the universal canonical passport by itself

## Cascade SATI

Official reference: [cascade-protocol/sati](https://github.com/cascade-protocol/sati)

Role in Deside:

- protocol registry
- identity input
- structured trust and registration source

Strength:

- SATI-native trust signals and attestations when available
- structured protocol-native registration model

Limit:

- separate identity substrate from the passport model

## SAID Protocol

Official reference: [kaiclawd/said](https://github.com/kaiclawd/said)

Role in Deside:

- protocol registry
- identity input
- protocol-native identity and reputation source

Strength:

- independent source of identity and reputation data
- source-specific metadata and registration signals

Limit:

- separate from the passport model

## SAP

Official reference: [Synapse SAP](https://github.com/OOBE-PROTOCOL/synapse-sap)

Role in Deside:

- protocol registry
- identity input
- additional source-specific registration surface

The relevant product point is that SAP can contribute source-specific
registration evidence and registry presence to an already resolved product
identity.

Source-specific object details belong behind the adapter and resolver contract.

## How Deside Reads Multi-Registry Identity

Different sources expose different onchain structures behind a canonical
agent identity: a Core asset, a source-specific PDA, a source-specific
mint, owner or authority references. That variability is one of the
reasons the model exists, and no registry object automatically replaces
another.

Deside is source-aware without being source-captured: it extracts entries
from multiple registries, preserves source-specific evidence, lets
identity resolution decide canonical attachment, and projects the
resolved agent in product. Cross-source identity rules, including
one-to-one owner correspondence and owner collections, are defined in
[Identity Resolution And Auth Boundaries](identity-resolution-and-auth-boundaries.md).

## Agent Wallet Is Not A Generic Self-Declared Field

Some registry records can contain wallet-like fields, service endpoints, or
operational hints.

Deside should not treat all of those as equivalent.

The current backend contract is stricter:

- `ownerWallet` is not a canonical operational `agentWallet`
- Metaplex can contribute a canonical `agentWallet`
- source-specific wallet hints from protocol registries can remain evidence or
  metadata
- those hints should not automatically overwrite the canonical `agentWallet`

This matters for authentication, messaging, and profile display.

If Deside says an agent has an operational `agentWallet`, that should come from
a source path the backend treats as canonical, not from an arbitrary service
field.

This is why the operational agent wallet is exclusive to the Metaplex Agent
Registry: wallet-like fields from other protocol registries stay as declared
evidence, never as the canonical `agentWallet`. Deside labels an
owner-shaped value from a non-Metaplex registry as plain `Owner` (with no
possession claim: across registries "owner" can be the collection authority
rather than the real owner — see Identity Resolution for those cases),
reserving `Agent` for the canonical operational agent wallet. See [Identity Resolution And Auth Boundaries](identity-resolution-and-auth-boundaries.md)
for the full label set.

## Wallet-Level Reputation Is Separate

Wallet-level reputation is not the same thing as passport or protocol-registry identity.

It is a separate layer.

When exposed by the public surface, wallet-level reputation can apply to:

- user wallets
- agent wallets

without being the same thing as registry identity.

### FairScale

In the current public model, FairScale is treated as a wallet-level reputation input rather than a passport anchor or protocol registry.

That distinction should remain explicit.

Deside does not expose protocol-internal registry scores as the cross-registry
product score.

When exposed, the product-level score comes from FairScale as wallet-level
reputation.

## Metadata Delivery Is Also Separate

Identity source and metadata transport are different concerns.

When supported sources expose offchain metadata, Deside can consume public metadata and images served over:

- `https://`
- `ipfs://`
- `ar://`
- public gateway-backed delivery

This is an implementation detail around delivery, not the same thing as choosing the canonical identity source.

## Product Rule

The product rule remains simple even if the source structure is not:

- one visible agent identity for each resolved agent in product
- one canonical starting point when a passport exists, without requiring
  one (no forced registry monoculture)
- protocol-native enrichment preserved around that identity
- cross-source projection only after identity resolution decides the
  records belong to the same agent
