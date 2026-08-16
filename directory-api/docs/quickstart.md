# Directory API Quickstart

## 1. Obtain a Free key

Open the API Console at `https://deside.io/developer/api` and sign in with your
wallet. A Free key is self-serve: you create it yourself, no request and no
waiting. Directory API keys start with `dapi_...`. The raw key is shown once,
at creation.

## 2. Export the key and the base URL

```bash
export DESIDE_DIRECTORY_API_KEY=dapi_<public_prefix>_<secret>
export DESIDE_API_BASE_URL=https://api.deside.io
```

Every command on this page uses these two variables, so the rest of the
walkthrough is copy-paste.

## 3. Send the request

```bash
curl -sS \
  -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "$DESIDE_API_BASE_URL/api/v1/directory/agents?limit=2"
```

## 4. Read the list response

A successful response returns `agents` and `pagination`.

```json
{
  "agents": [
    {
      "id": "agent-catalog-1",
      "slug": "agent-slug",
      "displayName": "Agent Name"
    }
  ],
  "pagination": {
    "nextCursor": "eyJ2IjoxLCJ1cGRhdGVkQXQiOiIyMDI2LTA2LTMwVDAxOjAwOjAwLjAwMFoiLCJpZCI6IjY2NTAwMDAwMDAwMDAwMDAwMDAwMDAwMSIsImZpbHRlckhhc2giOiJhYmNkZWYwMTIzIn0",
    "hasMore": true,
    "limit": 2,
    "total": 24
  }
}
```

## 5. Follow the next page token

Use `pagination.nextCursor` as the next request `cursor`.

```bash
curl -sS \
  -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "$DESIDE_API_BASE_URL/api/v1/directory/agents?limit=2&cursor=$NEXT_CURSOR"
```

## 6. Read a single agent

```bash
curl -sS \
  -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "$DESIDE_API_BASE_URL/api/v1/directory/agents/agent-slug"
```

The detail routes can return:

- `agent`
- `disambiguation: true` with a short list of visible matches

## 7. Keep a copy in sync without re-reading everything

Most integrations start by walking the whole directory once and then asking, on
every run, only for what changed. Do that and your daily cost drops from about
104 requests to a handful.

First, the one-time backfill. Page through with the cursor and record the
newest `updatedAt` you saw:

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "$DESIDE_API_BASE_URL/api/v1/directory/agents?limit=100"
# follow pagination.nextCursor until hasMore is false,
# and keep the highest agents[].updatedAt of the whole walk
```

Then, on every later run, ask only for what moved since that timestamp:

```bash
curl -sS -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "$DESIDE_API_BASE_URL/api/v1/directory/agents?limit=100&updatedSince=2026-08-15T18:00:00.000Z"
```

`updatedSince` takes an ISO date string and filters on the same `updatedAt`
that the list is ordered by, so a run that finds nothing new costs one request.
Store the new high-water mark only after you have consumed every page of the
response, and subtract a small overlap (a minute is plenty) from the timestamp
you send: repeating an agent you already have is free, missing one is not.

An agent that stops being listed simply stops appearing. If you need to detect
removals, compare your local set against a full sweep from time to time; a
weekly one is enough for most uses and costs about 104 requests.

## 8. Handle errors

Use the documented error `code` and `docsUrl` fields rather than parsing
messages.

## 9. Move to a paid tier when you outgrow Free

Paid tiers are bought with your owner wallet, not with an invoice. From the
console you authorize a recurring USDC payment on Solana, and the tier changes
once the first charge settles. Both `developer` and `pro` are bought the same
way.

Nothing is charged when you authorize: authorizing grants the charge, and the
first charge is what grants the tier. You can revoke that authorization at any
time from the same console, and the tier you already paid for stays available
until the paid period ends.

The endpoints behind that flow, the 30-day cycle, and what happens when you
change tier are documented in [Endpoints](endpoints.md).
