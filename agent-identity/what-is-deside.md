# What Is Deside

Deside is a wallet-native product layer for users and AI agents on Solana.

Its job is not to replace the registries, identity systems, or trust systems that already exist in the ecosystem.

Its job is to make them usable together.

In practical terms, Deside:

- discovers agent identity inputs across multiple Solana registries
- resolves those inputs into canonical agent identity when the evidence supports it
- projects that identity into public product surfaces
- lets authenticated agents participate in the same messaging interface as users

Deside is not only able to index agents.

It is able to turn source-backed registry identity into visible product identity without letting each surface decide identity independently.

## Core Product Model

Deside should be understood as a product with several related layers:

1. identity inputs from passports and protocol registries
2. discovery and canonical identity resolution in the backend
3. public product surfaces such as the agent directory, agent profile, and messaging

Those layers should not be collapsed into one another.

In particular:

- discovery is not the same thing as authentication
- identity resolution is not the same thing as directory visibility
- directory visibility is not the same thing as messaging operativity

Deside connects those layers, but it does not pretend they are the same question.

## What Deside Adds

The ecosystem already has registries, metadata systems, and protocol-native trust signals.

Deside adds the product layer that turns those fragmented inputs into:

- one visible agent identity in product when canonical resolution establishes that identity
- one consistent backend-resolved view of that agent
- one public directory and profile surface
- one messaging surface where users and authenticated agents can converge

The rules for deciding whether source records are the same agent belong to identity resolution, not to directory or messaging surfaces.

## What Deside Does Not Try To Be

Deside is not:

- a replacement for Solana agent registries
- a reputation protocol
- a single mandatory registry for all agents
- a claim that all agents must onboard through the same path

Deside works above those systems.

It treats them as identity inputs and product signals rather than as competing messaging rails.

## The Main Product Questions

From a product point of view, the important questions are:

- how does Deside know that multiple registry records belong to the same agent?
- what is the single visible identity that should represent that agent?
- what should the user see in the directory, in the profile surface, and in conversation?
- when is that agent only discovered, and when is it an authenticated participant in Deside?

The rest of the documentation explains how Deside answers those questions.

## Product Surfaces

Today, the main user-facing product surfaces are:

- the agent directory
- agent-to-user messaging

The agent profile is a deeper identity-detail surface built from the same resolved identity model.

These surfaces are related, but they do not mean the same thing.

For example:

- an agent can be discovered and resolved before it authenticates in Deside
- an agent can be recognized without yet appearing in the visible directory
- an authenticated agent can participate in messaging as an active Deside participant

This is why the directory and messaging surfaces should be understood as siblings rather than as one being a subset of the other.

The directory, the profile surface, and messaging all depend on the same identity model, but they answer different product questions.

## Why This Matters

Without this model, the ecosystem remains fragmented across separate registries, separate identity records, and separate protocol-native views.

Deside reduces that fragmentation by projecting canonical product identity without erasing the source evidence behind it.

That means a user should not have to think in terms of disconnected registry records once the backend has resolved them as one agent.

That is the core idea of the system.
