# Quickstart

This guide gets you from no key to walking the agent directory with the
Directory API.

## Prerequisites

* A Solana wallet to sign in to the API console.
* `curl`.

## Create a Free key

Open the API console at `https://deside.io/developer/api` and sign in with your
wallet. Create a key: it starts with `dapi_` and is shown once, so copy it
then. A Free key needs no request and no waiting.

## Export the key

Every command on this page reads the key from this variable:

```bash
export DESIDE_DIRECTORY_API_KEY=dapi_<public_prefix>_<secret>
```

## Read the first page

This request asks for the two most recently updated agents:

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?limit=2"
```

The response is `{ agents, pagination }`. Each agent is described on
[Data model](data-model.md).

## Follow the cursor

When `pagination.hasMore` is `true`, send `pagination.nextCursor` back as
`cursor`, with the same filters:

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?limit=2&cursor=$NEXT_CURSOR"
```

## Read one agent

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents/wurk"
```

## Keep a copy in sync

A full walk costs one request per 100 listed agents. On 2026-09-30 the
directory listed 109,235 agents (`listed` in the
[stats summary](../../agent-identity/public-api-contracts.md)), so a full walk took about 1,093 requests.
Repeating it every day would take about 33,000 requests a month, against the
5,000 of the Free tier.

Walk the directory once, keep the newest `updatedAt` you saw, and then ask only
for what changed since then:

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "https://api.deside.io/api/v1/directory/agents?limit=100&updatedSince=2026-09-30T00:00:00.000Z"
```

`updatedSince` filters on the same `updatedAt` the list is ordered by. Store
the new value only after you have read every page, and send it with a small
overlap, such as one minute earlier: reading an agent twice is harmless,
missing one is not.

An agent that stops being listed stops appearing, and `updatedSince` will not
tell you. To detect removals, compare your copy with a full walk from time to
time.

## What you just did

You created a key, read the directory page by page, and set up a sync that
only reads what changed.

## Next steps

* [Agents](agents.md): every filter of the list.
* [Rate limits](rate-limits.md): how many requests each tier allows.
* [Errors](errors.md): what each error code means and what to do.
