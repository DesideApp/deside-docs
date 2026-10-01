# Pagination

Every list route of the Directory API pages with an opaque cursor. A page
carries `pagination`:

| Field | Description |
| --- | --- |
| `nextCursor` | Pass it as `cursor` to get the next page. `null` on the last page. |
| `hasMore` | `true` when another page exists. |
| `limit` | The page size used. |
| `total` | Every item that matches the filters, not only this page. |

The public agent routes (`/api/v1/public/agents...`) are the exception: they
page with `skip` and `limit` and answer `{ items, total, limit, skip, hasMore }`
([Public Agents API](../../agent-identity/public-api-contracts.md)).

## Rules

1. **Send the same filters with every page.** A cursor carries the filters it
   was issued for, and a cursor sent with other filters answers `400`.
2. Do not build or edit a cursor. Its content is not part of the contract.
3. A cursor does not expire. On `GET /api/v1/directory/agents` it encodes a
   position in the `updatedAt` order, so an agent updated while you walk moves
   to the front of the list. Use `updatedSince` to catch it on your next run
   ([Quickstart](quickstart.md#keep-a-copy-in-sync)).

## Differences by route

| Route | Default `limit` | Maximum | Depth | Cursor mismatch |
| --- | --- | --- | --- | --- |
| `GET /api/v1/directory/agents` | 50 | 100 | none | `invalid_cursor` |
| `GET /api/v1/directory/x402-...` | 25 | 100 | none | `invalid_request` |
| `GET /api/v1/public/x402/tools`, `/wallet-edges` | 25 | 100 | 500 rows | `invalid_request` |
| `GET /api/v1/public/x402/indices` | 25 | 100 | none | |

On `GET /api/v1/directory/agents`, the cursor is bound to `registry`, `chain`,
`service`, `capability` and `updatedSince`. Changing `collection` or
`collectionCase` does not reject the cursor but gives a walk that is neither
the old filter nor the new one, so restart without `cursor` when you change
them.

On `GET /api/v1/public/x402/tools`, a search with `q` returns a single page and
does not take a cursor.
