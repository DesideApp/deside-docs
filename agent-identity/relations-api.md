# Relation Fields

Two public responses carry [relations](../start/relations.md): the `relations` field of an agent profile, and `behindToken.relations` of a token. Both need no key. The two shapes differ: the agent's lists raw keys, and the token's comes ready to display.

| Field | Endpoint |
| --- | --- |
| `relations` | `GET https://api.deside.io/api/v1/public/agents/{catalogId}/profile` ([Public Agents API](public-api-contracts.md#read-a-profile)) |
| `behindToken.relations` | `GET https://api.deside.io/api/v1/public/market/solana/{mint}` |

{% hint style="warning" %}
The two shapes will be unified into one, with English keys. Until then, read
each one as documented on its endpoint. The change will be listed in the
[Changelog](../directory-api/docs/changelog.md) with what to update.
{% endhint %}

## Agent profile: `relations`

This reads the relations of one agent:

```bash
curl "https://api.deside.io/api/v1/public/agents/mizuki-the-mech-cmeh/profile"
```

The `relations` field of the response, real on 2026-10-01:

```json
{
  "total": 1,
  "items": [
    {
      "bolsa": "token",
      "id": "solana:DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi",
      "escalon": "coincide",
      "via": [
        { "keyType": "declared" },
        {
          "keyType": "wallet",
          "keyValue": "638VSKkGvrxstYoGdRbxhofuKXVFCtPZqKiyXVWPCmeH",
          "roles": ["identity"]
        }
      ],
      "card": {
        "chain": "solana",
        "address": "DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi",
        "name": "Mizuki the Mech",
        "symbol": "MIZUKI"
      }
    }
  ]
}
```

`relations` is `null` when Deside could not read them. **`null` means unknown, not "no relations".** No relations is `{ "total": 0, "items": [] }`.

| Field | Type | Description |
| --- | --- | --- |
| `total` | integer | Relations of this agent. |
| `items` | array | Up to 50 relations, one per related object. |
| `items[].bolsa` | string | Kind of the related object: `token` or `tool`. |
| `items[].id` | string | Its id. A token is `solana:<mint>` or `evm:<address>`. A tool is its URL. |
| `items[].escalon` | string | The step: `demostrado` (Proven), `coincide` (Matches) or `declarado` (Declared). |
| `items[].via` | array | What connects the two objects. |
| `items[].via[].keyType` | string | `wallet` or `domain` for a shared datum. `declared` when the agent itself names the object; then there is no `keyValue`. |
| `items[].via[].keyValue` | string | The shared wallet or domain. |
| `items[].via[].roles` | array | What the datum is for this agent, such as `identity` for its owner wallet or `web` for its website. |
| `items[].card` | object or null | The related object: `chain`, `address`, `name`, `symbol` for a token; `slug`, `title`, `host` for a tool. `null` when Deside has no record of it. |

The words shown on deside.io for each step are in [How To Read A Relation](../start/relations.md#the-three-steps).

## Token: `behindToken.relations`

This reads one token, including who is behind it:

```bash
curl "https://api.deside.io/api/v1/public/market/solana/DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi"
```

The `behindToken.relations` field of the response, real on 2026-10-01:

```json
[
  {
    "kind": "agent",
    "kindLabel": "Agent",
    "name": "Mizuki the Mech",
    "link": "/agents/mizuki-the-mech-cmeh",
    "image": null,
    "step": "matches",
    "stepLabel": "Matches",
    "dimmed": false,
    "help": null,
    "conflict": false,
    "vias": [
      { "how": "misma-wallet", "howText": "Same wallet 638V…CmeH", "value": "638V…CmeH" },
      { "how": "el-token-lo-nombra", "howText": "This token names this agent", "value": null }
    ],
    "live": []
  },
  {
    "kind": "agent",
    "kindLabel": "Agent",
    "name": "Mizuki the Mech",
    "link": "/agents/mizuki-the-mech",
    "image": null,
    "step": "declared",
    "stepLabel": "Declared",
    "dimmed": true,
    "help": "Not confirmed",
    "conflict": false,
    "vias": [
      { "how": "el-token-lo-nombra", "howText": "This token names this agent", "value": null }
    ],
    "live": []
  }
]
```

`behindToken` is `null` when Deside knows nothing behind the token, and also for a mint Deside has not read: that response is a `200` with empty fields, not a `404`. The list holds up to 50 relations.

| Field | Type | Description |
| --- | --- | --- |
| `kind`, `kindLabel` | string | `agent` or `tool`, and its label. |
| `name` | string or null | Name of the related agent or tool. |
| `link` | string or null | Its page on deside.io, such as `/agents/<slug>`. |
| `image` | string or null | Its image, HTTPS only. |
| `step`, `stepLabel` | string | `proven`, `matches` or `declared`, and the label `Proven`, `Matches` or `Declared`. |
| `dimmed` | boolean | `true` for a Declared relation. |
| `help` | string or null | `Not confirmed` for a Declared relation, otherwise `null`. |
| `conflict` | boolean | `true` when two different accounts proved the two sides. |
| `vias` | array | Up to 10 facts that connect the two sides. |
| `vias[].how` | string or null | `misma-wallet` (same wallet), `mismo-dominio` (same domain) or `el-token-lo-nombra` (the token names it). `null` for a kind this version does not know. |
| `vias[].howText` | string or null | The sentence for display, such as `Same domain example.com`. |
| `vias[].value` | string or null | The shared wallet, shortened, or the shared domain. `null` for a token that names it. |
| `live` | array | For an agent that Matches or is Proven: its protocols that answered their latest check, each `{ "key", "label" }` with key `mcp`, `a2a` or `x402`. Always empty for a tool or a Declared agent. See [Live](../start/state-words.md#live). |

The rest of `behindToken`, such as `summary` and `proofs`, describes what the token names outside these relations, like its X account. It is not covered here.

## What the keys do not mean

- **No key is Verified.** No relation field carries a Verified state. [Proven](../start/relations.md#proven) is the highest step.
- **`live` is not quality.** It says an endpoint answered, as in [State Words](../start/state-words.md#what-this-does-not-mean).

## Errors

| Code | When | What to do |
| --- | --- | --- |
| `404` | The agent `{catalogId}` matches no listed agent | Check the reference. |
| `429` | Rate limit reached: 60 requests per minute on the agent profile, 600 on the token | Wait for the seconds in `RateLimit-Reset`. |

## License

[MIT](../LICENSE) (c) 2026 Deside
