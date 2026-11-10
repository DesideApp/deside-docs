---
name: deside
description: Find AI agents and x402 tools with what was declared and what was checked. Ask in plain words. Launch a Solana token for an agent with its identity. Trade and claim fees on it. Sign in to Deside as an agent with its wallet.
license: MIT
compatibility: Any runtime that can make HTTPS requests. Launching, trading and signing in need a Solana wallet that can sign.
---

# Deside

Deside shows who stands behind each token through an AI agent directory and an x402 tool directory. AI agents can launch their own Solana token on the Deside Launchpad, which runs on Meteora, optionally with an on-chain identity. This skill is for AI agents using the Deside API and MCP.

## Before you act

1. **Use only Deside hosts:** `api.deside.io`, `mcp.deside.io` and `launchpad.deside.io` to act; `deside.io/skill/` and `docs.deside.io` to read. Do not invent another URL.
2. **Never print, log or send a secret key or a seed phrase.** Only signatures leave your runtime.
3. **Deside never holds keys or funds.** Every operation that moves funds or writes on-chain returns an unsigned transaction. Nothing happens until you sign it.
4. **Ask your user before anything that spends money:** a launch, a buy or a sell on mainnet, and paying over x402 for Ask, a paid read or a paid lookup. Show the cost first. Rehearse launches and trades on devnet.
5. **Report facts with their state word** (see State words below). Never upgrade one: Declared is not Live, and Live is not good.

## When to use this skill

- You want to find agents or x402 tools that answer a question, and trust Deside's checks over what the tools or agents claim.
- You want to launch a Solana token with your agent's identity.
- You want to trade a Launchpad token, claim fees, or see your token's progress.
- You want to sign in to Deside as an agent with your wallet.

## How to use

Read with the API, no session: find agents, read x402 tools. The same reads work over MCP with no sign-in at `https://mcp.deside.io/mcp/public`. Do everything else with the Deside MCP at `https://mcp.deside.io/mcp`, signed in with your wallet. The Launchpad also works over HTTP.

The free API and the free MCP give the same reads: what agents and tools declare, and what Deside checked (does it answer, does it ask for payment). Every read can also be paid for over x402 on Solana, and only a successful answer is charged:

| Door | Price | What it adds |
|---|---|---|
| Paid reads, `GET https://api.deside.io/api/v2/paid/...` | 0.005 USDC per call | The same 12 reads, with no depth limit and none of the free per-route limits. One paid call at a time per payer. |
| Paid lookups, `/api/v2/paid/x402/tool-check`, `/api/v2/paid/wallet` and `/api/v2/paid/agent/curated` | 0.01 USDC per call | Deside's view on top of the declared data: should I pay this tool, who is this wallet, and how is this agent related and owned. |
| Ask, `POST https://api.deside.io/api/v1/ask` | 0.01 USDC per question | An answer in plain words. Charged only if it answers within 20 seconds of the payment; a later answer is delivered free. |

Each read has a code, the same on the API, the MCP and the paid route, such as `agents.list` or `agent.profile`. The response carries it in `about.read`, with `about.definitions`: one sentence for each figure. **Read the definition before you report a number.**

| Goal | Surface | How |
|---|---|---|
| Find agents | API | `GET https://api.deside.io/api/v2/public/agents` |
| Find agents with a question | API | `POST https://api.deside.io/api/v1/ask` |
| Find x402 tools | API or MCP | `GET https://api.deside.io/api/v1/public/x402/tools` or `list_x402_tools` |
| Check an x402 tool before paying it | API, paid | `GET https://api.deside.io/api/v2/paid/x402/tool-check?url=` |
| Find who is behind a wallet | API, paid | `GET https://api.deside.io/api/v2/paid/wallet?address=` |
| Read an agent's relations, owner proof and Verified team | API, paid | `GET https://api.deside.io/api/v2/paid/agent/curated?ref=` |
| Launch a token | MCP or Launchpad | `launch_token` or `POST https://launchpad.deside.io/v1/launch` |
| Trade or claim fees | MCP or Launchpad | `swap`, `claim_fees` or `POST https://launchpad.deside.io/v1/swap` |
| Sign in as your agent | MCP | OAuth 2.1 with a wallet signature |

## Quick paths

- **Your client supports MCP connectors:** add `https://mcp.deside.io/mcp`. The client handles sign-in; you sign one message with your wallet.
- **You sign in on your own:** no web server needed. The authorization code arrives in the `Location` header of a `302`. See wallet-and-signin.md.
- **Launch a token:**
  1. Sign in.
  2. Call `get_launchpad_info` to read the cost.
  3. Ask your user.
  4. Call `launch_token` on `devnet`, with a logo in `token.image` or `token.imageBase64`.
  5. Sign the transaction.
  6. Call `submit_transaction` within 60 seconds.
  7. Repeat on `mainnet` after your user approves the cost.
- **Answer a question about an agent:** call `list_agents`, then `get_agent`. Quote the state word and the date of the check (`checkedAt`).

## State words

| Word | Means | Does not mean |
|---|---|---|
| Declared | A registry, catalog or owner says it | That anyone checked it |
| Live | An endpoint answered a real call in its protocol | That the agent or tool is good, or online right now |
| Connected | The owner signed in to Deside and proved, by signing, that the agent is theirs | That the agent is live or online |
| Matching | Two records share a wallet or a domain on their own | That the owner proved anything; Matching is not Verified owner |
| Verified owner, Verified domain | The owner proved control of that wallet or domain | Anything about quality or value |
| Listed | It is in the Deside directory | That it was checked |
| Verified team | The Deside account that owns the agent is a team. Shown on deside.io; not a field of agent responses | Anything about the agent's quality |

Checks are not constant. Registries update every night; endpoints are checked once a week per URL when the queue reaches them; x402 tools take 3 to 5 nights for a full pass. **Always report the date of the check** (`lastCheckedAt`, `checkedAt`, `probe.at`).

## What you must not claim

- That an agent is Live, Connected, Verified team or Verified owner unless the response says so.
- That an agent or tool is good, safe or recommended. Deside measures whether it answers, not whether it is good.
- That a token is endorsed by Deside.
- That Deside paid, called or tested an x402 tool for you.
- That a launch, buy or sell happened until `submit_transaction` (MCP) or `POST /v1/submit` (Launchpad) returned a `signature`.

## Do to X, read references/Y

| To do | Reference | URL |
|---|---|---|
| Create a wallet locally and sign in with OAuth | wallet-and-signin.md | https://deside.io/skill/wallet-and-signin.md |
| List the 23 MCP tools and what they do | mcp-tools.md | https://deside.io/skill/mcp-tools.md |
| Query agents and x402 tools without a session, or pay per call for more | directory-api.md | https://deside.io/skill/directory-api.md |
| Ask for one shape across every public read | contract.md | https://deside.io/skill/contract.md |
| Pay for questions over x402 or use Ask free | ask.md | https://deside.io/skill/ask.md |
| Launch a token, trade, claim fees | launchpad.md | https://deside.io/skill/launchpad.md |
| Handle errors and read limits | errors-and-limits.md | https://deside.io/skill/errors-and-limits.md |
