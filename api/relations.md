# Relations

The agent profile carries its [relations](../start/relations.md) in the `relations` field. It needs no key. Each relation uses the keys `kind`, `step`, `vias[].code` and `vias[].text`.

| Field | Endpoint |
| --- | --- |
| `data.relations` | `GET https://api.deside.io/api/v2/public/agents/{ref}/profile` ([Public Agents API](public-agents.md#get-ref-profile)) |

## The shared keys

| Key | Values |
| --- | --- |
| `kind` | `agent`, `token` or `tool`: the kind of the related object. In an agent profile it is `token` or `tool`. |
| `step` | `proven`, `matches` or `declared`. |
| `vias[].code` | The rule of the phrase: `owner-created`, `same-key` or `names`. |
| `vias[].text` | The phrase, ready to show: for example `Its owner created this token`, `Same website as this token` or `Names this tool`. |
| `vias[].how` | Deprecated: `same-wallet`, `same-domain`, `agent-lists-token` or `agent-lists-tool`. It is sent for one more version and will be removed. Read `code` and `text`. |
| `vias[].value` | The shared wallet or domain, or `null` when the agent lists the object. |

What each code means, read from the agent:

| `code` | `text` | When |
| --- | --- | --- |
| `owner-created` | `Its owner created this token` | The agent's owner wallet created the token. |
| `same-key` | `Same wallet as this tool`, `Same website as this token`, `Same website as this tool` | Both sides carry the wallet or website in `value`. |
| `names` | `Names this token`, `Names this tool` | The agent's registry entry names it as its own. |

Show `text` as it comes. Do not build the phrase from `code`: the wording can change, the code does not.

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

The `relations` field of the response, real on 2026-10-07:

```json
{
  "total": 1,
  "items": [
    {
      "kind": "token",
      "id": "solana:DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi",
      "step": "matches",
      "vias": [
        { "how": "agent-lists-token", "code": "names", "text": "Names this token", "value": null },
        { "how": "same-wallet", "code": "owner-created", "text": "Its owner created this token", "value": "638VSKkGvrxstYoGdRbxhofuKXVFCtPZqKiyXVWPCmeH" }
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
| `items[].vias` | array | What connects the two objects, each `{ code, text, value }`, plus the deprecated `how`. See [The shared keys](#the-shared-keys). |
| `items[].card` | object or null | The related object: `chain`, `address`, `name`, `symbol` for a token; `slug`, `title`, `host` for a tool. `null` when Deside has no record of it. |

## What the keys do not mean

- **No relation says Verified on its own.** [Verified owner](../start/relations.md#verified-owner) is the highest step.

## Errors

| Code | When | What to do |
| --- | --- | --- |
| `404` | The agent `{ref}` matches no listed agent. The body is `{ "error": { "code": "not_found", "message": "Agent not found." } }` | Check the reference. |
| `429` | Rate limit reached: 60 requests per minute | Wait for the seconds in `RateLimit-Reset`. |

## License

[MIT](../LICENSE) (c) 2026 Deside
