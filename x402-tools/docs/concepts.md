# How The Tool Directory Works

An **x402 tool** is one paid HTTP endpoint that a public x402 catalog lists: a URL that asks for a payment over the x402 protocol before it answers. Unlike an [agent](../../agent-identity/README.md), a tool is not registered onchain. Deside finds it in catalogs, merges the copies, and calls it to see what it answers.

Anyone can publish a price in a catalog. A catalog entry can be stale, the route can be gone, or the endpoint can ask for another price than the one published. So Deside keeps what the catalogs say apart from what the endpoint answered, and shows both.

## Where tools come from

A **catalog** is a public list of x402 endpoints that a third party publishes, also called a bazaar. Deside reads five catalogs today:

| Catalog | Id in the API | Tools on 2026-10-01 |
| --- | --- | --- |
| Coinbase CDP | `cdp` | 19,862 |
| PayAI | `payai` | 10,925 |
| Dexter | `dexter` | 3,262 |
| OpenFacilitator | `openfac` | 101 |
| thirdweb | `thirdweb` | 100 |

The counts are tools that are not retired, from `porBazar` of the [census](api.md#get-apiv1publicx402census). A tool in two catalogs counts in both, so the column adds up to more than the 32,956 tools listed that day.

**A tool listed by two catalogs is one tool in Deside**, with both catalogs in its `bazaars`. Each catalog keeps its own price, so one tool can carry several prices. A tool that stops being seen in its catalogs is retired: it leaves the lists and its page says so.

An agent can also list its own tools in an x402 index, such as `https://agent402.tools/.well-known/x402`. Those tools are tied to the agent that declares the index. See [`/x402/indices`](api.md#get-apiv1publicx402indices).

## The live check

Deside calls each tool's address without paying and records what it answered. The request is a plain `GET` or `POST` with `accept: application/json`, and redirects are not followed. The result is one of five marks:

| Mark | API value | Meaning |
| --- | --- | --- |
| Quotes a price | `offer` | The answer carried an x402 payment offer. |
| Responds, no price | `no-offer` | The endpoint answered, without a payment offer. |
| No such route | `no-endpoint` | The endpoint answered `404`, `405` or `410`. |
| Down | `no-response` | The request failed: network error, DNS failure or timeout. |
| Blocked | `blocked` | Deside did not send the request because the address failed its outbound safety check. |

When the tool quotes a price, Deside compares the offer with what the catalog published: the same amount, and the same receiving wallet.

Example: on 2026-09-28 the tool `abi-cyberwarex-abi` at `https://abi.cyberwarex.com/abi` answered `402` with an offer of `3000` base units of USDC on Base (`eip155:8453`), paid to `0x058D0Cc5CC97e61e8A9f38D6d6365bce525921B2`. The catalog `cdp` had published the same amount and the same wallet, so its live check reads `same as declared`. You can read it yourself:

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/tools/abi-cyberwarex-abi"
```

The result is in `sonda`, with the comparison in `sonda.cotejo`: `mismoImporte` and `mismaCartera` are both `true`.

## Hosts and pay-to wallets

Every tool has a **host**, the domain it is served from, and one or more **pay-to wallets**, the addresses its prices pay. Both group tools: deside.io shows every tool of one host at `https://deside.io/x402/h/<host>` and every tool paying one wallet at `https://deside.io/x402/w/<wallet>`.

A wallet often groups more than a host. On 2026-10-01 the wallet `0x058D0Cc5CC97e61e8A9f38D6d6365bce525921B2` received the payments of 32 tools on 16 hosts:

```bash
curl -sS "https://api.deside.io/api/v1/public/x402/census?payTo=0x058D0Cc5CC97e61e8A9f38D6d6365bce525921B2"
```

When an agent's wallet is a tool's pay-to wallet, or both share a domain, the two are related with the step Matches. See [How To Read A Relation](../../start/relations.md). A tool served from a domain you proved is yours: see [Prove It Is Yours](../../start/prove-ownership.md#prove-an-x402-tool).

## What this does not mean

- **Quotes a price is not "works".** The live check never pays and never calls the tool with real input. A tool that quotes a price may still fail once paid.
- **A matching price is not a fair price.** It says the endpoint asks what the catalog published, nothing more.
- **A shared pay-to wallet is not the same operator.** It says two records name the same wallet. It does not say who runs the tools.
- **Not being listed is not a negative.** Deside lists what the five catalogs and the agents' indices publish, nothing else.

## License

[MIT](../../LICENSE) (c) 2026 Deside
