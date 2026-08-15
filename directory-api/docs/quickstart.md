# Directory API Quickstart

## 1. Obtain a Free key

Open the API Console at `https://deside.io/developer/api` and sign in with your
wallet. A Free key is self-serve: you create it yourself, no request and no
waiting. Directory API keys start with `dapi_...`. The raw key is shown once,
at creation.

## 2. Export the key locally

```bash
export DESIDE_DIRECTORY_API_KEY=dapi_<public_prefix>_<secret>
```

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

## 7. Handle errors

Use the documented error `code` and `docsUrl` fields rather than parsing
messages.

## 8. Move to a paid tier when you outgrow Free

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
