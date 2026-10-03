# Deside Docs

Deside is a Solana trading app that shows which agent, tools and website stand behind each token. It also runs two public directories, of AI agents and of x402 pay-per-call tools, and a launchpad where an agent launches its own token with its identity.

{% hint style="info" %}
These docs cover the three ways to use Deside from code:
- **API**: read the directories over HTTP, with no key. Ask is paid per question unless you are signed in with a wallet.
- **MCP**: everything Deside does, from Claude or your own agent, signed in with a Solana wallet.
- **Agent Token Launchpad**: launch a token for an agent, with its identity, in one signature.
{% endhint %}

## Quick start

Read the first page of the Agent Directory. No key needed:

```bash
curl "https://api.deside.io/api/v2/public/agents?limit=1"
```

To act (launch a token, trade, claim fees), connect the MCP instead. See [MCP](mcp/README.md).

## Which one you want

| You want to | Use | Sign-in |
|---|---|---|
| Read agents, x402 tools and their checks | [API](api/README.md) | None |
| Search, ask, launch and trade from Claude or an agent | [MCP](mcp/README.md) | Your Solana wallet |
| Launch a token for an agent | [Agent Token Launchpad](launchpad/README.md) | Your Solana wallet |
| Give an agent the whole guide in one file | [Skill](skill.md) | None |

**The API only reads. Anything that changes state goes through the MCP or the Launchpad, signed by your own wallet.** Deside never holds your key.

## Quick reference

| What | Where |
|---|---|
| API base | `https://api.deside.io` |
| API OpenAPI | `https://api.deside.io/openapi.json` |
| MCP endpoint | `https://mcp.deside.io/mcp` |
| Launchpad terms and fees | `https://launchpad.deside.io/v1/info` |
| Launchpad OpenAPI | `https://launchpad.deside.io/openapi.json` |
| Agent skill | `https://deside.io/skill.md` |
| llms.txt | `https://deside.io/llms.txt` |

## Official domains

Deside runs only on deside.io and its subdomains api.deside.io, mcp.deside.io, launchpad.deside.io and docs.deside.io.

{% hint style="danger" %}
Deside never asks for your secret key or seed phrase. Don't paste them anywhere that claims to be Deside.
{% endhint %}

## Next steps

- [API](api/README.md): every public route, with real responses.
- [MCP](mcp/README.md): sign-in, the tools and their errors.
- [Agent Token Launchpad](launchpad/README.md): how a launch works, fees and a real example.
- [Changelog](changelog.md): what changed in the contracts and what to do about it.
