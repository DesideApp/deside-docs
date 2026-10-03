# Relations

Two public responses carry [relations](../start/relations.md): the `relations` field of an agent profile, and `behindToken.relations` of a token. Both need no key, and both use the same English keys for a relation: `kind`, `step` and `vias[].how`, with the same values.

| Field | Endpoint |
| --- | --- |
| `relations` | `GET https://api.deside.io/api/v1/public/agents/{catalogId}/profile` ([Public Agents API](public-agents.md#get-public-agents-ref-profile)) |
| `behindToken.relations` | `GET https://api.deside.io/api/v1/public/market/solana/{mint}` |

<!-- REVISAR(modificar): /public/market esta clasificada como ruta INTERNA del front de trading (nivel-3/api-rutas-publicas.md:70) y no esta en el OpenAPI. O se publica (y entra en la portada de la API y en el OpenAPI) o sale esta seccion. Decision del owner. -->

## The shared keys

| Key | Values |
| --- | --- |
| `kind` | `agent`, `token` or `tool`: the kind of the related object. |
| `step` | `proven`, `matches` or `declared`. |
| `vias[].how` | `same-wallet`, `same-domain`, `agent-lists-token` or `agent-lists-tool`. |
| `vias[].value` | The shared wallet or domain, or `null` when the agent lists the object. |

What each step means:

- **Declared**: one side names the other. Nothing else confirms it.
- **Matched**: the two sides share a public datum, such as a wallet or a domain.
- **Verified owner**: the same owner proved control of both sides.

The words shown on deside.io for each step are in [How To Read A Relation](../start/relations.md#the-three-steps).

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
      "kind": "token",
      "id": "solana:DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi",
      "step": "matches",
      "vias": [
        { "how": "agent-lists-token", "value": null },
        { "how": "same-wallet", "value": "638VSKkGvrxstYoGdRbxhofuKXVFCtPZqKiyXVWPCmeH" }
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
| `items[].kind` | string | `token` or `tool`. |
| `items[].id` | string | Its id. A token is `solana:<mint>` or `evm:<address>`. A tool is its URL. |
| `items[].step` | string | `proven`, `matches` or `declared`. |
| `items[].vias` | array | What connects the two objects, each `{ how, value }`. |
| `items[].card` | object or null | The related object: `chain`, `address`, `name`, `symbol` for a token; `slug`, `title`, `host` for a tool. `null` when Deside has no record of it. |

## Token: `behindToken.relations`

This reads one token, including who is behind it:

```bash
curl "https://api.deside.io/api/v1/public/market/solana/DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi"
```

The `behindToken.relations` field of the response, real on 2026-10-01, trimmed to its first item:

```json
[
  {
    "kind": "agent",
    "kindLabel": "Agent",
    "name": "Mizuki the Mech",
    "link": "/agents/mizuki-the-mech-cmeh",
    "image": null,
    "step": "matches",
    "stepLabel": "Matched",
    "dimmed": false,
    "help": null,
    "conflict": false,
    "vias": [
      { "how": "same-wallet", "howText": "Same wallet 638V…CmeH", "value": "638V…CmeH" },
      { "how": "agent-lists-token", "howText": "This token names this agent", "value": null }
    ],
    "live": [],
    "proof": null
  }
]
```

`behindToken` is `null` when Deside knows nothing behind the token, and also for a mint Deside has not read: that response is a `200` with empty fields, not a `404`. The list holds up to 50 relations. Each item has the shared keys plus fields ready to display:

| Field | Type | Description |
| --- | --- | --- |
| `kind`, `kindLabel` | string | `agent` or `tool`, and its label. |
| `name` | string or null | Name of the related agent or tool. |
| `link` | string or null | Its page on deside.io, such as `/agents/<slug>`. |
| `image` | string or null | Its image, HTTPS only. |
| `step`, `stepLabel` | string | `proven`, `matches` or `declared`, and the label `Verified owner`, `Matched` or `Declared`. |
| `dimmed` | boolean | `true` for a Declared relation. |
| `help` | string or null | `Not confirmed` for a Declared relation, otherwise `null`. |
| `conflict` | boolean | `true` when two different accounts proved the two sides. |
| `vias` | array | Up to 10 facts that connect the two sides. |
| `vias[].how` | string or null | One of the shared values above. `null` for a kind this version does not know. |
| `vias[].howText` | string or null | The sentence for display, such as `Same domain example.com`. |
| `vias[].value` | string or null | The shared wallet, shortened, or the shared domain. |
| `live` | array | For a Matched or Verified owner agent: its protocols that answered their latest check, each `{ "key", "label" }` with key `mcp`, `a2a` or `x402`. Always empty for a tool or a Declared agent. See [Live](../start/checks.md#agents). |
| `proof` | object or null | For a Verified owner relation, the proof: `kind`, `text`, `since`, `atLabel`. Otherwise `null`. |

The rest of `behindToken`, such as `summary` and `proofs`, describes what the token names outside these relations, like its X account. It is not covered here.

## What the keys do not mean

- **No relation says Verified on its own.** [Verified owner](../start/relations.md#verified-owner) is the highest step.
- **`live` is not quality.** It says an endpoint answered, as in [State Words](../start/checks.md#what-we-do-not-measure).

## Errors

| Code | When | What to do |
| --- | --- | --- |
| `404` | The agent `{catalogId}` matches no listed agent. The body is `{ "error": "not_found" }` | Check the reference. |
| `429` | Rate limit reached: 60 requests per minute on the agent profile, 600 on the token | Wait for the seconds in `RateLimit-Reset`. |

## License

[MIT](../LICENSE) (c) 2026 Deside
