# Relations

The agent profile carries its [relations](../start/relations.md) in the `relations` field. It needs no key. Each relation uses the keys `kind`, `step` and `vias[].how`.

| Field | Endpoint |
| --- | --- |
| `relations` | `GET https://api.deside.io/api/v2/public/agents/{id}/profile` ([Public Agents API](public-agents.md#get-ref-profile)) |

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
curl "https://api.deside.io/api/v2/public/agents/mizuki-the-mech-cmeh/profile"
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

## What the keys do not mean

- **No relation says Verified on its own.** [Verified owner](../start/relations.md#verified-owner) is the highest step.
- **`live` is not quality.** It says an endpoint answered, as in [State Words](../start/checks.md#what-we-do-not-measure).

## Errors

| Code | When | What to do |
| --- | --- | --- |
| `404` | The agent `{id}` matches no listed agent. The body is `{ "error": { "code": "not_found" } }` | Check the reference. |
| `429` | Rate limit reached: 60 requests per minute | Wait for the seconds in `RateLimit-Reset`. |

## License

[MIT](../LICENSE) (c) 2026 Deside
