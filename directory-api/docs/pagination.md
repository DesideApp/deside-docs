# Directory API Pagination

## Model

Directory API uses cursor pagination.

The list endpoint accepts:

- `limit`
- `cursor`

The response includes:

- `pagination.nextCursor`
- `pagination.hasMore`
- `pagination.limit`
- `pagination.total`

## Behavior

- `limit` defaults to `50`
- `limit` cannot exceed `100`
- `nextCursor` is opaque
- cursors are tied to the current filter set
- changing the filter set invalidates the cursor
- `total` counts the current filter set
- `hasMore` indicates whether another page exists

## Practical use

When `pagination.nextCursor` is non-null, pass it back as the next `cursor`
value to continue reading the directory from the same filter set.

## Invalid cursor

If the cursor payload does not match the current request filters, the API
returns `invalid_cursor`.

## Example

```bash
curl -sS \
  -H "x-api-key: $DESIDE_DIRECTORY_API_KEY" \
  "$DESIDE_API_BASE_URL/api/v1/directory/agents?limit=25"
```

The developer contract does not use `skip` or `offset`.
