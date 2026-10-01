# x402 Tool Directory

The x402 Tool Directory is a public list of pay-per-call HTTP endpoints, one entry per endpoint, read from five public x402 catalogs. Unlike a single catalog, it merges the copies of a tool listed in several catalogs, and it calls every endpoint to see whether it answers and whether the price it asks matches the one it publishes.

{% hint style="info" %}
In this section you will find:

- what a tool is and which catalogs it comes from
- what the live check measures, and what its marks mean
- how hosts and pay-to wallets group tools
- the public routes, with parameters, real responses and errors
{% endhint %}

## Quick start

1. Read one tool that quoted a price on Solana mainnet. No key is needed:

   ```bash
   curl -sS "https://api.deside.io/api/v1/public/x402/tools?network=solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp&live=1&limit=1"
   ```

2. Open one tool by its slug:

   ```bash
   curl -sS "https://api.deside.io/api/v1/public/x402/tools/abi-cyberwarex-abi"
   ```

3. Count the tools that answered their live check:

   ```bash
   curl -sS "https://api.deside.io/api/v1/public/x402/census?live=1"
   ```

4. See the same tool as a person sees it: [deside.io/x402/t/abi-cyberwarex-abi](https://deside.io/x402/t/abi-cyberwarex-abi).

## Core concepts

### x402 tool

**An x402 tool is one paid HTTP endpoint that a public x402 catalog lists.** A tool listed by two catalogs is one tool here. See [How The Tool Directory Works](docs/concepts.md).

### Catalog

**A catalog is a public list of x402 endpoints published by a third party**, such as Coinbase CDP or PayAI. See [Where tools come from](docs/concepts.md#where-tools-come-from).

### Live check

**The live check is a call to the tool's address, without paying**, that records whether it quotes a price, responds without one, or fails. See [The live check](docs/concepts.md#the-live-check).

### Host and pay-to wallet

**A host is the domain a tool is served from, and a pay-to wallet is the address its price pays.** Both group tools. See [Hosts and pay-to wallets](docs/concepts.md#hosts-and-pay-to-wallets).

## Quick reference

| Item | Value |
| --- | --- |
| Base URL | `https://api.deside.io/api/v1/public/x402` |
| Authentication | None |
| Rate limit, lists | 30 requests per minute per IP |
| Deepest row on a public list | 500 |
| Page size | 25 by default, 100 at most |
| Tool page on the web | `https://deside.io/x402/t/<slug>` |
| Whole catalog with a key | [x402 data with an API key](../directory-api/docs/x402-keyed.md) |

## Next steps

- [How The Tool Directory Works](docs/concepts.md): catalogs, the live check, hosts and wallets, with real values.
- [x402 Tools API](docs/api.md): every public route, parameter and error.
- [Prove It Is Yours](../start/prove-ownership.md#prove-an-x402-tool): make a tool on your domain yours.

## License

[MIT](../LICENSE) (c) 2026 Deside
