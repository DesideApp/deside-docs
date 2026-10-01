# Authentication

The Directory API uses one credential per kind of caller. An API key reads data
on a tier; a console proof manages the project behind it. They do not mix: a
key never manages a project, and a console proof never reads data.

## Which credential each route needs

| Routes | Credential |
| --- | --- |
| `/api/v1/directory/agents...`, `/api/v1/directory/x402-...` | API key, `x-api-key: dapi_...` |
| `/api/v1/directory/console/nonce`, `/api/v1/directory/console/auth` | none: they issue the console proof |
| `/api/v1/directory/keys`, `/usage`, `/subscription` | console proof, `Authorization: Bearer <proof>` ([Owner console](console.md)) |
| `/api/v1/public/...` | none ([Access model](access-model.md)) |
| `/api/v1/ask` | none; a signed-in browser session selects the human lane ([Ask](ask.md)) |
| MCP, `https://mcp.deside.io/mcp` | MCP session ([MCP authentication](../../mcp/docs/authentication.md)) |

## API key

Send the key in the `x-api-key` header on every request:

```http
x-api-key: dapi_<public_prefix>_<secret>
```

A key is required on every tier, Free included. You create and revoke keys in
the API console. The raw key is shown once, at creation.

{% hint style="danger" %}
Do not put a `dapi_` key in a browser page, a public repository or a URL. Anyone
who has it spends your quota. Revoke a leaked key in the console.
{% endhint %}

A key can be limited to a list of allowed origins. A request from another
origin answers `origin_not_allowed`.

## Key errors

| Code | Status | When |
| --- | --- | --- |
| `missing_api_key` | 401 | No `x-api-key` header. |
| `invalid_api_key` | 401 | The key is unknown or malformed. |
| `api_key_revoked` | 403 | The key was revoked. |
| `api_key_blocked` | 403 | The key was blocked. |
| `project_blocked` | 403 | The key's project is blocked. |
| `origin_not_allowed` | 403 | The request origin is not in the key's allowed list. |

A request rejected for any of these reasons does not count against your quota.
The full error list is on [Errors](errors.md).
