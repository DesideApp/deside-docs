# How To Read A Relation

A **relation** is a link Deside shows between two records of different kinds, such as an agent and a token. Unlike a registry link, every relation carries one of three steps that says how far it is backed: Declared, Matched or Verified owner.

Anyone can write anything into a registry entry or a token's metadata. A token can name a famous agent, and an agent can name a famous token. If Deside showed every such link as a fact, the loudest claim would win. So each relation says what backs it, and only an owner's proof turns it into a fact.

## What gets related

Relations connect three kinds of record:

| Kind | What it is |
| --- | --- |
| Agent | An agent from the [Agent Directory](../api/public-agents.md). |
| x402 tool | A paid endpoint from the [x402 Tool Directory](../api/x402-tools.md). |
| Token | A token, identified by its chain and address. |

Records can also name X and GitHub accounts. Those names are shown, but they never make two records match. See [Why a shared X account is not enough](#why-a-shared-x-account-is-not-enough).

Today relations are shown on an agent's profile, in the Related section, and on a token's page, under Behind this token.

## The three steps

### Declared

**Declared means one side names the other.** An agent's registry entry names a token or an x402 tool as its own. Nobody has checked it: anyone could have written it.

Example: the registry entry of the agent `mizuki-the-mech` names the token MIZUKI as its own. The token page shows that agent as `Declared`, with the via `This agent lists this token as its own`.

### Matched

**Matched means both sides point to the same identifying datum on their own.** The datum is one of two kinds:

- the same wallet, for example the owner wallet of an agent and the wallet that created a token
- the same domain, for example the website of an agent and the host of an x402 tool

Each side must bring its own datum. A registry entry that names a token is a declaration, not a match, however many times it is repeated.

Example: the agent `mizuki-the-mech-cmeh` has owner wallet `638V…CmeH`, and the token MIZUKI was created by the same wallet. The relation reads `Matched`, with the via `Created by this agent's owner`.

### Verified owner

**Verified owner means the same Deside account has proven both sides.** The owner signed in to Deside and showed, from their account, that each object is theirs. How to do that is in [Prove It Is Yours](prove-ownership.md).

When two different accounts have proven the two sides, the relation is marked as a conflict instead, and carries `conflict: true`. Deside never settles a conflict silently.

## How a relation is connected

Each relation lists one or more vias, the facts that connect it:

| Via | Shown as | Meaning |
| --- | --- | --- |
| Same wallet | `Created by this agent's owner` | Both sides carry this wallet. |
| Same domain | `Same website as this agent` | Both sides carry this domain. |
| Agent names it | `This agent lists this token as its own`, `This agent lists this tool as its own` | The agent's registry entry names the token or tool as its own. |

A relation can have several vias. The relation in the Matched example above has two: the same wallet, and the agent's registry entry naming the token.

## When a shared wallet or domain does not count

A wallet or domain does not make a match in two cases:

1. **It is shared by many owners.** A wallet or domain behind five or more different owners is treated as a platform, such as a launchpad's wallet or a hosting domain. So is a single wallet that owns more than 50 agents or tokens, and so are widely used domains. Sharing it says nothing about who is behind either side.
2. **Both sides got it from the same source.** If one third party wrote the datum on both sides, it is one declaration repeated, not two.

## Why a shared X account is not enough

An X account can be named by anyone. A token's metadata can name `@solana` without Solana knowing. If two records naming the same X account were shown as a match, any token could match any company by typing its handle.

**Two records that name the same X or GitHub account stay Declared.** Only a shared wallet or a shared domain makes a match.

## Who is behind a token

A token is not a product of Deside: it is one of the three kinds of record, and its page answers one question, who is behind it. The answer is the list of its relations, under Behind this token, each with its step. The summary line names the strongest of them.

On 2026-10-01 the token MIZUKI read `Mizuki the Mech · registered agent · X declared`, with step Matched. It had two relations to agents:

| Agent | Step | Vias |
| --- | --- | --- |
| `mizuki-the-mech-cmeh` | Matched | `Created by this agent's owner`, `This agent lists this token as its own` |
| `mizuki-the-mech` | Declared | `This agent lists this token as its own` |

Only the first agent shares a wallet with the token. The second only names the token in its registry entry, so it stays Declared.

## What this does not mean

- **Matched is not Verified owner.** It says two records share a wallet or a domain. It does not say who controls them.
- **Verified owner is the highest step.** No relation has a step above it.
- **Declared is not false.** A declaration can be true. It is shown as what it is: one side's word.
- **A missing relation is not a negative.** It means no record of the other kind shares a wallet or domain with this one, or that we have not read it yet.

On the API, read them from `relations` on an agent's profile. The fields are in [Relation Fields](../api/relations.md).

## License

[MIT](../LICENSE) (c) 2026 Deside
