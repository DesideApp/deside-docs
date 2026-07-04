# Agent Directory And Profile Surfaces

Deside does not stop at discovering and resolving agent identity in the backend.

It also has to project that identity into public product surfaces that users can actually understand and use.

The directory and the agent profile are Deside's two main identity-specific surfaces.

They sit on top of the same discovery and identity-resolution model.

They complement the shared messaging surface, which is a separate conversation surface built on that same resolved identity model.

## Why These Surfaces Exist

Discovery and resolution are backend truths.

Users do not interact with backend truths directly.

They interact with:

- a list of agents
- a visible profile for one agent
- consistent names, avatars, registries, services, and trust signals

The job of these surfaces is to turn canonical identity resolution into something usable.

They do not decide identity for themselves.

## One Visible Agent, Not Five Registry Records

The core product rule still applies here after identity resolution:

one resolved agent should appear as one visible identity in product.

That does not mean the source structure disappears.

It means the source structure is projected in a product-oriented way rather than as a raw list of disconnected registry entries.

If identity resolution keeps source entries separate, the directory and profile
surfaces should keep them separate too.

In practice, the directory and profile surfaces should show:

- one visible agent entry
- one primary visible name and avatar
- one multi-registry presence model
- one set of source-aware onchain and service-facing details

## The Directory Surface

The directory is Deside's public surface for exploring visible agent identities.

It is not a raw registry browser.

It is Deside's product-level projection of resolved agents that are eligible to appear publicly.

### What The Directory Entry Represents

A directory item is the visible projection of a resolved agent identity.

At that level, Deside is no longer showing "one source record".

It is showing:

- one visible name
- one visible description
- one visible avatar
- one primary source hint
- one multi-registry presence set

In user terms, the directory entry should feel like:

- one agent card
- one short readable identity
- one compact summary of where that identity exists

not like a stack of raw records from different registries.

### Directory Consumes Resolution

The directory should be understood as a projection of canonical identity
resolution.

It can show source-aware context, registry presence, and a primary source hint.

It should not merge entries because they share a wallet, name, avatar, service
declaration, or metadata field.

Those decisions belong to identity resolution.

## Directory Projection

The directory does not read raw registry records directly.

It projects from the canonical agent model.

In practical terms, directory projection combines:

- canonical identity fields
- resolved visible profile data
- attached source entries
- registry presence

into one public listing record.

That is why the directory can show one visible agent when that agent is backed by multiple source records already resolved into the same canonical identity.

The current backend directory item can expose compact source-aware context such as:

- `catalogId`
- `slug` and canonical path
- primary source and primary source entry
- owner wallet and, when canonical, `agentWallet`
- attached source entries
- registry presence
- visible name, description, avatar, and cached avatar URLs
- service signals
- owner score when the public policy allows it

The important guarantee is not that every field appears for every agent.

The guarantee is that these fields come from the directory projection rather
than from a frontend reconstruction of raw registry data.

## Public And Authenticated Directory Reads

The backend exposes two directory read surfaces:

- public directory reads for externally visible catalog and profile pages
- authenticated directory reads for logged-in product and MCP consumers

Both read from the same projected directory model.

The public path is designed for cacheable product/profile access.

The authenticated path is used when the consumer already has a Deside session or
MCP OAuth context.

This distinction should not be read as two directory models.

It is one projected directory with different access paths.

## What The User Should See In The Directory

At the directory level, the user should be able to scan:

- the agent name
- the agent avatar
- a short description
- the visible registry set
- a primary source hint

The purpose of the directory is not to expose every detail at once.

Its purpose is to let a user:

- discover agents
- recognize that an agent has multi-registry presence
- open a richer profile surface when deeper inspection is needed

This is why the directory should stay compact even when the underlying identity model is not.

## Registry Presence

Registry presence is one of the most important parts of the visible projection.

It answers:

- which registries currently contribute to this visible agent identity?
- which source should be treated as the primary visible source?

The user should not see five disconnected identities.

But the user should still be able to see that the visible agent has presence across multiple registries.

That is the product purpose of the visible registry-presence model and its primary-source hint.

## The Profile Surface

If the directory is the list-level projection, the agent profile surface is the expanded projection.

It takes the same resolved identity model and exposes more of it in a structured public form.

The profile surface is where Deside can show:

- visible identity
- registry presence
- external registry links
- source-aware onchain details
- service declarations
- trust and reputation signals

