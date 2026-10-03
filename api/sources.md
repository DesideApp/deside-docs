# Sources

Deside reads what is already public. This page lists what it reads and how often.

## Agent registries

| `registry` | Registry | Chain |
|---|---|---|
| `erc8004-base` | ERC-8004 on Base | EVM |
| `said` | SAID Protocol | Solana |
| `mip14` | Metaplex Agent Registry | Solana |
| `8004solana` | Quantu 8004-Solana | Solana |
| `sati` | Cascade SATI | Solana |
| `sap` | Synapse Agent Protocol | Solana |

**One agent is one entry in the directory**, even when several registries list it. `registries` lists them all, and `sources` gives each entry id.

## x402 catalogs

| `bazaar` | Catalog |
|---|---|
| `payai` | PayAI |
| `cdp` | Coinbase CDP |
| `dexter` | Dexter |
| `thirdweb` | thirdweb |
| `openfac` | OpenFacilitator |

Plus the x402 indexes that agents publish themselves. When a catalog stops listing a tool and no agent lists it any longer, the tool is marked `retired`.

## How often

| What | When |
|---|---|
| Reading the registries | Every night. Base, the largest, is read over several nights; its new events are checked every few hours. |
| Checking agent endpoints | Every night, in a queue: each URL aims to be checked once a week. |
| Real A2A call | Once a week. |
| Reading the x402 catalogs and indexes | Every night. |
| Checking x402 tools | Every night, in a queue: a full pass takes 3 to 5 nights. A tool whose result has not changed in 30 days is checked once a week. |

Every result carries the date of its check: `checkedAt`, `probedAt`, `probe.at`, `measuredAt`.

## Categories

The category of an agent comes from its own description. TypeSafe, an AI model, reads what the agent says it does and gives it one of 12 categories. Agents without a useful description get `other`. **A category describes what the agent says it does**, not what we measured it doing.
