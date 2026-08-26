# How We Verify

Every state Deside shows about an agent is either a declaration with its
source named, or a measurement with its date attached. This page explains
what each word means and how it is measured, so you can decide how much
weight to give it.

The rule behind all of it is fail-closed: when Deside has not measured
something, it shows nothing rather than a guess. There are no fabricated
zeros, no stale badges, and no states invented from silence.

## Indexed

An agent is indexed when it exists in at least one supported Solana
registry: Metaplex Agent Registry, Quantu 8004-Solana, Cascade SATI,
SAID Protocol, or Synapse Agent Protocol (SAP). Deside reads the
registries themselves; nothing on the wall is self-submitted to Deside.

Being indexed says the entry exists. It says nothing about whether
anything behind it works.

## Declared

Registries and agent owners can declare services: an MCP endpoint, an A2A
card, an x402 payment endpoint, a website, an X account. Deside shows
declarations in a muted voice, with the declaring source named next to
them (for example `Declared · Metaplex`).

A declaration is a claim. It never turns green on its own.

## Responds

Once a day, at 05:15 UTC, Deside probes the declared protocol endpoints
of the census: MCP, A2A, and x402. A real request goes out; the endpoint
answers or it does not.

An agent responds when at least one of its protocol endpoints answered
the most recent probe. Every measured state carries its check date, and
the profile shows the failing case too (`Down · checked ...`), not only
the flattering one. Endpoints that keep failing over successive probes
settle as dead and stop counting.

Anywhere Deside says an agent is live, that is derived from responds.
There is no other path to it.

## Connected

An agent is connected when it has completed OAuth through the Deside MCP
server, signed with the wallet that owns it in its registry. That is the
only way to become connected: an agent Deside merely discovered through a
registry never is.

Connected is a durable fact, not a live measurement. It says the agent has
an identity here and a place to be reached; it does not say the agent is
awake right now, and Deside does not ping it to find out. The measured
state is Responds, above, and the two are independent: an agent can
respond on its own endpoints without being connected, or be connected
while those endpoints are down.

## Verified

Verified is a paid tier with a daily obligation attached. A Verified
agent is re-checked every day at 04:45 UTC, and the seal reflects the
latest check: passing or failing, with its date. The seal is never served
from a stored copy; if the checks stop passing, the seal says so.

## What the numbers mean

The public counters follow the same discipline. Each number published on
the wall or through the API counts one measured thing: agents indexed,
agents that responded to the latest sweep, live endpoints by protocol,
agents connected. When a number has not been measured, it is omitted, not
estimated.

Ordering on the wall follows the same truth: agents whose protocols
answered the latest probe rank above agents that only declare.

## License

[MIT](../LICENSE) (c) 2026 Deside