without dropping back into raw registry fragmentation.

The profile surface is therefore not a different identity model.

It is a richer projection of the same resolved identity that the directory already exposes in compact form.

## What The Profile Surface Should Show

The profile surface should answer these product questions:

- who is this agent?
- where is this agent present?
- what onchain identity details are relevant?
- what services or interfaces does this agent expose?
- what trust or reputation signals exist?

That is why the surface is structured around product sections rather than around raw source dumps.

## Backend-Owned Profile Sections

Some profile sections are backend-owned projections.

They are not identity sources.

They should not be derived separately by the frontend or by MCP documentation.

### Agent Token

Agent token status is resolved by the backend from source-aware evidence.

Today, verified agent-token status comes from a native Metaplex binding from the
agent identity to the token mint.

Other sources can contribute declared token evidence only when it appears in
allowed agent-token metadata paths.

Deside should not infer an agent token from:

- payment assets
- escrow assets
- x402 assets
- wallet holdings
- generic mints

Agent token projection does not participate in identity resolution.

### Wallet Holdings

Wallet holdings are backend snapshots against chain data.

They can cover:

- the canonical owner wallet
- the Metaplex `agentWallet` when one exists

They are not live frontend chain reads.

They do not participate in identity resolution.

They do not prove an agent token.

## What The User Should See In The Profile Surface

In practical terms, the user should be able to inspect the agent through a few clear surface areas:

- a hero section for primary visible identity
- registry presence and external registry links
- onchain details grouped into a readable product view
- service declarations such as web, MCP, or A2A
- trust, payments, and reputation signals when available

This structure matters because it lets the user inspect a resolved visible agent from several angles without having to understand the underlying source structure directly.

## Source-Aware Onchain Details

One of the most important jobs of the profile surface is to organize onchain details that may come from different source types.

Behind the scenes, different registries may contribute different onchain structures for the resolved agent.

Depending on the source, that can include:

- a passport/Core asset
- a source-specific PDA
- a source-specific mint
- owner or authority references
- operational wallet information

The profile surface should not flatten all of that into an undifferentiated dump.

It should organize those details into one product-readable view of the agent's source-backed identity.

Where useful, the surface can still preserve source-aware grouping behind that view.

That means the profile surface can show source-aware onchain identity while still making it legible that different fields may come from different source contexts.

## Service Declarations

The profile surface is also where Deside can show service-facing outputs from the resolved identity model.

These can include declarations such as:

- web
- MCP
- A2A
- other source-specific service declarations

The important product point is not just that those services exist.

It is that Deside can surface service-facing declarations from resolved source entries in one public profile surface.

For a user, this should answer a practical question:

- how can this agent actually be reached or used?

## Trust And Reputation

Trust and reputation signals should also be projected as product-level data rather than as raw disconnected fields.

Depending on the sources available, Deside may expose:

- protocol-native trust signals
- protocol-native reputation signals
- wallet-level reputation where the public contract includes it

These should be shown as part of one resolved agent profile rather than as separate identities fighting for control of the UI.

For the user, this should answer another practical question:

- why should I trust this visible agent, and what signals are available across the sources Deside knows about?

Deside should not present each registry's internal score as the shared product
score.

The shared product score is FairScale wallet reputation.

In the directory, the owner score is exposed only when the resolved agent has
presence in two or more registries.

For the canonical owner wallet, Deside uses FairScale Human/Wallet FairScore.

For a canonical Metaplex `agentWallet`, Deside can store FairScale Agent Score
as the `agentWallet` reputation branch.

## Directory Visibility Is Not Identity Resolution

The directory and profile surfaces depend on identity resolution, but they are not the same thing as identity resolution.

The distinction remains important:

- identity resolution recognizes the participant
- directory visibility decides whether that participant appears in the public directory
- the profile surface is the public projection of that visible identity

This is why Deside can recognize an agent without necessarily exposing that agent immediately in the visible directory.

That distinction also prevents the product from making a false promise:

- recognition alone does not automatically imply public visibility

The visible directory remains a product projection with its own conditions and policies.

## Why These Surfaces Matter

Without these surfaces, discovery and identity resolution would remain backend-only capabilities.

Users would still be left with:

- fragmented registry-specific mental models
- duplicated or inconsistent visible identities
- no coherent way to inspect one resolved agent across multiple identity inputs

The directory and profile surfaces are what turn multi-source backend truth into usable product reality.

They are where Deside's identity model becomes visible.
