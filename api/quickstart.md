# Quickstart

This guide gets you from zero to reading the Agent Directory in about 2 minutes. You need no key and no account.

## Prerequisites

- `curl`, or any HTTP client.

## Read the first page

The list returns 20 agents by default, up to 100 with `limit`:

```bash
curl "https://api.deside.io/api/v2/public/agents?limit=5"
```

The agents come in `data`, and `page.hasMore` says whether there are more.

## Filter

Ask only for agents with a live MCP endpoint, on Solana:

```bash
curl "https://api.deside.io/api/v2/public/agents?live=mcp&chain=solana"
```

Every filter is in [Public agents](public-agents.md#parameters).

## Read the next page

Add `skip`. The list stops at 500 rows deep: narrow the filters to reach the rest.

```bash
curl "https://api.deside.io/api/v2/public/agents?limit=100&skip=100"
```

## Read one agent

Use its `slug` or `id`. `/profile` adds what we read on-chain:

```bash
curl "https://api.deside.io/api/v2/public/agents/blinkcodes/profile"
```

## What you just did

You listed, filtered and paged the Agent Directory and read one full profile, with no key. Each IP has 30 list requests a minute, 60 single reads a minute and 500 a day.

## Next steps

- [Public agents](public-agents.md): every filter and field.
- [Checks](../start/checks.md): what each state means.
- [Errors and limits](errors-and-limits.md): every code and what to do.
