# Prove It Is Yours

This guide shows how to prove, from your Deside account, that an agent, a web domain, an x402 tool or a token is yours. The same steps, in short, are on [deside.io/verified](https://deside.io/verified).

{% hint style="info" %}
What each proof gives you:

- an agent you prove becomes [Connected](checks.md#agents)
- a domain you prove makes your account a [Verified domain](#verified-domain)
- a token or tool you prove can lift its relations to [Verified owner](relations.md#verified-owner)
{% endhint %}

## Prerequisites

- A Deside account. Sign in at [deside.io](https://deside.io), then open Account.
- For an agent: the wallet that owns it in its registry.
- For a domain: access to the website's files or to its DNS.
- For a token through X: the X account linked in Identities, under Link your accounts.

## Prove an agent with its owner wallet

In Account, open Wallets and link the agent's owner wallet by signing a message with it. One signature covers every agent that wallet owns in a registry.

The agent stays Connected while the wallet stays linked to your account. Unlinking the wallet clears it.

## Prove a domain

In Account, open Identities, add your domain and choose one of two ways. You only need one. Deside gives you a 64-character code for that domain.

### With a file

Publish a plain text file at this address, with your code on one line and nothing else:

```
https://<your-domain>/.well-known/deside-domain-challenge.txt
```

The file must be served as `text/plain` and be at most 256 bytes. Open the address in a browser: you should see only the code. Then press Check file.

### With a DNS record

Where you manage your domain's DNS, add one TXT record:

| Type | Name | Value |
| --- | --- | --- |
| TXT | `_deside-challenge.<your-domain>` | your code |

If your DNS provider appends the domain to the name by itself, enter only `_deside-challenge`. DNS can take up to 48 hours to reach everyone. Then press Check DNS record.

### Hosting subdomains

A subdomain you rent from a host, such as `name.vercel.app`, can be proven with the file only, because its DNS belongs to the host. **The proof covers that exact host.** It says nothing about the host's other subdomains.

### How long a proof lasts

We read every proven domain once a day. The proof falls in these cases:

| Case | When |
| --- | --- |
| The file or record is gone | for 7 days |
| We cannot read the domain | for 14 days |
| A different code is found | on two reads about a day apart |
| The domain no longer resolves | on two reads about a day apart |
| Another account proves the same domain | when that proof stands |

A domain you add but do not prove within 7 days is dropped. Prove it again and the proof comes back.

## Prove an x402 tool

An x402 tool served from a domain you proved is yours, with nothing more to do. A tool on another host does not become yours by being listed in your files.

## Prove a token

A token links to your account in any of three ways:

1. **Your wallet created it.** If a wallet you linked in Wallets signed the token's creation, the token links by itself.
2. **Your token, plus your website or your X.** In Identities, add the token's chain and address under Your token. It links when the token's own metadata names a website on a domain you proved, or an X account you linked in Identities.
3. **Your owner list.** List the token in `/.well-known/deside.json` on a domain you proved. See [List what is yours](#list-what-is-yours-on-your-domain).

Your token accepts Solana, Base and Ethereum addresses, up to 100 tokens. Each one shows its state:

| What Identities says | Meaning |
| --- | --- |
| `Linked to your account.` | The token is yours in Deside. |
| `Not linked yet: the token's website has to be a domain you proved.` | None of the three ways applies yet. |
| `Another account says this token is theirs too, so it is not linked to anyone.` | Two accounts claim it. |
| `We do not have this token yet. It links once we read it.` | Deside has not read this token. |

**Adding a token under Your token does not prove it by itself.** The token has to name a domain you proved or an X account you linked, or have been created by your wallet.

## List what is yours on your domain

This step is optional. Publish a JSON file at `https://<your-domain>/.well-known/deside.json`, served as `application/json`, up to 64 KB. We read it once a day with your domain, and when you press Check now on that domain in Identities.

This is a valid file:

```json
{
  "version": 2,
  "proof": "<your 64-character code>",
  "agents": [{ "registry": "erc8004-base", "id": "1234" }],
  "tools": [{ "host": "api.example.com", "pathPrefix": "/v1/" }],
  "tokens": [{ "chain": "solana", "address": "<your token address>" }],
  "x": [{ "handle": "yourhandle" }],
  "github": [{ "login": "yourlogin" }]
}
```

| Key | Type | Required | Rules |
| --- | --- | --- | --- |
| `version` | integer | Yes | `1` or `2`. |
| `proof` | string | Yes | Your code for this domain. A file with another account's code is ignored. |
| `agents` | array | No | Each item is `{ "registry", "id" }`. `registry` is one of `mip14`, `8004solana`, `sati`, `said`, `sap`, `erc8004-base`, `erc8004-ethereum`. `id` is the agent's id in that registry. |
| `tools` | array | No | Each item is `{ "host", "pathPrefix" }`. `host` has no scheme, port or `www.`. `pathPrefix` starts with `/`. On a hosting subdomain, only that exact host. |
| `tokens` | array | No | Each item is `{ "chain", "address" }`. `chain` is `solana`, `base` or `ethereum`. |
| `x` | array | No | Version 2 only. Each item is `{ "handle" }` (X handle, lowercase, no @, up to 10 items). |
| `github` | array | No | Version 2 only. Each item is `{ "login" }` (GitHub login, lowercase, up to 10 items). |

Each list takes up to 100 items. Items cannot carry other keys, wildcards or names. Any other top-level key makes the whole file invalid.

**The file does not prove a domain.** Only the code file or the TXT record does. Of the lists, only `tokens` makes an object yours today, and only when the token names that domain as its website.

## Check what a proof needs

To see what a token, agent or tool needs to become yours, call this endpoint. While unclaimed, it returns the steps for each way:

```bash
curl https://api.deside.io/api/v1/public/claim/token/solana:DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi
```

Response:

```json
{
  "objectType": "token",
  "objectId": "solana:DwquZcs2JtPe2w9xfyqF9wDnySQXLBHTMawusJ8Uk1mi",
  "state": "unclaimed",
  "vias": [
    {
      "via": "wallet",
      "steps": ["link-wallet"],
      "value": null
    },
    {
      "via": "x",
      "steps": ["link-x", "your-token"],
      "value": "mizukimech"
    }
  ]
}
```

Each via is a way to claim it. Once claimed, `state` becomes `proven` and `vias` disappears.

## Verified domain

**Verified domain means a Deside account has proven a web domain.** It is a state of the account, shown inside Identities. It is not a state of an agent or of a [relation](relations.md).

Verified domain is given automatically when a domain proof stands, and it falls when the proof falls.

## Common errors

### `Your file was rejected`

The `deside.json` file broke one of the rules in the table above: wrong content type, over 64 KB, not valid JSON, an unknown top-level key, a `version` other than `1` or `2`, or a `proof` that is not your code. Identities shows the reason. Nothing from a rejected file counts.

A single bad item does not reject the file: that item is skipped and listed under the summary.

### `No /.well-known/deside.json found`

The file returned `404` or `410`. Check the address and that the file is at the root of the proven domain.

## License

[MIT](../LICENSE) (c) 2026 Deside
