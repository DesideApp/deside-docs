# Changelog

This page records changes to the contracts of the API, the MCP and the Launchpad, newest first, and corrections to these docs. A change that breaks a client says what to do.

## 2026-10-06

- Launchpad statistics added, with no key: `GET /v1/stats` (totals, totals per version and one row per token), `GET /v1/stats/history` and `GET /v1/tokens/{mint}/history` (points per `hour` or `day`, at most 2000 per call, never filled in). The statistics routes allow 10 requests a minute per IP. See [Statistics reference](launchpad/docs/stats.md).
- `GET /v1/tokens/{mint}` and `get_token` carry a `launchpad` block for tokens launched here: version, launch and graduation dates, volume, fees by recipient, locked liquidity, price, market cap and holders. The other fields do not change.

## 2026-10-04

- The Agent Token Launchpad is called the Deside Launchpad in these docs. Its routes, tools and URLs do not change.
- The Ask example is a real response.

## 2026-10-03

- `POST /api/v1/ask` is listed in the PayAI x402 catalog, and its `402` carries `extensions.bazaar` with the input and output schema.
- `search_agents`, `agent_trust_card` and `get_directory_stats` on the MCP answer with the v2 agent shape with `catalogId` in place of `id`. `wallet`, `primarySource`, `primarySourceEntryId`, `sourceEntries`, `registryPresence`, `registry` and `publicReceipts` are gone: read `sources`, `registries`, `wallets`, `status`, `endpoints` and `services`. See [Tools Reference](mcp/tools.md#directory).
- the agent routes moved to `/api/v2/public/agents`: `GET /`, `/stats`, `/{ref}` and `/{ref}/profile` with no key. Answers come in `{data}` and errors in `{error:{code,message}}`. Field names changed: `catalogId` is `id`, `canonicalPath` is `path`, `curationPublic.state` is `status`. `stats-summary` is `/stats`. The profile no longer carries raw registry data, `receipts`, `registryStatus`, `collectionBadges` or the retired Verified badge. `/api/v1/public/agents` and all its routes, `receipts` included, are retired and answer `404`: move to v2. Each agent carries `team` and `handles` (`x`, `github`). See [Public agents](api/public-agents.md).
- the paid Directory API with keys is closed: `/api/v1/directory/agents`, `/directory/x402-tool-profiles`, `/directory/x402-resources` and the key console answer `404`. Use the [public routes](api/README.md#routes), which need no key.
- `POST /api/v1/ask` is paid over x402: 0.01 USDC per answered question on Solana, for callers without a Deside session. Without a payment it answers `402` with the offer, and `GET https://api.deside.io/.well-known/x402` lists it. deside.io and users signed in with a wallet stay free. See [Ask](api/ask.md#pay-with-x402).
- `topHosts` is no longer in `GET /api/v1/public/x402/census`. `GET /api/v1/public/x402/wallet-edges`, `/public/x402/wallet-revenue`, `/directory/x402-wallet-edges` and `/directory/x402-wallet-revenue` are closed and answer `404`.
- the Launchpad tools of the MCP name their errors: `PAYMENT_REQUIRED` (402), `forbidden` (403), `NOT_FOUND` (404), `CONFLICT` (409) and `transaction_failed` (422). Before, all of them came as `INVALID_INPUT`. If you matched on `INVALID_INPUT` plus `status`, match on the new codes.
- `POST /oauth/register` answers `201` instead of `200`, and a `401` from `/mcp` carries `WWW-Authenticate` with `resource_metadata`.
- `search_agents` takes only integers in `limit` and `offset`. `launch_token` over the MCP takes `imageBase64` up to 950,000 characters.
- the protected resource metadata of the MCP names it `deside` instead of `deside-dm`.
- `token_card` added to the MCP: read one Solana token, its market data and who is behind it.
- `search_agents`, `agent_trust_card`, `get_directory_stats` and `token_card` share a limit of 20 calls a minute and 100 a day per OAuth client, answered with `RATE_LIMITED` (429).

## 2026-10-02

- the docs are organized in six parts: Start, Agent Token Launchpad, API, MCP, Skill and Changelog. The old pages of Agent Identity, Directory API and x402 Tool Directory are now under API and Start; old links redirect.
- one Deside MCP: the Launchpad tools moved to `https://mcp.deside.io/mcp`, where you sign in with your Solana wallet and the tools use that wallet. `https://launchpad.deside.io/mcp` answers `410` with the new address. Connect to `mcp.deside.io` instead. The REST routes of the Launchpad do not change.
- `list_my_launches` is `list_launches`. On the MCP, `wallet` is optional and defaults to your session wallet. Call the new name.
- `update_agent_identity` and `POST /v1/agent-identity/update` added to the Launchpad: change the EIP-8004 registration of an agent identity your wallet owns in one unsigned transaction.
- `register_agent_identity` and `POST /v1/agent-identity` added: add the agent identity to a token already launched here, by its creator.
- `acceptTerms: true` is required on `launch_token` and `register_agent_identity`. Without it the Launchpad answers `400`.
- the TypeScript SDK `@desideapp/mcp-sdk` and the mini-agent example are no longer documented. Use any MCP client against `https://mcp.deside.io/mcp`.
- the MCP scopes are `deside:read` and `deside:write`. `dm:read` and `dm:write` are older names for the same scopes and are still accepted.
- the step `proven` of a relation is shown as "Verified owner" on deside.io. The key does not change.
- the x402 catalogue answers in English: `GET /api/v1/public/x402/tools`,
  `/tools/:slug`, `/census` and `/indices`, the Directory API `x402-tool-profiles`, the tool
  cards of Ask and the x402 row of the agent profile. Same values, new names: `sonda` is
  `probe` (`verdict`, `at`, `httpStatus`, `quote{amount,asset,network,payTo}`,
  `comparison{sameAmount,sameWallet,undeclaredNetwork}`), `bloque` is `tier`, `completitud`
  is `completeness`, `tipo` is `type`, `descriptorDeLlamada` is `callDescriptor`, `agente`
  is `agent` and `indice` is `index{agents,indexes,served,wellKnown}`. In the census
  `porBazar`, `porRed` and `porSonda` are `byBazaar`, `byNetwork` and `byVerdict`. In the
  index groups `indices`, `agentes` and `contenido` are `indexes`, `agents` and
  `content{state,tools,quotesPrice,respondsNoPrice,down,notChecked}`. In the agent profile
  `x402Estado` and `x402Contenido` are `x402State` and `x402Content`. Values: `vivo`,
  `muerto` and `sin-tools` are `live`, `down` and `no-tools`; a wallet `family` of
  `desconocida` is `unknown`; `walletCoincidences[].claim` `coincide` is `matches`. The
  verdicts do not change. The group filter is `?index=`; `?indice=` is a deprecated alias
  and will be removed. The old keys are no longer served.

## 2026-10-01: one shape for relations

The agent profile `relations` now uses English keys. See [Relation Fields](api/relations.md).

What changed in the agent profile `relations`:

* `items[].bolsa` is now `items[].kind`.
* `items[].escalon` is now `items[].step`, with `proven`, `matches` or
  `declared` instead of `demostrado`, `coincide` or `declarado`.
* `items[].via` is now `items[].vias`, each `{ how, value }`.
  `keyType` became `how`: `wallet` is `same-wallet`, `domain` is
  `same-domain`, and `declared` is `agent-lists-token` or
  `agent-lists-tool`. `keyValue` became `value`. `roles` is no longer sent.

What changed in the claim:

* `GET /api/v1/public/claim/{objectType}/{objectId}` and the `claim` field
  of the agent profile answer `{ state, vias: [{ via, steps, value }] }`,
  instead of `{ estado, vias: [{ via, pasos, valor }] }`.
* The claim route answers `404` with `{ "error": "not_found" }` when the
  agent or the token does not exist. Before, it answered `200` with
  `unclaimed`. An agent asked by slug comes back with its `catalogId` in
  `objectId`.

What to update: read the new keys and values above. A client that matched
the old Spanish values gets no match today.

## 2026-10-01: docs restructured

The docs are now organized by product: [Agent Directory](start/checks.md),
[x402 Tool Directory](api/x402-tools.md) and the Agent Token Launchpad, with
the Directory API and MCP under Developer Access. The public agents routes
are on one page, [Public Agents API](api/public-agents.md).
The public x402 routes moved to [x402 Tools API](api/x402-tools.md).
[Ask](api/ask.md) moved to the Agent Directory. New page:
x402 data with an API key (retired on 2026-10-03; see [x402 Tools API](api/x402-tools.md)).

Corrections to what the docs said before:

* `services` in the agent item is `{ kind, url, declared, checked, checkedAt,
  source }`. The docs described an older shape with `type`, `label`,
  `confidence` and `status`.
* `capabilities` also carries the agent's skills, as `{ id, source }`.
* `curationPublic` does not carry `descCategory`. Read `category` on the item.
* `fairscale` in the agent item is always `null`.
* `DELETE /directory/keys/{keyId}` on an unknown key answers
  `invalid_request`. There is no `key_not_found` code.
* `project_not_found` is not returned by the owner console. `GET /usage`
  without a project answers `invalid_request`.
* The `{id}` of the agent routes also accepts a previous slug, a registry
  entry address or a wallet. The redirect from `/profile` and `/trust` drops
  the suffix.
* The cursor of `GET /directory/agents` is not bound to `collection` or
  `collectionCase`.
* A full walk of the directory costs about 1,093 requests, not 104.
* The MCP endpoint is `https://mcp.deside.io/mcp`, not `/api/v1/mcp`.
* `byCategory` in the stats summary has 13 keys, not 11.
* The list of future MCP tool names was removed: they do not exist.

## 2026-09-30

* Added `GET /api/v1/public/x402/indices` and the `indice` filter of
  `/public/x402/tools`. The tool profile gains `indice`.
* Added `category` to the agent item and the profile.

## 2026-09-29

* Changed: an A2A endpoint stops counting as live when its last successful
  check is more than 8 days old ([Checks](start/checks.md#how-we-check-that-an-agent-is-live)).

## 2026-09-28

* Added `evidence` to `curationPublic.liveEndpoints[]`.
* Added `completitud` to x402 tools, and the list of `/public/x402/tools`
  orders tools by `bloque` and then by what they have complete.

## 2026-09-27

* Changed: the x402 live-check result `verified` is now `offer`, in
  `sonda.veredicto` and in the census `porSonda`. Read `offer` where you read
  `verified`.
* Changed: the keyed tool profile serves `walletCoincidences` instead of
  `identities`. Read `walletCoincidences`.
* Added `logo` and `bloque` to x402 tools.

## 2026-09-24

* Added `declaredByAgent`, `description` and `tipo` to the x402 tool list item,
  and `agente` to the profile.

## 2026-09-23

* Added the `network` and `live` filters and the `sonda` live-check result to
  `/public/x402/tools` and `/public/x402/census`.

## 2026-09-06

* Added the `chain` filter to `GET /api/v1/directory/agents`.

## 2026-09-03

* Added `GET /api/v1/public/x402/wallet-edges` and
  `GET /api/v1/directory/x402-wallet-edges`.

## 2026-09-02

* Added the public x402 tool catalog: `GET /api/v1/public/x402/tools`,
  `/tools/{slug}` and `/census`.
* Added `GET /api/v1/directory/x402-tool-profiles` and `/{slug}`.

## 2026-09-01

* Added `GET /api/v1/directory/x402-resources` and `/census`.

## Earlier

- the stats summary renamed its headline counter on 2026-09-21: `listed` is
  the number of agents in the Deside catalogue. `indexed` is kept as a
  deprecated alias of the same number. `registered` was removed: read
  `listed` instead. Documented `byChain` and `byCategoryByChain`, which split
  the counters by `solana` and `evm`
- the agent token in the public profile is `declared` or `none` since
  2026-09-27. `verifiedBy` is always `null`, and a native Metaplex binding is
  named in `nativeBy`
- documented that agent verification is not offered: the `verified*` fields
  stay for compatibility, `verified` is `false` and the check fields carry no
  data. The pages no longer describe a paid verification
- the public agent item renamed `isConnected` to `mcpSessionActive` on 2026-09-27, in the
  list card and in the profile. Same fact and same value: the agent itself has an active
  session with Deside through the MCP. `isConnected` is no longer served. Connected is
  still `ownerProven`, the owner's proof, and the two facts stay independent
- `connected` changed meaning on 2026-09-05, in the list item, in the trust
  facts, in the `?connected=` filter and in the `connected` counter of the
  stats summary: it is now `true` when the agent's owner has proved ownership
  by linking, with a signature, the wallet that owns the agent. Before it
  meant that the agent had completed OAuth through Deside's MCP, which is a
  fact about the agent, not about its owner. The public agent item gains
  `ownerProven` with the same meaning; `isConnected` stays and keeps meaning
  the agent's own MCP session. The counter in the stats summary follows on
  its next daily snapshot
- documented the public product surface: `GET /api/v1/public/agents/stats-summary`
  now has a page, with what each counter counts and in which unit. The headline
  number changed meaning on 2026-08-21: `indexed` counts AGENTS in the Deside
  catalogue, not entries across registries, so it dropped from 10,825 to 10,690
  and now matches the `total` of the public list. `registered` is kept as a
  deprecated alias of the same number and will be removed
- stated the two rules that make the counters readable: `endpoints` is counted
  in URLs and never converts to agents (one URL can be declared by many
  agents), and an absent key means not measured, never `0`

- documented how liveness and verification are actually measured (Trust): the
  census handshake with its two-strike anti-flapping versus the paid daily
  check that invokes safe tools with rotation; the five failure-semantics
  rules (a failing tool never touches liveness, never removes the badge, a
  single failure is not a verdict, passes are sealed to the URL they probed,
  and the owner is notified on change)

- Ask moved to its own address: `POST /api/v1/ask` is now canonical. The old
  `POST /api/v1/directory/agents/ask` keeps answering as an alias and will be
  retired. Rationale: Ask has its own regime (no key, no quota, its own rate
  limits, and an announced pay-per-question machine lane), so it gets its own
  street instead of living inside the key-protected read surface's path

- named the collision that will confuse everyone sooner or later:
  `/directory/subscription` is the API plan of a project, and
  `/directory/agents/:catalogId/subscription` belongs to the owner of one
  agent, outside this API. Same word, same prefix, different person and different credential
- listed the agent-owner rail as deliberately out of scope, so that finding its
  eleven routes does not read as an undocumented part of this API

- corrected the auth of the owner routes: keys and usage use the
  console proof, not a session cookie. The previous text said `protectRoute`,
  which has not been true since the console got its own signature login
- documented the console proof itself: nonce, signature, short-lived bearer,
  its own audience, and why it is a separate credential family
- documented the subscription and billing routes: the five endpoints, the
  unsigned-transaction flow, the 30-day cycle, why the tier only changes once
  the first charge settles, why cancelling stays open when selling is closed,
  that changing tier means cancel and subscribe again, and the actionable
  error codes
- documented `POST /directory/agents/ask`: no API key, no quota, human and
  headless lanes with their own rate limits, never a `500`, and the announced
  `402` for the paid machine lane
- added the two filters that the list route really accepts, `collection` and
  `collectionCase`, and stated that there is no free-text search on it
- documented the list ordering and what invalidates a cursor
- added the five item fields that were exposed but undocumented: `connected`,
  `primaryWalletSource`, `collectionBadges`, `channels` and `curationPublic`,
  the last one being where the measured facts live
- documented the verified fields in the trust response (`verified`,
  `verifiedCheck`, `verifiedCheckedAt`, `verifiedFailed`) and how to read
  them: a positive fact when present, no information when absent

- added `socialLinks` to `DirectoryAgentListItemV1` and the profile contract:
  the agent's own declared website, X, and GitHub links, resolved and
  sanitized by the backend

- aligned the capability filter vocabulary with the accepted server set
  (removed `support` and `automation` until the server accepts them)
- corrected the trust `fairscale` example (`scoreKind` values such as
  `fairscore`, added `walletClassification`)
- reworded the public catalog auth row: it is an open product surface, not a
  legacy surface

- published the initial Directory API doc set
- documented the API-key read surface, pagination, errors, limits, data
  product fields, and boundary notes
- reconciled `api_key_revoked` to the `403` status contract
- documented the trust facts endpoint, including `connected`, receipt
  families, declared services, and FairScale attribution rules

## Notes

This changelog records changes to the public contracts of the API, the MCP and the Launchpad, and corrections to these docs.
