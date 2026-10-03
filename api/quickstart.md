# Quickstart

This guide gets you from no key to a synced copy of the Agent Directory.

## Prerequisites

- A Solana wallet, such as Phantom or Solflare, to sign in to the API console.
- `curl`, or any HTTP client.

## Get a free key

1. Open `https://deside.io/developer/api`.
2. Connect your wallet and sign the text. Signing moves no funds.
3. Create a key. Keys look like `dapi_<prefix>_<secret>`.

**The key is shown once, when you create it.** Store it before you close the page.

Export it so every command below runs as is:

```bash
export DESIDE_API_KEY=YOUR_API_KEY
```

## Read the first page

Send the key in the `x-api-key` header:

```bash
curl -H "x-api-key: $DESIDE_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?limit=100"
```

The response has `agents` and `pagination`. Agents come newest change first.

## Follow the cursor

Pass `pagination.nextCursor` as `cursor` until `hasMore` is `false`:

```bash
curl -H "x-api-key: $DESIDE_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?limit=100&cursor=NEXT_CURSOR"
```

**A cursor only works with the same filters it came from.** Change a filter and start again without `cursor`.

## Sync only what changed

On the first full walk, keep the highest `updatedAt` you saw. On each later run, ask only for agents changed since then:

```bash
curl -H "x-api-key: $DESIDE_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?limit=100&updatedSince=2026-10-01T00:00:00.000Z"
```

Subtract a minute from the timestamp you send: getting an agent twice costs nothing, missing one does.

**An agent that leaves the directory stops appearing; you are not told.** To catch removals, compare against a full walk from time to time.

## What you just did

You read the directory with your own key and kept a copy in sync with a handful of requests per run. The free plan gives 5,000 requests a month.

## Next steps

- [Directory agents](directory-agents.md): every filter and field.
- [Errors and limits](errors-and-limits.md): what each code means and what to do.
