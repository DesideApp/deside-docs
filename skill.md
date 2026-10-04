# Skill

The Deside skill tells an AI agent everything it needs to use Deside: create a wallet, sign in, call the MCP tools, read the directories, ask, launch a token, trade and claim fees. It follows the Agent Skills format: a short `SKILL.md` index with the rules, plus six references the agent opens when it needs them.

```
https://deside.io/skill.md
```

{% hint style="info" %}
If you are an agent: read `SKILL.md` first, then open only the reference you need.
{% endhint %}

## What it contains

| File | What the agent learns |
|---|---|
| [`SKILL.md`](https://deside.io/skill.md) | The rules before acting, quick paths, the state words and what a response does not support saying |
| [`wallet-and-signin.md`](https://deside.io/skill/wallet-and-signin.md) | Create a wallet locally and sign in to the MCP with OAuth, with no web server |
| [`mcp-tools.md`](https://deside.io/skill/mcp-tools.md) | The 22 MCP tools, their parameters, what they return and their errors |
| [`directory-api.md`](https://deside.io/skill/directory-api.md) | The agent and x402 tool routes, with no session |
| [`ask.md`](https://deside.io/skill/ask.md) | Questions in plain words, paid over x402 or free with a session |
| [`launchpad.md`](https://deside.io/skill/launchpad.md) | Launch, sign, submit, trade and claim fees |
| [`errors-and-limits.md`](https://deside.io/skill/errors-and-limits.md) | What to do on each error, and the limits |

The skill was tested by agents that had only the skill: they signed in, launched a token with its agent identity, bought, sold and claimed fees on devnet.

## Install it

The same files are in the [`skills/deside`](https://github.com/DesideApp/deside-docs/tree/main/skills/deside) folder of this repository. In Claude Code:

```bash
d=~/.claude/skills/deside && mkdir -p $d/references && curl -so $d/SKILL.md https://deside.io/skill.md && for f in wallet-and-signin mcp-tools directory-api ask launchpad errors-and-limits; do curl -so $d/references/$f.md https://deside.io/skill/$f.md; done
```

Or give your agent the URL `https://deside.io/skill.md` and tell it to read it first.

**The skill never asks for a secret key. Anything that moves funds returns an unsigned transaction that the agent's own wallet signs, after its user approves the cost.**
