# Directory API Quickstart

## 1. Obtain a Free key

Open the gated API Console at `https://deside.io/developer/api` or request
preview access through `support@deside.io`. Directory API keys start with
`dapi_...`.

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
