# Relations

A [relation](../start/relations.md) is a link Deside shows between two records, such as an agent and an x402 tool, with a step that says how far it is backed. Relations are Deside's own view, not something an agent declares. **The free API and the free MCP do not return relations.** Over the API you read them in the two [paid lookups](paid-lookups.md).

## Where relations appear

| Lookup | Field | What it relates |
| --- | --- | --- |
| [`GET /api/v2/paid/x402/tool-check`](paid-lookups.md#get-x402-tool-check) | `tool.declaredBy[]` | Listed agents that declare the tool. |
| [`GET /api/v2/paid/x402/tool-check`](paid-lookups.md#get-x402-tool-check) | `wallets[].matchingAgents[]` | Listed agents that declare a wallet the tool is paid to. |
| [`GET /api/v2/paid/wallet`](paid-lookups.md#get-wallet) | `relations.items[]` | The agents, tools and other records related through that wallet. |

## The shared keys

| Key | Values |
| --- | --- |
| `step` | `declared`, `matches` or `proven`. |
| `label` | The word to show for the step: `Declared`, `Matching` or `Verified owner`. Show it as it comes. |
| `how` | What links the two sides: `same-wallet`, or for a tool, `declares-this-url` or `lists-it-in-its-x402-index`. |
| `checkedAt` | When Deside last saw the link, where the lookup gives it. |

What each step means:

| `step` | `label` | Meaning |
| --- | --- | --- |
| `declared` | Declared | One side names the other. Nothing else confirms it. |
| `matches` | Matching | The two sides share a public datum, such as a wallet. Not proven by the owner. |
| `proven` | Verified owner | The same owner proved, from their Deside account, that both sides are theirs. |

## What the keys do not mean

- **No relation says anything about quality.** A step says how far a link is backed, not whether the agent or the tool is good.
- **An empty list means we found no relation**, not that none exists.
- **A wallet shared by many owners does not relate them.** The wallet lookup says so in `sharedWallet`. See [How To Read A Relation](../start/relations.md).

## License

[MIT](../LICENSE) (c) 2026 Deside
