# Services And Capabilities

Services and capabilities are the two filters of
`GET /api/v1/directory/agents` that describe what an agent offers. A service
is a channel the agent declares, such as an MCP endpoint. A capability is a
role or skill, derived from those declarations and from the agent's skills.

**Both are declared or derived, never tested.** Whether an endpoint answers is
in `channels[].checked` and `curationPublic.liveEndpoints` (see
[Data model](data-model.md#channels-and-services)).

## The `service` filter

`service` matches agents that declare that service. Accepted values:

| Value | Matches agents that declare |
| --- | --- |
| `web` | a website |
| `mcp` | an MCP endpoint |
| `a2a` | an A2A endpoint |
| `x402` | an x402 endpoint |
| `api` | an API |
| `contact` | a contact channel |

Any other value answers `invalid_request`.

## The `capability` filter

Accepted values, and what each one matches:

| Value | Matches agents that |
| --- | --- |
| `mcp_server` | declare an MCP endpoint |
| `a2a_task_receiver` | declare an A2A endpoint |
| `x402_acceptor` | declare an x402 endpoint |
| `payments` | declare an x402 endpoint, or have a `payments` or `transfer` skill |
| `identity` | are present in at least one registry |
| `trading`, `defi`, `content`, `analytics` | have at least one skill that Deside files under that category |

Any other value answers `invalid_request`.

## The `capabilities` field

Each agent carries a `capabilities` array. Entries come in two forms.

Derived roles carry `id`, `label`, `source` and `confidence`, which is always
`derived`:

| `id` | `source` | Present when the agent |
| --- | --- | --- |
| `mcp_server` | `serviceSignals` | declares an MCP endpoint |
| `a2a_task_receiver` | `serviceSignals` | declares an A2A endpoint |
| `x402_acceptor`, `payments` | `serviceSignals` | declares an x402 endpoint |
| `identity` | `registryPresence` | is present in at least one registry |

Skills carry only `id` and `source`. `id` is the skill as declared, and
`source` is `registry` when a registry declared it or `web` when it came from
the agent's website. Up to 60 skills are listed.

## What this does not mean

* A `mcp_server` capability does not mean the MCP endpoint answers. It means
  the agent declares one.
* A skill is the agent's own claim. Deside does not test skills.
