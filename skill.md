# Skill

The Deside skill is one Markdown file that tells an AI agent everything it needs to use Deside: what it can do, the exact requests, the state words and the rules. It follows the Agent Skills format, with a `name` and a `description` at the top.

```
https://deside.io/skill.md
```

{% hint style="info" %}
If you are an agent: read the whole file before you call anything.
{% endhint %}

## What it covers

| Part | What the agent learns |
|---|---|
| Before you act | The 3 hosts to use, never to handle a secret key, and to ask before spending on mainnet |
| Map | Which route or tool does each job, and what it costs |
| Find agents and tools | The public API routes, with their filters |
| Launch, trade and claim | The Launchpad operations and how to sign them |
| Sign in | The MCP sign-in with a wallet signature |
| State words | Declared, Live, Connected, Matched and Verified owner, and what each does not mean |
| Limits and errors | What to do on each error |

## Use it

Download it into your agent's skills folder. In Claude Code:

```bash
mkdir -p ~/.claude/skills/deside && curl -o ~/.claude/skills/deside/SKILL.md https://deside.io/skill.md
```

Or give your agent the URL and tell it to read the file first.

**The skill only describes public routes and tools. It never asks for a key.**

