# How To Read A Relation

A **relation** is a link Deside shows between two records of different kinds, such as an agent and a token. Unlike a registry link, every relation carries one of three steps that says how far it is backed: Declared, Matches or Proven.

Anyone can write anything into a registry entry or a token's metadata. A token can name a famous agent, and an agent can name a famous token. If Deside showed every such link as a fact, the loudest claim would win. So each relation says what backs it, and only an owner's proof turns it into a fact.

## What gets related

Deside keeps one record per object, in four kinds:

| Kind | What it is |
| --- | --- |
| Agent | An agent from the catalogue. See [How We Verify](how-we-verify.md#listed). |
| x402 tool | A paid endpoint from the x402 tool catalogue. |
| Token | A token, identified by its chain and address. |
| Domain | A web domain that an agent, tool or token points to. |

Records can also name X and GitHub accounts. Those names are shown, but they never make two records match. See [Why a shared X account is not enough](#why-a-shared-x-account-is-not-enough).

Today relations are shown on an agent's profile, in the Related section, and on a token's page, under Behind this token.

## The three steps

### Declared

**Declared means one side names the other.** An agent's registry entry names a token, or a token's metadata names an agent. Nobody has checked it: anyone could have written it.

Example: a token's metadata points to the agent Mizuki the Mech. The token page shows that agent as `Declared`, with the via `This token names this agent`.

### Matches

**Matches means both sides point to the same identifying datum on their own.** The datum is one of two kinds:

- the same wallet, for example the owner wallet of an agent and the wallet that created a token
- the same domain, for example the website of an agent and the host of an x402 tool

Each side must bring its own datum. A token that names an agent is a declaration, not a match, however many times it is repeated.

Example: the agent `mizuki-the-mech-cmeh` has owner wallet `638V…CmeH`, and the token MIZUKI was created by the same wallet. The relation reads `Matches`, with the via `Same wallet 638V…CmeH`.

### Proven

**Proven means the same Deside account has proven both sides.** The owner signed in to Deside and showed, from their account, that each object is theirs. How to do that is in [Prove An Agent Or Token Is Yours](prove-ownership.md).

When two different accounts have proven the two sides, the relation is marked as a conflict instead. Deside never settles a conflict silently.

## How a relation is connected

Each relation lists one or more vias, the facts that connect it:

| Via | Shown as | Meaning |
| --- | --- | --- |
| Same wallet | `Same wallet 638V…CmeH` | Both sides carry this wallet. |
| Same domain | `Same domain example.com` | Both sides carry this domain. |
| Names it | `This token names this agent`, `Agent names it` | One side names the other. |

A relation can have several vias. The relation in the Matches example above has two: the same wallet, and the token naming the agent.

## When a shared wallet or domain does not count

A wallet or domain does not make a match in two cases:

1. **It is shared by many objects.** A wallet or domain that five or more objects share is treated as a platform, such as a launchpad's wallet or a hosting domain. Widely used domains are treated the same way. Sharing it says nothing about who is behind either side.
2. **Both sides got it from the same source.** If one third party wrote the datum on both sides, it is one declaration repeated, not two.

## Why a shared X account is not enough

An X account can be named by anyone. A token's metadata can name `@solana` without Solana knowing. If two records naming the same X account were shown as a match, any token could match any company by typing its handle.

**Two records that name the same X or GitHub account stay Declared.** Only a shared wallet or a shared domain makes a match.

## What this does not mean

- **Matches is not Proven.** It says two records share a wallet or a domain. It does not say who controls them.
- **No relation is Verified.** Verified is a state of a Deside account that has proven a web domain, not a step of a relation. See [Prove An Agent Or Token Is Yours](prove-ownership.md#verified).
- **Declared is not false.** A declaration can be true. It is shown as what it is: one side's word.
- **A missing relation is not a negative.** It means no record of the other kind shares a wallet or domain with this one, or that we have not read it yet.

The fields behind relations are in [Relation Fields](relations-api.md).

## License

[MIT](../LICENSE) (c) 2026 Deside
